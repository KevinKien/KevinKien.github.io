---
layout: post
title: Cách Hoạt Động Của CDN
date: 2023-3-3
categories: ["system"]
---

CDN là viết tắt của Content Delivery Network, là một hệ thống các máy chủ được phân bố trên khắp thế giới, được sử dụng để phân phối nội dung trên Internet đến người dùng cuối nhanh hơn và ổn định hơn.

Khi một người dùng truy cập một trang web, yêu cầu tải các tài nguyên từ máy chủ web chứa trang đó, chẳng hạn như hình ảnh, video, CSS, JavaScript, vv. Khi sử dụng CDN, thay vì yêu cầu tải các tài nguyên từ một máy chủ duy nhất, người dùng sẽ tải các tài nguyên từ máy chủ CDN gần nhất với vị trí của họ trên thế giới. Điều này giúp tăng tốc độ tải trang web, giảm độ trễ và tăng trải nghiệm của người dùng.

CDN có thể được sử dụng để phân phối nội dung đa phương tiện lớn như video trực tuyến, trò chơi trực tuyến và các ứng dụng web phức tạp khác. Các công ty sử dụng CDN để tăng tốc độ và độ ổn định của trang web của họ, đồng thời giảm chi phí băng thông và tải cho máy chủ của họ.

# Cách Hoạt Động Của CDN

![Imgur](https://i.imgur.com/L85xIRm.png)

1. Bob gõ www.myshop.com trong trình duyệt. Trình duyệt tra cứu tên miền trong bộ đệm DNS cục bộ.

2. Nếu tên miền không tồn tại trong bộ đệm DNS cục bộ, trình duyệt sẽ chuyển đến trình phân giải DNS để phân giải tên miền. Trình phân giải DNS thường nằm trong Nhà cung cấp dịch vụ Internet (ISP).

3. Trình phân giải DNS phân giải tên miền theo cách đệ quy. Cuối cùng, nó yêu cầu máy chủ tên có thẩm quyền giải quyết tên miền.

4. Nếu không sử dụng CDN, máy chủ tên có thẩm quyền sẽ trả về địa chỉ IP cho www.myshop.com. Nhưng với CDN, máy chủ có thẩm quyền trỏ đến www.myshop.cdn.com (tên miền của máy chủ CDN).

5. Trình phân giải DNS yêu cầu máy chủ tên có thẩm quyền phân giải www.myshop.cdn.com.

6. Máy chủ định danh có thẩm quyền trả về tên miền cho bộ cân bằng tải của CDN www.myshop.lb.com.

7. Trình phân giải DNS yêu cầu bộ cân bằng tải CDN giải quyết www.myshop.lb.com. Bộ cân bằng tải chọn máy chủ edge CDN tối ưu dựa trên địa chỉ IP của người dùng, ISP của người dùng, nội dung được yêu cầu và tải của máy chủ.

8. Bộ cân bằng tải CDN trả về địa chỉ IP của máy chủ edge CDN cho www.myshop.lb.com.

9. Bây giờ cuối cùng chúng ta cũng có được địa chỉ IP thực để truy cập. Trình phân giải DNS trả lại địa chỉ IP cho trình duyệt.

10. Trình duyệt truy cập máy chủ edge CDN để tải nội dung. Có hai loại nội dung được lưu trong bộ nhớ cache trên máy chủ CDN: nội dung tĩnh và nội dung động. Cái trước chứa các trang tĩnh, hình ảnh, video; cái thứ hai bao gồm các kết quả của tính toán.

11. Nếu bộ nhớ cache của máy chủ edge CDN không chứa nội dung, nó sẽ chuyển lên máy chủ CDN khu vực. Nếu nội dung vẫn không được tìm thấy, nó sẽ chuyển lên máy chủ CDN trung tâm hoặc thậm chí chuyển đến máy chủ web. Đây được gọi là mạng phân phối CDN, nơi các máy chủ được triển khai theo địa lý.
