---
layout: post
title: <Django Web Programming> Polls Project
date: 2020-03-24
excerpt: "Polls Project programmed by python django"
category: [Python]
comment: true
---

## Polls Project

### 1. Create Project

### 2. Install Django

`$ pip install django`

### 3. Django Project start

`$ django-admin startproject config .`

### 4. Test Web service

`$ python manage.py runserver`

### 5. Create App [polls]

`$ python manage.py startapp polls`

### 6. Create First View

`polls/views.py` 

```python
from django.http import httpResponse;

def index(request):
	return httpResponse("Hello, world. You're at the polls index.");
```

`polls/urls.py`

```python
from django.urls import path;
from . import views;

urlpatterns = [
	path('', views.index, name='index');
];
```

--------------------------

##### path(route, view, kwargs, name)

`route` : 주소를 의미

`view` : `route`주소로 접급했을 때 호출할 뷰

`kwargs` : 뷰에 전달할 인자들

`name` : `route`의 이름을 의미

--------------------

`config/urls.py`

```python
from django.contrib import admin;
from django.urls import path, include;

urlpatterns = [
	path('polls/', include('polls.urls/')),
	path('admin/', admin.site.urls)
];
```

---------------

##### include()

`include` : 최상위 `urls.py`가 다른 app에 포함된 `urls.py`를 참조하도록 하는 함수

-------------

`$ python manage.py runserver`

이후 서버를 실행시키고 첫 화면이 설정한대로 출력되는지 확인

### 7. Create Database

`config/settings.py`

`[ENGINE]` : 어떤 종류의 데이터베이스를 사용할지 결정 (`settings.py`파일 내부에 존재하는 설정)

`django.db.backends.mysql`

`django.db.backends.sqlite3`

`django.db.backends.postgresql`

`django.db.backends.oracle` 등을 기본적으로 사용할 수 있다.

--------------

`$ python manage.py migrate` (데이터베이스를 만들고 초기화)

### 8. Create Model

`polls/models.py`

```python
from django.db import models;

class Question(models.Model):
	question_text = models.CharField(max_length=200);
	pub_date = models.DateTimeField("date published");
class Choice(models.Model):
	question = models.ForeignKey(Question, on_delete=models.CASCADE);
	choice_text = models.CharField(max_length=200);
	votes = models.IntegerField(default=0);
```

`config/settings.py` ([INSTALLED_APPS]에 polls 앱 추가)

```python
INSTALLED_APPS = [
	'polls.apps.PollsConfig',   # 'polls'라고만 써도 무방
	'django.contrib.admin',
	'django.contrib.auth',
	...
];
```

`$ python manage.py makemigrations polls`

위 명령으로 앱의 변경사항을 추적해 데이터베이스에 적용할 내용을 만든다. 

[결과값은 `polls/migrations/0001_initial.py`에 기록된다.]

`$ python manage.py sqlmigrate polls 0001`

`$ python manage.py migrate polls 0001`

위 두 명령을 실행하면 데이터베이스에 테이블을 생성하고 초기화 할 수 있다.

### 9. Add function in Model

`polls/models.py`

```python
from django.db import models;
import datetime;
from django.utils import timezone;

class Question(models.Model):
	question_text = models.CharField(max_length=200);
	pub_date = models.DateTimeField("date published");
	def __str__(self):
		return self.question_text;
    def was_published_recently(self):
        return self.pub_date >= timezone.now() - datetime.timedelta(days=1);
class Choice(models.Model):
	question = models.ForeignKey(Question, on_delete=models.CASCADE);
	choice_text = models.CharField(max_length=200);
	votes = models.IntegerField(default=0);
	def __str__(self):
		return self.choice_text;
```

### 10. Check Administrator Page

`$ python manage.py createsuperuser`

`$ python manage.py runserver`

위 명령을 실행하고 `127.0.0.1:8000/admin`으로 접속하면 로그인 페이지를 확인할 수 있다.

`polls/admin.py`

```python
from django.contrib import admin;
from .models import Question;

admin.site.register(Question);
```

다음과 같이 파일을 설정하면 관리자 페이지에 `POLLS` 항목과 `Question`이 생성된 것을 확인할 수 있다.

(나머지는 관리자 페이지 UI를 통해 투표를 등록할 수 있다.)

### 11. Add other Views

`투표 목록` : 등록된 투표의 목록을 표시하고 상세 페이지로 이동하는 링크 제공

`투표 상세` : 투표의 상세 항목을 보여줌

`투표 기능` : 선택한 답변을 반영

`투표 결과` : 선택한 답변을 반영한 후 결과를 보여줌

------------------

`polls/views.py`

