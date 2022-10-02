---
title: "[Character set & Encoding]Part2. Python 在編碼上的操作與例子"
subtitle: "Character Encoding in Python"
date: 2021-12-12T18:36:14+08:00
draft: false
url: "posts/15/211212/Character_Encoding/Part2"
summary: <p align="center">上一篇紀錄了編碼與 Unicode 的脈絡與區別，這裡紀錄一下在 Python 上的操作轉換</p> <img src="/blog/images/15_211212/15_211212_python字符編碼操作例子.png" width="70%"/>
author: "linooohon"
email: "linooohon@gmail.com"
toc: true
tags: ["Encoding", "Decoding", "Unicode", "ACSII", "UTF/UCS", "OS", "Binary", "Hexadecimal", "Python"]
categories: ["2021", "encoding", "python"]
tocNum: false
---

本篇紀錄一下，字符(字串)編碼在 Python 上的實現。

## 前提:
- 基本編碼知識
- 理解 ASCII, Unicode, UTF-8 之間的差別


## Basic:

### 1. Python3 的字串是以什麼編碼？
    - UTF-8(實踐 Unicode)，所以 Python 支持多語言字串

```python
print("嗨Hiおはよう")
# 嗨Hiおはよう
```

### 2. `ord(str_)`: 得到對應的 Unicode 碼點，但是以十進位表示
```python
print(ord('A')) # 65
print(ord('a')) # 97
print(ord('林')) # 26519
```

- 用途:
    1. HTML Code
    ```htmlembedded
    # 65 -> &#65;
    # 97 -> &#97;
    # 26519 -> &#26519;
    <!-- 使用方式 -->
    <p>&#65;</p> 
    <p>&#97;</p>
    <p>&#26519;</p>
    ```


### 3. `format(ord(str_), 'x')`: 得到對應的 Unicode 碼點，搭配 format(), 將碼點轉換成十六進位，所以會得到 Unicode 最普遍使用的十六進位表示碼點
```python
print(format(ord('A'), 'x')) # 41 -> U+0041
print(format(ord('a'), 'x')) # 61 -> U+0061
print(format(ord('林'), 'x')) # 6797 -> U+6797
print(format(ord('🤢'), 'x')) # 1f922 -> U+1f922
print(format(ord('📸'), 'x')) # 1f4f8 -> U+1f4f8
print(format(ord('臺'), 'x')) # 81fa -> U+81fa
print(format(ord('灣'), 'x')) # 7063 -> U+7063
```

- 用途:
     1. 得到真正的 Unicode Code Point(主流的十六進位碼 Unicode HEX)
     ```
     # 41 -> U+0041
     # 61 -> U+0061
     # 6797 -> U+6797
     # 1f922 -> U+1f922
     # 1f4f8 -> U+1f4f8
     # 81fa -> U+81fa
     # 7063 -> U+7063
     ```
     2. 單純印出: `\u + 4碼` or `超過4碼補零到8碼，並改成大寫U`
     ```python
      print('\u0041')
      print('\u0061')
      print('\u6797')
      print('\U0001f922')
      print('\U0001f4f8')
      print('\u81fa')
      print('\u7063')
      print('\u81fa\u7063')
      
      """
      A
      a
      林
      🤢
      📸
      臺
      灣
      臺灣
      """
     ```
     3. HTML Code
     ```htmlembedded
      # 41 -> &#x41; or &#x0041;
      # 61 -> &#x61; or &#x0061;
      # 6797 -> &#x6797;
      # 1f922 -> &#x1f922;
      # 1f4f8 -> &#x1f4f8;
      
      <!-- 使用方式 -->
      <p>&#x41;</p> 
      <p>&#x61;</p>
      <p>&#x6797;</p>
      <p>&#x1f922;</p>
      <p>&#x1f4f8;</p>

      <!-- 顯示 -->
      A
      a
      林
      🤢
      📸
     ```
     4. CSS Code
     ```htmlembedded
     # 41 -> \41
     # 61 -> \61
     # 6797 -> \6797
     # 1f922 -> \1f922
     # 1f4f8 -> \1f4f8
     <!-- 使用方式 -->
    
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>Document</title>
        <style>
            .testing:hover:after {
                content: '\1f4f8'
            }
        </style>
    </head>
    <body>
        <p>&#x41;</p>
        <p>&#65;</p>
        <p class="testing">&#97;</p>
        <p>&#26519;</p>
        <p>&#x1f922;</p>
    </body>
    </html>
     ```
     

### 4. `chr(int_)`: 給 Unicode十進位，得到對應字串
```python
print(chr(65)) #A
print(chr(97)) # a
print(chr(26519)) # 林
```

### 5. `str_.encode(encodingrule_)`: 給字串然後取得對應編碼規則的編碼，並以 bytes 形式顯示，但如果是 ASCII 的守備範圍，那就不以 bytes 顯示，而是直接顯示。
```python
print('ABC'.encode('ascii'))
print('ABC'.encode('utf8'))
print(type('ABC'.encode('utf8')))
print('臺灣'.encode('utf8'))
print('臺灣'.encode('big5'))

"""
b'ABC'
b'ABC'
b'ABC'
<class 'bytes'>
b'\xe8\x87\xba\xe7\x81\xa3'
b'\xbbO\xc6W'
"""
```

