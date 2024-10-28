---
layout: post
title: Leetcode Practice
date: 2024-10-28
categories: ["Leetcode"]
---

# 1. Reverse vowels of a string

## Đề bài: 
Cho 1 string `s`, Đảo ngược tất cả các ký tự vowels trong string và return lại string đã được đảo ngược.
Vowels là các ký tư sau: 'a', 'e', 'i', 'o', và 'u'. Bao gồm cả chữ in hoa và in thường. 

Ví dụ 1: 
- Input: s = "IceCreAm"
- Output: "AceCreIm"

Các ký tự vowels trong trong s là I, e, e, A. Sau khi đảo ngược sẽ được kết quả "AceCreIm"

Ví dụ 2: 
- Input: s = "leetcode"
- Output: "leotcede"

## Lời giải: 

### Ý tưởng: 
- Xác định các ký tự vowels trong string
- Đảo ngược các ký tự vowels đó
- Ghi đè vào các vị trí trong string

### Code

```
def solution(s):
    vowels = set('aeiouAEIOU')
    vowels_chars = [char for char in s if char in vowels]

    result = []

    for char in s:
        if char in vowels:
            result.append(vowels_chars.pop())
        else:
            result.append(char)
    
    return ''.join(result)
```
