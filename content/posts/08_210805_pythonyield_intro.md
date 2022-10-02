---
title: "[Python]觀念(2) yield, generator?，那 Scrapy 裡的 yield? "
subtitle: "yield 與 generator"
date: 2021-08-05T18:36:14+08:00
draft: false
url: "posts/08/210805/python的yield和scrapy的yield"
summary: <p align="center">繼上篇 Iterator 紀錄，這次對 yield 的概念在此統整一下，同時也追了一下 Scrapy 的 source code</p> <img src="/blog/images/08_210805/08_210805_pythonyield_intro.png" />
author: "linooohon"
email: "linooohon@gmail.com"
toc: true
tags: ["Yield", "Python", "Scrapy", "Generator"]
categories: ["2021", "python"]
---

<h1 style="text-align:center">
Python 的 yield 是什麼?，Scrapy 裡的 yield?
</h1>

> 最近研究了一下 yield 的概念，在此統整一下

![Python 的 yield 是什麼?](/blog/images/08_210805/08_210805_pythonyield_intro.png)

## **< 關於 yield >**:
> 談到 yield 就會談到 generator，但 generator 是什麼後面解釋！

在 python 的定義中，function 中只要出現了 yield（yield expressions），那麼事實上定義的是一個 generator function。
跟一般的 function 不一樣，呼叫 generator function 就是會 return "generator iterator"。


### 所以,
有 yield 的 function 被單純呼叫的時候會 return "generator iterator"，然後這時候還不會執行 function 定義的事情，那什麼時候執行？
大概有兩種執行方式:
1. 調用 Built-in Function `next()`
2. 使用 for loop (實際 for loop 的步驟也是 magic method `__next__()` 在處理)


### 例子測試一下:

> 確實是 return generator iterator
```py
def get_string():
    return "happy coding"
    
    
def get_generator():
    yield "happy coding"
    
    
if __name__ == '__main__':
    
    result = get_string()
    print(result, type(result)) # happy coding <class 'str'>
    
    result = get_generator()
    print(result, type(result)) # <generator object get_generator at 0x10c424d60> <class 'generator'>
```

