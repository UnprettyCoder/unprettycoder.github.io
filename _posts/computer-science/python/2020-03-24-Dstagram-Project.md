---
layout: post
title: <Django Web Programming> Dstagram Project
date: 2020-03-24
excerpt: "Dstagram Project programmed by python django"
category: [Python]
comment: true
---

## Dstagram Project

### Infra-structure of the project

1. `photo_list`: 사진 목록, 각 사진별 작성자, 텍스트 설명, 댓글달기 버튼이 함께 출력
2. `photo_create_view`, `photo_update_view` : 사진을 추가 할 때는 사진과 텍스트 설명을 입력, 수정 할 때는 기존의 정보를 그대로 출력하고 수정
3. `detail_view`: 사진의 상세 정보를 확인, 수정/삭제, 댓글 기능을 이용가능
4. `photo_delete_view`: 사진을 삭제 가능, 삭제 확인 메시지 출력/확인하면 사진 삭제
5. `login_view`, `logout_view`: 로그인, 로그아웃 기능 제공
6. `register`: 회원가입을 위한 뷰, 회원 가입을 할 수 있도록 폼 출력 [ModelForm 사용]

### Create Project

`$ pip install django`

`$ django-admin startproject config .`

`$ python manag.py migrate`

`$ python manage.py createsuperuser`

### Create Photo App

`$ python manage.py startapp photo`

`.config/settings.py`

```python
INSTALLED_APPS = [
	'django.contrib.admin',
	'django.contrib.auth',
	# ... ,
	'photo'
];
```

### Create Model

`.photo/models.py`

```python
from django.db import models;

from django.contrib.auth.models import User;

class Photo(models.Model):
	author = models.ForeignKey(User, on_delete=models.CASCADE, related_name='user_photos');
	photo = models.ImageField(upload_to='photos/%Y/%m/%d', default='photos/no_image.png');
	text = models.TextField();
	created = models.DateTimeField(auto_now_add=True);
	updated = models.DateTimeField(auto_now=True);
```

`author`: ForeignKey를 사용하여 User 테이블과 관계 생성, `on_delete`는 연결된 모델이 삭제될 경우 현재 모델의 값을 어떻게 할 지 정하는 인자이다.

|  on_delete  |                       action                        |
| :---------: | :-------------------------------------------------: |
|   CASCADE   |    연결된 객체가 지워지면 하위 객체도 함께 삭제     |
|   PROTECT   | 하위 객체가 남아있으면 연결된 객체가 지워지지 않음  |
|  SET_NULL   |    연결된 객체만 삭제하고 필드 값을 NULL로 설정     |
| SET_DEFAULT | 연결된 객체만 삭제하고 필드 값을 DEFAULT값으로 변경 |
|    SET()    | 연결된 객체만 삭제하고 필드 값을 지정한 값으로 변경 |
| DO_NOTHING  |                 아무일도 하지 않음                  |

`photo`: 사진 필드, `upload_to`는 사진이 업로드 될 경로, 업로드가 진행되지 않을 경우 `default`값으로 대체됨

`text`: 사진에 대한 설명을 저장한 텍스트 필드

`created`: 글 작성 일자를 저장하기 위한 날짜시간 필드, `auto_now_add`는 객체 추가 시 자동으로 값을 설정할 지 여부

`updated`: 글 수정 일자를 저장하기 위한 날짜시간 필드, `auto_now`는 객체 수정 시 자동으로 값을 설정할 지 여부

`.photo/models.py`

```python
class Photo(models.Model):
	# ...
	class Meta:
		ordering = ['-updated'];
```

`Meta`의 `ordering`은 정렬 기준이다. ['-updated']로 설정했으므로 수정 일자 기준 내림차순 정렬된다.

`.photo/models.py`

```python
class Photo(models.Model):
	# ...
	def __str__(self):
		return self.author.username + " " + self.created.strftime("%Y-%m-%d %H:%M:%S");
    	# __str__()에는 작성자 + 작성일자의 문자열 반환
```

`.photo/models.py`

```python
from django.urls import reverse;

class Photo(models.Model):
	# ...
	def get_absolute_url(self):
		return reverse('photo:photo_detail', args=[str(self.id)]);
    	# 객체의 상세 페이지 주소를 반환
```

`$ python manage.py makemigrations photo`

