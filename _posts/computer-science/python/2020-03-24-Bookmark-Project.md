---
layout: post
title: <Django Web Programming> Bookmark Project
date: 2020-03-24
excerpt: "Bookmark Project programmed by python django"
category: [Python]
comment: true
---

## BookMark Project

----------------------

이번 프로젝트에서 만들어 볼 것은 간단한 북마크 서비스이다.

목록 페이지, 북마크 추가, 북마크 상세, 북마크 수정, 북마크 삭제, 관리자 페이지를 생성/조작한다.

-------------------------

### Create Project

`$ pip install django`

`$ django-admin startproject config .`

`$ python manage.py migrate`

`$ python manage.py createsuperuser`

`$ python manage.py runserver`

Django 프로젝트 기본 생성/테스트 과정이다.

### Add BookMark application

`$ python manage.py startapp bookmark`

### Create Model

`bookmark/models.py`

```python
from django.db import models;

class Bookmark(models.Model):
	site_name = models.CharField(max_length=100);
	url = models.URLField('Site URL');
```

`config/settings.py`

```python
INSTALLED_APPS = [
	'django.contrib.admin',
	'django.contrib.auth',
	# ... ,
	'bookmark'
];
```

`$ python manage.py makemigrations bookmark`

`$ python manage.py migrate bookmark`

### Enroll Models in Admin Page

`bookmark/admin.py`

```python
from django.contrib import admin;

from .models import Bookmark;

admin.site.register(Bookmark);
```

`$ python manage.py runserver`

`127.0.0.1:8000/admin`에 접속하여 관리자 페이지에 모델이 정상적으로 등록되었는지 확인

(관리자 페이지에서 북마크를 추가할 수 있다. 그러나 원하는대로 출력되지는 않는다.)

### Add `__str__()` method in Models

`bookmark/models`

```python
from django.db import models;

class Bookmark(models.Model):
    # ...
    def __str__(self):
        # printed value when printing Object
        return "name : " + self.site_name + ", addr : " + self.url;
```

(이제 관리자 페이지에서 목록을 확인하면 `name : google, addr : google.com`과 같은 형식으로 출력된다.)

### Create ListView

`bookmark/views.py`

```python
from django.views.generic.list import ListView;

from .models import Bookmark;

class BookmarkListView(ListView):
	model = Bookmark;
```

Django에서 제공하는 Generic view를 이용하여 클래스형 뷰로 구성한다. [simple]

### Connect URL

`config/urls.py`

```python
from django.contrib import admin;
from django.urls import path, include;

urlpatterns = [
	path('bookmark/', include('bookmark.urls')),
	path('admin/', admin.site.urls)
];
```

`bookmark/urls.py`

```python
from django.urls import path;
from .views import BookmarkListView;

urlpatterns = [
	path('', BookmarkListView.as_view(), name='list')
];
```

`$ python manage.py runserver`

(아직 `localhost:8000/bookmark/`에 접속해도 템플릿 파일이 없다는 오류 메시지를 출력한다.)

### Make Templates [bookmark_list.html]

`bookmark/templates/bookmark/bookmark_list.html`

```html
{% raw %}
<!-- <html><head> ... -->
<body>
    <div class="btn-group">
        <a href="#" class="btn btn-info">Add Bookmark</a>
    </div>
    <p></p>
    <table class="table">
        <thread>
            <tr>
                <th scope="col">#</th>
                <th scope="col">Site</th>
                <th scope="col">URL</th>
                <th scope="col">Modify</th>
                <th scope="col">Delete</th>
            </tr>
        </thread>
        <tbody>
        	{% for bookmark in object_list %}
            	<tr>
            		<td>{{ forloop.counter }}</td>
                     <td><a href="#">{{ bookmark.site_name }}</a></td>
                     <td><a href="{{ bookmark.url }}" targer="_blank">{{ bookmark.url }}</a><</td>
                     <td><a href="#" class="btn btn-success btn-sm">Modify</a></td>
                     <td><a href="#" class="btn btn-danger btn-sm">Delete</a></td>
            	</tr>
            {% endfor %}
        </tbody>
    </table>
</body>
{% endraw %}
```

