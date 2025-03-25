---
layout: post
title: SAMLStorm Critical Authentication Bypass in xml-crypto and Node.js libraries
date: 2025-03-25
categories: ["CVE-2025-29775","CVE-2025-29774","SAML protocol"]
---

Lỗ hổng SAMLStorm ảnh hưởng đến thư viện xml-crypto Node.js (phiên bản 6.0.0 và trước đó, CVE-2025-29775 & CVE-2025-29774), với bản vá được giới thiệu trong phiên bản 6.0.1 và được backport đến phiên bản 3.2.1 và 2.1.6. Nó cũng ảnh hưởng đến các thực thi SAML của Node.js bao gồm @node-saml/node-saml, samlify, saml2-js, samlp, saml2-suomifi và các gói khác. Tổng thể, các gói này có hơn 500k lượt tải mỗi tuần.

Chi tiết kỹ thuật đầy đủ của lỗi này, cách nó hoạt động và các bước khắc phục cho dịch vụ không phải WorkOS được đề cập bên dưới.

## How this zero-day enables full account takeovers

Trước khi đi vào chi tiết về giao thức SAML và cách thức hoạt động của lỗ hổng này, trước tiên cần xác định mức độ ảnh hưởng tiềm tàng ở cấp độ cao.

Bất kỳ công ty nào cung cấp dịch vụ SSO thông qua SAML và sử dụng thư viện xml-crypto đều có nguy cơ bị tấn công. Trong trường hợp xấu nhất, một kẻ tấn công bên ngoài có thể giả mạo các assertion tùy ý cho một nhà cung cấp danh tính SAML (IdP), dẫn đến nguy cơ chiếm quyền kiểm soát hoàn toàn tài khoản trong các nhà cung cấp dịch vụ bị ảnh hưởng, tùy thuộc vào các biện pháp bảo mật của họ.

Cuộc tấn công này không yêu cầu bất kỳ sự tương tác nào từ người dùng, có nghĩa là kẻ tấn công có thể truy cập trái phép vào ứng dụng mục tiêu với các đặc quyền nâng cao.

## SAML basics: understanding the attack surface

Có hai lỗ hổng tương tự nhưng khác biệt trong xml-crypto, được đại diện bởi CVE-2025-29775 và CVE-2025-29774, lỗ hổng đầu tiên có thể khai thác qua node-saml và lỗ hổng thứ hai thì không. Để ngắn gọn, bài viết này chỉ tập trung vào lỗ hổng và vector khai thác ảnh hưởng đến việc sử dụng node-saml, và đã được báo cáo cho WorkOS.

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

Để một nhà cung cấp dịch vụ SAML có thể an toàn tin tưởng một khẳng định trong phản hồi SAML, nó phải xác thực chữ ký mật mã của IdP. Chuỗi tin cậy này bắt đầu với chứng chỉ X.509 của IdP, mở rộng đến chữ ký trên khối SignedInfo, bao gồm bản tóm lược của khẳng định trong SignedInfo, và kết thúc với chính khẳng định đó.

Bức tranh dưới đây minh họa cách chuỗi tin cậy bị đứt trong cuộc tấn công SAMLStorm.

