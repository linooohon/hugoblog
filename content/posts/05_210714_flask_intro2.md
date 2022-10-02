---
title: "「Flask」02: Static File handling, Database connection"
subtitle: "Flask 基本操作 02!(EN)"
date: 2021-07-14T18:36:14+08:00
draft: false
url: "posts/05/210714/「「Flask」02: Static File handling, Database connection"
summary: <p align="center">上次紀錄了 Flask 基本處理，這次紀錄 Static File 和 DB 連線(EN)</p> <img src="/blog/images/04_05_210710/04_05_210710_flask_intro.png" width="30%"/>
author: "linooohon"
email: "linooohon@gmail.com"
toc: true
tags: ["Flask", "Python", "Web"]
categories: ["2021", "python", "flask"]
---

<h1 style="text-align:center">
「Flask」02: Static File handling, Database connection
</h1>

> 前陣子用 Flask 實做了一個 LifePlaylist App 這裡簡單紀錄一下基本的東西，以後回頭更快重拾記憶🤧 -  第二篇

![上次紀錄了 Flask 基本處理，這次](/blog/images/04_05_210710/04_05_210710_flask_intro.png)

## **< Intro >**
"A lightweight Python Web Framework ! " ----> 
[Flask Docs](https://flask.palletsprojects.com/en/latest/quickstart/)


## **< Handle Static File >**

### Basic:
Path: /static/images/01.jpg

```bash
## HTML:
<img src="/static/images/01.jpg">
## Browser:
<img src="/static/images/01.jpg">
```
### Change the path:
Path: /views/assets/01.jpg
```bash
app = Flask(__name__, static_folder="views/assets/")
app = Flask(__name__, static_folder="images/")

## or
app = Flask(__name__, static_folder="./views/assets/")
app = Flask(__name__, static_folder="./images/")

## HTML:
<img src="{{ url_for('static', filename='01.jpg') }}">
# <img src="01.jpg">
## Browser:
<img src="/assets/01.jpg">
<img src="/images/01.jpg">
```

### Change the path that show in browser:
```bash
app = Flask(__name__, static_url_path="/Hello", static_folder="views/assets/")
## HTML:
<img src="{{ url_for('static', filename='01.jpg') }}">
# <img src="/Hello/01.jpg">
## Browser:
<img src="/Hello/01.jpg">
```
> But actually image is still under the path `views/assets/`

## **< Database Connection >**

### Flask-SQLAlchemy 
- Also support SQLAlchemy
- Raw SQL / ORM

> Flask-SQLAlchemy: [Docs](https://flask-sqlalchemy.palletsprojects.com/en/2.x/)

> SQLAlchemy: [Docs](https://docs.sqlalchemy.org/en/14/glossary.html#term-2.0-style)

```
pip install flask-sqlalchemy
```
```
from flask_sqlalchemy import SQLAlchemy
```

### Setting URI:
```
app.config['SQLALCHEMY_DATABASE_URI'] = [DB_TYPE]+[DB_CONNECTOR]://[USERNAME]:[PASSWORD]@[HOST]:[PORT]/[DB_NAME]
```
### SQLite e.g.:
```bash
app.config['SQLALCHEMY_DATABASE_URI'] = "sqlite:////db/testing.db"   ("sqlite:///" + absolute path + name.db)
## folder: /db/
## db name: testing.db
## 'sqlite:////absolute/path/to/foo.db'


## -----------
import os
basedir = os.path.abspath(os.path.dirname(__file__))
def create_sqlite_uri(db_name):
    return "sqlite:///" + os.path.join(basedir, db_name)
```

### MySQL e.g.:
- Need pymysql
```
pip install pymysql
```
```bash
app.config['SQLALCHEMY_DATABASE_URI'] = 
mysql+pymysql://[user_name]:[user_password]@[HOST]:3306/[db_name]
## e.g.

mysql+pymysql://root:password@localhost:3306/myapp
```

### PostgreSQL e.g.:
- Need psycopg2-binary
```
pip install psycopg2-binary
```
```bash
app.config['SQLALCHEMY_DATABASE_URI'] = 
postgresql://[user_name]:[user_password]@[HOST]:5432/[db_name]
## e.g.
## no password
postgresql://linpinhung:linpinhung@localhost:5432/app_pro

## HOST should be different depend on where your db is, for example, I set it in Docker, then my HOST will depend on what I configure.
postgresql://name:name@db:5432/app_docker

```


### Config:
```
app.config['SQLALCHEMY_ENGINE_OPTIONS'] = {
    'pool_pre_ping': True,
    'pool_recycle': 300,
    'pool_timeout': 900,
    'pool_size': 10,
    'max_overflow': 5,
    .....: ...,
    ....
    ....
}
```
- SQLALCHEMY_ECHO(For debug):
    - Default: False
    - If True, print Raw SQL in console
- SQLALCHEMY_RECORD_QUERIES
    - If True, can check query detail
```
from flask_sqlalchemy import get_debug_queries

def after_request(response):
    for query in get_debug_queries():
        print(query.statement) 
        print(query.parameters) 
        print(query.duration)
        print(query.context) 
    return reponse
```
> get_debug_queries can also use in other ways.


- SQLALCHEMY_TRACK_MODIFICATIONS:
    - Mostly set with False.
    - Flask recommend using SQLAlchemy events directly [Signalling Support](https://flask-sqlalchemy.palletsprojects.com/en/master/signals/)

- SQLALCHEMY_BINDS
    - For handling multiples database.


### Basic Model Fromat:

> model/models.py
```python
from flask_login import UserMixin
from sqlalchemy.sql import func

from app import db


class Playlist(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    artist = db.Column(db.String(500))
    artist_spotify_uri = db.Column(db.String(500))
    artist_spotify_image_url = db.Column(db.String(500))
    song = db.Column(db.String(500))
    artist_genres = db.Column(db.JSON)
    date = db.Column(db.DateTime(timezone=True), default=func.now())  # 拿到當下時間
    user_id = db.Column(db.Integer, db.ForeignKey('user.id'))  # 拿使用者 id 當 fk
    # user_id = db.Column(db.Integer, db.ForeignKey('user.id'), nullable=False)  # 拿使用者 id 當 fk
    test_migrate_col = db.Column(
        db.String(120), nullable=True)  # 預設就是 nullable=True 可以不寫

    def __init__(self, artist, song, user_id):
        self.artist = artist
        self.song = song
        self.user_id = user_id
        
        
class User(db.Model, UserMixin):
    id = db.Column(db.Integer, primary_key=True)
    email = db.Column(db.String(150), unique=True)
    password = db.Column(db.String(150))
    first_name = db.Column(db.String(150))
    playlists = db.relationship('Playlist')
    # playlists = db.relationship('Playlist', backref='user')

    def __init__(self, email, password, first_name):
        self.email = email
        self.password = password
        self.first_name = first_name
```



### Data Manipulation(CRUD):
- Get:
```
self.model.query.get(id)
Playlist.query.get(id)
```
- Get All:
```
self.model.query.all()
Playlist.query.all()
```
- Get with filter:
```bash
## method 1
self.model.query.filter_by(**filter_dic).all()

## method 2
my_filters = {'artist':'xxxxx', 'song':'xxxxx'}
query = session.query(self.model)
for attr,value in my_filters.iteritems():
    query = query.filter( getattr(self.model,attr)==value )
results = query.all()
```
- Delete:
```
row = self.get_data(id)
db.session.delete(row)
db.session.commit()
```

- Insert:
```bash
new_top10_data = Dashboard(
            dashboard_artist=i.artist, dashboard_song=i.song, artist_spotify_uri=i.spotify_uri, artist_spotify_image_url=i.spotify_image_url, song_youtube_url=i.youtube_url, artist_genres=i.genres)
db.session.add(new_top10_data)
db.session.commit()

## -----------------
## Usage:
   ##  Dic: **Dic
   ##  List: *List
def insert_data(self, insert_dic):
    new_data = self.model(**insert_dic)
    db.session.add(new_data)
    db.session.commit()
```






## **< The End >**:

### Review:
- Static File handling try to use `url_for` will be better.
- `static_url_path` can hide the real path.
- DB connection should careful for the `path` / `host, user, password` / `dependencies`


### Reference:
- [Max行銷誌](https://www.maxlist.xyz/2019/11/10/flask-sqlalchemy-setting/)
- [FlaskSQLAlchemy](https://flask-sqlalchemy.palletsprojects.com/en/2.x/)
- [Sean Yeh in Python Everywhere -from Beginner to Advanced](https://medium.com/seaniap/python-web-flask-%E4%BD%BF%E7%94%A8%E9%9D%9C%E6%85%8B%E6%AA%94%E6%A1%88-ac00e863a470)

## **< Next 上一篇 >**:
<a href="/blog/posts/04/210710/「Flask」01: Basic, Route, Templates, GET, POST">「Flask」01: Basic, Route, Templates, GET, POST</a>