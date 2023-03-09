---
layout: post
title: Cách Hoạt Động Của HTTPS
date: 2023-3-2
categories: ["system"]
---

HTTPS là viết tắt của Hypertext Transfer Protocol Secure, là một giao thức truyền tải dữ liệu an toàn trên Internet. Nó được sử dụng để bảo mật việc truyền tải thông tin giữa trình duyệt web và máy chủ web.

HTTPS sử dụng giao thức mã hóa SSL/TLS để tạo ra một kênh truyền tải dữ liệu mã hóa giữa trình duyệt web của người dùng và máy chủ web. Điều này đảm bảo rằng thông tin được truyền tải an toàn và không bị lộ ra ngoài cho bất kỳ ai khác.

Khi trang web sử dụng HTTPS, trình duyệt của người dùng sẽ hiển thị một biểu tượng khóa màu xanh hoặc một biểu tượng "An toàn" trên thanh địa chỉ. Điều này cho phép người dùng biết rằng họ đang kết nối với một trang web được bảo vệ bằng HTTPS và thông tin của họ đang được bảo mật.

# Cách Hoạt Động Của HTTPS

![Imgur](https://i.imgur.com/0Vlp1QE.png)

Dữ liệu được mã hóa và giải mã như thế nào?

Bước 1 - Client (trình duyệt) và Server thiết lập kết nối TCP.

Bước 2 - Client gửi “client hello” đến Server. Tin nhắn chứa một tập hợp các thuật toán mã hóa cần thiết và phiên bản TLS mới nhất mà nó có thể hỗ trợ. Server phản hồi bằng “server hello” để trình duyệt biết liệu nó có thể hỗ trợ các thuật toán và phiên bản TLS hay không.

Sau đó, Server sẽ gửi chứng chỉ SSL cho client. Chứng chỉ chứa khóa công khai, tên máy chủ, ngày hết hạn, v.v. Client xác thực chứng chỉ.

Bước 3 - Sau khi xác thực chứng chỉ SSL, client sẽ tạo khóa phiên và mã hóa nó bằng khóa chung. Server nhận khóa phiên được mã hóa và giải mã nó bằng khóa riêng.

Bước 4 - Bây giờ cả client và server đều giữ cùng một khóa phiên (mã hóa đối xứng), dữ liệu được mã hóa được truyền trong một kênh hai chiều an toàn.

Tại sao HTTPS chuyển sang mã hóa đối xứng trong quá trình truyền dữ liệu? Có hai lý do chính:

1. Bảo mật: Mã hóa bất đối xứng chỉ có một chiều. Điều này có nghĩa là nếu server cố gửi dữ liệu được mã hóa trở lại client, thì bất kỳ ai cũng có thể giải mã dữ liệu bằng khóa chung.

2. Tài nguyên server: Mã hóa bất đối xứng tạo thêm khá nhiều chi phí. Nó không phù hợp để truyền dữ liệu trong các phiên dài.
