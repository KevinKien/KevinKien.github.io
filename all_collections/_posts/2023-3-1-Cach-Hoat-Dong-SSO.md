---
layout: post
title: Cách Hoạt Động SSO
date: 2023-3-1
categories: ["system"]
---

# SSO (Single Sign-On) là gì?

SSO (Single Sign-On) là một phương thức xác thực duy nhất cho phép người dùng đăng nhập một lần và truy cập vào nhiều hệ thống khác nhau mà không cần phải nhập lại thông tin đăng nhập. Điều này giúp giảm thiểu sự phiền toái và thời gian mất để đăng nhập vào nhiều hệ thống và tăng tính bảo mật thông tin người dùng bằng cách sử dụng các phương thức xác thực như mã thông báo truy cập hoặc chứng chỉ số.

Các bạn cũng hay gặp SSO trong khi sử dụng một trang web bất kỳ và khi thực hiện login bạn sẽ được yêu cầu thực hiện đăng nhập bằng google hay facebook hay là github. Thì đó chính là trang web đã sử dụng SSO. 

# Cách hoạt động của SSO như thế nào

![Imgur](https://i.imgur.com/xIdpZ9q.png)

Ở hình trên là cách hoạt động của SSO:

- Bước 1: Người dùng truy cập Gmail hoặc bất kỳ dịch vụ email nào. Gmail nhận thấy người dùng chưa đăng nhập và do đó chuyển hướng họ đến máy chủ xác thực SSO, máy chủ này cũng tìm thấy người dùng chưa đăng nhập. Kết quả là người dùng được chuyển hướng đến trang đăng nhập SSO, nơi họ nhập thông tin đăng nhập của mình.

- Bước 2-3: Máy chủ xác thực SSO xác thực thông tin đăng nhập, tạo phiên chung cho người dùng và tạo mã thông báo.

- Bước 4-7: Gmail xác thực mã thông báo trong máy chủ xác thực SSO. Máy chủ xác thực đăng ký hệ thống Gmail và trả về “hợp lệ”. Gmail trả lại tài nguyên được bảo vệ cho người dùng.

- Bước 8: Từ Gmail, người dùng điều hướng đến một trang web khác do Google sở hữu, chẳng hạn YouTube.

- Bước 9-10: YouTube nhận thấy người dùng chưa đăng nhập và sau đó yêu cầu xác thực. Máy chủ xác thực SSO tìm thấy người dùng đã đăng nhập và trả về mã thông báo.

- Bước 11-14: YouTube xác thực mã thông báo trong máy chủ xác thực SSO. Máy chủ xác thực đăng ký hệ thống YouTube và trả về "hợp lệ". YouTube cho phép người dùng sử dụng những chức năng của mình.
