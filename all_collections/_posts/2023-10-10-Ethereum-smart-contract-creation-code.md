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

Không có gì bên trong đặc tả EVM, điều đó nói rằng bố cục cần phải init code, runtime code và constructor parameters. Nó có thể là init code, constructor parameters và sau đó runtime code. Điều này là quy ước đơn giản để solidity sử dụng. Tuy nhiên, init code cần là phần đầu tiên cho EVM để biết được nơi bắt đầu thực thi. 

# Solidity creationCode

Solidity có 1 cơ chế để lấy bytecode sẽ được triển khai trong smart contract creation transaction thông qua creationCode keyword. 

Điều này không bao gồm constructor arguments, cái sẽ bao gồm 1 phần của bytecode chạy trong triển khai hợp đồng. Init Code  (creationCode) và arguments có cấu trúc như thế nào sẽ được giải thích trong bài viết này.

```
contract ValueStorage {
    uint256 public value;
    constructor(uint256 value_) {
        value = value_;
    }
}

contract GetCreationCode {
    function get() external returns (bytes memory creationCode) {
        creationCode = type(Simple).creationCode;
    }
}
```

# Init code

Init code là 1 phần của creation code chịu trách nhiệm cho triển khai 1 contract. 

## Payable constructor contract

```
pragma solidity 0.8.17;// optimizer: 200 runs

contract Minimal {
    constructor() payable {

    }
}
```

Sau khi nhận được kết quả biên dịch, copy giá trị input từ remix sau khi thực thi giao dịch triển khai. 

![](https://static.wixstatic.com/media/935a00_25371a89bdbb40228a009c0da2704f5c~mv2.png/v1/fill/w_740,h_399,al_c,q_85,usm_0.66_1.00_0.01,enc_auto/935a00_25371a89bdbb40228a009c0da2704f5c~mv2.png)


