---
title: "01「Git 介紹」什麼是 git rebase"
subtitle: "git rebase 基本使用場景"
date: 2021-06-15T18:36:14+08:00
draft: false
url: "posts/02/210615/01「Git 介紹」什麼是 git rebase"
summary: <p align="center">最近又整理了一下自己對 git 的概念，分享一下</p> <img src="/blog/images/02_210615/02_210615_git_rebase.png" width="70%"/>
author: "linooohon"
email: "linooohon@gmail.com"
toc: true
tags: ["Git"]
categories: ["2021", "git"]
---

<h1 style="text-align:center">
「Git 介紹」Ch1: 什麼是 git rebase
</h1>

> 最近自己對 git 的概念又整理了一下，先來介紹一下 git rebase

![Git 介紹」Ch1: 什麼是 git rebase](/blog/images/02_210615/02_210615_git_rebase.png)

## 一. (What?) git rebase 可以做什麼 ?
### 除了 `git merge` 的另一種選擇
   - merge 的常見三種方式: `git rebase`, `git merge`, `git cherry-pick`
     - 一般做 merge 常見大概會有 rebase、merge、cherry-pick，這篇只先針對 rebase 做分享，rebase 可以做的事情算是滿多的。
   - 與 git merge 的差別 ?
     - 最主要在使用 `git merge` 的時候，如果你有注意，其實都會再產生一個做 merge 這件事情的 commit，在 `git log --graph --all` 線圖上就不會是線性，線會轉彎然後合起來，最常見的情況是會產生小耳朵(小耳朵link)。然而在使用 `git rebase` 做 merge 時，它會是用接點的方式接在前面，rebase 顧名思義，重新定義 base(它的根)，而當然實際 git 的實作， 不是單純拿原本的 commit 接上去，其實是複製了一份過去，SHA-1 值也是重新產生，是會改變的。


### 有一個狀況 git merge 還是會保持一條線
> (除非parent branch 的 commit 始終包含在 child branch 裡，parent branch 從來沒變更過，現在要把 parent 的進度更新成 child 的進度，這時候的 merge 會走 ff(fast-forward)，也就是說不會產生額外的 commit，但這裡不討論這個，先假設 branch 間已經有分歧了，也是合作比較常見的狀況)。

> 如果在上述情況下 git merge 不想走 ff 可以 `git merge --no-ff` 這樣一定會產生一個 commit， git 會請你再留一個 commit message。

## 二. 以下分享 `git rebase`, `git merge` 的差異

### Ⅰ. 先來看 `git merge`，接下來藉由圖片和實例讓大家參考: 