### 6. `bytes_.decode(encodingrule_)` : 給 bytes 然後取得對應編碼規則的字串。

```python
print(b'ABC'.decode('ascii'))
print(b'ABC'.decode('utf8'))
print(b'\xe8\x87\xba\xe7\x81\xa3'.decode('utf8'))
print(b'\xbbO\xc6W'.decode('big5'))
print(b'\x82\xe0\x82\xb5\x82\xe0\x82\xb5'.decode('shift-jis'))

"""
ABC
ABC
臺灣
臺灣
もしもし
"""
```


### 7. `-*- coding: encodingrule_ -*-`

```python
# -*- coding: utf-8 -*-
此為 Python2 時，申明此文件是以 `UTF-8` 編碼, 基本上在 Python3 已不需要，除非有特殊需求，有不同編碼在專案中應用，才會使用。不然 Python3 by default 就是 `UTF-8` 編碼
```


```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
如果 shebang 有出現，那 coding 就放第二行
```

```python
# -*- coding: cp1252 -*-
如果想要其他編碼可以自己申明
```


### 8. 只要看到 `\x` 就知道是 byte 的呈現，一個 `\x`+`碼號` 就代表 1 byte

```python
print('臺灣'.encode('utf8'))
# b'\xe8\x87\xba\xe7\x81\xa3'

# 臺 佔 3 bytes \xe8 \x87 \xba
# 灣 佔 3 bytes \xe7 \x81 \xa3
# 所以臺灣共佔 6 bytes
# 那為什麼 6 bytes 請先了解 ASCII, Unicode, UTF-8 之間的關係，就會知道
```

### 9. `r'str_'`

以 r 開頭的字串，可以完整呈現每個字符
ex:
```python
print(r'123\n')
print(len(r'123\n'))
print('123\n')
print(len('123\n'))

# 會是這樣:
"""
123\n
5
123

4
"""
```


### 10. 特別釐清: Unicode 的碼點與 UTF-8 的編碼是兩回事，看一下在 Python 裡的範例，分辨差別。

(如果還不清楚編碼是什麼，可以先看這篇)

> <a href="/blog/posts/14/211206/Character_Encoding字元編碼/Part1">[Character set & Encoding]Part1.字元編碼 ASCII, Unicode, UCS, UTF</a>

依照前面的這些範例，
1. 先來看一下 `臺` 的 Unicode 的碼點怎麼獲取?

```python
string_ = "臺"
unicode_hex_code_point = format(ord(string_), 'x')
print(unicode_hex_code_point)
# 印出
# 81fa 
```
81fa 就是`臺`這個字 在 Unicode 的字符集定義中`臺`的碼點 -> U+81fa


2. 再來看一下 `臺` 的 UTF-8 實現的編碼怎麼獲取

```python
string_ = "臺"
bytes_ = string_.encode('utf-8')
print("".join([format(i, "x") for i in bytes_]))
# 印出
# e887ba
```

為了比較出差別，我這邊也把編碼過後的`臺` 換成 16進位表示法，得到 e887ba
所以可以看出來，同樣都是 16 進位表示，81fa 和 e887ba，完全不一樣，如果能了解這裡的區別，編碼的基本概念我覺得就算清楚了，基本上就是 Unicode 的碼位歸碼位，在 UTF-8 的規則下，還會再進行一次編碼，所以兩者長的不一樣。

同理，轉二進位比較也不一樣:
```python
string_ = "臺"
unicode_hex_code_point = format(ord(string_), 'b')
print(unicode_hex_code_point) # 1000000111111010

bytes_ = string_.encode('utf-8')
print(bytes_) # b'\xe8\x87\xba'
print("".join([format(i, "b") for i in bytes_])) # 111010001000011110111010
```


### 11. 一些轉換補充

1. 字符串轉 bytes, 16進位、2進位表示
```python
string_ = "臺" 
binary_converted = []
ba_ = bytearray(string_, "utf-8")
print(ba_) # bytearray(b'\xe8\x87\xba')
for i in ba_:
    binary_ = format(i, "x")
    binary_converted.append(binary_)
print("".join(binary_converted)) # e887ba
```

```python
string_ = "臺" 
map(bin, bytearray(string_, "utf-8"))
print("".join(map(bin, bytearray(string_, "utf-8")))) # 0b111010000b100001110b10111010
```

2. 十進位數字轉成以二進位表示字串

```python
print(bin(9))
print(type(bin(9)))

print(format(9, "b"))
print(type(format(9, "b")))

print("{0:b}".format(9))
print(type("{0:b}".format(9)))

"""
0b1001
<class 'str'>
1001
<class 'str'>
1001
<class 'str'>
"""

```

學習過程有參考網路資源，如紀錄中有雷同請見諒！
### 12. References:
- [Python Docs - Unicode HOWTO](https://docs.python.org/3/howto/unicode.html)
- [Python Docs - Literals](https://docs.python.org/3/reference/lexical_analysis.html#literals)


**< 上一篇 >**
<a href="/blog/posts/14/211206/Character_Encoding字元編碼/Part1">[Character set & Encoding]Part1.字元編碼 ASCII, Unicode, UCS, UTF</a>