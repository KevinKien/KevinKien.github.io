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


