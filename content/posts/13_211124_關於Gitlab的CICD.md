---
title: "[CI/CD]幫 Data Team 設置 GitLab 的 CI/CD"
subtitle: "GitLab CI/CD & CentOS Docker Install"
date: 2021-11-24T18:36:14+08:00
draft: false
url: "posts/13/211124/GitLab_CI-CD/"
summary: <p align="center">最近在公司開始整理一些 Legacy Code，在標記的同時，也開始設置 Python 版的 Monorepo，這裡紀錄一下目前幫 Data Team 初期設置的 GitLab CI/CD</p> <img src="/blog/images/13_211124/13_211124_gitlabcicd.png" width="20%"/>
author: "linooohon"
email: "linooohon@gmail.com"
toc: true
tags: ["Data", "GitLab", "CI/CD", "Docker", "Linux", "Python"]
categories: ["2021", "cicd"]
---

![https://linooohon.com/blog](/blog/images/13_211124/13_211124_gitlabcicd.png)

## **< 需求 >**:
### GitLab CI/CD
**設置 Data Team monorepo 在 GitLab 的 CI/CD ( 包括 Crawler - Zyte Scrapy 的 Deploy 設置)**


## **< 注意 >**:
1. GitLab 需要設定 `runner` ( 一般在 local 自己設定也可以啟動，而以目前公司為例，必須進到 AWS EC2 裡設定)。
2. 為了設定 Python環境，目前我先使用 "Docker" executor 的 runner，如此可以設定目標版本，`python:3.9.5`，(還有其他種的 executor 像是 "shell" 等等.. [GitLab Executors](https://docs.gitlab.com/runner/executors/)。
3. `runner` 設定時候的 tags 在後來 `.gitlab-ci.yml` 裡定義 Jobs 時標註的 tags 雙方要一樣，不然 CI/CD 是不會跑起來的。
4. 必須讓 docker engine 跑起來，所以也要在 EC2 裡下載 Docker Engine。
5. 在 CI/CD 的時後，也需要一些環境變數，可以在 GitLab 的 repo 裡的 Settings > CI/CD > Variables 這邊可以設置。


## **< 以下稍微紀錄重點 >**:

### 1. GitLab CI/CD 須設置 `.gitlab-ci.yml`


> 一些設定

- `image`: 我 runner 使用的是 docker executor，所以這樣設置

```yml
# image 的設定就會是:
image: python:3.9.5
```

- `tags`: 讓 Job 找到 runner 就需要 tags

```yml
tags:
  - python3.9.5_ci_runner
```

- `stages`: 定義 CI/CD 時候 Pipeline 的每個 Job(也就是在 GitLab 上那一個個的圈圈)

```yml
# 基本會類似像這樣
stages:
  - build
  - lint
  - test
  - deploy crawler
```

- `before_script`: 這個可以跑在每個 Job 的 `script` 之前

- `extends`: 可以放在 Jobs 一起定義 tags 和 image

```yml
.aws_ci_builder:
  tags:
    - python3.9.5_ci_runner
  image: python:3.9.5
```

- `artifacts`: 可以給下一個 Job 使用

- `script`: 就跑 script

- `needs` 和 `dependencies`: needs 是不管如何都會執行的 Job, 而 dependencies 則顧名思義，它依賴於那個 Job，接下來它才能夠執行。假設有一個 Job 叫做 `build`

```yml
# dependencies
dependencies:
  - build
```

如果 `needs` 需要某個 job 的 artifacts 可以這樣設置:
```yml
needs:
  - job: build
    artifacts: true
```

### 2. Docker 的 Install

下載步驟(以我公司 CentOS 為例)：

官方推薦 Set up docker's repositories 的方式下載

**1. 所以在下載之前要先 Set up 好 Repositories**

1-1. 下載 yum-utils
```bash
sudo yum install -y yum-utils

# 之所以下載 yum-utils 是因為我們要使用它的 yum-config-manager 功能
```


1-2. Set up repo
```bash
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```


**2. 設定完畢後就可以下載 Docker Engine**


- 選擇1. 直接下載最新版的 Docker Engine
```bash
sudo yum install docker-ce docker-ce-cli containerd.io
```


- 選擇2. 自行選擇版本

```bash
# 先列出版本清單
yum list docker-ce --showduplicates | sort -r
```

```bash
# 會顯示像這樣
docker-ce.x86_64    3:20.10.9-3.el7                            docker-ce-test
docker-ce.x86_64    3:20.10.9-3.el7                            docker-ce-stable
docker-ce.x86_64    3:20.10.8-3.el7                            docker-ce-test
docker-ce.x86_64    3:20.10.8-3.el7                            docker-ce-stable
docker-ce.x86_64    3:20.10.7-3.el7                            docker-ce-test
docker-ce.x86_64    3:20.10.7-3.el7                            docker-ce-stable
docker-ce.x86_64    3:20.10.6-3.el7                            docker-ce-test
docker-ce.x86_64    3:20.10.6-3.el7                            docker-ce-stable
docker-ce.x86_64    3:20.10.5-3.el7                            docker-ce-test
```

```bash
# 選擇要裝的版本，然後給他裝下去
sudo yum install docker-ce-<VERSION_STRING> docker-ce-cli-<VERSION_STRING> containerd.io
```

<VERSION_STRING> 要找第二欄的 `3:20.10.9-3.el7` 這邊，但不是全部放，只要冒號後面一直到 hyphen 之前就好，舉例: 我要 `3:20.10.9-3.el7`，那就會是

> <VERSION_STRING> 要替換成 `20.10.9`

```bash
sudo yum install docker-ce-20.10.9 docker-ce-cli-20.10.9 containerd.io
```


**3. 啟動 Docker Engine**

```
sudo systemctl start docker
```

**4. 如果要確認 Docker Engine 有沒有 active，可以下**


```bash
sudo systemctl is-active docker
# 如果顯示 active 那就是對了
```

如果想設置 EC2 runner 啟動的時候 Docker Engine 就啟動，那可以
```bash
sudo systemctl enable docker.service
sudo systemctl enable containerd.service
```

[官方](https://docs.docker.com/engine/install/linux-postinstall/#configure-docker-to-start-on-boot)



### 3. yml檔的 script 操作:

**- 如果 script 是需要互動的輸入 Key 或是 Password 之類的，就可以很簡單的 echo 它:**

```yml
# 以 Zyte Deploy 工具 `shub` 為例 
echo $ZYTE_DEPLOY_API_KEY | shub login
```

**- yml 檔裡如果有 script 需要多行**:

```yml
script:
  - |
    cat << EOF > example.txt
    123
    456
    EOF
```

**- yml 檔裡如果要用 GitLab 的 Environment Variables:**

```yml
$變數名
```

## **< The End >**:

結束！


### Self Checklist:
- [x]How to setting `.gitlab-ci.yml` file.
- [x]How to setting `GitLab runner`.
- [x]How to write the basic script. 
- [X]Basic Docker concept.
