---
title: "[Flask] Sendgrid/Gmail - <三種寄信設定>"
subtitle: "<p style='font-size: 17px;'>Flask Email Setting in three ways.</p>"
date: 2022-03-15T18:36:14+08:00
draft: false
url: "posts/21/220316/Flask_Email_Setting_三種寄信設定"
summary: <p align="center">最近重新設定 flask app 的寄信設定，紀錄一下</p><img src="/blog/images/21_220315/21_220315_cover.png" width="30%"/>
author: "linooohon"
email: "linooohon@gmail.com"
toc: true
tags: ["Flask", "Python", "Web", "Sendgrid", "Email"]
categories: ["2022", "python", "flask"]
tocNum: false
---

---

> 此篇完整 source code 放在 Github Repo - [flask-mail-setting](https://github.com/linooohon/flask-mail-setting) 供參考！

過去曾經使用 [flask-mail](https://pythonhosted.org/Flask-Mail/) 這個模組，但它很久沒維護了，想說找其他的試試來重新設定寄信，發現有 [flask-mailman](https://github.com/waynerv/flask-mailman) 可以使用，嘗試改用它發信，同時也熟悉一下 [sengrid](https://sendgrid.com/) 的設定，接下來會大概紀錄三種設定:


- 1. 使用 `flask_mailman` module 以 gmail 為 mail server 寄信
- 2. 使用 `flask_mailman` module 以 sendgrid 為 mail server 寄信
- 3. 單純使用 `sendgrid` module 提供的 Web API 完成寄信


#### **一. < 使用 `flask_mailman` module 以 gmail 為 mail server 寄信 >**

> Pre-request:

- 使用 module & version
    - flask
    - flask-mailman
    - python-dotenv
```
flask==2.0.3
flask-mailman==0.3.0
python-dotenv==0.19.2
```

- 需有一個 Email Account，如果是 Google Account 要去關閉 [低安全性應用程式存取權](https://myaccount.google.com/u/1/lesssecureapps)



##### Step.1 建環境
```bash
mkdir flask_mail_demo
cd flask_mail_demo
python3 -m venv venv # 因為是 demo 所以這裡就快速用 venv 建 Virtual Environment，平時開發我自己比較喜歡 poetry，更能清楚管理 dependencies
source ./venv/bin/activate
pip install flask
touch gmail_mailman.py
```

##### Step.2 然後在 `gmail_mailman.py` 寫 code 建立 Flask App，簡單啟動一個 flask server
```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def index():
    return 'Flask App 啟動了！'


if __name__ == '__main__':
    app.run()

# 如此執行 `python gmail_mailman.py` 就可以 run flask app 在 http://127.0.0.1:5000/
```


##### Step.3 從第二步延伸: 在 `flask_mail_demo/` 底下
```bash
加上 `.env` file # 放 gmail password 
加上 `config.py` file # 設定 mail 發送參數
```


##### Step.4 `.env` file 內容為 gmail password:

```
GMAIL_PASSWORD=<換成自己的 gmail 密碼>
```


##### Step.5 `config.py` file 內容為:

```python
import os
import dotenv
dotenv.load_dotenv()

class GmailConfig():
    DEBUG = True
    TESTING = False
    
    # MAIL Config
    MAIL_SERVER='smtp.gmail.com'
    MAIL_PORT=465
    MAIL_USE_SSL=True
    MAIL_USERNAME='lifeplaylistforfun@gmail.com' # MAIL_USERNAME 一定要放上 email 帳號
    MAIL_DEFAULT_SENDER='lifeplaylistforfun@gmail.com'
    MAIL_PASSWORD=os.getenv('LIFEPLAYLISTFORFUN_PASSWORD')
    
config = {
    'gmail': GmailConfig,
}
```

1. 把 env load 進來
2. 設計 `GmailConfig()` class
3. config variable 可以切換多個 config class，後面會用到。

MAIL_SERVER='smtp.gmail.com' `gmail server` \
MAIL_PORT=465 `使用 port 465` \
MAIL_USE_SSL=True `SSL 要為 True` \
MAIL_USERNAME=<mail 帳號> \
MAIL_PASSWORD=<mail 密碼> \
MAIL_DEFAULT_SENDER=<Default 寄信人 mail 帳號>

Refer to:
- https://support.google.com/mail/answer/7126229?hl=en#zippy=%2Cstep-change-smtp-other-settings-in-your-email-client


##### Step.6 回到 `gmail_mailman.py` 修改裡面的內容:


```python
from flask import Flask
from flask_mailman import Mail
from flask_mailman import EmailMessage
import config

app = Flask(__name__)
app.config.from_object(config.config['gmail'])

mail = Mail()
mail.init_app(app)

def send_email():
    subject = 'hello'
    from_email = 'lifeplaylistforfun@gmail.com'
    to = 'linooohon@gmail.com'
    html_content = '<h1>Test Mail</h1><p>From flask_mailman module, using gmail config</p>'
    msg = EmailMessage(subject, html_content, from_email, [to])
    msg.content_subtype = 'html'
    response = msg.send()
    print(response)

@app.route('/')
def index():
    send_email()
    return 'Flask app 已啟動，沒報錯的話信已寄出'


if __name__ == '__main__':
    app.run()
```

1. 把 flask_mailman 的 source code import 進來使用
2. 修改 config 的參數，改成自己的 mail，import 進來，並給 app 使用
3. 把 app 帶入 Mail instance
4. 設定好 `send_email` function，一樣把 `to` 改成收得到信的 mail，並放到 `@app.route('/')` 底下
5. 記得先進 Virtual Environment
6. 執行 `python gmail_mailman.py`，並訪問 `http://127.0.0.1:5000/`
7. 確定路徑沒錯，沒有錯誤訊息的話，去信箱就會看到信了！

<img src="/blog/images/21_220315/21_220315_flaskemail_02.png" width="40%">
<br/>
<img src="/blog/images/21_220315/21_220315_gmail_mailman.png" width="40%">



#### **二. < 使用 `flask_mailman` module 以 sendgrid 為 mail server 寄信 >**


跟 1. 相似，只是 mail 參數設定不同，以及需要去 sendgrid 官網註冊一個帳號，拿取 `Secret Key(Token)` 也就是 MAIL_PASSWORD 要帶入的地方


> Pre-request:

- 使用 module & version
    - flask
    - flask-mailman
    - python-dotenv
```
flask==2.0.3
flask-mailman==0.3.0
python-dotenv==0.19.2
```

- 需有一個 Email Account，如果是 Google Account 要去關閉 [低安全性應用程式存取權](https://myaccount.google.com/u/1/lesssecureapps)


- 需要去 [sendgrid - sign up](https://signup.sendgrid.com/) 註冊一個帳號

##### Step.1 在 sendgrid 註冊完帳號後，在裡面操作拿取 `Secret Key` (API Key)

Go to `Settings` -> `API Keys`：然後申請一個，拿到後保存下來。
<img src="/blog/images/21_220315/21_220315_sendgrid_01.png" width="20%">



##### Step.2 拿完 `Secret Key` 還必須要註冊一個 Single Sender Verification

Go to `Settings` -> `Sender Authentication`：然後申請一個，拿到後保存下來。

<img src="/blog/images/21_220315/21_220315_sendgrid_02.png" width="20%">

<img src="/blog/images/21_220315/21_220315_sendgrid_03.png" width="30%">


##### Step.3 在剛剛的 `flask_mail_demo/` 底下修改 `.env` `config.py`


- `.env`
```bash
GMAIL_PASSWORD=<換成 mail 密碼>
MAIL_PASSWORD=<換成剛剛從 Sendgrid 拿到的 API Key>
```

- `config.py`

```python
import os
import dotenv
dotenv.load_dotenv()

class GmailConfig():
    DEBUG = True
    TESTING = False
    
    # MAIL Config
    MAIL_SERVER='smtp.gmail.com'
    MAIL_PORT=465
    MAIL_USE_SSL=True
    # MAIL_USE_TLS=True
    MAIL_USERNAME='lifeplaylistforfun@gmail.com' # MAIL_USERNAME 一定要放上 email 帳號
    MAIL_DEFAULT_SENDER='lifeplaylistforfun@gmail.com'
    MAIL_PASSWORD=os.getenv('LIFEPLAYLISTFORFUN_PASSWORD')

class SendgridConfig():
    DEBUG = True
    TESTING = False
    
    # MAIL Config
    MAIL_SERVER='smtp.sendgrid.net'
    MAIL_PORT=587
    MAIL_USE_TLS=True
    MAIL_USERNAME='apikey' # MAIL_USERNAME 一定要放上 apikey
    MAIL_DEFAULT_SENDER='lifeplaylistforfun@gmail.com'
    MAIL_PASSWORD=os.getenv('MAIL_PASSWORD')

config = {
    'gmail': GmailConfig,
    'sendgrid': SendgridConfig,
}
```

MAIL_SERVER='smtp.sendgrid.net' \
MAIL_PORT=587 `port 587` \
MAIL_USE_TLS=True `TLS 為 True` \
MAIL_USERNAME='apikey' `MAIL_USERNAME 一定要放上 apikey` \
MAIL_DEFAULT_SENDER=<mail 帳號> \
MAIL_PASSWORD=os.getenv('MAIL_PASSWORD')




##### Step.4 一樣在 `flask_mail_demo/` 底下建立 flask app

```bash
touch sendgrid_mailman.py
```

- 裡面的內容跟 Gmail 是一樣的，把 config 調整成 sendgrid 的就行:

```python

from flask import Flask
from flask_mailman import Mail
from flask_mailman import EmailMessage
import config

app = Flask(__name__)
app.config.from_object(config.config['sendgrid'])

mail = Mail(app)
mail.init_app(app)

def send_email():
    subject = 'hello'
    from_email = 'lifeplaylistforfun@gmail.com'
    to = 'linooohon@gmail.com'
    html_content = '<h1>Test Mail</h1><p>From flask_mailman module, using sendgrid config</p>'
    msg = EmailMessage(subject, html_content, from_email, [to])
    msg.content_subtype = 'html'
    response = msg.send()
    print(response)

@app.route('/')
def index():
    send_email()
    return 'Flask app 已啟動，沒報錯的話信已寄出'


if __name__ == '__main__':
    app.run()
```
1. 一樣該改成自己 mail 的都要改，記得進 Virtual Environment。
2. 執行 `python sendgrid_mailman.py`，並訪問 `http://127.0.0.1:5000/`
3. 路徑沒錯，沒報錯的話就會看到信，成功！

<img src="/blog/images/21_220315/21_220315_sendgrid_mailman.png" width="30%">


#### **三. < 單純使用 `sendgrid` module 提供的 Web API 完成寄信 >**

> Pre-request:

- 使用 module & version
    - flask
    - sendgrid
    - python-dotenv
```
flask==2.0.3
sendgrid==6.9.7
python-dotenv==0.19.2
```

- 需有一個 Email Account，如果是 Google Account 要去關閉 [低安全性應用程式存取權](https://myaccount.google.com/u/1/lesssecureapps)


##### Step.1 與前一個拿 sendgrid 的 Secret Key 一樣

- 這裡就使用跟上面一樣的 `Secret Key`，就用 `SendgridConfig()` class

##### Step.2 一樣在 `flask_mail_demo/` 底下建立 flask app

> 其實可以不用建立 flask app，這是個單純的寄信 API，但這裡還是 demo 啟動 Flask App 來寄出信。

```bash
touch sendgrid_webapi.py
```

內容:

```python
from flask import Flask
from sendgrid.helpers.mail import Mail
from sendgrid import SendGridAPIClient, Content
import config

app = Flask(__name__)

def send_email():
    subject = 'hello'
    to_emails = 'linooohon@gmail.com'
    from_email = 'lifeplaylistforfun@gmail.com'
    mail_html = Content('text/html', '<h1>Test Mail</h1><p>From Sendgrid Web API</p>')
    msg = Mail(from_email, to_emails, subject, mail_html)
    try:
        sg = SendGridAPIClient(config.config['sendgrid']().MAIL_PASSWORD)
        response = sg.send(msg)
        print(response.status_code)
        print(response.body)
        print(response.headers)
    except Exception as e:
        print(e.message)

@app.route('/')
def index():
    send_email()
    return 'Flask app 已啟動，沒報錯的話信已寄出'


if __name__ == '__main__':
    app.run()
```

1. 一樣該改成自己 mail 的都要改，記得進 Virtual Environment。
2. 執行 `python sendgrid_webapi.py`，並訪問 `http://127.0.0.1:5000/`
3. 路徑沒錯，沒報錯的話就會看到信，成功！

<img src="/blog/images/21_220315/21_220315_sendgrid_webapi.png" width="30%">

