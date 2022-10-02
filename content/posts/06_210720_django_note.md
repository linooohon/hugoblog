---
title: "[Django] - part1.2: Official Docs Note"
subtitle: "「Django」part1.2: (EN)"
date: 2021-07-20T18:36:14+08:00
draft: false
url: "posts/06/210720/Django-Note"
summary: <p align="center">關於 Django 的官方文件筆記(EN)</p> <img src="/blog/images/06_210720/06_210720_django.png" width="30%"/>
author: "linooohon"
email: "linooohon@gmail.com"
toc: true
tags: ["Django", "Python", "Web"]
categories: ["2021", "python", "django"]
---

<h1 style="text-align:center">
「Django」part1.2: Official Docs Note(EN)
</h1>

> 關於 Django 的官方文件筆記(EN)，紀錄起來，讓自己可以快速統整 👐 

![關於 Django 的官方文件筆記(EN)](/blog/images/06_210720/06_210720_django.png)

## **< Intro >**
"A popular python web framework ! " ----> 
[Django Docs](https://docs.djangoproject.com/en/3.2/intro/tutorial01/)

## **< Quick Start >**
```bash
pip install Django
```
### 1. Create Django Project
```
django-admin startproject mysite
```
### 2. Run locally
> http://127.0.0.1:8000/
```
cd mysite
python manage.py runserver
```
- Changing the port
```bash
# http://127.0.0.1:8080/
python manage.py runserver 8080
```
```bash
# http://0.0.0.0:8000/
python manage.py runserver 0:8000
```

## **< Create App >**
### 1. Create App `polls`
> Under the same path with `manage.py`

```
python manage.py startapp polls
```

### 2. Write first View in App
> In the `polls/views.py`
```python
from django.shortcuts import render

# Create your views here.
from django.http import HttpResponse

def index(request):
    return HttpResponse("Hello, world. You're at the polls index.")
```

### 3. Create URLconf to configure urlpatterns in `polls`
- Under `polls/`
- Create `urls.py`
> In the `polls/urls.py`
```python
from django.urls import path

from . import views

urlpatterns = [
    path('', views.index, name='index'),
]
```

### 4. Point the root URLconf at `polls.urls` module ( Setting in `mysite/urls.py` )
> In the `mysite/urls.py`
```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('polls/', include('polls.urls')),
]
```

### 5. Run locally to see `polls/`
>  http://localhost:8000/polls/

```
python manage.py runserver
```


## **< DB Setup >**

### - Creates db table
```
python manage.py migrate
```
> This command looks at the `INSTALLED_APPS` setting and creates any necessary database tables according to the database settings in `mysite/settings.py` and the database migrations shipped with the app.


### - Creating models
> In the `polls/models.py`
```python
from django.db import models

# Create your models here.
class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')


class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)
```


### - Activating models
> We need to include app in project, so we need to reference to its configuration class in the 
`mysite/settings.py`
```python
INSTALLED_APPS = [
    'polls.apps.PollsConfig',
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]
```
But why is `polls.apps.PollsConfig`, it's because polls configuration file is in `polls/apps.py` and the Class is called `PollsConfig`, so we need to write `polls.apps.PollsConfig`.


### - Make migrations
```
python manage.py makemigrations polls
```
- Telling Django that you’ve made some changes to your models and that you’d like the changes to be stored as a migration.
```bash
# You will see somthing like this.
Migrations for 'polls':
  polls/migrations/0001_initial.py
    - Create model Question
    - Create model Choice
```
```bash
# And migrations folder
```

### - Get the Raw SQL for the migration you give.
```
python manage.py sqlmigrate polls 0001
```

### - Check for any problems in project
```
python manage.py check
```

### - Create model tables in daatbase:
> So run migrate again.
```
python manage.py migrate
```

## **< So, Basic Migrate Workflow is:>**

1. Change models in `models.py`
2. Activate models in INSTALLED_APPS
3. `python manage.py makemigrations` to create migrations for new changes.
4. `python manage.py migrate` to apply changes to the database.


## **< Django python shell >**

### - Run this command to enter:
```bash
python manage.py shell
```

### - Can use django framework to edit project, add some change in database here.
```python
>>> from polls.models import Choice, Question
>>> Question.objects.all()
# <QuerySet []>
>>> from django.utils import timezone
>>> q = Question(question_text="What's new?", pub_date=timezone.now())

>>> q.save()
>>> q.id
>>> q.question_text
# "What's new?"
>>> q.pub_date
# datetime.datetime(2012, 2, 26, 13, 0, 0, 775217, tzinfo=<UTC>)
>>> q.question_text = "What's up?"
>>> q.save()
>>> q.question_text
# "What's up?"
>>> Question.objects.all()
# <QuerySet [<Question: Question object (1)>]>
```
> Because usually we don't want `Question.objects.all()` to show `<QuerySet [<Question: Question object (1)>]>` for us, we want more clear infos for the model object we choose, so we can add `__str__()` in models.py class
### - Adding `__str__()` method to both Question and Choice in `polls/models.py`

```python
import datetime

from django.db import models
from django.utils import timezone


# Here we just return text
class Question(models.Model):
    # ...
    def __str__(self):
        return self.question_text
    
    def was_published_recently(self):
        return self.pub_date >= timezone.now() - datetime.timedelta(days=1)

class Choice(models.Model):
    # ...
    def __str__(self):
        return self.choice_text
```

### - Testing and playing around with shell again:
```bash
python manage.py shell
```


```python
>>> from polls.models import Choice, Question
>>> Question.objects.all()
# <QuerySet [<Question: What's up?>]>
# So now we can see the clear info for what we choose

>>> Question.objects.filter(id=1)
# <QuerySet [<Question: What's up?>]>


>>> Question.objects.filter(question_text__startswith='What')
# <QuerySet [<Question: What's up?>]>


>>> from django.utils import timezone
>>> current_year = timezone.now().year
>>> Question.objects.get(pub_date__year=current_year)
# <Question: What's up?>


>>> Question.objects.get(pk=1)
# pk is primary key
# <Question: What's up?>
>>> q = Question.objects.get(pk=1)
>>> q.was_published_recently()
# True


>>> q = Question.objects.get(pk=1)
>>> q.choice_set.all()
# <QuerySet []>


# Create three choices.
>>> q.choice_set.create(choice_text='Not much', votes=0)
# <Choice: Not much>
>>> q.choice_set.create(choice_text='The sky', votes=0)
# <Choice: The sky>
>>> c = q.choice_set.create(choice_text='Just hacking again', votes=0)

# Choice objects have API access to their related Question objects.
>>> c.question
# <Question: What's up?>
>>> q.choice_set.all()
# <QuerySet [<Choice: Not much>, <Choice: The sky>, <Choice: Just hacking again>]>
>>> q.choice_set.count()
# 3

# Use double underscores to separate relationships.
# Find all Choices for any question whose pub_date is in this year
>>> Choice.objects.filter(question__pub_date__year=current_year)
# <QuerySet [<Choice: Not much>, <Choice: The sky>, <Choice: Just hacking again>]>

# Let's delete one of the choices. Use delete() for that.
>>> c = q.choice_set.filter(choice_text__startswith='Just hacking')
>>> c.delete()

```


> So basically, we can use `python manage.py shell` to write some python code and also Django framework syntex.



## **< Login to Django admin >**

### - Creating an admin user
```
python manage.py createsuperuser
```
```
Username: admin
Email address: xxxxxx@xxxx
Password: **********
Password (again): *********
Superuser created successfully.
```

```
python manage.py runserver
```
> Go to http://127.0.0.1:8000/admin/

> And enter username and password.

![linooohon@gmail.com](/blog/images/06_210720/06_210720_django_adminlogin.png)


> Will see the Django administration

![linooohon@gmail.com](/blog/images/06_210720/06_210720_django_admininterface.png)

### - Register our own models in Django admin interface.

- In the `polls/admin.py`
```
from django.contrib import admin

# Register your models here.
from .models import Question, Choice
admin.site.register(Question)
admin.site.register(Choice)
```


## **< The End >**:

### Review:
- "Project" and "App".
- Remember Migration Workflow.
- Using django python shell.
- Create user for django admin interface and login.


### Reference:
- [Django Official Docs](https://docs.djangoproject.com/en/3.2/intro/tutorial01/)

## **< Next 下一篇 >**:
<!-- <a href="/blog/posts/01「Git 介紹」什麼是 git rebase">「Flask」02</a> -->