`$ python manage.py runserver`

`localhost:8000/bookmark/`에 접속하여 관리자 페이지에서 추가한 북마크가 정상 출력되는지 확인

### Implement Sub-Functions of Bookmark

`bookmark/views.py`

```python
from django.views.generic.edit import CreateView;
from django.urls import reverse_lazy;

class BookmarkCreateView(CreateView):
	model = Bookmark;
	fields = ['site_name', 'url'];
	success_url = reverse_lazy('list');
	template_name_suffix = '_create';
```

`bookmark/urls.py`

```python
from .views import BookmarkListView, BookmarkCreateView;

urlpatterns = [
	path('', BookmarkLustView.as_view(), name='list'),
	path('add/', BookmarkCreateView.as_view(), name='add')
];
```

`bookmark/templates/bookmark/bookmark_create.html`

```html
{% raw %}
<body>
	<form action="" method="post">
		{% csrf_token %}
        <!-- CSRF[Cross Site Request Forgery] 공격으로부터 서버를 지키기 위한 옵션 -->
		{{ form.as_p }}
		<input type="submit" value="Add" class="btn btn-info btn-sm">
	</form>
</body>
{% endraw %}
```

`$ python manage.py runserver`

`localhost:8000/bookmark/add/`에 접속하면 북마크를 추가할 수 있는 페이지가 출력된다.

`bookmark/templates/bookmark/bookmark_list.html`

```html
{% raw %}
<a href="{% url 'add' %}" class="btn btn-info">Add Bookmark</a>
{% endraw %}
```

`bookmark_list.html`파일의 `Add Bookmark` 링크가 작동할 수 있도록 `href`를 설정해준다.

### Create Bookmark Detail Page

`bookmark/views.py`

```python
from django.views.generic.detail import DetailView;

class BookmarkDetailView(DetailView):
	model = Bookmark;
```

`bookmark/urls.py`

```python
from django.urls import path;
from .views import *;

urlpatterns = [
	path('', BookmarkLustView.as_view(), name='list'),
	path('add/', BookmarkCreateView.as_view(), name='add'),
	path('detail/<int:pk>/', BookmarkDetailView.as_view(), name='detail')
];
```

`bookmark/templates/bookmark/bookmark_detail.html`

```html
{% raw %}
<body>
	{{ object.site_name }}<br>
	{{ object.url }}
</body>
{% endraw %}
```

`bookmark/templates/bookmark/bookmark_list.html`

```html
{% raw %}
<a href="{% url 'detail' pk=bookmark.id %}">{{ bookmark.site_name }}</a>
{% endraw %}
```

`bookmark_list.html`에 상세 정보를 보여줄 수 있도록 `href`를 수정한다.

### Bookmark Edit-Function Implementation

`bookmark/views.py`

```python
from django.views.generic.edit import CreateView, UpdateView;

class BookmarkUpdateView(UpdateView):
	model = Bookmark;
	fields = ['site_name', 'url'];
	template_name_suffix = '_update';
```

`bookmark/urls.py`

```python
urlpatterns = [
	# ...
	path('update/<int:pk>', BookmarkUpdateView.as_view(), name='update')
]
```

`bookmark/templates/bookmark/bookmark_update.html`

```html
{% raw %}
<body>
	<form action="" method="post">
	{% csrf_token %}
	{{ form.as_p }}
	<input type="submit" value="Update" class="btn btn-info btn-sm">
</body>	
{% endraw %}
```

`bookmark/templates/bookmark/bookmark_list.html`

```html
{% raw %}
<a href="{% url 'update' pk=bookmark.id %}" class="btn btn-success btn-sm">Modity</a>
{% endraw %}
```

