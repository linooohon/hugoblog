---
title: "[Python]Multi-processing 操作方式: fork, multiprocessing, subprocess"
subtitle: "---藉由 Python code 處理多任務子程序---"
date: 2021-10-20T18:36:14+08:00
draft: false
url: "posts/12/211020/Python_multiprocessing_usage/"
summary: <p align="center">Multi-processing 在 Python 上實作的方式紀錄</p> <img src="/blog/images/12_211020/12_211020_python_multiprocessing使用方式.png" width="50%"/>
author: "linooohon"
email: "linooohon@gmail.com"
toc: true
tags: ["Multiprocessing", "Subprocess", "OS", "Python"]
categories: ["2021", "python"]
tocNum: false
---

![https://linooohon.com/blog](/blog/images/12_211020/12_211020_python_multiprocessing使用方式.png)

## **< Code Example>**:

### 1. `os` module, os.fork()

- 先從 Unix/Linux 系統的 fork 調用來看，MacOS 是類 Unix 所以 fork 調用是支持的，在 Python 可以直接用 `os` module 創建子程序:


```python
import os
import time

print(f'程序 ({os.getpid()}) 開始...')
print("開始呼叫 os.fork()")

print("======================")

pid = os.fork()
print(f"呼叫 os.fork() 後的結果，現在 return : {pid}")

if pid == 0:
    print("pid == 0, 子程序返回: ", pid)
    print(f'子程序說話: 我是子程序 ({os.getpid()}) 然後我的父程序是 ({os.getppid()}).')
else:
    print("pid != 0, 父程序返回子程序 pid: ", pid)
    print(f'原本程序說話: 我是程序 ({os.getpid()}) 我創造了子程序 ({pid}).')
    print("======父程序睡五秒=======")
    time.sleep(5)

print("========最後一行, 現在是:", pid, "=============")

# 輸出會像這樣
"""
程序 (19810) 開始...
開始呼叫 os.fork()
======================
呼叫 os.fork() 後的結果，現在 return : 19826
pid != 0, 父程序返回子程序 pid:  19826
原本程序說話: 我是程序 (19810) 我創造了子程序 (19826).
======父程序睡五秒=======
呼叫 os.fork() 後的結果，現在 return : 0
pid == 0, 子程序返回:  0
子程序說話: 我是子程序 (19826) 然後我的父程序是 (19810).
========最後一行, 現在是: 0 =============
"""

# 並在等待一下後，才會出現
"""
========最後一行, 現在是: 19826 =============
"""

```


```python
# 假設我們在呼叫 os.fork() 後 time.sleep(20) 看一下 `ps -f`
# 會發現實際真的有 父程序和子程序這兩個 pid

import os
import time

print(f'程序 ({os.getpid()}) 開始...')
pid = os.fork()
time.sleep(20)
if pid == 0:
    print("pid == 0, 子程序返回: ", pid)
    print(f'子程序說話: 我是子程序 ({os.getpid()}) 然後我的父程序是 ({os.getppid()}).')
else:
    print("pid != 0, 父程序返回子程序 pid: ", pid)
    print(f'原本程序說話: 我是程序 ({os.getpid()}) 我創造了子程序 ({pid}).')

print("========最後一行, 現在是:", pid, "=============")

# 輸出會像這樣
"""
程序 (83196) 開始...
pid == 0, 子程序返回:  0
子程序說話: 我是子程序 (83216) 然後我的父程序是 (83196).
========最後一行, 現在是: 0 =============
pid != 0, 父程序返回子程序 pid:  83216
原本程序說話: 我是程序 (83196) 我創造了子程序 (83216).
========最後一行, 現在是: 83216 =============
"""
```


`ps -f` :
```
  UID PID   PPID    C  STIME  TTY        TIME 
  501 83196 11135   0  3:55PM ttys017    0:00.04 
  501 83216 83196   0  3:55PM ttys017    0:00.00 
```


### 2. `multiprocessing` module, 實例化 Process(), 啟動一個子程序

- 雖然 MacOS 有 fork 調用，但在 windows 沒有 fork 調用，那如何做到跨平台，使用 python 的標準函式庫裡的 `multiprocessing` 模組就可以支援:

```python
"""啟動一個子程序(進程)(process)並等待其結束然後再 print 出 "子程序結束"
"""

from multiprocessing import Process
import os


# 子程序要做的事
def child_procss(name):
    print("跑子程序", name, "pid 是:", os.getpid())
    print(1)
    print(2)
    print(3)


if __name__ == '__main__':
    print("主程序", os.getpid())
    p = Process(target=child_procss, args=('print123',))
    print("子程序將要開始")
    p.start()
    p.join()  # 加 join 是因為如此的話 可以等待子程序(進程)結束後再繼續往下運行
    print("子程序結束")

    
"""
主程序 67494
子程序將要開始
跑子程序 print123 pid 是: 67511
1
2
3
子程序結束
"""
```



### 3. `multiprocessing` module, 實例化 Pool(), 使用 apply_async(), 啟動大量子程序

- 同樣也是標準函式庫`multiprocessing` 使用裡面的 `Pool()`

```python
"""啟動大量程序(進程)(process), 使用 Pool
"""
from multiprocessing import Pool
import os, time, random

def long_time_task(name):
    print('跑任務 %s (%s)...' % (name, os.getpid()))
    start = time.time()
    time.sleep(random.random() * 3)
    end = time.time()
    print('任務 %s 跑了 %0.2f 秒' % (name, (end - start)))

if __name__=='__main__':
    print('主程序開始 %s.' % os.getpid())
    p = Pool(35)
    for i in range(35):
        p.apply_async(long_time_task, args=(i,))
    print('apply_async() 非同步, 正在等待所有子程序從運行直到完畢...')
    p.close() # join 前要加 close
    p.join() # 加 join 是因為如此的話，可以等待全部子程序(進程)結束後再繼續往下運行
    print('所有子程序結束。')
```

#### - close()
close() 必須加，不加的話會 raise error
```
raise ValueError("Pool is still running")
ValueError: Pool is still running
```
加了的話意思就是避免其他 Task 再進來這個 Pool, 同時確保所有 Job 做完後，子程序都會關閉。


#### - join()
join() 必須加，為了確保主程序關閉之前，子程序都要跑完，所以加這個。<br/> 不加的話，一旦主程序結束，子程序也就沒了。所以通常前面一會放 close() 或是你想直接終止的時候就會放 terminate()，總之 join() 就是確保等待子程序關閉。



#### - 關於 Pool()
```
p = Pool(8)
```

可以發現我設定 `p = Pool(8)` 表示設定我這個 Pool 可以同時跑 8 個 process
，如果我不設定 by default 就是當下 CPU 守備範圍內的核心，通常看 CPU 核心就能推估。或著用 python 標準函式庫裡的 module 可以看目前電腦核心數量:

- 查看自己目前 CPU 核心數量:
```python
import multiprocessing
cpu_amount = multiprocessing.cpu_count()
print(cpu_amount)

import os
cpu_amount = os.cpu_count()
print(cpu_amount)
```



#### - `multiprocessing` module, Pool 裡的其他種使用方式:


- map()

```python
p = Pool()
p.map(func, arg) # 會確保子程序都結束，才會跑下面一行，也就是阻塞在這
print("阻塞") # 這行會在子程序都結束後才會印出來
```

```python
# 假設有 return，可直接拿到 因為 map 是 return `list`
p = Pool()
results = p.map(func, arg) 
print(results) 
```



- map_async() : 實現非同步

```python
p = Pool()
p.map_async(func, arg) # 不會停在這，也就是說下一行不會等子程序結束
print("不阻塞") #直接印出來
pool.close()
pool.join()
```

```python
# 假設有 return，要呼叫 get(), 因為 map_async 是 return `multiprocessing.pool.MapResult`
p = Pool()
results = p.map_async(func, arg)
print(results.get()) 
pool.close()
pool.join()
```


- starmap()

```python
p = Pool()
p.starmap(func, arg) # 這裡的 arg 必須是 iterable, 如果是 list，則裡面的 element 也必須是 iterable, 而 map() 的話 list 裡面不用是 iterable
pool.close()
pool.join()
```

- chunksize : 每個子程序可以處理的任務數量
```python
p = Pool()
p.map(func, arg, chunksize=10)
```


- callback

```python
def print_func(pool_outputs):
    print("callback", pool_outputs)

p = Pool()
pool_outputs = p.map_async(func, arg, callback=print_func)
print(pool_outputs.get())
pool.close()
pool.join()

```

- maxtasksperchild : 設定每個子程序完成多少任務後，就會再重啟新的子程序
```python
p = Pool(5, maxtasksperchild=50)
```


### 4. `subprocess` module, subprocess.call()


ex1:
```python
import subprocess

print('$ curl https://www.google.com')
r = subprocess.call(['curl', 'https://www.google.com'])
print('Exit code:', r)

# 效果和 `curl https://www.google.com` 一樣
```

ex2:
```python
import subprocess

print('$ nslookup www.python.org')
r = subprocess.call(['nslookup', 'www.python.org'])
print('Exit code:', r)

# 效果和 `nslookup www.python.org` 一樣
```

### 5. `subprocess` module, subprocess.Popen(), communicate(), 輸入輸出

ex1:
```python
import subprocess

print('$ date -R')
p = subprocess.Popen(['date', '-R'], stdin=subprocess.PIPE, stdout=subprocess.PIPE, stderr=subprocess.PIPE)
output, err = p.communicate()
print(output)
print(output.decode('utf-8'))
print('Exit code:', p.returncode)
```

ex2:
```python
import subprocess

print('$ nslookup')
p = subprocess.Popen(['nslookup'], stdin=subprocess.PIPE, stdout=subprocess.PIPE, stderr=subprocess.PIPE)
output, err = p.communicate(b'set q=mx\twitter.com\nexit\n')
print(output.decode('utf-8'))
print('Exit code:', p.returncode)


"""
$ nslookup

> set q=mx
  twitter.com
  exit
"""

# 輸出
"""
$ nslookup
Server:         192.168.1.1
Address:        192.168.1.1#53

Non-authoritative answer:
twitter.com     mail exchanger = 20 alt2.aspmx.l.google.com.
twitter.com     mail exchanger = 30 aspmx2.googlemail.com.
twitter.com     mail exchanger = 10 aspmx.l.google.com.
twitter.com     mail exchanger = 30 aspmx3.googlemail.com.
twitter.com     mail exchanger = 20 alt1.aspmx.l.google.com.

Authoritative answers can be found from:


Exit code: 0
"""
```


### 6. `subprocess` module, subprocess.run(), stdout 輸出到檔案

subprocess.run() 是建立在 subprocess.Popen() 之上
```python
import subprocess

with open('output.txt', 'w') as f:
    subprocess.run(['date', '-R'], stdout=f)

# 如此 output.txt 就會產生
```


### 7. `subprocess` module, subprocess.run(), stdout 輸出回 subprocess.PIPE 繼續用


```python
import subprocess

result = subprocess.run(['date', '-R'], stdout=subprocess.PIPE)
print(result)
print("------")
print(result.stdout)
print("------")
print(result.stdout.decode('utf-8'))
```

### 8. `subprocess` module, subprocess.check_output()

subprocess.check_output() 更簡化了 subprocess.run()

```python
import subprocess

result = subprocess.check_output(['date', '-R'])
print(result)
print(result.decode("utf-8"))
```


### 9. `multiprocessing` module, Process() 搭配 Queue() 達成不同子程序間的資料流交換

```python
from multiprocessing import Process, Queue
import os
import time

# 資料寫進 Queue
def write(q):
    print(os.getpid(), "子程序寫入")
    for value in ['A', 'B', 'C']:
        print(f'放 {value} 到 queue...')
        q.put(value)
        time.sleep(1)

# 從 Queue 拿資料
def read(q):
    print(os.getpid(), "子程序讀取")
    while True:
        value = q.get(True)
        print(f'拿到 {value} 來自 queue.')

if __name__ == '__main__':
    # 主程序創造 Queue, 傳給兩個子程序
    q = Queue()
    pw = Process(target=write, args=(q,))
    pr = Process(target=read, args=(q,))
    # 啟動子程序pw，寫入:
    pw.start()
    # 啟動子程序pr，讀取:
    pr.start()
    # 等待 pw 結束:
    pw.join()
    # pr 程序是 無窮loop，所以等待 pw 結束後，就把它終止:
    pr.terminate()

# 輸出    
"""
92372 子程序寫入
放 A 到 queue...
92373 子程序讀取
拿到 A 來自 queue.
放 B 到 queue...
拿到 B 來自 queue.
放 C 到 queue...
拿到 C 來自 queue.
"""
```


以上，結束。
有參考網路一些例子，在這裡針對基本使用統整紀錄一下。


## **< The End >**:

結束！


### Self Checklist:
- [x]What is Multi-processing.
- [x]How to handle Multi-processing with python code.
- [x]Basic concept of Master-Worker Pattern.
- [X]How to use `os` module.
- [X]How to use `subprocess` module.
- [X]How to use `multiprocessing` module.
- [X]What is `fork` in Unix/Linux System.
- [X]How to use `Pool()`, `Process()`, `Queue()` in `multiprocessing` module.
- [X]How to use `map_async()`, `apply_async()` in `multiprocessing` module.
- [X]How to use `call()`, `Popen()`, `run()`, `check_output()` in `subprocess` module.
