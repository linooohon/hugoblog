---
title: "[Python]觀念(1) - Iteration, Iterable, Iterator"
subtitle: "Python 自我心得"
date: 2021-07-27T18:36:14+08:00
draft: false
url: "posts/07/210727/Python 觀念(1) - Iteration, Iterable, Iterator"
summary: <p align="center">Python Iteration 如何運行，for loop 本質 - Iteration, Iterable, Iterator</p> <img src="/blog/images/07_210727/07_210727_python_iteration.png" width="40%"/>
author: "linooohon"
email: "linooohon@gmail.com"
toc: true
tags: ["Python", "Iterator"]
categories: ["2021", "python"]
---


<h1 style="text-align:center">
Python 觀念紀錄 - Iteration, Iterable, Iterator
</h1>

> 在此對 Iteration, Iterable, Iterator 的觀念做個紀錄。

![Python 觀念紀錄 - Iteration, Iterable, Iterator](/blog/images/07_210727/07_210727_python_iteration.png)


## 了解 Iteration (迭代)

### **< Iteration >**: 
是個廣義名詞, 意思是每次去 "遍歷(走訪)" 1 個 object, 對裡面每個 item 做想操作的事的這個過程，ex: 像是在程式碼中操作 loop，我們可以說這個是一種 Iteration。

### **< Iterable, Iterator兩個名詞必須分清楚 >**:
- Iterable
- Iterator

#### Iterable (可迭代物件):
> 在 Python 中 Iterable 和 Iterator 有特別意義，先解釋 Iterable。

##### Python對它的定義: 

是一個物件，這個物件裡要有 `__iter__()` 方法或是 `__getitem__()` 方法(要能支持從 0 開始的index)，這兩個其中一個條件達成，它就是定義上的 Iterable。

> 為什麼要有這兩個方法，因為 Python 說要成為 Iterable 就必須支持 Iteration Protocol 或是 Sequence Protocol。

`__iter__()` 對應 Iteration Protocol。

`__getitem__()` 對應 Sequence Protocol。

##### 特徵:
1. 是一個物件。
2. 可以 for loop 遍歷的 objects 它都是定義上 Iterable 的範圍。


##### 基本上：
2. 當今天迭代的 Iterable 是 String, 那每次就是得到單個字符。
3. 當今天迭代的 Iterable 是 文件，那每次得到的就是每一行的字符串。
4. 當今天迭代的 Iterable 是 Dictionary, 那每次得到的就是該次的 Key。


```
dic, list, tuple, range, string, set, numpy array, dataframe, bytes 都是 Iterable
```