`bookmark/models.py`

```python
from django.urls import reverse;

class Bookmark(models.Model):
	# ...
	def get_absolute_url(self):
		return reverse('detail', args=[str(self.id)]);
```

### Bookmark Delete-Function Implementation

`bookmark/views.py`

```python
from django.views.generic.edit import CreateView, UpdateView, DeleteView;

class BookmarkDeleteView(DeleteView):
	model = Bookmark;
	success_url = reverse_lazy('list');
```

`bookmark/urls.py`

```python
urlpatterns = [
	# ...
	path('delete/<int:pk>', BookmarkDeleteView.as_view(), name='delete')
];
```

`bookmark/templates/bookmark/bookmark_list.html`

```html
{% raw %}
<a href="{% url 'delete' pk=bookmark.id %}" class="btn btn-danger btn-sm">Delete</a>
{% endraw %}
```

`bookmark_list.html`의 Delete 부분 `href`를 수정하여 목록 제거 작업을 수행할 수 있도록 한다.

(`Delete`버튼을 누르면 삭제 페이지로 이동하지만 아직 오류 메시지 출력)

`bookmark/templates/bookmark/bookmark_confirm_delete.html`

```html
{% raw %}
<body>
	<form action="" method="post">
		{% csrf_token %}
		<div class="alert alert-danger">Do you wnate to delete Bookmark "{{ object }}"?</div>
		<input type="submit" value="Delete" class="btn btn-danger">
	</form>
</body>
{% endraw %}
```

삭제를 실수로 누를 때를 대비하여 재확인 페이지를 구현하여 확인한다.

------------------

## Set Design

### Expend Template

`config/setting.py`

```python
TEMPLATES = [
	{
		'BACKEND': 'django.template.backends.django.DjangoTemplates',
		'DIRS': [os.path.join(BASE_DIR, "templates")],
		'APP_DIRS': True,
		'OPTIONS': {
			'context_processors': [
				'django.template.context_processors.debug',
				'django.template.context_processors.request',
				'django.contrib.auth.context_processors.auth',
				'django.contrib.messages.context_processors.messages'
			]
		}
	}
];
```

`templates/base.html`

```html
{% raw %}
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>{% block title %}{% endblock %}</title>
</head>
<body>
	{% block content %}
	
	{% endblock %}
</body>
</html>
{% endraw %}
```

`block`: 기준 템플릿에 다른 템플릿에서 껴넣을 공간을 `block` 태크를 사용해 만들어두고 하위 템플릿에서는 이 블록에 껴넣을 내용을 결정하여 내용을 채운다.

`bookmark/templates/bookmark/bookmark_confirm_delete.html`

```html
{% raw %}
{% extends 'base.html' %}

{% block title %}Confirm Delete{% endblock %}

{% block content %}
<form action="" method="post">
	{% csrf_token %}
	<div class="alert alert-danger">Do you want to delete Bookmark "{{object}}"?</div>
	<input type="submit" value="Delete" class="btn btn-danger">
</form>

{% endblock %}
{% endraw %}
```

`extends`: 상속해 올 기준이 되는 템플릿을 지정

`bookmark/templates/bookmark/bookmark_create.html`

```html
{% raw %}
{% extends 'base.html' %}

{% block title %}Bookmark Add{% endblock %}

{% block content %}
<form action="" method="post">
	{% csrf_token %}
	{{ form.as_p }}
</form>
{% endblock %}
{% endraw %}
```

`bookmark/templates/bookmark/bookmark_detail.html`

```html
{% raw %}
{% extends 'base.html' %}

{% block title %}Detail{% endblock %}

{% block content %}
{{ object.site_name }}<br>
{{ object.url }}
{% endblock %}
{% endraw %}
```

`bookmark/templates/bookmark/bookmark_list.html`