```python
def details(request, question_id):
	return HttpResponse("You're looking at question %s." %question_id);

def results(request, question_id):
    response = "You're looking at the results of question %s.";
    return HttpResponse(response %question_id);

def vote(request, question_id):
    return HttpResponse("You're voting on question %s." %question_id);

# 아직 특별한 기능없이 값만 출력하는 뷰
```

`polls/urls.py`

```python
from django.urls import path;
from . import views;

urlpartterns = [
	# ex : /polls/
	path('', views.index, name='index'),
	# ex : /polls/5/
	path('<int:question_id>/', views.detail, name='detail'),
	# ex : /polls/5/results/
	path('<int:question_id>/results/', views.results, name='results'),
	# ex : /polls/5/vote/
	path('<int:question_id>/vote/', views.vote, name='vote')
];
```

`polls/views.py`

```python
from .models import Question

def index(request):
	lastest_question_list = Question.objects.order_by('-pub_date')[:5];
	
    output = ', '.join([q.question_text for q in lastest_question_list]);
    return HttpResponse(output);

# At first, Modify index Page
```

여기까지 만들면 기능이 있는 뷰를 만들었지만 MTV 패턴에 따르지는 않는다. 템플릿을 만들어 python code와 HTML을 분리해야한다. 그를 위해 `polls`디렉터리 하위에 `templates`디렉터리를 생성한다.(HTML code가 들어갈 디렉터리)

--------------

`polls/templates/polls/index.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Title</title>
</head>
<body>
    {% if latest_question_list %}
    	<ul>
        	{% for question in latest_question_list %}
        		<li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>
        	{% endfor %}
    	</ul>
    {% else %}
    	<p>No polls are available.</p>
    {% endif %}
</body>
</html>
```

`polls/views.py`

```python
from django.http import HttpResponse;
from django.template import loader;

from .models import Question;

def index(request):
	latest_question_list = Question.objects.order_by('-pub_date')[:5];
	template = loader.get_template('polls/index.html');
	context = { 'latest_question_list': latest_question_list };
	return HttpResponse(template.render(context, request));
```

`polls/views.py` [render() 이용]

```python
from django.shortcuts import render;

from .models import Question;

def index(request):    
    latest_question_list = Question.objects.order_by('-pub_date')[:5];
    context = { 'latest_question_list': latest_question_list };    
    return render(request, 'polls/index.html', context);
```

### 12. Error Page

`polls/views.py`

```python
from django.http import Http404;

def detail(request, question_id):
	try:
        question = Question.objects.get(pk=question_id);
    except Question.DoesNotExist:
        raise Http404("Question does not exist.");
        # 상세 정보를 불러올 수 없는 경우 404 에러를 발생시킨다.
    return render(request, 'polls/detail.html', {'question': question});
```

`polls/templates/polls/detail.html`

```html
<body>
	{{ question }}
</body>
```

`polls/views.py` [django.shortcuts 이용]

```python
from django.shortcuts import render, get_object_or_404;

def detail(request, question_id):
	question = get_object_or_404(Question, pk=question_id);
    # get_object_or_404()에 의해 try-except문을 하나의 함수로 표현
	return render(request, 'polls/detail.html', {'question': question});
```

`polls/templates/polls/detail.html`

```html
<body>
	<h1>{{ question.question_text }}</h1>
    <ul>
    {% for choice in question.choice_set.all %}
        <li>{{ choice.choice_text }}</li>
    {% endfor %}
    </ul>
</body>
```

### 13. Delete Hard-coded URLs

`polls/templates/polls/index.html`

```html
<!-- Hard-coded url -->
<li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>

<!-- delete Hard-coding -->
<li><a href="{% url 'detail' question.id %}"> question.question_text </a></li>
```

### 14. Set URL Namespace

`polls/urls.py`

```python
app_name = 'polls';
```

`polls/templates/polls/index.html`

```html
<li><a href="{% url 'polls:detail' question.id %}">{{ question_text }}</a></li>
```

### 15. Create simple Form

`polls/templates/polls/detail.html`

```html
<h1>{{ question.question_text }}</h1>

{% if error_message %}
	<p><strong>{{ error_message }}</strong></p>
{% endif %}
<form action="{% url 'polls:vote' question.id %}" method="post">
{% csrf_token %}
{% for choice in question.choice_set.all %}
	<inpyt type="radio" name="choice" id="choice{{ forloop.counter }}" value="{{ choice.id }}">
	<label for="choice{{ forloop.counter }}">{{ choice.choice_text }}</label><br>
{% endfor %}
<input type="submit" value="Vote">
</form>
```

`polls/views.py`

```python
from django.http import HttpResponse, HttpResponseRedirect;
from django.urls import reverse;

from .models import Question, Choice;

def vote(request, question_id):
    question = get_object_or_404(Question, pk=question_id);
    try:
        selected_choice = question.choice_set.get(pk=request.POST['choice']);
    except (KeyError, Choice.DoesNotExist):
        # Redisplay the question voting form.
        return render(request, 'polls/detail.html',{
            'question': question,
            'error_message': "You didn't select a choice."
        });
    else:
        selected_choice.votes += 1;
        selected_choice.save();
        
    	return HttpResponseRedirect(reverse('polls:results', args=(question.id)));
```

