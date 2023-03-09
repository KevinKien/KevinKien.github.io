---
layout: post
title: API Recon
date: 2023-3-5
categories: ["API", "Pentest"]
---

Recon là một bước rất quan trọng trong quá trình pentest và khi pentest API cũng vậy. Để tìm kiếm được các thông tin như tài liệu về API, các endpoint private hay các key để thực hiện xác thực, .... Do vậy, nhờ các thông tin khi mà mình recon được thì sẽ giúp cho việc pentest tìm kiếm các lỗ hổng dễ dàng hơn.

Khi thực hiện recon API thì có 2 cách để thực hiện là Passive Recon và Active Recon.

# Passive Recon

Passive API Reconnaissance là việc thu thập thông tin về một mục tiêu mà không tương tác trực tiếp với các hệ thống của mục tiêu. Khi sử dụng phương pháp này, mục tiêu là tìm và tài liệu hóa thông tin công khai về bề mặt tấn công của mục tiêu.

Thường thì, thu thập thông tin bằng cách sử dụng thông tin tình báo nguồn mở (OSINT), đó là dữ liệu được thu thập từ các nguồn có sẵn công khai. Tìm kiếm các điểm cuối API, thông tin thông tin xác thực, thông tin phiên bản, tài liệu API. Bất kỳ endpoint API nào được phát hiện sẽ trở thành mục tiêu trong quá trình active recon sau này. Thông tin liên quan đến xác thực sẽ giúp kiểm tra như một người dùng được xác thực hoặc như một quản trị viên. Tài liệu API sẽ cho bạn biết chính xác cách kiểm tra API mục tiêu. Cuối cùng, việc khám phá mục đích kinh doanh của API có thể cung cấp cho bạn cái nhìn về những lỗ hổng về logic kinh doanh có thể xảy ra.

Khi thu thập thông tin bằng OSINT, tình cờ phát hiện ra một lỗ hổng dữ liệu quan trọng, chẳng hạn như API keys, thông tin xác thực, JSON Web Tokens (JWT) và các bí mật khác có thể dẫn đến một chiến thắng ngay lập tức. Những phát hiện có rủi ro cao khác bao gồm rò rỉ thông tin cá nhân (PII) hoặc dữ liệu người dùng nhạy cảm, chẳng hạn như số Bảo hiểm xã hội (SSN), tên đầy đủ, địa chỉ email và thông tin thẻ tín dụng. Những phát hiện như thế này nên được báo cáo ngay lập tức vì chúng là một điểm yếu nguy hiểm.

Các kỹ thuật để thực hiện passice recon gồm Google Dorking, Git Dorking, API Repositories, Way Back Machine và Shodan.

## Google Dorking

Bằng cách search với google sẽ là 1 cách nhanh nhất để có thể tìm kiếm các thông tin về API cho mục tiêu cụ thể.