`makemigrations` 명령을 이용해 모델의 변경사항을 기록

`$ pip install pillow`

`ImageField`를 이용하기 위해 `pillow` 패키지를 설치

`$ python manage.py migrate photo 0001`

`migrate` 명령을 이용해 데이터베이스에 적용

### Enroll Model in Admin Site

`photo/admin.py`

```python
from django.contrib import admin;

from .models import Photo;

admin.site.register(Photo);
```

`python manage.py runserver`

### Upload Directory Management

`config/settings.py`

```python
MEDIA_URL = '/media/';
# MEDIA_URL : 파일을 브라우저로 서빙할 때 보여줄 가상 URL [Security]

MEDIA_ROOT = os.path.join(BASE_DIR, 'media');
# 이제 어떤한 앱에서 업로드하더라도 프로젝트 루트 및에 'media'라는 디렉터리에 업로드된다. [각 앱별로]
# EX) "Dstagram/media/photos/YEAR/MONTH/DAY/example.png"
```

### Customizing Admin Page

`photo/admin.py`

```python
class PhotoAdmin(admin.ModelAdmin):
	list_display = ['id', 'author', 'created', 'updated'];
	raw_id_fields = ['author'];
	list_filter = ['created', 'updated', 'author'];
	search_fields = ['text', 'created'];
	ordering = ['-updated', '-created'];
	
admin.site.register(Photo, PhotoAdmin);
```

`list_display`: 관리자 페이지 목록에 보일 필드를 설정

`list_filter`: 필터 기능을 사용할 필드를 선택

`search_fields`: 검색 기능을 통해 검색할 필드를 선택

`ordering`: 모델의 기본 정렬값이 아닌 관리자 사이트 기본 정렬값을 설정

### Create View 

`photo/views.py` <functional view>

```python
from django.shortcuts import render;

from .models import Photo;

def photo_list(request):
	photos = Photo.objects.all();
	return render(request, 'photo/list.html', {'photos': photos});
```

`photo/views.py` <classical view>

```python
from django.views.generic.edit import CreateView, DeleteView, UpdateView;
from django.shortcuts import redirect;

class PhotoUploadView(CreateView):
	model = Photo;
	fields = ['photo', 'text'];
	template_name = 'photo/upload.html';
	
	def form_valid(self, form):
		form.instance.author_id = self.request.user.id;
		if form.is_valid():
			form.instance.save();
			return redirect('/');
		else:
			return self.render_to_response({'form': form});
        
class PhotoDeleteView(DeleteView):
    model = Photo;
    success_url = '/';
    template_name = 'photo/delete.html';
    
class PhotoUpdateView(UpdateView):
    model = Photo;
    fields = ['photo', 'text'];
    template_name = 'photo/update.html';
```

### Connect URL

`photo/urls.py`

```python
from django.urls import path;
from django.views.generic.detail import DetailView;

from .views import *;
from .models import Photo;

app_name = 'photo';

urlpatterns = [
	path('', photo_list, name='photo_list'),
	path('detail/<int:pk>/', DetailView.as_view(model=Photo, template_name='photo/detail.html'), name='photo_detail'),
	path('upload/', PhotoUploadView.as_view(), name='photo_upload'),
	path('delete/<int:pk>/', PhotoDeleteView.as_view(), name='photo_delete'),
	path('update/<int:pk>/', PhotoUpdateView.as_view(), name='photo_update')
];
```

`config/urls.py`

```python
from django.contrib import admin;
from django.urls import path, include;

urlpatterns = [
	path('admin/', admin.site.urls),
	path('', include('photo.urls'))
];
```

### Distribute & Expand Template

