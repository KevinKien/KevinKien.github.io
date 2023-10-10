---
layout: post
title: Ethereum smart contract creation code
date: 2023-10-10
categories: ["Smart Contract"]
---

# Mở đầu

Ở 1 cấp độ cao hơn, Ví triển khai hợp đồng gửi 1 giao dịch tới null address với dữ liệu giao dịch được gồm có 3 phần: 

```
<init code> <runtime code> <constructor parameters>
```

Chúng được gọi là ```creation code```. EVM bắt đầu thực thi init code. Nếu init code được encoded đúng cách, việc thực thi này sẽ được lưu vào ```runtime code``` trên blockchain. 

Không có gì bên trong đặc tả EVM
