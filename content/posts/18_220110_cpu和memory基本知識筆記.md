---
title: "CPU 與 Memory 基礎概念/名詞釐清 Note"
subtitle: "CPU, Memory, RAM, ROM, Work Flow, Memory Hierarchy..."
date: 2022-01-10T18:36:14+08:00
draft: false
url: "posts/18/220110/Character_Encoding字元編碼/Part1"
summary: <p align="center">CPU, Memory, RAM, ROM...</p> <img src="/blog/images/18_220110/18_220110_cpu_memory.png" width="50%"/>
author: "linooohon"
email: "linooohon@gmail.com"
toc: true
tags: ["CPU", "Memory", "RAM"]
categories: ["2022", "cpu", "memory"]
tocNum: false
---



> 此筆記參考網路文章與資源，內文描述與圖片有雷同請見諒，僅作為筆記之用。


## **< 一. CPU(central processing unit):中央處理器/處理器/電腦大腦(心臟) >**
### @ 作用
```
負責各部門的運作以及處理各種算術運算與邏輯運算
```
### @ 組成
- Main(2 parts):
    - CU(Control Unit)(控制單元)
    - ALU(Arithmatic and Logical Unit)(算術邏輯單元)
- Others:
    - Register(暫存器) - 好幾種
        - Buffer Register(緩衝暫存器)
        - Accumulator(累加器)
        - Address Register(位址暫存器)
        - Instruction Register(指令暫存器)
        - Program Counter(程式計數器)
    - Bus(匯流排) - 好幾種
        - ex: Data Bus(資料匯流排)

### @ Jobs
```
    - CU: 指揮及協調電腦系統中其他部門的工作，並且監督與管制主記憶體與算術邏輯單元及輸入／輸出裝置間的資料與指令的傳輸。負責傳出 Signal 給 CPU 內的部門來控制其動作，總之 CPU 的動作其實都是 CU 規劃的。
        - ex: 把 Instruction Register 中的 Instruction 送到 Decoder 中。
        - ex: 指定 Data 要放進去哪個 Register。
        - ex: 控制 ALU 該做哪一種 Operation、Program Counter 的計算。
    - ALU: 在控制單元的指揮之下，負責執行所有的算術運算，如加、減、乘、除等。以及執行邏輯運算如某數是否大於、等於或小於另一數。(英文來說就是對讀入的指令做各種的 Operation)。ALU 最多只能從兩個 Register 做讀取，只能輸出到一個 Register，也就是最多有兩個輸入和一個輸出。
        - Logic Operation: AND、OR、XOR、NAND…等等。
        - Shift Operation: 將指令的位元向左移或向右移。
        - Arithmetic Operation: 做運算，A + B、A — B、A * B或 A / B …等等。
    - Register: 
        - Buffer Register: 暫存由主記憶體中讀出以待處理的資料。
        - Accumulator: 儲存算術或邏輯運算執行時暫存的結果。
        - Address Register: 暫存所要讀取之資料或指令在主記憶體中的儲存位址。
        - Instruction Register: 暫存下一次所要執行的指令。
        - Program Counter: 暫存下一個等待執行的指令在主記憶體中的位址。
    - Bus: 是電腦系統中不同部門間用來傳送資料的通道，Bus 亦有好幾種，其中資料匯流排(databus)是負責整個CPU中資料傳送的任務。而資料匯流排的寬度越寬，則一次所能傳送的資料量就越大，電腦的處理速度也就越快。例如Intel 80286處理器的匯流排寬度為十六位元，一般稱為十六位元電腦。而80486則為32位元。有些超級電腦的資料匯流排的寬度則高達128個位元。
```
    
### @ Notes
```
在某些電腦系統中，CPU包含了控制單元，算術邏輯單元和主記憶體，而一般的微電腦系統其CPU則只包含控制單元及算術邏輯單元。主記憶體是否為CPU的一部分是隨電腦系統的不同而有不同的定義的。
```