`templates/base.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
	
	<link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.4.1/css/bootstrap.min.css" integrity="sha384-Vkoo8x4CGsO3+Hhxv8T/Q5PaXtkKtu6ug5TOeNV6gBiFeWPGFN9MuhOf23Q9Ifjh" crossorigin="anonymous">
	<script src="https://code.jquery.com/jquery-3.4.1.slim.min.js" integrity="sha384-J6qa4849blE2+poT4WnyKhv5vZF5SrPo0iEjwBvKU7imGFAV0wwj1yYfoRSJoZ+n" crossorigin="anonymous"></script>
	<script src="https://cdn.jsdelivr.net/npm/popper.js@1.16.0/dist/umd/popper.min.js" integrity="sha384-Q6E9RHvbIyZFJoft+2mJbHaEWldlvI9IOYy5n3zV9zzTtmI3UksdQRVvoxMfooAo" crossorigin="anonymous"></script>
	<script src="https://stackpath.bootstrapcdn.com/bootstrap/4.4.1/js/bootstrap.min.js" integrity="sha384-wfSDF2E50Y2D1uUdj0O3uMBJnjuUD4Ih7YwaYd1iqfktj0Uod8GCExl3Og8ifwB6" crossorigin="anonymous"></script>
    <title>Dstagram {% block title %}{% endblock %}</title>
</head>
<body>

<div class="container">
	<header class="header clearfix">
		<nav class="navbar navbar-expand-lg navbar-light bg-light">
			<a class="navbar-brand" href="/">Dstagram</a>
			<ul class="nav">
				<li class="nav-item"><a href="/" class="active nav-link">Home</a></li>
				
				{% if user.is_authenticated %}
				<li class="nav-item"><a href="#" class="nav-link">Welcome, {{ user.get_username }}</a></li>
				<li class="nav-item"><a href="{% url 'photo:photo_upload' %}" class="nav-link"> Upload</a></li>
				<li class="nav-item"><a href="#" class="nav-link">Logout</a></li>
				{% else %}
				<li class="nav-item"><a href="#" class="nav-link">Login</a></li>
				<li class="nav-item"><a href="#" class="nav-link">Signup</a></li>
				{% endif %}
			</ul>
		</nav>
	</header>
	{% block content %}
	{% endblock %}
	
	<footer class="footer">
		<p>&copy; 2018 Baepeu. Powered By Django 2</p>
	</footer>
</div>

</body>
</html>
	
```

`config/settings.py`

```python
TEMPLATES = [
	{
		'BACKEND': 'django.template.backends.django.DjangoTemplates',
		'DIRS': [os.path.join(BASE_DIR, "templates")],
		'APP_DIRS': True,
		'OPTIONS': {
			'context_processors':[
				'django.template.context_processors.debug',
				'django.template.context_processors.request',
				'django.contrib.auth.context_processors.auth',
				'django.contrib.messages.context_processors.messages'
			]
		}
	}
];
```

`photo/templates/photo/list.html`

```html
{% extends 'base.html' %}

{% block title %}- List{% endblock %}

{% block content %}
	{% for post in photos %}
		<div class="row">
			<div class="col-md-2"></div>
			<div class="col-md-8 panel panel-default">
				<p><img src="{{post.photo.url}}" style="width:100%;"</p>
				<button type="button" class="btn btn-xs btn-info">
					{{post.author.username}}</button>
				<p>{{post.text|linkbreaksbr}}</p>
				<p class="text-right">
					<a href="{% url 'photo:photo_detail' pk=post.id %}" class="btn btn-xs btn-success">댓글달기</a>
				</p>
			</div>
			<div class="col-md-2"></div>
		</div>
	{% endfor %}
{% endblock %}
```

`$ python manage.py runserver`

`photo/templates/photo/upload.html`

```html
{% extends 'base.html' %}

{% block title %}- Upload{% endblock %}

{% block content %}
<div class="row">
	<div class="col-md-2"></div>
	<div class="col-md-8 panel panel-default">
		<form action="" method="post" enctype="multipart/form-data">
			{{ form.as_p }}
			{% csrf_token %}
			<input type="submit" class="btn btn-primary" value="Upload">
		</form>
	</div>
	<div class="col-md-2"></div>
</div>
{% endblock %}
```

`enctype`: form 태그로 작성한 정보를 어떤 형태로 인코딩해서 서버로 전달할 것인지 결정하는 옵션

| enctype                           | Discribtion                                                  |
| --------------------------------- | ------------------------------------------------------------ |
| application/x-www-form-urlencoded | 기본 옵션, 모든 문자열을 인코딩해 전달                       |
| multipart/form-data               | 파일 업로드 시 사용하는 옵션, 데이터를 문자열로 인코딩하지 않고 전달 |
| text/plain                        | 띄어쓰기만 +로 변환하고 특별한 인코딩 없이 전달              |

`photo/templates/photo/detail.html`