```html
{% raw %}
{% extends 'base.html' %}

{% block title %}Bookmark List{% endblock %}

{% block content %}
<div class="btn-group">
	<a href="{% url 'add' %}" class="btn btn-info">Add Bookmark</a>
</div>
<p></p>
<table class="table">
        <thread>
            <tr>
                <th scope="col">#</th>
                <th scope="col">Site</th>
                <th scope="col">URL</th>
                <th scope="col">Modify</th>
                <th scope="col">Delete</th>
            </tr>
        </thread>
        <tbody>
        	{% for bookmark in object_list %}
            	<tr>
            		<td>{{ forloop.counter }}</td>
                     <td><a href="{% url 'detail' pk=bookmark.id %}">{{ bookmark.site_name }}</a></td>
                     <td><a href="{{ bookmark.url }}" targer="_blank">{{ bookmark.url }}</a><</td>
                     <td><a href="{% url 'update' pk=bookmark.id %}" class="btn btn-success btn-sm">Modity</a></td>
                     <td><a href="{% url 'delete' pk=bookmark.id %}" class="btn btn-danger btn-sm">Delete</a></td>
            	</tr>
            {% endfor %}
        </tbody>
</table>
{% endblock %}
{% endraw %}
```

`bookmark/templates/bookmark/bookmark_update.html`

```html
{% raw %}
{% extends 'base.html' %}

{% block title %}Bookmark Update{% endblock %}

{% block content %}
<form action="" method="post">
	{% csrf_token %}
	{{ form.as_p }}
	<input type="submit" value="Update" class="btn btn-info btn-sm">
</form>
{% endblock %}
{% endraw %}
```

지금까지 템플릿의 확장을 이용하지 않고 작성했던 `html`파일들을 모두 `base.html`에서 확장한 형태로 수정한다.

### Using Bootstrap

`https://getbootstrap.com/`(부트스트랩 공식 홈페이지)에 접속하여 부트스트랩을 이용하도록 한다.

`Bootstrap`: One of css Frameworks <design>

홈페이지 `Nav`의 `Documentation`에 들어가서 CSS, JS 참고 코드를 긁어서 `base.html`의 `<head>`에 추가한다.

`templates/base.html`

```html
{% raw %}
<head>
	<!-- skip other contents -->
	<link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.4.1/css/bootstrap.min.css" integrity="sha384-Vkoo8x4CGsO3+Hhxv8T/Q5PaXtkKtu6ug5TOeNV6gBiFeWPGFN9MuhOf23Q9Ifjh" crossorigin="anonymous">
	<script src="https://code.jquery.com/jquery-3.4.1.slim.min.js" integrity="sha384-J6qa4849blE2+poT4WnyKhv5vZF5SrPo0iEjwBvKU7imGFAV0wwj1yYfoRSJoZ+n" crossorigin="anonymous"></script>
	<script src="https://cdn.jsdelivr.net/npm/popper.js@1.16.0/dist/umd/popper.min.js" integrity="sha384-Q6E9RHvbIyZFJoft+2mJbHaEWldlvI9IOYy5n3zV9zzTtmI3UksdQRVvoxMfooAo" crossorigin="anonymous"></script>
	<script src="https://stackpath.bootstrapcdn.com/bootstrap/4.4.1/js/bootstrap.min.js" integrity="sha384-wfSDF2E50Y2D1uUdj0O3uMBJnjuUD4Ih7YwaYd1iqfktj0Uod8GCExl3Og8ifwB6" crossorigin="anonymous"></script>
</head>
{% endraw %}
```

이것을 적용한 것 만으로도 페이지의 디자인이 훨씬 더 아름다워진다. <class attr>

`bootstrap`은 HTML 태그의 `class` 속성을 기준으로 css를 적용한다. [class= "btn btn-info btn-sm" 등]

### Create Navigation Bar in `base.html`

`templates/base.html`