##### 那 `__iter__()` 和 `__getitem__()` 是什麼？
它們是 Python Magic Methods
`__iter__()`: [Docs](https://docs.python.org/3/reference/datamodel.html#object.__iter__)
`__getitem__()`: [Docs](https://docs.python.org/3/reference/datamodel.html#object.__getitem__)

當 `__iter__()` 被呼叫時，會 return 一個 Iterator(新的 Iterator)
當 `__getitem__()` 在 class 裡被定義, 使用object[index]形式, 就會呼叫該方法:
```python=
class Hi():
    def __init__(self, name):
        self.name = name
    def __getitem__(self, key):
        return "__getitem__ is called"

hi = Hi("Phil")
print(hi[2])  

# 輸出:
# __getitem__ is called
```

那什麼是 Iterator，接著講：


#### Iterator (迭代器):
- Iterator: [Docs](https://docs.python.org/3/library/stdtypes.html#iterator-types)

##### Python 對它的定義:

它是一個物件，這個物件裡要有 `__iter__()` 方法和 `__next__()` 方法，兩個方法都必須要有，那它就是定義上的 Iterator。

##### 特徵:
1. 是一個物件。
2. Iterable 其實可以 for loop 是因為 Python 私底下幫忙轉換為 Iterator 然後做處理。
3. Iterator 才可以調用 `next()`, Iterable 不行。


```
而, dic, list, tuple, range, string, set, numpy array, dataframe, bytes 都不是 Iterator
```

> 從下面程式碼可以發現他們裡面都沒有 `__next__()`，所以不會是 Iterator

```py
import numpy as np
import pandas as pd

# dic
dic_ = {'name': 'phil', 'phone': '0933333333'}
print("===============dic===============")
print(hasattr(dic_, '__iter__')) # True
print(hasattr(dic_, '__getitem__')) # True
print(hasattr(dic_, '__next__')) # False


# list
list_ = ['a', 1, {'key': 'value'}, (1, 2), {1, 2}]
print("===============list===============")
print(hasattr(list_, '__iter__')) # True
print(hasattr(list_, '__getitem__')) # True
print(hasattr(list_, '__next__')) # False


# tuple
tuple_ = ('a', {'key': 'value'}, 2)
print("===============tuple===============")
print(hasattr(tuple_, '__iter__')) # True
print(hasattr(tuple_, '__getitem__')) # True
print(hasattr(tuple_, '__next__')) # False

# range
range_ = range(1)
print("===============range===============")
print(hasattr(range_, '__iter__')) # True
print(hasattr(range_, '__getitem__')) # True
print(hasattr(range_, '__next__')) # False

# string
string_ = 'a'
print("===============string===============")
print(hasattr(string_, '__iter__')) # True
print(hasattr(string_, '__getitem__')) # True
print(hasattr(string_, '__next__')) # False

# set
set_ = {'a', 2, 3, (1, 2)}
print("===============set===============")
print(hasattr(set_, '__iter__')) # True
print(hasattr(set_, '__getitem__')) # False
print(hasattr(set_, '__next__')) # False

# numpy array
np_array_ = np.array([1, 2, 3, 4])
print("===============np_array===============")
print(hasattr(np_array_, '__iter__')) # True
print(hasattr(np_array_, '__getitem__')) # True
print(hasattr(np_array_, '__next__')) # False

# dataframe
df_ = pd.read_csv("1.csv")
print("===============dataframe===============")
print(hasattr(df_, '__iter__')) # True
print(hasattr(df_, '__getitem__')) # True
print(hasattr(df_, '__next__')) # False


# bytes
bytes_1 = bytes(1)
print("===============bytes===============")
print(hasattr(bytes_1, '__iter__')) # True
print(hasattr(bytes_1, '__getitem__')) # True
print(hasattr(bytes_1, '__next__')) # False

```


##### 那如何成為 Iterator:  `iter()`, `next()`
> 根據 Iterator 成立的條件，我們如果 call Builtin Function `iter()`
裡面帶 Iterable，就可以 return Iterator.

> 所以
1. `iter()` 會 return Iterator
2. Iterator 裡有 `next()` 所以可以使用 `next()`


> `next()` 的作用:

Iterator 可以被 `next()` 調用，並不斷的返回下一個資料，直到沒有資料的時候，會拋出 StopIteration error, 而這是什麼意思呢？等等再底下有個例子可以解釋。

在 Python Docs [Glossary](https://docs.python.org/3.9/glossary.html#term-iterator) 裡面，寫道，Iterator is an object representing a stream of data，可以看到 Iterator 是代表一個資料流的概念，也就是說程式它本身其實不知道整個長度，像是 `dict`, `list`, `str` 是一開始就知道長度的，但 Iterator 的概念是`流`, 每當我需要的時候，我才通過 `next()` 拿到下一個我要的產出(資料), 以我的觀點 Iterator 有點像是一個 被動、懶惰的代表, 我喊它，它才動, 我不喊它, 它永遠都不動，所以他的計算是被動的，只有在需要 return 下一個資料的時候它才會行動(計算)。


> ex: 範例
```python=
ss_ = "abc"
iter_ss_ = iter(ss_)
print(hasattr(iter_ss_, '__iter__')) # True
print(hasattr(iter_ss_, '__getitem__')) # False
print(hasattr(iter_ss_, '__next__')) # True

print(next(iter_ss_))
print(next(iter_ss_))
print(next(iter_ss_))


# 輸出:
"""
True
False
True
a
b
c
"""

```
> 底下這裡就會報錯因為不是 Iterator
```python=
ss_ = "abc"
next(ss_)

# 輸出
"""
True
False
True
Traceback (most recent call last):
TypeError: 'str' object is not an iterator
"""
```

### **< 關於 `iter()`, `__iter__()`, `next()`, `__next__()` >** :

最一開始我也搞的滿模糊的，但要理解可以先拆分：

#### Built-in Function: `iter()` `next()`
`iter()`, `next()` : Python Built-In Function [Docs](https://docs.python.org/3/library/functions.html)

所謂 Built-In Function, 你只要 call 它，Python 就會照著他原本寫好的劇本去跑。

#### Magic Methods: `__iter__()` `__next__()`
`__iter__()`, `__next__()`: Python Magic Methods(Dunders)

所謂 Magic Methods, 就是由 class 裡所提供的一些 methods, Python 將這些 methods 特徵弄成前後各兩個底線，而這些 methods 之所以很 magic(自認為..)，是因為..其實我們做的很多事情，都是他們幫我們代勞的。

```
代勞是什麼意思呢？
```

看一下這個例子:
```py
number = 1
print(number + 2)   #3
print(number.__add__(2)) #3

# 會發現都是 print 出 3
# 當我們用 Int 這個 type 做加法, `+` 和 使用 `__add__()` 是一樣的結果。
```

```py
如果我們用 `help()` 看一下 Int class
會發現它所定義的東西就是 `+` 要做的事
"""
__add__(self, value, /)
    Return self+value.
"""
```

#### 那意思是我們可以自己客製化 object 修改這個 object 的 `+` 的功能
```py
class CustomPlus:
    def __init__(self, num):
        self.num = num
    def __add__(self, value):
        print("__add__ is called ")
        return self.num - value

cp = CustomPlus(1)
print(cp.__add__(2))
# __add__ is called 
# -1

print(cp + 2)
# __add__ is called 
# -1
# 可以發現 cp + 2 也是走 class 裡的 __add__() 所以結果就也會變成 -1
```

#### 所以，結論是:

```py
使用 `+` 的背後，其實 python 是去找 `Int` class 所定義的 `__add__()` 然後叫它做事
```

#### 在 Python 中有數不清像 `+` 這樣由 `__add__()` 代勞的事情，
詳細可以看 [Docs 3. Data model 這頁](https://docs.python.org/3/reference/datamodel.html#special-method-names)

#### 那同理，回到主題:
其實 `iter()` `next()` 是會去找當下的 type 所能使用的 `__iter__()`, `__next__()` 代勞，白話一點就是 call `iter()` `next()` 的下一瞬間，Python 就是會設法 call `__iter__()`, `__next__()`。


#### 一個小結論:
```py
call `iter()` `next()` 可以說是等價於 call `__iter__()`, `__next__()`
但,
call `__iter__()`, `__next__()` 不等於 call `iter()` `next()`
因為就像上面說的 magic methods 可以客製化
但 Built-In Function 就是照著劇本走，
所以當你把 `__iter__()`, `__next__()` 改成不照著 `iter()` `next()` 劇本規定的時後，再來使用 `iter()` `next()` 就肯定會報錯
看一下底下 `iter()` 的例子
```

#### 用 `iter()` 再舉個例子:

```py
s = "abc"
iter_s = iter(s)
__iter__s = s.__iter__()
print(type(iter_s))
print(type(__iter__s))

# 輸出
"""
<class 'str_iterator'>
<class 'str_iterator'>
"""
```
```
這裡沒問題，兩個就是做一樣的事，但如果底下我們自定義呢?
```

> 這裡自訂義 class Foo:

```py
class Foo:
    def __iter__(self):
        return "Foo"

foo = Foo()
print(foo.__iter__())

# 輸出不會報錯
# Foo 
```
> 如果改成 `iter()` 這時底下這裡就會出錯，因為 `iter()` 的劇本就是堅持要拿到 `Iterator`, 你給他 `str` 就肯定噴錯
```py
class Foo:
    def __iter__(self):
        return "Foo"

foo = Foo()
print(iter(foo))

# 輸出會報錯
"""
Traceback (most recent call last):
print(iter(foo))
TypeError: iter() returned non-iterator of type 'str'
"""

```


> 小結論: 
1. 所有可 for loop 的 object 都是 Iterable。
2. 所有可作用於 `next()` 的 object 都是 Iterator。
3. `list`, `str`, `dict` 這些是 Iterable 不是 Iterator, 但可以使用 `iter()` return 一個 Iterator。
4. 可以自己做 class 做出 Iterator, 本質上 Iterator 可以 for loop 是由 `__iter__()` return self, 也就是 return object 自己去實現的，如此才能不斷繼續調用自己的 `__next__()` 如果 class 裡只實現 `__next__()`，那是不能使用 for loop 的，只能調用 next() 一個一個呼叫。
5. 那 `next()` return 什麼？ 你讓它 return 什麼它就 return 什麼。

```py
iterator_ = iter([[1], 2, {}])
print(type(next(iterator_)))
print(type(next(iterator_)))
print(type(next(iterator_)))

"""
<class 'list'>
<class 'int'>
<class 'dict'>
"""

# 當然也是可以客製化 class 裡的 `__next__()`
```

#### 關於前面提到 for loop 的概念:
- for loop 的本質上就是通過不斷調用 `next()` 實現的。

ex:
```py
for x in [1, 2, 3, 4, 5]:
    pass
```
實際上相當於是:

```py
# 先得到 Iterator
iterator_ = iter([1, 2, 3, 4, 5])
while True:
    try:
        x = next(iterator_) #獲得下一個值
    except StopIteration: # 遇到 StopIteration 就退出 loop
        break
```


```py
iterator_ = iter([1, 2, 3, 4, 5])
while True:
    try:
        print("front======")
        x = next(iterator_) #獲得下一個值
        print("back======")
        print(x)
    except StopIteration: # 遇到 StopIteration 就退出 loop
        print("到底了======")
        break
        
        
# 如果仔細這樣 print
"""
front======
back======
1
front======
back======
2
front======
back======
3
front======
back======
4
front======
back======
5
front======
到底了======
"""

# 會發現 第 6 次 call next(), 就出現 exception
```

所以 for 語句是如何 loop 的:
- 1. 先判斷是否為可迭代物件(Iterable)，不是的話直接報錯: TypeError: 'xxxx' object is not iterable，是的話調用 `__iter__()`，並 return 一個迭代器(Iterator)
- 2. 不斷的調用迭代器(Iterato)的 `__next__()` 方法，每次都按照順序返回迭代器(Iterator)中的一個值
- 3. 迭代到最後，已經沒有元素了，這時就會拋出 StopIteration，但是這個異常就像上面的例子一樣會做處理，Python 自己會處理掉，不會給 developer 看。



## 收尾:
- Iteration(迭代): 可以指一個概念，輪廓，操作過程，是一個廣義名詞，在程式中的例子有 for loop
- Iterable(可迭代物件): 一個物件，根據 Python 的定義，得出結論:
    1. 如果可以 for loop 它，就是 Iterable 的範疇
    2. 但 Iterable 可以 for loop 是因為 Python 幫我們把它轉變成 Iterator
- Iterator(迭代器): 一個物件，根據 Python 的定義，得出結論：
    1. Python for loop 的原理就是需要 Iterator，然後不斷調用 Iterator 的 `__next__()`
    2. 產生 Iterator 的方式:
        - iter()
        - 自己客製化 class
        - 產生 generator iterator (或是叫做 generator object / generator)(是一種 Iterator，下篇會敘述)



## The End:

### **< Self Checklist >**:
- [x]Explain what is `Iteration`.
- [x]Explain what is `Iterable`.
- [x]Explain what is `Iterator`.
- [x]Can you tell what is the difference between `Iterable` and `Iterator`.
- [x]Explain what is `Built-in Function`.
- [x]Explain what is `Magic Methods`.
- [x]Explain how Python `for loop` works.
- [x]Use `hasattr()`, `next()`, `iter()`


### **< Reference >**:
- [Python `__iter__()`](https://docs.python.org/3/reference/datamodel.html#object.__iter__)
- [Python `__getitem__()`](https://docs.python.org/3/reference/datamodel.html#object.__getitem__)
- [Python Iterator](https://docs.python.org/3/library/stdtypes.html#iterator-types)
- [Python Glossary](https://docs.python.org/3.9/glossary.html)
- [Python Built-In Function](https://docs.python.org/3/library/functions.html)
- [Python Docs 3. Data model](https://docs.python.org/3/reference/datamodel.html#special-method-names)
- [廖雪峰 Python 3.x](https://www.kancloud.cn/smilesb101/python3_x/296095)
- [迭代那件小事：深入了解 Iteration / Iterable / Iterator / __iter__ / __getitem__ / __next__ / yield](https://medium.com/citycoddee/python%E9%80%B2%E9%9A%8E%E6%8A%80%E5%B7%A7-6-%E8%BF%AD%E4%BB%A3%E9%82%A3%E4%BB%B6%E5%B0%8F%E4%BA%8B-%E6%B7%B1%E5%85%A5%E4%BA%86%E8%A7%A3-iteration-iterable-iterator-iter-getitem-next-fac5b4542cf4)

### **< Next 下一篇 >**:
<a href="/blog/posts/08/210810/python的yield和scrapy的yield">Python 觀念(2) yield 是什麼?，那Scrapy 裡的 yield?</a>