```html
{% extends 'base.html' %}
{% block title %}
	{{object.text|truncatechars:10}}
{% endblock %}

{% block content %}
	<div class="row">
		<div class="col-md-2"></div>
		<div class="col-md-8 panel panel-default">
			<p><img src="{{object.photo.url}}" style="width:100%;"></p>
			<button type="button" class="btn btn-outline-primary btn-sm">
				{{object.author.username}}</button>
			<p>{{object.text|linebreaksbr}}</p>
			
			<a href="{% url 'photo:photo_delete' pk=object.id %}" class="btn btn-outline-danger btn-sm float-right">Delete</a>
			<a href="{% url 'photo:photo_update' pk=object.id %}" class="btn btn-outline-success btn-sm float-right">Update</a>
		</div>
		<div class="col-md-2"></div>
	</div>
{% endblock %}
```

`photo/templates/photo/update.html`

```html
{% extends 'base.html' %}

{% block title %}- Update{% endblock %}

{% block content %}
<div class="row">
	<div class="col-md-2"></div>
	<div class="col-md-8 panel panel-default">
		<form action="" method="post" enctype="multipart/form-data">
			{{ form.as_p }}
			{% csrf_token %}
			<input type="submit" class="btn btn-primary" value="Update">
		</form>
	</div>
	<div class="col-md-2"></div>
</div>
{% endblock %}
```

`photo/templates/photo/delete.html`

```html
{% extends 'base.html' %}

{% block title %}- Delete{% endblock %}

{% block content %}
<div class="row">
	<div class="col-md-2"></div>
	<div class="col-md-8 panel panel-default">
		<div class="alert alert-info">
			Do you want to delete {{object}}?
		</div>
		<form action="" method="post">
			{{ form.as_p }}
			{% csrf_token %}
			<input type="submit" class="btn btn-danger" value="Confirm">
		</form>
	</div>
	<div class="col-md-2"></div>
</div>
{% endblock %}
```

`config/urls.py`

```python
from django.conf.urls.static import static;
from django.conf import settings;

urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT);
```

`static`을 사용하여 `MEDIA_URL`에 해당하는 주소를 가진 요청에 대해서 `MEDIA_ROOT`에서 찾도록 `urlpatterns`에 추가

### Create Account Application

`$ python manage.py startapp accounts`

`config/settings.py`

```python
INSTALLED_APPS = [
	'django.contrib.admin',
	# ... ,
	'photo',
	'accounts'
];
```

### Add Login/Logout Function

`accounts/urls.py`

```python
from django.urls import path;
from django.contrib.auth import views as auth_view;

urlpatterns = [
	path('login/', auth_view.LoginView.as_view(), name='login'),
	path('logout/', auth_view.LogoutView.as_view(template_name='registration/logout.html'), name='logout')
];
```

`config/urls.py`

```python
urlpatterns = [
	path('admin/', admin.site.urls),
	path('', include('photo.urls')),
	path('accounts/', include('accounts.urls'))
];
```

`accounts/templates/registration/login.html`

```html
{% extends 'base.html' %}
{% block title %}- Login{% endblock %}

{% block content %}
<div class="row">
	<div class="col-md-2"></div>
	<div class="col-md-8 panel panel-default">
		<div class="alert alert-info">Please enter your login informations.</div>
		<form action="" method="post">
			{{form.as_p}}
			{% csrf_token %}
			<input class="btn btn-primary" type="submit" value="Login">
		</form>
	</div>
	<div class="col-md-2"></div>
</div>
{% endblock %}
```

`accounts/templates/registration/logout.html`

```html
{% extends 'base.html' %}
{% block title %}- Logout{% endblock %}

{% block content %}
<div class="row">
	<div class="col-md-2"></div>
	<div class="col-md-8 panel panel-default">
		<div class="alert alert-info">You have been successfully loggedout.</div>
		<a class="btn btn-primary" href="{% url 'login' %}">Click to Login</a>
	</div>
	<div class="col-md-2"></div>
</div>
{% endblock %}
```

`templates/base.html`

```html
<li class="nav-item"><a href="{% url 'logout' %}" class="nav-link">Logout</a></li>

<li class="nav-item"><a href="{% url 'login' %}" class="nav-link">Login</a></li>
```

`base.html`에서 `login, logout`의 `href`속성을 수정한다.

로그인 후 이동할 페이지가 '/profile'로 기본값 설정되어있기 때문에 이 설정을 수정한다.

`config/settings.py`