![Imgur](https://i.imgur.com/BIB9b51.png)

Tuy nhiên, trong nhiều trường hợp bạn sẽ không nhận được kết quả gì. Nếu có quá nhiều kết quả thì bạn sẽ sử dụng các kỹ thuật của Goolge Dorking để khám phá vụ thể hơn.

- inurl:"/wp-json/wp/v2/users"
- intitle:"index.of" intext:"api.txt"
- inurl:"/api/v1" intext:"index of /"
- ext:php inurl:"api.php?action="
- intitle:"index of" api_key OR "api key" OR apiKey -pool

## GitDorking

Github là một trong những nơi cực kỳ hữu ích khi tìm kiếm các endpoint hay các thông tin về API. Đây là nơi các lập trình viên lưu trữ code, cho nên sẽ có nhiều lập trình viên vô tình hoặc cố ý sẽ upload nhiều thông tin hữu ích cho kẻ tấn công lên như là API Key, JWT token, password, ...

Bạn có thể tìm kiếm các keyword sau trên github: “api key,” “api keys”, “apikey”, “authorization: Bearer”, “access_token”, “secret” hoặc “token”. Sau đó tìm kiếm trong các repo để tìm kiếm những thứ mà bạn có thể thấy.

![Imgur](https://i.imgur.com/HMJzWAz.png)

Code chứa source code hiện tại, tệp readme và các tệp khác. Tab này sẽ cung cấp tên của nhà phát triển cuối cùng đã committed với tệp, thời điểm commit, contributors và mã nguồn thực tế.

![Imgur](https://i.imgur.com/3k7vs3P.png)

Bạn cũng nên để ý cả những commit từ history, trong đó đôi khi bạn sẽ tìm kiếm được nhiều điều thú vị.

![Imgur](https://i.imgur.com/szDO67p.png)

## TruffleHog 

Đây là 1 công cụ giúp bạn tìm các secret, api key, ... trong các source code và công cụ cũng hỗ trợ scan các repo luôn bằng lệnh sau:

```
sudo docker run -it -v "$PWD:/pwd" trufflesecurity/trufflehog:latest github --org=target-name
```

![Imgur](https://i.imgur.com/Pv1RcOe.png)

## API Directories

Programmableweb.com là nguồn cung cấp thông tin liên quan đến API (https://www.programmableweb.com/apis/directory). Để thu thập thông tin về mục tiêu của bạn, hãy sử dụng Thư mục API, cơ sở dữ liệu có thể tìm kiếm được với hơn 23.000 API. Bạn có thể tìm các endpoint API, thông tin phiên bản, thông tin logic nghiệp vụ, trạng thái của API, mã nguồn, SDK, bài viết, tài liệu API và nhật ký thay đổi.

![Imgur](https://i.imgur.com/BDE51lS.png)

![Imgur](https://i.imgur.com/uTLtjTv.png)

## Shodan

Shodan thường xuyên quét toàn bộ IPv4 trên toàn thế giới để tìm các hệ thống có port open và công khai thông tin thu thập được trên https://shodan.io. Bạn có thể sử dụng Shodan để khám phá các API và nhận thông tin về các cổng đang mở của mục tiêu, điều này sẽ hữu ích nếu bạn chỉ có một địa chỉ IP hoặc tên của tổ chức để làm việc. 

- hostname:"targetname.com"
- "content-type: application/json"
- "content-type: application/xml"
- "wp-json"

## The Wayback Machine

Wayback Machine là kho lưu trữ các trang web khác nhau theo thời gian. Điều này rất tốt cho việc theo dõi Passive API vì điều này cho phép bạn kiểm tra các thay đổi lịch sử đối với mục tiêu của mình. Dựa vào đó bạn có thể tìm kiếm các API. Nếu API không được quản lý tốt theo thời gian, thì có khả năng bạn có thể tìm thấy các điểm cuối đã ngừng hoạt động vẫn tồn tại mặc dù nhà cung cấp API tin rằng chúng đã ngừng hoạt động. Chúng được gọi là API Zombie.

![Imgur](https://i.imgur.com/OvQdbbr.png)

# Active Recon

Là quá trình tương tác trực tiếp với mục tiêu chủ yếu thông qua việc sử dụng các công cụ scan để tìm kiếm các API của mục tiêu và bất kỳ thông tin hữu ích nào. Các công cụ giúp cho active recon như: nmap, OWASP Amass, gobuster, kiterunner và DevTools.

## Nmap

Nếu bạn làm về pentest thì không quá còn quá xa lạ với nmap, nó sẽ giúp bạn scan các port đang được open trên server. 

```
nmap -sV --script=http-enum <target> -p 80,443,8000,8080
```

## Directory Brute-force with Gobuster

Gobuster sẽ giúp cho bạn thực hiện brute force các URI của mục tiêu và cả subdomain nữa. Và từ đó giúp bạn tìm kiếm ra các URI ẩn hoặc các subdomain mà bạn chưa biết. 

```
$ gobuster dir -u target-name.com:8000 -w /home/hapihacker/api/wordlists/common_apis_160
========================================================
Gobuster
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
========================================================
[+] Url:                     http://192.168.195.132:8000
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /home/hapihacker/api/wordlists/common_apis_160
[+] Negative Status codes:   404
[+] User Agent:              gobuster
[+] Timeout:                 10s
========================================================
09:40:11 Starting gobuster in directory enumeration mode
========================================================
/api                (Status: 200) [Size: 253]
/admin                (Status: 500) [Size: 1179]
/admins               (Status: 500) [Size: 1179]
/login                (Status: 200) [Size: 2833]
/register             (Status: 200) [Size: 2846]
```

## Kiterunner

Kiterunner hiện là công cụ tốt nhất để khám phá các tài nguyên và điểm cuối API. Mặc dù các công cụ cũng brute force như Gobuster để khám phá các đường dẫn URL, nhưng nó thường dựa vào các yêu cầu HTTP GET tiêu chuẩn. Kiterunner sẽ không chỉ sử dụng tất cả các phương thức yêu cầu HTTP phổ biến với API (GET, POST, PUT và DELETE) mà còn bắt chước các cấu trúc đường dẫn API phổ biến. Nói cách khác, thay vì yêu cầu GET /api/v1/user/create, Kiterunner sẽ thử POST /api/v1/user/create, bắt chước một yêu cầu thực tế hơn.

```
kr scan HTTP://127.0.0.1 -w ~/api/wordlists/data/kiterunner/routes-large.kite
```

![Imgur](https://i.imgur.com/xRsZZvL.png)

## DevTools

Một công cụ ngay trên trình duyệt, nó sẽ giúp bạn tìm kiếm các API mà website try cập vào. Từ DevTools bạn có thể search các keyword như api, graphql. 

![Imgur](https://i.imgur.com/xVMl0Je.png)

## OWASP Amass

OWASP Amass là một công cụ dòng lệnh có thể ánh xạ mạng bên ngoài của mục tiêu bằng cách thu thập OSINT từ hơn 55 nguồn khác nhau. Bạn có thể đặt nó để thực hiện quét passice hoặc active. Amass sẽ thu thập dữ liệu từ các công cụ tìm kiếm (chẳng hạn như Google, Bing và HackerOne), các nguồn chứng chỉ SSL (chẳng hạn như GoogleCT, Censys và FacebookCT), các API tìm kiếm (chẳng hạn như Shodan, AlienVault, Cloudflare và GitHub) và kho lưu trữ web Wayback.

Qua đó, thì bạn hãy tận dụng công cụ để thực hiện scan các subdomain hay brute force các URI về API.

```
amass enum -active -d target-name.com |grep api
legacy-api.target-name.com
api1-backup.target-name.com
api3-backup.target-name.com
```

```
amass enum -active -brute -w /usr/share/wordlists/API_superlist -d [target domain] -dir [directory name] 
```