![](https://cdn.prod.website-files.com/621f84dc15b5ed16dc85a18a/67d433b16eda7a1fa45cb1fa_Chain%20of%20trust.png)

Trong thư viện xml-crypto, quá trình kiểm tra băm của assertion đảm bảo rằng giá trị băm trong khối SignedInfo khớp chính xác với assertion, trong khi kiểm tra chữ ký xác minh rằng chữ ký hợp lệ dựa trên chứng chỉ của IdP và khối SignedInfo. Trong điều kiện hoạt động bình thường, chữ ký bảo vệ assertion khỏi bị chỉnh sửa, vì giá trị băm của assertion được bao gồm trong khối SignedInfo.

Vấn đề cốt lõi của lỗ hổng này là hai quá trình kiểm tra đánh giá tài liệu SAML theo cách khác nhau. Các tài liệu SAML trải qua quá trình chuẩn hóa (canonicalization), một bước loại bỏ các bình luận XML. Tuy nhiên, kiểm tra băm của assertion được thực hiện trên tài liệu chưa được chuẩn hóa (vẫn giữ nguyên bình luận), trong khi kiểm tra chữ ký được thực hiện trên tài liệu đã chuẩn hóa (đã loại bỏ bình luận). Sự không đồng nhất này tạo ra cơ hội khai thác bằng cách chèn một giá trị băm giả mạo bên trong một bình luận, như minh họa bên dưới.

```
<DigestValue><!-- forged_digest -->legitimate_digest</DigestValue>
```

Trong đoạn mã dễ bị tấn công, quá trình kiểm tra băm (digest check) của SAML assertion lấy phần tử con đầu tiên của nút DigestValue. Thông thường, đây là giá trị băm mong đợi, nhưng kẻ tấn công có thể chèn một bình luận XML chứa giá trị băm giả mạo của một assertion tùy ý trước giá trị băm hợp lệ. Kết quả là quá trình kiểm tra băm của assertion vô tình sử dụng giá trị băm do kẻ tấn công cung cấp.

Trong khi đó, quá trình kiểm tra chữ ký (signature check) được thực hiện trên khối SignedInfo sau khi nó đã được chuẩn hóa (canonicalized). Vì bình luận độc hại bị loại bỏ trong quá trình chuẩn hóa, chỉ còn lại giá trị băm của assertion hợp lệ ban đầu, cho phép kiểm tra chữ ký thành công.

Điều này phá vỡ chuỗi tin cậy giữa chứng chỉ và assertion, cho phép kẻ tấn công giả mạo các assertion tùy ý mà nhà cung cấp dịch vụ dễ bị tấn công sẽ chấp nhận là hợp lệ.

## Understanding if your application may be impacted 

Bất kỳ khách hàng nào của một nhà cung cấp dịch vụ (Service Provider - SP) sử dụng phiên bản dễ bị tấn công của thư viện xml-crypto đều có nguy cơ bị tấn công. Tuy nhiên, mức độ rủi ro phụ thuộc vào từng nhà cung cấp dịch vụ cụ thể và việc kẻ tấn công có danh tính trong một nhà cung cấp danh tính (Identity Provider - IdP) hay không. Nếu kẻ tấn công có một danh tính từ bất kỳ IdP nào được kết nối với nhà cung cấp dịch vụ, họ có thể khai thác lỗ hổng này đối với bất kỳ nhà cung cấp danh tính SAML nào, miễn là nhà cung cấp dịch vụ phụ thuộc vào xml-crypto.

Nếu kẻ tấn công không có danh tính trong IdP, họ vẫn có thể khai thác lỗ hổng này—nhưng chỉ đối với các nhà cung cấp danh tính có ký metadata của họ. Nhiều IdP lớn ký metadata của họ. Mặc dù đây không phải là một lỗ hổng bảo mật trong các IdP này, nhưng nó vẫn làm lộ thông tin mà kẻ tấn công có thể lợi dụng để khai thác lỗi trong xml-crypto.

Nếu một ứng dụng không tự xác minh quyền sở hữu tài khoản, kẻ tấn công có thể giả mạo xác thực cho bất kỳ người dùng nào trong tổ chức, bao gồm cả quản trị viên, và leo thang đặc quyền bằng cách sử dụng thuộc tính SAML hoặc phân quyền nhóm từ IdP. Ngoài ra, nếu một ứng dụng không giới hạn những người dùng mà IdP được phép xác thực, kẻ tấn công có thể giả mạo xác thực cho bất kỳ người dùng nào trong ứng dụng của nhà cung cấp dịch vụ, bất kể ranh giới tổ chức.

## Recommendations for impacted organizations

### Short-term recommendations 

Đối với các công ty

WorkOS đã vá lỗ hổng này và xác nhận rằng không có hệ thống hay khách hàng nào bị ảnh hưởng. Tuy nhiên, các hệ thống xác thực không thuộc WorkOS có thể vẫn đang gặp rủi ro. Chúng tôi khuyến nghị các bước sau:
- Liên hệ với bất kỳ ứng dụng nào mà bạn xác thực thông qua SAML SSO để hỏi xem họ có bị ảnh hưởng không và rủi ro còn lại là gì.
- Xem xét nhật ký kiểm tra (audit logs) trong các ứng dụng bị ảnh hưởng để phát hiện bất kỳ đăng nhập hoặc hoạt động đáng ngờ nào.

Đối với các nhà cung cấp dịch vụ
- Kiểm tra xem triển khai SAML của bạn có sử dụng gói xml-crypto hay không.
- Nếu có, hãy xem xét nhật ký SAML để tìm dấu hiệu khai thác. Cụ thể, hãy kiểm tra xem có bình luận XML được nhúng trong trường DigestValue của phản hồi hay không, ví dụ:

```
<DigestValue><!-- forged_digest -->legitimate_digest</DigestValue>
```

### Long-term recommendations

Đối với các công ty
- Thường xuyên đánh giá mức độ rủi ro của tổ chức bạn đối với các ứng dụng được bảo vệ bằng SSO và duy trì một danh sách cập nhật về tất cả các nhà cung cấp này, bao gồm thông tin liên hệ bảo mật và loại dữ liệu được lưu trữ bởi từng nhà cung cấp.
- Lựa chọn các nhà cung cấp hỗ trợ đầy đủ khả năng ghi nhật ký kiểm tra bảo mật, giúp bạn có thể tự đánh giá quyền truy cập vào dữ liệu của mình khi cần.

Đối với các nhà cung cấp dịch vụ
- Đảm bảo rằng các tenant được bảo vệ bởi SSO được cô lập đúng cách và áp dụng nguyên tắc đặc quyền tối thiểu (Principle of Least Privilege) đối với IdP. Ví dụ, WorkOS giảm thiểu nguy cơ truy cập chéo giữa các tenant theo mặc định bằng cách hỗ trợ giới hạn miền danh tính theo từng tổ chức.
- Nhìn chung, việc kiểm tra dữ liệu nhận được có tuân theo các tiêu chuẩn mong đợi trước khi xử lý là một cơ chế phòng thủ quan trọng chống lại các lỗ hổng chưa được biết đến. Hãy áp dụng nguyên tắc này vào phản hồi SAML để đảm bảo cấu trúc của chúng khớp với mong đợi, đặc biệt là đối với chứng chỉ, chữ ký và giá trị băm trong chuỗi tin cậy. Ví dụ, thư viện node-saml không bị ảnh hưởng bởi phương thức khai thác trong CVE-2025-29774 vì nó kiểm tra xem số lượng tham chiếu trong khối SignedInfo có khớp với giá trị mong đợi là một hay không, trước khi chuyển phản hồi đến xml-crypto để xác thực thêm.