`polls/views.py`

```python
def results(request, question_id):
	question = get_object_or_404(Question, pk=question_id);
	return render(request, 'polls/results.html', {'question': question});
```

`polls/templates/polls/results.html`

```python
<h1>{{ question.question_text }}</h1>

<ul>
{% for choice in question.choice_set.all %}
	<li>{{ choice.choice_text }} -- {{ choice.votes }} votes {{ choice.votes|pluralize }}</li>
{% endfor %}
</ul>

<a href="{% url 'polls:detail' question.id %}">Vote again</a>
```

### 16. Using Generic View

`Generic view` : Django에서 미리 준비되어 있는 뷰 [보편적인 틀]

`polls/views.py`

```python
from django.views import generic;

from .models import Question, Choice;

class IndexView(generic.ListView):
    template_name = 'polls/index.html';
    context_object_name = 'latest_question_list';
	
    def get_queryset(self):
        # Return the last five published questions.
        return Question.objects.order_by('-pub_date');

class DetailView(generic.DetailView):
    model = Question;
    template_name = 'polls/detail.html';
    
class ResultsView(generic.DetailView):
    model = Question;
    template_name = 'polls/results.html';
```

이전에 작업했던 뷰를 `함수형 뷰[functional view]`라고 하고, 지금 작성한 뷰가 `클래스형 뷰[classical view]`이다.

`polls/urls.py`

```python
urlpatterns = [
	path('', views.IndexView.as_view(), name='index'),
	path('<int:pk>/', views.DetailView.as_view(), name='detail'),
	path('<int:pk/results/', views.ResultsView.as_view(), name='results'),
	path('<int:question_id>/vote/', views.vote, name='vote')
];
```

### 17. Using Static Files

`Static File` : `css`, `js`와 같은 파일을 의미, static 디렉터리를 만들고 그 안에 파일을 저장한 후 사용 가능

`polls/static/polls/style.css`

```css
body {
	background: white url("images/background.png") no-repeat;
	background-position: right bottom;
}

li a {
	color: green;
}
```

`polls/templates/polls/index.html`

```html
<head>
	...
	{% load static %}
	<link rel="stylesheet" type="text/css" href="{% static 'polls/style.css' %}"
</head>
```

### 18. Customizing Admin Form

`polls/admin.py`

```python
class QuestionAdmin(admin.ModelAdmin):
	fieldsets = [
		(None,					{'fields': ['question_text']}),
		('Date information',	 {'fields': ['pub_date']})
	];
	
admin.site.register(Question, QuestionAdmin);
```

`polls/admin.py` <StackedInline>

```python
from .models import Question, Choice;

class ChoiceInline(admin.StackedInline):
	model = Choice;
	extra = 3;
	
class QuestionAdmin(admin.ModelAdmin):
	fieldsets = [
		(None,					{'fields': ['question_text']}),
		('Date information',	 {'fields': ['pub_date']})
	];
	
	inlines = [ChoiceInline];
```

`polls/admin.py` <TabularInline>

```python
class ChoiceInline(admin.TabularInline):
	model = Choice;
	extra = 3;
```

### 19. Admin Page List Customizing

`polls/admin.py`

```python
class QuestionAdmin(admin.ModelAdmin):
	fieldsets = [
		(None,					{'fields': ['question_text']}),
		('Date information',	 {'fields': ['pub_date']})
	];
	
	list_display = ('question_text', 'pub_date', 'was_published_recently');
	
	inlines = [ChoiceInline];
```

`polls/models.py`

```python
class Question(models.Model):
	# ...
    def was_published_recently(self):
        return self.pub_date >= timezone.now() - datetime.timedelta(days=1);
    
    was_published_recently.admin_order_field = 'pub_date';
    # admin_order_field -> 정렬 기준을 결정
    was_published_recently.boolean = True;
    # boolean -> 값 대신 아이콘 사용 여부
    was_published_recently.short_description = 'Published recently?'
    # short_description -> 항목 헤더 이름 설정
```

`polls/admin.py`

```python
class QuestionAdmin(admin.ModelAdmin):
	fieldsets = [
		(None, 				{'fields': ['question_text']}),
		('Date information', {'fields': ['pub_date']})
	];
	
	list_display = ('question_text', 'pub_date', 'was_published_recently');
	
	inlines = [ChoiceInline];
	
	list_filter = ['pub_date'];
    # list_filter -> 필터 기능
	search_fields = ['question_text'];
    # search_fields -> 검색 기능
```

