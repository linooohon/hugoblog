---
title: "「Flask」01: Basic, Route, Templates, GET, POST"
subtitle: "Flask 基本操作!(EN)"
date: 2021-07-10T18:36:14+08:00
draft: false
url: "posts/04/210710/「Flask」01: Basic, Route, Templates, GET, POST"
summary: <p align="center">前陣子實做了 Flask App 紀錄一下基本處理! (EN)</p> <img src="/blog/images/04_05_210710/04_05_210710_flask_intro.png" width="30%"/>
author: "linooohon"
email: "linooohon@gmail.com"
toc: true
tags: ["Flask", "Python", "Web"]
categories: ["2021", "python", "flask"]
---

<h1 style="text-align:center">
「Flask」01: Basic, Route, Templates, GET, POST
</h1>

> 前陣子用 Flask 實做了一個 LifePlaylist App 這裡簡單紀錄一下基本的東西，以後回頭更快重拾記憶🤧

![前陣子實做了 Flask App 紀錄一下基本處理!](/blog/images/04_05_210710/04_05_210710_flask_intro.png)

## **< Intro >**
"A lightweight Python Web Framework ! " ----> 
[Flask Docs](https://flask.palletsprojects.com/en/latest/quickstart/)
## **< Quick start to run a Flask App >**


### Install:
```
pip install Flask
```

### Create python file:
```
touch main.py
```

### Import:
```
from flask import Flask
```

### Create instance:
```
app = Flask(__name__)
```

### Route:
```
@app.route('/')
def index():
    return 'Hi Flask App ! '
```

### Setting only when you run python file directly, then execute `app.run()`:
```
if __name__ == '__main__':
    app.run()
```

### Now run Flask App:
```
python main.py
```
> And Flask Default Port:5000: `http://127.0.0.1:5000` will run on local.

> Hi Flask App ! 


## **< Handling Route >**

### Example:
```bash
@app.route('/')
def xxxxxx():
```
### Basic:
```
@app.route('/login')
def login():
    return 'This is login page.'
```

### Handling Multiple routes with same function:
```
@app.route('/index')
@app.route('/')
def home():
    return 'Welcome Home.'
```

### Routing parameters:
```
@app.route('/<number>')
def number():
    return f'Your route is /{number} .'
```

## **< Use Template(Jinja2) >**
### Import:
```
from flask import render_template
```
### Create xxx.html:
```
mkdir templates
touch templates/welcome.html
```
### Change return:
```
@app.route("/welcome")
def home():
    return render_template('welcome.html')
```

### Pass parameters：
```
@app.route("/welcome/<yourname>")
def home(yourname):
    return render_template('welcome.html', yourname=yourname)
```

### welcome.html:
> Use {{}}
- Basic:
```
{{ yourname }}
```
- List:
```
{{ list.0 }}
```
- Dict:
```
{{ Dict.key }}
```
- If statement:
```
{% if <xxxx> %}
    Hi, {{ xxxx }}!
{% else %}
    Oops...
{% endif %}
```
- For loop:
```
{% for item in <xxxx> %}
    <li>{{ item }}</li>
{% endfor %}
```
- Use base template:

```
## Create base.html
```
```
{% extends "base.html" %}
```
- Put own content(child templates):
```
e.g.: {% block content %}{% endblock %}
e.g.: {% block body %}{% endblock %}
e.g.: {% block custom %}{% endblock %}
```

## **< GET, POST >**
### Import:
```
from flask import request
```
### Setting route with GET method:

- Settting Route:
> Remember is "methods" not "method"

> `**locals()` can pass all args.
```
from flask import Flask, request, render_template

app = Flask(__name__)

@app.route('/',methods=['GET'])
def index():
    q = request.args.get('q')
    q2 = request.args.get('q2')
    return render_template('get.html', **locals())

if __name__ == '__main__':
    app.run()
```
> Remember is "templates" not "template"
- templates/get.html
```
{{ q }}
{{ q2 }}
```
- Run local and enter URL 
```Python
http://127.0.0.1:5000/?q=123&q2=456

## Should appear 123 456
```

### Setting route with POST method:

- Settting `/form` Route:
```
from flask import Flask, request, render_template

app = Flask(__name__)

@app.route("/form")
def form():
    return render_template('form.html')
    
if __name__ == '__main__':
    app.run()
```

- Create form.html
> Setting `action="{{url_for('submit')}}"` for POST route
```
<!DOCTYPE html>
<html lang="en">
<body>
    <form method="POST" action="{{url_for('submit')}}">
    <input type="text" name="name" placeholder="Name">
    <input type="text" name="email" placeholder="Email">
    <button type="submit">Submit</button>
</body>
</html>
```

- Handle `/submit` Route:
```
@app.route("/submit", methods=['POST'])
def submit():
    name = request.values['name']
    email = request.values['email']
    return render_template('submit.html',**locals())
```
- Create submit.html
```
<!DOCTYPE html>
<html lang="en">
<body>
    {{ name }}
    {{ email }}
</body>
</html>
```

- Run local and enter URL 
```Python
http://127.0.0.1:5000/form

## Should appear form, fill out name and email, then submit.
## Should go to `http://127.0.0.1:5000/submit`, and show name and email.
```

### Handle both GET and POST with one route:
```
from flask import Flask, request, render_template
from flask import redirect, url_for

@app.route("/form", methods=["POST","GET"])
def form():
    if request.method == "POST":
        name = request.form['name']
        email = request.form['email']
        return redirect(url_for('submit',**locals()))
    else:
        return render_template('form.html')

@app.route("/submit", methods=['POST'])
def submit():
    name = request.values['name']
    email = request.values['email']
    return render_template('submit.html',**locals())
```

## **< The End >**:

### Review:
- Remember Route Method settings is "methods" not "method".
- Remember folder name is "templates" not "template".
- Jinja2 syntax ?
- **locals() ?
- redirect & url_for ?


### Reference:
- [Jinja2](https://jinja.palletsprojects.com/en/3.0.x/templates/?highlight=enabled%20line)
- [Flask](https://flask.palletsprojects.com/en/latest/quickstart/)
- [Sean Yeh in Python Everywhere -from Beginner to Advanced](https://medium.com/seaniap/tagged/flask)



## **< Next 下一篇 >**:
<a href="/blog/posts/05/210714/「「Flask」02: Static File handling, Database connection">「Flask」02: Static File handling, Database connection</a>