```python
LOGIN_REDIRECT_URL = '/';
```

### Add SignUp Function

`accounts/forms.py`

```python
from django.contrib.auth.models import User;
from django import forms;

class RegisterForm(forms.ModelForm):
	password = forms.CharField(label='Password', widget=forms.PasswordInput);
	password2 = forms.CharField(label='Repeat Password', widget=forms.PasswordInput);
	
	class Meta:
		model = User;
		fields = ['username', 'first_name', 'last_name', 'email'];
		
	def clean_password2(self):
		cd = self.cleaned_data;
		if cd['password'] != cd['password2']:
			raise forms.ValidationError('Passwords not matched!');
		return cd['password2'];
```

`accounts/views.py`

```python
from django.shortcuts import render;
from .forms import RegisterForm;

def register(request):
	if request.method == 'POST':
		user_form = RegisterForm(request.POST);
		if user_form.is_valid():
			new_user = user_form.save(commit=False);
			new_user.set_password(user_form.cleaned_data['password']);
			new_user.save();
			return render(request, 'registration/register_done.html', {'new_user': new_user});
	else:
		user_form = RegisterForm();
	return render(request, 'registraion/register.html', {'form': user_form});
```

`accounts/urls.py`

```python
from .views import register;

urlpatterns = [
	# ... ,
	path('register/', register, name='register')
];
```

`accounts/templates/registration/register.html`

```html
{% extends 'base.html' %}

{% block title %}- Registration{% endblock %}

{% block content %}
<div class="row">
	<div class="col-md-2"></div>
	<div class="col-md-8 panel panel-default">
		<div class="alert alert-info">Please enter your account informations.</div>
		<form action="" method="post">
			{{form.as_p}}
			{% csrf_token %}
			<input class="btn btn-primary" type="submit" value="Register">
		</form>
	</div>
	<div class="col-md-2"></div>
</div>
{% endblock %}
```

`accounts/templates/registration/register_done.html`

```html
{% extends 'base.html' %}

{% block title %}- Registration Done{% endblock %}

{% block content %}
<div class="row">
	<div class="col-md-2"></div>
	<div class="col-md-8 panel panel-default">
		<div class="alert alert-info">Registration Success. Welcome, {{new_user.username}}</div>
		<a class="btn btn-info" href="/">Move to main</a>
	</div>
	<div class="col-md-2"></div>
</div>
{% endblock %}
```

`templates/base.html`

```html
<a href="{% url 'register' %}" class="nav-link">Signup</a>
```

### Comment Function Implementation with DISQUS

`DISQUS`: 댓글 시스템을 직접 만들지 않아도 댓글 시스템을 사용할 수 있도록 시스템을 빌려주는 사이트

`https://disqus.com/`

회원가입 후 온라인 소셜 댓글 시스템 사이트 생성 [웹 사이트 UI를 이용]

### Disqus Application Install

`$ pip install django-disqus`

`config/settings.py`

```python
INSTALLED_APPS = [
	'django.contrib.admin',
	# ... ,
	'photo',
	'accounts',
	'disqus',
	'django.contrib.sites'
];
```

`$ python manage.py migrate`

`config/settings.py`

```python
DISQUS_WEBSITE_SHORTNAME = 'dstagram-django';
# DISQUS_WEBSITE_SHORTNAME에는 DISQUS에서 사이트 생성할 때 입력한 이름을 기입한다.
SITE_ID = 1;
```

`photo/templates/photo/detail.html`

```html
<div class="row">
	<div class="col-md-2"></div>
	<div class="col-md-8 panel panel-default">
		{% load disqus_tags %}
		{% disqus_show_comments %}
	</div>
	<div class="col-md-2"></div>
</div>
```

### Control authority

`photo/view.py`

```python
from django.contrib.decorators import login_required;
from django.contrib.auth.mixins import LoginRequiredMixin;

@login_required
def photo_list(request):
# decorators는 함수형 뷰에 사용된다.

# Mixin은 클래스형 뷰에 사용된다.
class PhotoUploadView(LoginRequiredMixin, CreateView):

class PhotoDeleteView(LoginRequiredMixin, DeleteView):

class PhotoUpdateView(LoginRequiredMixin, UpdateView):
```

### AWS S3 Connect

### Using Heroku, Deploy App

이 2가지 부분에 대해서는 별도로 공부하도록 한다.
