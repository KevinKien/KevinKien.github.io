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

sau khi copy chúng ra được giá trị

```
0x6080604052603f8060116000396000f3fe6080604052600080fdfea2646970667358221220d03248cf82928931c158551724bebac67e407e6f3f324f930c4cf1c36e16328764736f6c63430008110033
```

Đoạn code trên được chia thành 2 phần

![](https://static.wixstatic.com/media/935a00_b7ef73f4ed484fea99a315c0efd0f691~mv2.png/v1/fill/w_740,h_240,al_c,q_85,usm_0.66_1.00_0.01,enc_auto/935a00_b7ef73f4ed484fea99a315c0efd0f691~mv2.png)

Nó có vẻ giống như chúng ta chia bytecode ở 1 nơi nào đó ngẫu nhiên, nhưng nó sẽ được giải thích cụ thể sau

Nếu copy và paste phần đầu tiên vào evm code, và convert bytecode thành mnemonics, chúng ta lấy được [output](https://www.evm.codes/playground?fork=merge&unit=Wei&codeType=Bytecode&code=%276080604052603f8060116000396000f3fe%27) 

```
// allocate free memory pointer
PUSH1 0x80
PUSH1 0x40
MSTORE

// length of the runtime code
PUSH1 0x3f 
DUP1

// where the runtime code begins
PUSH1 0x11 
PUSH1 0x00// copy the runtime code from calldata into memory
CODECOPY

// runtime code is deployed at this step
PUSH1 0x00
RETURN
INVALID
```

Trong phần code được highlight, được gọi là runtime code, có kích thước là 63 bytes (0x3f trong hexadecimal). Nó bắt đầu tại 17th (0x11 trong hexadecimal) trong bộ nhớ. Điều này giải thích nơi giá trị của 0x3f và 0x11 lấy từ trong mnemonics. 

Trên level cao hơn, theo dõi 3 hành động ở trong init code: 
- con trỏ bộ nhớ trống, cái mà theo dõi vị trí bộ nhớ còn trống tiếp theo để ghi.
- runtime code thì sẽ copy vào vị trị bộ nhớ đó sử dụn "CODECOPY" opcode
- Cuối cùng, vùng nhớ có chứa runtime code trả về EVM, lưu trữ hợp đồng mới theo dạng runtime bytecode.

## Non-payable constructor contract

```
pragma solidity 0.8.17;// optimizer: 200 runs
contract Minimal {
    constructor() {

    }
}
```

hãy nhìn vào bytecode khi constructor không có payable và nhìn sự khác biệt. Đây là kết quả biên dịch

```
6080604052348015600f57600080fd5b50603f80601d6000396000f3fe6080604052600080fdfea2646970667358221220a6271a05446e269126897aea62fd14e86be796da8d741df53bdefd75ceb4703564736f6c63430008070033
```

Chúng được chia thành init và runtime code

![](https://static.wixstatic.com/media/935a00_87200a2c332346488c67cc4d575205c7~mv2.png/v1/fill/w_740,h_148,al_c,q_85,usm_0.66_1.00_0.01,enc_auto/935a00_87200a2c332346488c67cc4d575205c7~mv2.png)