### @ CPU 流程圖
![](https://i.imgur.com/ssG5VY7.jpg)

> [圖片來源](https://medium.com/@leo19866/cpu%E8%88%87%E8%A8%98%E6%86%B6%E9%AB%94-memory-%E7%9A%84%E9%81%8B%E4%BD%9C%E6%A9%9F%E5%88%B6-2e33bd9b0858)


### @ 一次工作時，資料進來在CPU的流程是做了什麼?

```
1. 由 Input devices(Keyboard、Mouse、或者其他I/O裝置) 經由 Data Bus 從 Memory 傳輸給 CPU。
2. CPU 將收到的指令放到 Instruction Register 中。
3. Control Unit 再從 Instruction Register 中把它讀取出來(一條指令)。
4. Control Unit 讀出來後傳給 Decoder 做解碼 (decoding) 的動作，把指令轉換成 CPU 可以明白的 Machine Language(Machine Code的組成)，CPU 到這邊才真正了解到底要做什麼。
5. CPU 依照指令內容到 Cache 中去尋找有沒有需要用到的 Program 或 Data。
    1. 如果沒有，再到 Main Memory (也就是 Violate Memory 揮發性記憶體) 中去找。
    2. 如果再沒有，再到 External Memory (Secondary Storage, 也就是 Hard Disk) 中去找。
6. 等到一條指令需要用到的 Program 和 Data 都找到後，Program 和 Data 會先被送入 ALU 中執行各種 Operation, Control Unit 會依照指令內容告訴 AUL 該執行哪一種 Operation。
    - 若是在處理過程中需要把資料暫時儲存起來，Control Unit 會叫 CPU 把資料暫時存到 Data Register, 等到一條指令的處理結束後，Data Register 也會跟著被清空。
7. 一條指令執行完成後，CPU 會把該條執行的結果輸出到 Output Device。
8. Control Unit 的 Program Counter 會繼續依照指令內容來決定下一條指令的 Address，並存著。
9. CPU 再去看 Program Counter 存的 Address，根據該 Address 去 Main Memory 找那條指令，並把他存在 Instruction Register，等待準備執行，重複 Step.3 - step.9


整個過程會不斷地重複: 1.找到指令放進 Instruction Register -> 2.解碼 -> 3.執行 -> 4.Program Counter 繼續給出下一個指令位址
```


### @ References
- [國家教育研究院](https://terms.naer.edu.tw/detail/1302340/)
- [NUTNCSIE10412](https://sites.google.com/site/nutncsie10412/ge-ren-jian-jie/ji-yi-ti)
- [Leo Lin Medium](https://medium.com/@leo19866/cpu%E8%88%87%E8%A8%98%E6%86%B6%E9%AB%94-memory-%E7%9A%84%E9%81%8B%E4%BD%9C%E6%A9%9F%E5%88%B6-2e33bd9b0858)
- [Wiki - Machine Code](https://en.wikipedia.org/wiki/Machine_code)
- [電腦組成](https://sites.google.com/site/nutncsie10412/ge-ren-jian-jie/dian-nao-nei-bu-gou-cheng)


## **< 二. Memory: 記憶體/內存 >**
### @ 組成(大致分兩種)
- Volatile Memory(揮發性記憶體/暫存記憶體): 電源供應中斷後，資料會消失, 速度快
    - RAM(Random Access Memory)(隨機存取記憶體)
        - DRAM(Dynamic RAM)(動態隨機存取記憶體)
        - SRAM(Static RAM)(靜態隨機存取記憶體)
            - ex: Cache
- Non-volatile Memory(非揮發性記憶體/永久儲存記憶體/NVM): 電源供應中斷後，資料會保存, 速度慢
    - ROM(Read-Only-Memory)(唯讀記憶體)
        - PROM(Programmable ROM)(可邊程式唯讀記憶體)
        - EPROM(Erasable Programmable ROM)(可抹除可編程式唯讀記憶體)
        - EEPROM(Electrically Erasable Programmable ROM)(電子抹除式可複寫唯讀記憶體)
    - Flash Memory(快閃記憶體/快閃儲存器/閃存)
        - NOR Flash
        - NAND Flash
            - ex: 隨身碟
            - ex: 記憶卡
            - ex: SSD(Solid-state Drive/Solid-state Disk)(固態硬碟)


- **補充**
> - HDD(Hard Disk Drive/Hard Disk/硬碟/硬盤/磁盤/傳統硬碟/機械硬盤)
> - 光碟
> - 磁帶 \
> **這些雖然也都是非揮發性儲存裝置，但以 Wiki 上的解釋，並不把他們歸類為 NVM** \
> 參考 [(Wiki - NVM)](https://zh.wikipedia.org/wiki/%E9%9D%9E%E6%8F%AE%E7%99%BC%E6%80%A7%E8%A8%98%E6%86%B6%E9%AB%94)


- **補充: Main Memory 和 External Memory**
```
- Main Memory: 就我查到的是指 RAM、ROM、有時候包括 Flash 和 Cache。
但其實常常 Main Memory 最常專指 RAM。
且有時候也稱 Primary Storage。

- External Memory: 包括
    - Non-volatile Memory(flash, SSD)
    - HHD(硬碟)
    - CD
```


### @ RAM(Random Access Memory)(Main Memory/隨機存取記憶體)
- 負責電腦運作時 CPU 的資料暫存和快取
```
1. 隨機存取的意思:
當要對這個記憶體內的任何位置的資料作讀取或寫入時，其所花費的時間與位置無任何關係。
與之相對的是 -> 循序存取(Sequential Access)

2. RAM 被稱為 Main Memory:
裡面存放的內容會隨著不同做法而改變，最普遍的做法就是把 Program 和 Data 一起放在裡面，所以當電腦開機之後，它若是想要執行程式或者是處理資料，就必須先把資料放到 RAM 中，接著在來使用，而當它想要保持現有的執行狀態，比方說一直執行某個程式，它也必須占用 RAM 的空間，所以當你發現開太多程式讓系統變得緩慢時，那就是 RAM 裡的空間已經不太夠用了
```

- **DRAM(Dynamic RAM)(動態隨機存取記憶體)**
    - Main Memory 的 RAM 最下層主要都以 DRAM 去做。
    - DRAM 便宜(便宜簡單的電容)。
    - 同體積 DRAM 的容量是 SRAM 的好幾倍。
```
何謂動態:
不穩定，使用 "Capacitance"(電容)的電量區別0和1來存資料。會漏電，要定期充電，否則資料會消失。
``` 
---

<div style="display: flex; justify-content: center;">
    <img width="50%" src="https://i.imgur.com/4lcNqUL.png"/>
</div>

> [圖片來源](https://kopu.chat/2017/05/06/dram%E3%80%81flash%E8%B2%B4%E5%88%B0%E7%82%B8%EF%BC%8C%E4%BD%A0%E9%82%84%E6%90%9E%E4%B8%8D%E6%87%82%E8%A8%98%E6%86%B6%E9%AB%94%E7%9A%84%E5%B7%AE%E7%95%B0%E5%97%8E%EF%BC%9F/#:~:text=Flash%EF%BC%88%E5%BF%AB%E9%96%83%E8%A8%98%E6%86%B6%E9%AB%94,%E4%B9%9F%E6%AF%94NAND%20Flash%20%E8%B2%B4%E3%80%82)

---

- **SRAM(Static RAM)(靜態隨機存取記憶體)**
    - Cache 是 SRAM 的一種。
    - SRAM 貴(貴又複雜的邏輯閘)。
```
何謂靜態:
穩定，使用 "flip-flop"(邏輯閘)斷路0通路1來存資料，速度快，不用定期充電。
```



### @ ROM(Read-Only-Memory)(唯讀記憶體)
- 通常存放 Bios、Bootstrap、 Operating System (作業系統)…等等不會被使用者變動的資料。
```
ROM 是在製造的過程中，就把資料一併寫進 IC 裡，所以當一顆 ROM 完成的時候，它內部的資料就已經定型了。
如此一來，它就可以在不用電力供給的狀況下來儲存資料了，但是這種做法也會帶來一些限制，正如其名， ROM 內部的資料就只能夠被寫入一次，而且只能夠被讀取，不能改寫。
所以 ROM 內部放的通常都是不會被使用者所變動到的資料，比方說:Bios、Bootstrap、 Operating System (作業系統)…等等，
這些資料通常會在開機之後的第一時間被載入 Main Memory(RAM) ，接著傳給 CPU 執行。
```

- **PROM(Programmable ROM)(可邊程式唯讀記憶體)**
```
PROM 與 ROM 的差異就在於: PROM 可以在完成之後再把資料寫入，一樣還是只能寫一次，而且不能改。
```
- **EPROM(Erasable Programmable ROM)(可抹除可編程式唯讀記憶體)**
```
可以進行的消除的 PROM。
不過修改時必須先將它從電腦上拆下來並用強紫外線來照射，所以必須要有一台專門的機器再才可以進行消除。
```

- **EEPROM(Electrically Erasable Programmable ROM)(電子抹除式可複寫唯讀記憶體)**
```
用電壓來進行消除的動作的，這讓它在進行修改時，可以不用從電腦上拆下來，也不用有專門的機器。
改善了 EPROM 所有的缺點，所以當 EEPROM 出現之後，很快就取代了所有的 EPROM。
```


### @ References
- [CPU與記憶體(Memory)的運作機制](https://medium.com/@leo19866/cpu%E8%88%87%E8%A8%98%E6%86%B6%E9%AB%94-memory-%E7%9A%84%E9%81%8B%E4%BD%9C%E6%A9%9F%E5%88%B6-2e33bd9b0858)



## **< 三. Memory Hierarchy: 記憶體階層 >**

單純討論資料存取(存與取)，其實在電腦領域範疇，有各種存取。

ex: 在 [一.CPU] 提到的 Register 也是存取，是在記憶體階層的最頂端。
ex: 在 [二.Memory] 提到的 DRAM, SRAM, ROM, Cache, SSD 也都是存取。
ex: Web 領域使用到的 Persistent Cookies, 也會是存在 HHD(Hard Disk Drive/硬碟)，硬碟也是一種存取，他在記憶體階層的底端。

> 所以可以廣義的說，其實都是記憶體，只是特性、結構、用法有所不同。
> 而平常溝通常說的記憶體:
> 我的認知大部分指的是 RAM 或是說 Main Memory。

記憶體階層的快慢/容量大小:[(Wiki-記憶體階層)](https://zh.wikipedia.org/wiki/%E8%A8%98%E6%86%B6%E9%AB%94%E9%9A%8E%E5%B1%A4)

<div style="display: flex; justify-content: center;">
    <img width="50%" src="https://i.imgur.com/KLuRrih.png"/>
</div>

> [圖片來源](https://kopu.chat/2017/05/06/dram%E3%80%81flash%E8%B2%B4%E5%88%B0%E7%82%B8%EF%BC%8C%E4%BD%A0%E9%82%84%E6%90%9E%E4%B8%8D%E6%87%82%E8%A8%98%E6%86%B6%E9%AB%94%E7%9A%84%E5%B7%AE%E7%95%B0%E5%97%8E%EF%BC%9F/#:~:text=Flash%EF%BC%88%E5%BF%AB%E9%96%83%E8%A8%98%E6%86%B6%E9%AB%94,%E4%B9%9F%E6%AF%94NAND%20Flash%20%E8%B2%B4%E3%80%82)



## **< The End >**

### Self Checklist:
- [x]What is CPU.
- [x]What is Memory.
- [x]What is the difference between Volatile Memory and Non-volatile Memory。
- [x]What is the difference between RAM and ROM。
- [x]What is Register in CPU.
- [x]What is the difference between HHD and SSD.
- [x]Understand Memory Hierarchy.
- [x]Understand basic work flow between memory and CPU inside.