如果是 `git merge` 的情況:
![](https://i.imgur.com/3Jb3U5g.png)
<center>"圖1(Sourcetree)"</center>


![](https://i.imgur.com/ElOabeG.png)

<center>"git log --graph --all --oneline 也是一樣的"</center>



- 目前我在 `master` 加了 `add 01 project` commit 接著在 `master` commit 完後，建了一個分支 `add-02project`，並 commit 了三次，停在 `feat: add 02.js`，接著我繼續從 `add-02project` 分支建了 `add-03project` 分支，並 commit 一次 `feat: add 03.html` ，然後我回頭從 `master` 建 `add-04project` 分支 commit 一次 `feat: add 04.html`，以上都只是我亂建的不重要，只是為了講解一下。
        
> 接下來重點是我想讓 `add-04project` branch 與 `add-03project` branch 做 merge，<br>這裡如果我先做 `git merge`：

- 目前 HEAD 在 add-04project，下指令
```
$ git merge add-03project
```
> 意思是 : merge branch `add-03project` into branch `add-04project`

接著，

![](https://i.imgur.com/AXL1adk.png)
<center>"圖2"</center>
        
> 你會看到線圖產生變化，多了一個 commit `cbcfe7b`  是 for merge 的，然後線圖也還是原本的線，只是接在一起了。



### Ⅱ. 那現在來看如果使用 `git rebase` 

  - 原本的情況和圖1一樣，HEAD 一樣在 `add-04project` branch :
![](https://i.imgur.com/3Jb3U5g.png)
<center>"圖1"</center>
  
在這裡下 
  ```
  $ git rebase add-03project
  ``` 
  > 字面意思是請幫我把我當前的 branch 的根換成 `add-03project` branch，所以實際上它會把當前 branch 進度複製接到 `add-03project` branch 上，這時候會呈現如下:
  
![](https://i.imgur.com/WsdMcBl.png)
<center>"圖3"</center>

- 會發現變成一條線了，且 `add-04project` commit 內容也都在，但 SHA-1 會重新計算。

## 三. rebase 反悔了怎麼辦：

### `git reset --hard <SHA-1值>`
> 記得在使用 git rebase 的時候，如果反悔了剛剛的操作，不能用 `git reset --hard HEAD^` 做相對路徑切換， 它是不會切回原樣的，要記得直接用絕對路徑切換，下 `git reflog` 看想切回去的地方，然後下 `git reset --hard <SHA-1值>` 也就是直接放 commit ID，或著是 reflog 上的 HEAD@{x}，`git reset --hard HEAD@{x}`

```bash

$ git reset --hard <SHA-1值>  #找你想要回去的點
$ git reset --hard HEAD@{x}   #找你想要回去的點
$ git reset --hard ORIG_HEAD  #反悔離當下最近的危險操作點

'''
要注意的是 ORIG_HEAD 是在危險操作時紀錄上一個點，如果沒有在操作過後的當下馬上使用 git reset --hard ORIG_HEAD
ORIG_HEAD 很有可能會被蓋過去就不能用了，這時後就要使用其他兩個指令，可以自己選想回去的地方
'''
```


## 四. `git rebase -i` (會進入 rebase 的互動模式，之後分享一下)
- 我把它稱作小小時光機，但就像 [Back to the Future](https://www.imdb.com/title/tt0088763/) 演的那樣，很酷也容易不小心出問題。
- 可以做滿多事，但也要注意有時會更動到歷史紀錄，在多人協作專案，更是要小心操作。
   
## 五. (Why?) 為什麼需要 git rebase 

### 先說結論：

 - 直接先說我自己認為的結論，`git rebase` 只是工具，不管是 git merge、git rebase 還是其他 git 指令，其實都是因應情況、團隊開發模式等情況下，去做合理使用，只要在合適的情況使用 rebase 我相信是沒問題的，使用 rebase 有時可以讓專案的進度 (commit 顆粒) 更清晰。


### 那什麼是合適的情況 ? :
以下情況我根據自身經驗認為會使用或是可以使用:
#### 1. 遠端 master(main) branch 的 code 進度更新了， 這時候自己的分支開發了一個功能，準備推上去，可以使用 `rebase` 讓自己的 branch 進度更新，最後再推上去自己的分支
```
$ git fetch upstream
$ git rebase upstream/master
$ git push origin 自己的 branch
```
#### 2. 同理更新自己的 fork repo:
```
$ git checkout master
$ git fetch upstream
$ git rebase upstream/master
$ git push origin master
```
#### 3. 整理 git 歷史紀錄，可以修改、合併、變更、刪除等等..
```
# 使用互動模式
$ git rebase -i <SHA-1>
```

---
## The End:



### Review:
- `git rebase` 與 `git merge` 的差異
- `git rebase` 反悔如何處理
- `git rebase` 使用時機的取捨
- 在團隊協作中善用工具，follow 大家的共識

### Reference:
- [git](https://git-scm.com/docs/git-rebase)
- [Git Branching!](https://learngitbranching.js.org/?locale=zh_TW)
- [王雨峰的博客](http://wangyufeng.org/2020/03/15/post-20200315/)
- [高見龍-為你自己學 Git](https://gitbook.tw/chapters/branch/merge-with-rebase.html)


## Next 下一篇:
<a href="/blog/posts/03/210620/02「Git 介紹」git rebase -i">02「Git 介紹」git rebase -i</a>



