---
layout: post
title: SAMLStorm Critical Authentication Bypass in xml-crypto and Node.js libraries
date: 2025-03-25
categories: ["CVE-2025-29775","CVE-2025-29774","SAML protocol"]
---

Lỗ hổng SAMLStorm ảnh hưởng đến thư viện xml-crypto Node.js (phiên bản 6.0.0 và trước đó, CVE-2025-29775 & CVE-2025-29774), với bản vá được giới thiệu trong phiên bản 6.0.1 và được backport đến phiên bản 3.2.1 và 2.1.6. Nó cũng ảnh hưởng đến các thực thi SAML của Node.js bao gồm @node-saml/node-saml, samlify, saml2-js, samlp, saml2-suomifi và các gói khác. Tổng thể, các gói này có hơn 500k lượt tải mỗi tuần.

Chi tiết kỹ thuật đầy đủ của lỗi này, cách nó hoạt động và các bước khắc phục cho dịch vụ không phải WorkOS được đề cập bên dưới.

## How this zero-day enables full account takeovers

Trước khi đi sâu vào chi tiết của giao thức SAML và cách thức hoạt động của lỗ hổng này, điều quan trọng là trước tiên cần nêu ra tác động tiềm năng ở cấp độ cao.

Bất kỳ công ty nào cung cấp dịch vụ SSO qua SAML mà sử dụng thư viện xml-crypto đều có nguy cơ. Trong trường hợp xấu nhất, một kẻ tấn công bên ngoài có thể tạo ra các khẳng định tùy ý cho một nhà cung cấp danh tính SAML (IdP), có thể dẫn đến việc chiếm quyền truy cập đầy đủ vào các nhà cung cấp dịch vụ bị ảnh hưởng, tùy thuộc vào các biện pháp bảo mật của họ.
Lỗ hổng này không yêu cầu sự tương tác của người dùng, nghĩa là kẻ tấn công có thể giành quyền truy cập không được ủy quyền vào ứng dụng mục tiêu với đặc quyền được nâng cao.

## SAML basics: understanding the attack surface

Có hai lỗ hổng tương tự nhưng khác biệt trong xml-crypto, được đại diện bởi CVE-2025-29775 và CVE-2025-29774, lỗ hổng đầu tiên có thể khai thác qua node-saml và lỗ hổng thứ hai thì không. Để ngắn gọn, bài viết này chỉ tập trung vào lỗ hổng và vector khai thác ảnh hưởng đến việc sử dụng node-saml, và đã được báo cáo ban đầu cho WorkOS.

Để hiểu về lỗ hổng này, trước tiên chúng ta cần hiểu một chút về cách thức hoạt động của SAML.