## **< 幾個名詞先釐清 >**:
在 [Python Glossary](https://docs.python.org/3.9/glossary.html#term-generator) 裡定義了三個詞
- generator
- generator iterator
- generator expression


### generator, generator iterator, generator expression
> 裡面寫道 (這裡只先做名詞區分不解釋)：

- generator
    - 是一個 function 會回傳 generator iterator。
    - 而通常 generator 指的是一個大家常說的詞叫做 `generator function`，但是在某些文檔裡也有人把它用來指 `generator iterator`，
- generator iterator
    - 是一個被 generator function 創造出來的物件。(就我查詢的過程中，在某些文章裡會寫 `generator object` 特別指出當下這個 generator 是被創造出來的 object, 而不是 function)
- generator expression
    - 是一個 expression 會回傳 iterator。


<p style="color: #d19169">名詞看到會有點疑惑，但把它搞清楚才是最佳解，以後看別人的文章或是文件，才不會模模糊糊:</p>

Glossary 對於 generator 的補充:

<p style="color: #42bac7">Usually refers to a generator function, but may refer to a generator iterator in some contexts. In cases where the intended meaning isn’t clear, using the full terms avoids ambiguity.</p>


> 所以接下來我會用 generator function 和 generator iterator 這兩個來區別 function 與 object。

> 關於 Generator 留到後面做解釋

```
現在只要先知道 function 裡有 yield 就是一個會回傳 generator iterator 的 generator function。
```

```
既然知道 yield 和 generator 的關係了，底下來個例子
```

## **< (generator)generator function 例子解釋 >**:
> 用這個 function 來理解一下，看到對岸的朋友解釋的文章，覺得挺好的，記錄下來！

```py
def foo():
    print('foo function is starting...')
    while True:
        res = yield 4
        print('res:', res)

g = foo()
print(next(g))
print('*'*10)
print(next(g))
```

> 執行結果:
```py
foo function is starting...
4
********************
res: None
4
```


### 1. `g = foo()`
```py
g = foo()
```

因為 foo() 帶有 yield 所以根據定義，就算呼叫了 foo(), 他也不會開始執行。

但是這裡他會得到一個生成器物件 "g" (generator iterator)。

> 執行結果: 什麼都沒印


### 2. `print(next(g))`
```py
print(next(g))
```
直到調用 next 方法 foo function 才正式開始執行，
先執行 foo 的 print('foo function is starting...') 然後進入 while loop，

這時候遇到 yield，先把它想成 return ，所以這時候 return 一個 4, program(程序)就停了。
這裡很大的重點是，"並沒有把 4 assign(指派/賦值) 給 res"。


> print(next(g)) 執行結果: 
- next(g): print 了 'foo function is starting...'
- return 了 4, 所以又 print 了 4
```
foo function is starting...
4
```

### 3. `print('*'*10)`
```py
print('*'*10)
```
這行只是印開心的順便做一些區隔: print 10個 '*'

> print('*'*10) 執行結果:
```
********************
```

### 4. `print(next(g))`
```py
print(next(g))
```
這裡是第二次調用，每次調用會接續上一次停止的地方，也就是正準備把 4 assign(指派/賦值) 給 res 的時候，但注意這邊因為上一次調用 `next()` 的最後一個步驟是 return 4 出去，所以這時候 "=" 右邊已經是 None 了。

> print(next(g)) 執行結果: 
- 這時候 None 被 assign 給 res, res 就是 None，所以 print('res:', res) -> res: None。
- 接著還在 loop 裡，又會跑到 yield 4, 所以 return 4 出來，所以最後還會 print 出 4。
```
res: None
4
```

### 如果最後一行改成這樣: `print(g.send(7))`
```python
g = foo()
print(next(g))
print('*'*20)
print(g.send(7))   ## 改這行
```

7 就會 assign 給 res
所以 print('res:', res) -> res: 7

> 執行結果:
```
foo function is starting...
4
********************
res: 7
4
```

## **< Key Point >**:
1. 每次調用 next() 他會記住上次停的地方，這次就會從那裡開始。
2. 而 generator iterator 也是 iterator 的一種，所以當然想處理 generator iterator 也是有兩種方式：
    - 一次次自己調用 next()
    - for loop它
3. 使用 send() 可以接續上一次 yield 停住的地方並 assign 給左邊。
4. generator 是 iterator 的一種，但 iterator 卻不一定是 generator。這個不難理解，依我自己的看法(這裡只是隨意舉例!)像是陸軍和特種部隊都會一樣的基本能力，但特種部隊擁有陸軍所沒有的特性, 能力(在程式碼這裡就會是: 定義, 功能, 佔用空間等等..)。

### isinstance 可以查看它是不是，這裡不多說

> 也可以用 isinstance，來確認它是不是 Iterable, Iterator, Generator

```python=
import pandas as pd
import numpy as np
from collections.abc import Iterator, Iterable, Generator

# dic
dic_ = {'name': 'phil', 'phone': '0933333333'}
print("===============dic===============")
print(isinstance(dic_, Iterable))
print(isinstance(dic_, Iterator))
print(isinstance(dic_, Generator))

# list
list_ = ['a', 1, {'key': 'value'}, (1, 2), {1, 2}]
print("===============list===============")
print(isinstance(list_, Iterable))
print(isinstance(list_, Iterator))
print(isinstance(list_, Generator))

# tuple
tuple_ = ('a', {'key': 'value'}, 2)
print("===============tuple===============")
print(isinstance(tuple_, Iterable))
print(isinstance(tuple_, Iterator))
print(isinstance(tuple_, Generator))

# range
range_ = range(1)
print("===============range===============")
print(isinstance(range_, Iterable))
print(isinstance(range_, Iterator))
print(isinstance(range_, Generator))

# string
string_ = 'a'
print("===============string===============")
print(isinstance(string_, Iterable))
print(isinstance(string_, Iterator))
print(isinstance(string_, Generator))

# set
set_ = {'a', 2, 3, (1, 2)}
print("===============set===============")
print(isinstance(set_, Iterable))
print(isinstance(set_, Iterator))
print(isinstance(set_, Generator))

# numpy array
np_array_ = np.array([1, 2, 3, 4])
print("===============np_array===============")
print(isinstance(np_array_, Iterable))
print(isinstance(np_array_, Iterator))
print(isinstance(np_array_, Generator))

# dataframe
df_ = pd.read_csv("1.csv")
print("===============dataframe===============")
print(isinstance(df_, Iterable))
print(isinstance(df_, Iterator))
print(isinstance(df_, Generator))

# bytes
bytes_1 = bytes(1)
print("===============bytes===============")
print(isinstance(bytes_1, Iterable))
print(isinstance(bytes_1, Iterator))
print(isinstance(bytes_1, Generator))

```





### Generator 到底是什麼，目的，用法(再次解析):
Python 中有一個東西叫做 List Comprehensions, 是 Python 可以簡潔的創造 list 的方式
```py
list_ = [x * x for x in range(1, 11) if x % 2 == 0]
print(list_)
# [4, 16, 36, 64, 100]
```

- 列出資料夾底下所有文件和資料夾
```py
import os 
l = [d for d in os.listdir('.')]
print(type(l)) # 是個 list
```

<p style="color: #d19169">但是，如果今天我們要的內容很多，list 就會變得非常佔用記憶體，所以當我們要做的事，可以依照某種規則算出來，那我們就可以 loop 整個過程去持續推算下一個元素，這樣就不必創造很大很大的 list, 在 Python 中，這樣邊 loop 邊計算還真的有，就是 Generator(生成器)</p>

要創建 generator 除了上面有提過的 generator function，這裡再補充一個，只要把 List Comprehensions 改成小括號 `()` 就行了。
就是所謂的 generator expression [Docs](https://docs.python.org/3.9/glossary.html#term-generator-expression)
```py
list_ = (x * x for x in range(1, 11) if x % 2 == 0)
print(list_)
# <generator object <genexpr> at 0x106a674a0>
```
那當然 generator 直接 print 沒東西，我們就是透過上面提過的 `next()`

```py
print(next(list_))
print(next(list_))
print(next(list_))
print(next(list_))
print(next(list_))
print(next(list_))
"""
4
16
36
64
100
Traceback (most recent call last):
  File "/Users/linpinhung/Desktop/TMPF/06.py", line 82, in <module>
    print(next(list_))
StopIteration
"""
```

但一直用 next()，太麻煩，基本上都用 for loop 實現，且用 for loop 也不需關心 `StopIteration`, Python 幫忙解決，這也就是上一篇提過的 Iterator 的特性之一，Generator Iterator 當然也會有 Iterator 的特性。

```py
for i in list_:
    print(i)

"""
4
16
36
64
100
"""
```

### Generator(Generator Iterator)與 Iterator的差異:
- 創建 Generator 我們用 `generator expression` 或是 `generator function` 而不使用 class 創建, 但創建 Iterator 我們用 `iter()` 或是自己創建 class 寫 `__iter__()`, `__next__()`。
- Generator 用 yield 關鍵字，Iterator 沒有。
- Generator 在每次 yield 停住的時候會儲存狀態(ex: 變數)，但 Iterator 不會，他就是做 iteration 而已。
- Generator Function 裡可以不只有一個 yield。
- Generator 是 Iterator 的 subclass, 我們可以透過 `issubclass()` 證明
```py
import collections,types
print(issubclass(types.GeneratorType,collections.abc.Iterator)) #True
print(issubclass(collections.abc.Generator,collections.abc.Iterator)) #True

```

## **< Scrapy >**:
### Scrapy 框架裡常用的 yield, 會有兩種情況。
> 一個是 yield item, 一個是 yield Request

自己在工作上有使用 Scrapy 的機會，
而出於好奇也想知道 Scrapy 在這裡怎麼會用 yield,
仔細去追 Scrapy 的 source code，就會發現以 `scrapy/crawler.py` 底下的 class CrawlerRunner 為例:

```py
# 根據官方文檔
import scrapy
from twisted.internet import reactor
from scrapy.crawler import CrawlerRunner
from scrapy.utils.log import configure_logging
from scrapy.utils.project import get_project_settings

class MySpider1(scrapy.Spider):
    # Your first spider definition
    ...

class MySpider2(scrapy.Spider):
    # Your second spider definition
    ...

configure_logging()
settings = get_project_settings()
runner = CrawlerRunner(settings)
runner.crawl(MySpider1)
runner.crawl(MySpider2)
d = runner.join()
d.addBoth(lambda _: reactor.stop())

reactor.run() # the script will block here until all crawling jobs are finished
```

最一開始是呼叫了 class CrawlerRunner 的 crawl 方法：
這時後仔細看 source code 往下追，

> `scrapy/crawler.py` [source code](https://github.com/scrapy/scrapy/blob/06f3d12c1208c380f9f1a16cb36ba2dfa3c244c5/scrapy/crawler.py)

```python
from scrapy.core.engine import ExecutionEngine


class Crawler:
def crawl(self, *args, **kwargs):
    
    #(略)
    
    try:
        self.spider = self._create_spider(*args, **kwargs)
        self.engine = self._create_engine()
        start_requests = iter(self.spider.start_requests())
        yield self.engine.open_spider(self.spider, start_requests)

    # (略)


def _create_spider(self, *args, **kwargs):
        return self.spidercls.from_crawler(self, *args, **kwargs)
        
def _create_engine(self):
    return ExecutionEngine(self, lambda _: self.stop())

# (略)

```


追到 `scrapy/core/engine.py`[source code](https://github.com/scrapy/scrapy/blob/06f3d12c1208c380f9f1a16cb36ba2dfa3c244c5/scrapy/core/engine.py)
```py
class Slot:
    def __init__(
        self,
        start_requests: Iterable,
        close_if_idle: bool,
        nextcall: CallLaterOnce,
        scheduler,
    ) -> None:
        self.closing: Optional[Deferred] = None
        self.inprogress: Set[Request] = set()
        self.start_requests: Optional[Iterator] = iter(start_requests)
        self.close_if_idle = close_if_idle
        self.nextcall = nextcall
        self.scheduler = scheduler
        self.heartbeat = LoopingCall(nextcall.schedule)

    # (略)
    
class ExecutionEngine:
    def _next_request(self) -> None:
        assert self.slot is not None  # typing
        assert self.spider is not None  # typing

        if self.paused:
            return None

        while not self._needs_backout() and self._next_request_from_scheduler() is not None:
            pass

        if self.slot.start_requests is not None and not self._needs_backout():
            try:
                request = next(self.slot.start_requests)
            except StopIteration:
                self.slot.start_requests = None
            except Exception:
                self.slot.start_requests = None
                logger.error('Error while obtaining start requests', exc_info=True, extra={'spider': self.spider})
            else:
                self.crawl(request)

        if self.spider_is_idle() and self.slot.close_if_idle:
            self._spider_idle()
```

其實看到 yield 就了解 generator 會有 next() 的實作，但是還是想親眼見到，所以：
看到第 32 行，找到 next()，也就安心了...
而 Scrapy 本身是在 Twisted 這個 asynchronous networking 的 library 架構下做處理，但這邊我還不是很理解，也不多說，這邊就記錄一下而已！




## **< The End >**:

### Self Checklist:
- [x]Explain what is `yield`.
- [x]Explain what is `generator function`.
- [x]Explain what is `generator iterator`.
- [x]Figure out confusing terms`generator`, `generator function`, `generator iterator`, `generator object`.
- [x]Can you tell what is the difference between `iterator` and `generator`.
- [x]Explain what is `next()`.
- [x]Explain what is `send()`.
- [x]How to make `generator iterator`.


### Reference:
- [generator expression](https://docs.python.org/3.9/glossary.html#term-generator-expression)
- [Python Glossary](https://docs.python.org/3.9/glossary.html)
- [廖雪峰 Python 3.x](https://www.kancloud.cn/smilesb101/python3_x/296095)
- [迭代那件小事：深入了解 Iteration / Iterable / Iterator / __iter__ / __getitem__ / __next__ / yield](https://medium.com/citycoddee/python%E9%80%B2%E9%9A%8E%E6%8A%80%E5%B7%A7-6-%E8%BF%AD%E4%BB%A3%E9%82%A3%E4%BB%B6%E5%B0%8F%E4%BA%8B-%E6%B7%B1%E5%85%A5%E4%BA%86%E8%A7%A3-iteration-iterable-iterator-iter-getitem-next-fac5b4542cf4)
- [Python Generators vs Iterators – Comparison Between Python Iterators and Generators](https://data-flair.training/blogs/python-generator-vs-iterator/)

## **< Next 下一篇 >**:
<!-- <a href="/blog/posts/08/210805/python的yield和scrapy的yield">Python 觀念(2) yield 是什麼?，那Scrapy 裡的 yield?</a> -->