```html
{% raw %}
<!-- <body> 내부 -->
<div class="container">
	<nav class="navbar navbar-expand-lg navbar-light bg-light">
		<a class="navbar-brand" href="#">Django bookmark</a>
        <button class="navbar-toggler" type="button" data-toggle-"collapse" data-target="#navbarSupportedContent" aria-controls="navbarSupportedContent" aria-expanded="false" aria-label="Toggle navigation">
            <span class="navbar-toggle-icon"></span>
        </button>
        
        <div class="collapse navbar-collapse" id="navbarSupportedContent">
            <ul class="navbar-nav mr-auto">
                <li class="nav-item aciton">
                	<a class="nav-link" href="#">Home <span class="sr-only">(current)</span></a>
                </li>
            </ul>
        </div>
    </nav>
    <p></p>
    <div class="row">
        <div class="col">
            {% block content %}
            {% endblock %}
            
            {% block pagination %}
            {% endblock %}
        </div>
    </div>
</div>
{% endraw %}
```

### Add Paging-Function

`bookmark/views.py`

```python
class BookmarkListView(ListView):
	model = Bookmark;
	paginate_by = 6; # how many objects printed at one page [6]
```

`bookmark/templates/bookmark/bookmark_list.html`

```html
{% raw %}
{% block pagination %}
	{% if is_paginated %}
		<ul class="pagination justify-content-center pagination-sm">
		{% if page_obj.has_previous %}
			<li class="page-item">
				<a class="page-link" href="{% url 'list' %}? page={{ page_obj.previous_page_number }}" tabindex="-1">Previous</a>
			</li>
		{% else %}
			<li class="page-item disabled">
				<a class="page=link" href="#" tabindex="-1">Previous</a>
			</li>
		{% endif %}
		
		{% for object in page_obj.paginator.page_range %}
			<li class="page-item {% if page_obj.number == forloop.counter %}disabled{% endif %}">
				<a class="page-link" href="{{ request.path }}?page={{ forloop.counter }}">{{ forloop.counter }}</a>
			</li>
		{% endfor %}
		
		{% if page_obj.has_next %}
			<li class="page-item">
				<a class="page-link" href="{% url 'list' %}?page={{ page_obj.next_page_number }}"> Next</a>
			</li>
		{% else %}
			<li class="page-item disabled">
				<a class="page-link" href="#">Next</a>
			</li>
		{% endif %}
		</ul>
	{% endif %}
{% endblock %}
{% endraw %}
```

이제 북마크 목록 페이지에 1 페이지에 6 항목씩 출력되는 페이징 기능이 추가되었다.

### Using Static Files

`config/settings.py`

```python
STATICFILES_DIRS = [os.path.join(BASE_DIR, 'static')];
```

이후 프로젝트 하위 디렉터리로 `static` 디렉터리를 생성한다.

`static/style.css`

```css
body {
	width: 100%;
}
```

`templates/base.html`

```html
{% raw %}
{% load static %}
<link rel="stylesheet" href="{% static 'style.css' %}">
{% endraw %}
```

----------------

## Deploy Service

`https://github.com/`[깃허브]에서 배포하는 과정

깃허브를 통해 배포하기 위해 프로젝트 하위에 `.gitignore`파일을 생성하고, 다음과 같이 작성한다.

`.gitignore` (여기에 입력된 파일은 커밋하지 않는다. [와일드카드 문자 사용가능])

```txt
*.pyc
*~
/venv
__pycache__
db.sqlite3
.DS_Store
```

`.config/settings.py`

```python
DEBUG = False;

ALLOWED_HOSTS = ['*'];
```

`$ git init`

`$ git add -A`

`$ git commit -m "Bookmark Service"`

`$ git remote add origin https://github.com/unprettycoder/bookmark.git`

`$ git push -u origin master`

로그인 후 코드 업로드

`pythonanywhere`[파이썬 애니웨어]를 이용하여 배포할 수도 있다.

`Address` : `https://www.pythonanywhere.com/`

`PythonAnywhere`에 대해서는 차후 다시 공부하도록 한다.