![](https://cdn.prod.website-files.com/621f84dc15b5ed16dc85a18a/67d22e2584f34364f3f615f5_image%20(3).png)

Hình ảnh mô tả luồng xác thực SAML (Security Assertion Markup Language) giữa ba thành phần chính:
- Identity Provider (IdP) – Nhà cung cấp danh tính, nơi quản lý thông tin đăng nhập của người dùng.
- End User Browser – Trình duyệt của người dùng.
- Service Provider (SP) – Nhà cung cấp dịch vụ, ứng dụng hoặc hệ thống mà người dùng muốn truy cập.

Dưới đây là giải thích chi tiết từng bước:

### SP-Initiated Flow (Dòng khởi tạo từ SP)
- Người dùng truy cập Service Provider (SP)
  - Người dùng mở trình duyệt và truy cập ứng dụng/dịch vụ yêu cầu xác thực.
  - Nếu người dùng chưa đăng nhập, SP sẽ kích hoạt quy trình xác thực SAML.
- SP chuyển hướng trình duyệt của người dùng đến IdP bằng SAML Request
  - SP gửi một yêu cầu xác thực SAML (SAML AuthnRequest) đến IdP thông qua trình duyệt của người dùng.
  - Trình duyệt chuyển tiếp yêu cầu này đến IdP.
 
### IdP-Initiated Flow (Dòng khởi tạo từ IdP)
- IdP nhận yêu cầu SAML và kiểm tra thông tin
  - IdP nhận được yêu cầu và kiểm tra xem người dùng đã đăng nhập hay chưa.
  - Nếu chưa, IdP sẽ yêu cầu người dùng xác thực.
- Người dùng xác thực trên IdP
  - Người dùng nhập thông tin đăng nhập (tên người dùng, mật khẩu hoặc sử dụng xác thực đa yếu tố - MFA).
  - Nếu xác thực thành công, IdP tiếp tục quy trình.
- IdP tạo phản hồi SAML (SAML Response) với Assertion
  - Sau khi xác thực thành công, IdP tạo một SAML Response, bao gồm một SAML Assertion.
  - SAML Assertion chứa thông tin danh tính của người dùng (ví dụ: email, tên, vai trò) và trạng thái xác thực.
  - Phản hồi này được ký điện tử để đảm bảo tính toàn vẹn và xác thực.
- IdP gửi phản hồi SAML lại cho SP thông qua trình duyệt
  - IdP chuyển tiếp phản hồi SAML đến trình duyệt của người dùng.
  - Trình duyệt chuyển tiếp phản hồi này đến SP.
- SP xác thực phản hồi SAML và cấp quyền truy cập
  - SP kiểm tra tính hợp lệ của phản hồi SAML (chữ ký số, thời gian hết hạn, Issuer).
  - Nếu phản hồi hợp lệ, SP xác nhận danh tính người dùng và thiết lập phiên đăng nhập.
  - Người dùng được cấp quyền truy cập vào ứng dụng hoặc dịch vụ.

Đây là ví dụ về SAML response:

```
<saml2p:Response Destination="acsurl" ID="id1857861521424404880641928" ...>
    ...
    <saml2:Assertion ID="id1857861521593646366230134" ...>
        <saml2:Issuer>samlissuer</saml2:Issuer>
        <ds:Signature xmlns:ds="http://www.w3.org/2000/09/xmldsig#">
            <ds:SignedInfo>
                <ds:CanonicalizationMethod Algorithm="http://www.w3.org/2001/10/xml-exc-c14n#" />
                <ds:SignatureMethod Algorithm="http://www.w3.org/2001/04/xmldsig-more#rsa-sha256" />
                <ds:Reference URI="#id1857861521593646366230134">
                    ...
                    <ds:DigestValue>puw8MLNZ67893HzfgbpLjGPfsdSBJueFbcSw2neguIuk=</ds:DigestValue>
                </ds:Reference>
            </ds:SignedInfo>
            <ds:SignatureValue>assertionsignaturevalue</ds:SignatureValue>
            <ds:KeyInfo>
                <ds:X509Data>
                    <ds:X509Certificate>x509certificate</ds:X509Certificate>
                </ds:X509Data>
            </ds:KeyInfo>
        </ds:Signature>
        <saml2:Subject xmlns:saml2="urn:oasis:names:tc:SAML:2.0:assertion">
            ...
        </saml2:Subject>
        ...
        <saml2:AttributeStatement>
            <saml2:Attribute Name="id">
                <saml2:AttributeValue ...>idp-id</saml2:AttributeValue>
            </saml2:Attribute>
        </saml2:AttributeStatement>
    </saml2:Assertion>
</saml2p:Response>
```

## How an attacker would exploit this vulnerability

Có một số điều kiện chung cần được đáp ứng để khai thác một nhà cung cấp dịch vụ:

1. Nhà cung cấp dịch vụ phải sử dụng xml-crypto.
2. Kẻ tấn công cần bất kỳ cặp chữ ký và bản tóm lược hợp lệ nào từ một nhà cung cấp danh tính.
3. Nhà cung cấp dịch vụ cần tin tưởng chứng chỉ của nhà cung cấp danh tính trong cấu hình SAML của mình.
4. Kẻ tấn công cần có quyền truy cập vào URL ACS, SP Entity ID và IdP Entity ID:
   4.1. URL ACS – Điểm cuối mà Nhà cung cấp danh tính gửi phản hồi xác thực của mình.
   4.2. SP Entity ID – Một URI xác định đối tượng của phản hồi SAML.
   4.3. IdP Entity ID – Một URI xác định người cấp phát của phản hồi SAML.

Những điều kiện này dẫn đến các trường hợp khai thác sau:

### Attack path 1: full access without an account

Cách thức tấn công của kẻ tấn công không có danh tính trong nhà cung cấp danh tính:

![](https://cdn.prod.website-files.com/621f84dc15b5ed16dc85a18a/67d22e6201a27204cb96e60f_image%20(4).png)

1. Truy cập ứng dụng mục tiêu bằng một email chuyển hướng đến một nhà cung cấp danh tính có siêu dữ liệu được ký công khai. Lưu ý rằng đây không phải là lỗ hổng bảo mật trong các nhà cung cấp danh tính này—they chỉ cung cấp thông tin mà kẻ tấn công có thể sử dụng để khai thác lỗ hổng trong xml-crypto.
2. Khi cố gắng đăng nhập, một yêu cầu SAML sẽ được phát hành, từ đó bạn có thể lấy URL ACS và SP Entity ID, điều này cần thiết để tạo phản hồi SAML
3. Lấy thông tin chứng chỉ và chữ ký, cũng như IdP Entity ID, từ siêu dữ liệu công khai
4. Tạo một phản hồi SAML với chứng chỉ và giá trị đã ký từ siêu dữ liệu
5. Chỉnh sửa khẳng định SAML
6. Tính toán lại giá trị DigestValue và chèn nó dưới dạng chú thích trước giá trị tiêu hóa hiện có trong nút DigestValue
7. Gửi yêu cầu với phản hồi SAML đã tạo đến URL ACS
8. Nhà cung cấp dịch vụ xác thực phản hồi, và bạn đã xác thực

Lưu ý rằng trường hợp khai thác này một phần dựa trên luồng khởi tạo bởi SP để lấy URL ACS và SP Entity ID, nhưng đôi khi URL ACS và SP Entity ID là có thể đoán trước và được tài liệu hóa.Đây là ví dụ về một phản hồi SAML đã bị can thiệp:

```
<saml2p:Response Destination="acsurl" ID="id1857861521424404880641928" ...>
    ...
    <saml2:Assertion ID="id1857861521593646366230134" ...>
        <saml2:Issuer>samlissuer</saml2:Issuer>
        <ds:Signature xmlns:ds="http://www.w3.org/2000/09/xmldsig#">
            <ds:SignedInfo>
                <ds:CanonicalizationMethod Algorithm="http://www.w3.org/2001/10/xml-exc-c14n#" />
                <ds:SignatureMethod Algorithm="http://www.w3.org/2001/04/xmldsig-more#rsa-sha256" />
                <ds:Reference URI="#id1857861521593646366230134">
                    ...
                    <ds:DigestValue><!--3YjA3OTNjZWQ1GI5YjljNjgzOWZiZWI5OWY1ZTk1ZDk=-->puw8MLNZ67893HzfgbpLjGPfsdSBJueFbcSw2neguIuk=</ds:DigestValue>
                </ds:Reference>
            </ds:SignedInfo>
            <ds:SignatureValue>assertionsignaturevalue</ds:SignatureValue>
            <ds:KeyInfo>
                <ds:X509Data>
                    <ds:X509Certificate>x509certificate</ds:X509Certificate>
                </ds:X509Data>
            </ds:KeyInfo>
        </ds:Signature>
        <saml2:Subject xmlns:saml2="urn:oasis:names:tc:SAML:2.0:assertion">
            ...
        </saml2:Subject>
        ...
        <saml2:AttributeStatement>
            <saml2:Attribute Name="id">
                <saml2:AttributeValue ...>idp-id</saml2:AttributeValue>
            </saml2:Attribute>
        </saml2:AttributeStatement>
    </saml2:Assertion>
</saml2p:Response>
```

### Attack path 2: abusing legitimate access for privilege escalation

Có một biến thể của cuộc tấn công này ảnh hưởng đến tất cả các nhà cung cấp dịch vụ sử dụng xml-crypto, bất kể có hay không có siêu dữ liệu đã ký công khai. Tuy nhiên, nó yêu cầu kẻ tấn công phải có một danh tính trong nhà cung cấp danh tính và ít nhất một ứng dụng đã được cấp phép.

![](https://cdn.prod.website-files.com/621f84dc15b5ed16dc85a18a/67d43360725b4244f4ddac53_Scenario%202.png)

1. Cố gắng đăng nhập vào nhà cung cấp dịch vụ bằng luồng khởi tạo bởi IdP hoặc SP.
2. Chặn yêu cầu đến URL ACS chứa phản hồi SAML.
3. Chỉnh sửa khẳng định SAML theo nhu cầu.
4. Tính toán lại bản tóm lược của khẳng định SAML và chèn giá trị bản tóm lược mới dưới dạng chú thích trong nút DigestValue của chữ ký SAML.
5. Thay thế phản hồi SAML trong yêu cầu bằng phản hồi SAML đã tạo của bạn.
6. Chuyển tiếp yêu cầu đến URL ACS.
7. Nhà cung cấp dịch vụ xác thực phản hồi, và bạn đã xác thực.

## Breaking the chain of trust: How xml-crypto fails

