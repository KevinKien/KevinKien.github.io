---
layout: post
title: HTB BlackSky Hailstorm
date: 2024-11-15
categories: ["AWS Security", "Pentest"]
---

# Summary

Các bài BlackSky Hailstorm của HTB tập chung chủ yếu là pentest và tìm bug trên ứng dụng được xây dựng dựa trên hạ tầng của AWS. Qua bài này mình cũng học được nhiều tư duy về các lỗ hổng misconfigure trên AWS và làm sao để bảo vệ, audit hay là best practice cho các dịch vụ trên AWS.

# Write up

## Grand Leakage

Bài này thì khá đơn giản, truy cập vào IP mà đề bài cho rồi thực hiện view source thì mình thấy file css vs js được lấy từ link s3 public. 

![](https://github.com/KevinKien/KevinKien.github.io/blob/main/assets/img/grandleak1.png?raw=true)

Mình sẽ dùng aws cli thực hiện xem bucket s3 này có những file gì. 

![](https://github.com/KevinKien/KevinKien.github.io/blob/main/assets/img/grandleak2.png?raw=true)

Khá đơn giản vì s3 public nên mình có thể download file flag.txt về và lấy được flag với lệnh:

```
aws s3 cp s3://bucket/flag.txt . --no-sign-request
```

## Just a tester

Ở trong bucket s3, mình thấy thư mục `webadmin` mình sẽ truy cập vào để xem có những file gì? 

![](https://raw.githubusercontent.com/KevinKien/KevinKien.github.io/refs/heads/main/assets/img/justatester1.png)

2 file public key và private key được để ngay trong này, mình sẽ lấy về thử ssh xem được không.

![](https://raw.githubusercontent.com/KevinKien/KevinKien.github.io/refs/heads/main/assets/img/justatester2.png)

Sau khi kéo 2 file ssh key về mình chạy `chmod 600 id_rsa` và sau đó thử thực hiện ssh. 

![](https://raw.githubusercontent.com/KevinKien/KevinKien.github.io/refs/heads/main/assets/img/justatester3.png)

Done, flag ở ngay trong thư mục ngoài luôn.

## Meta-reality

Từ bài trên mình đã vào được server, vì là ec2 của AWS thì mình sẽ tìm kiếm IAM role thông qua IMDS (Instance Metadata Service). Mình run câu lệnh sau:

```
curl http://169.254.169.254/latest/meta-data/
```

Sau khi chạy lệnh trên mình được kết quả sau

![](https://raw.githubusercontent.com/KevinKien/KevinKien.github.io/refs/heads/main/assets/img/Meta-reality1.png)

Từ đây mình có thể lấy được role name trong IAM/security-credentials với câu lệnh sau

```
curl http://169.254.169.254/latest/meta-data/iam/security-credentials
```

Sau khi lấy được role name thì mình get credentials

![](https://raw.githubusercontent.com/KevinKien/KevinKien.github.io/refs/heads/main/assets/img/Meta-reality2.png)

Sau đó mình gán vào biến trên hệ điều hành để dễ dàng call

```
webadmin@ip-10-0-0-9:~$ export AWS_ACCESS_KEY_ID=ASIA4Sxxxxxx
webadmin@ip-10-0-0-9:~$ export AWS_SECRET_ACCESS_KEY=OYEw6HTm3Axxxxx
webadmin@ip-10-0-0-9:~$ export AWS_SESSION_TOKEN=IQoJb3JpZ2xxxxxx
```

Mình chạy lệnh sau để thực hiện xem kỹ hơn về iam role `web01`

![](https://raw.githubusercontent.com/KevinKien/KevinKien.github.io/refs/heads/main/assets/img/Meta-reality3.png)

Lệnh trên thì mình lấy được account id là `864899859472`

Mình tiếp tục sử dụng tool để recon xem có các dịch vụ nào đang gán cho role `web01` này bằng tool `weirdAAL` với lệnh: 

```
python3 weirdAAL.py -m recon_all -t web01
```

Sau khi chạy xong thì mình lấy được `DesscribeSnapshot` đang allow trên EC2. Mình sẽ thực hiện list `desscribe-snapshots` xem có những thông tin gì

![](https://raw.githubusercontent.com/KevinKien/KevinKien.github.io/refs/heads/main/assets/img/Meta-reality5.png)

Tại đây mình thấy được là con EC2 này được cấu hình làm proxy và mình thấy được snapshot này được owner đang public. Từ đây mình có thể tìm xem được là thuộc tính `CreateVolumePermissions` có được public hay không.

![](https://raw.githubusercontent.com/KevinKien/KevinKien.github.io/refs/heads/main/assets/img/Meta-reality6.png)

Snapshot này cho phép tất cả mọi người có thể tạo EC2 từ snapshot này. 

Mình thực hiện tạo 1 instance trên aws và thêm volume từ snap id ở trên, vì snapshot public nên mình có thể thực hiện gán vào instance của mình. Sau đó mình sẽ mount vào volume từ instance mới tạo và lấy các file trên đó.

![](https://raw.githubusercontent.com/KevinKien/KevinKien.github.io/refs/heads/main/assets/img/Meta-reality7.png)

Sau khi tạo xong thì mình vào server và mount vào volume từ snapshot trên.

![](https://raw.githubusercontent.com/KevinKien/KevinKien.github.io/refs/heads/main/assets/img/Meta-reality8.png)

Từ đó mình vào lấy được ssh key của user root.

![](https://raw.githubusercontent.com/KevinKien/KevinKien.github.io/refs/heads/main/assets/img/Meta-reality9.png)

Từ đây thì khá là dễ dàng :))))

## That's an unexpected one

Sau khi có root ở trên, mình tìm kiếm các file trên server thì mình thấy được code trong thư mục `/opt/deployment`

Mình truy cập thì thấy thư mục .git, tuy nhiên trong code không có nhiều thông tin lắm, mục đích của mình vẫn là làm sao để lấy được AWS key. 

Mình có ý tưởng là check git log xem thế nào

![](https://raw.githubusercontent.com/KevinKien/KevinKien.github.io/refs/heads/main/assets/img/thatit1.png)

Yeah!!, có 1 đoạn mô tả 'remove env config'

Mình thử đọc đoạn commit đó xem nội dung như thế nào

![](https://raw.githubusercontent.com/KevinKien/KevinKien.github.io/refs/heads/main/assets/img/thatit2.png)

vậy là mình lấy được AWS key và thực hiện cấu hình key vào aws cli bằng lệnh

```
aws configure
```

Mình thực hiện call identity xem key này được gắn cho user nào

![](https://raw.githubusercontent.com/KevinKien/KevinKien.github.io/refs/heads/main/assets/img/thatit3.png)

User cho role này là `daneil`, mình export vào biến môi trường trên server để thực hiện chạy rool recon `weirdAAL` và được kết quả

![](https://raw.githubusercontent.com/KevinKien/KevinKien.github.io/refs/heads/main/assets/img/thatit4.png)

Gateway API được cho phép call từ bên ngoài. Mình sẽ tìm xem url cho api này là như thế nào

![](https://raw.githubusercontent.com/KevinKien/KevinKien.github.io/refs/heads/main/assets/img/thatit5.png)

theo như tài liệu của aws thì url mặc định sẽ là

```
https://{api_id}.execute-api.{region}.amazonaws.com
```

từ đó mình có thể tạo lại link API như sau:

```
[https://pe1rbyoaod.execute-api.us-east-2.amazonaws.com](https://pe1rbyoaod.execute-api.us-east-2.amazonaws.com/)
```

Tiếp đến mình thực hiện tìm xem stage_name của API là gì

![](https://raw.githubusercontent.com/KevinKien/KevinKien.github.io/refs/heads/main/assets/img/thatit6.png)

stage_name được đặt mặc định là `default`

Vậy mình thực hiện truy vấn API như sau

![](https://raw.githubusercontent.com/KevinKien/KevinKien.github.io/refs/heads/main/assets/img/thatit7.png)

OK, đã được mình thực hiện đặt authen là aws key và set key của daniel vào header

![](https://raw.githubusercontent.com/KevinKien/KevinKien.github.io/refs/heads/main/assets/img/thatit8.png)

Tuy nhiên vẫn lỗi, mình thực hiện chuyển sang method `POST`

![](https://raw.githubusercontent.com/KevinKien/KevinKien.github.io/refs/heads/main/assets/img/thatit9.png)

Như trong code trong thư mục /opt/deployment/Freight-Tracking.php mình có thấy đoạn sau

```
$result = $client->testInvokeMethod([
    'body' => 'url=http://10.0.0.11/jobs/invoice.docx',
    'clientCertificateId' => '103248',
    'httpMethod' => 'POST',
    'pathWithQueryString' => '/',
    'resourceId' => $_ENV['resourceId'],
    'restApiId' => $_ENV['restApiId'],
    'stageVariables' => ['default','staging','prod']
]);
```

Mình thử thêm biến url vào trong url với 1 link google xem

![](https://raw.githubusercontent.com/KevinKien/KevinKien.github.io/refs/heads/main/assets/img/thatit10.png)

Response trả về nội dung của site google,com. Đoạn này mình thử với lỗi SSRF xem. thực hiện đọc file /etc/passwd

![](https://raw.githubusercontent.com/KevinKien/KevinKien.github.io/refs/heads/main/assets/img/thatit11.png)

OK, great thế là có lỗi SSRF và có thể đọc được file trên hệ thống. Giờ thì get key và flag thôi.

![](https://raw.githubusercontent.com/KevinKien/KevinKien.github.io/refs/heads/main/assets/img/thatit12.png)

## Variety of policies

Nào check xem key trên identity như thế nào

![](https://raw.githubusercontent.com/KevinKien/KevinKien.github.io/refs/heads/main/assets/img/varietypolicy.png)


![](https://raw.githubusercontent.com/KevinKien/KevinKien.github.io/refs/heads/main/assets/img/varietypolicy2.png)



![](https://raw.githubusercontent.com/KevinKien/KevinKien.github.io/refs/heads/main/assets/img/varietypolicy3.png)



![](https://raw.githubusercontent.com/KevinKien/KevinKien.github.io/refs/heads/main/assets/img/varietypolicy4.png)



![](https://raw.githubusercontent.com/KevinKien/KevinKien.github.io/refs/heads/main/assets/img/varietypolicy5.png)
