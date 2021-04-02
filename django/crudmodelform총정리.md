## 시작

- 가상환경 설정 / 라이브러리 설치 / .gitignore추가 / freeze > requiremets.txt 저장
- 프로젝트와 app을 만들기 / settings , urls.py , base.html 만들기

### base.html

```html
{% load bootstrap5 %}
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  {% bootstrap_css %}
  {% block css %}{% endblock css %}
  <title>Document</title>
</head>
<body>
  {% include 'nav.html' %}
  <div class="container">
    {% block content %}
    {% endblock content %}
  </div>
  {% bootstrap_javascript %}
</body>
</html>
```

- css 는 따로 작성 안했지만 추가해 줬고 nav.html 을 연결 시켜줬다.

### nav.html

```html
{% load static %}

<nav class="navbar navbar-expand-lg navbar-light bg-light">
  <div class="container-fluid">

    <img src=" {% static 'megabox.png' %} " alt="logo" width="80" height="50">
    <a class="navbar-brand" href="{% url 'movies:index' %}">Navbar</a>
    <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarSupportedContent" aria-controls="navbarSupportedContent" aria-expanded="false" aria-label="Toggle navigation">
      <span class="navbar-toggler-icon"></span>
    </button>
    <div class="collapse navbar-collapse" id="navbarSupportedContent">
      <ul class="navbar-nav me-auto mb-2 mb-lg-0">
        <li class="nav-item">
          <a class="nav-link active" aria-current="page" href="{% url 'movies:index' %}">Home</a>
        </li>
    </div>
  </div>
</nav>
```

- static 이미지를 추가해주었기 load static을 상단에 써주었고 project 폴더 안에 static폴더를 생성하여 image파일을 넣어줘서 nav 왼쪽 에 위치하도록 하였다. 그렇기에 settings.py 안에 다음과 같은 코드를 추가했다.

  ```python
  STATIC_URL = '/static/'
  
  STATICFILES_DIRS=[
      BASE_DIR / 'pjt05' / 'static',
  ]
  ```

### Model.py

```python
from django.db import models

# Create your models here.
class Movie(models.Model):
    title = models.CharField(max_length=100)
    overview = models.TextField()
    image=models.ImageField(blank=True)
    poster_path = models.CharField(max_length=500)
```

- 모델에 Movie class를 선언해줬는데 어제 수업시간에 배운 image 업로드를 써보고 싶어서 ImageField를 추가해줬다.

- 모델을 만들어줬다면 `python manage.py makemigrations` 로 데이터를 구조화 해주고 
  `python manage.py migrate` 데이터 스키마 완성 후  `python manage.py loaddata movies.json` 으로 데이터를 db에 넣어준다. 

- image 파일을 첨부한 것을 폴더에 따로 저장하기 위해 BASE_DIR 에 media 파일을 만들어서 그 안에 첨부파일이 저장되도록 하였다. 따라서 setting.py에 다음과 같은 코드를 추가하였다.

  ```python
  MEDIA_ROOT = BASE_DIR / 'media'
  MEDIA_URL = '/media/'
  ```

### forms.py

```python
from django import forms
from .models import Movie

class MovieForm(forms.ModelForm):
    title = forms.CharField(
        label='제목',
        widget=forms.TextInput(
            attrs={
                'class': 'my-title form-control',
                'placeholder': 'Enter the Title',
                'maxlength': 100,
            }
        )
    )
    overview = forms.CharField(
        label='내용',
        widget=forms.Textarea(
            attrs={
                'class': 'my-overview form-control',
                'placeholder': 'Enter the Overview',
                'rows': 10,
                'cols': 100,
            }
        ),
        error_messages={
            'required': '데이터를 입력해줄래...?',
        }
    )
    poster_path = forms.CharField(
        label='포스터경로',
        widget=forms.TextInput(
            attrs={
                'class': 'my-poster_path form-control',
                'placeholder': 'Enter the poster_path',
                'maxlength': 500,
            }
        )
    )
    
    class Meta:
        model = Movie
        fields = '__all__'
```

- html에서 보여줄 title / overview / poster_path 를 설정
- 선언할 때 form을 기억해 두자!!

### admin.py

```python
from django.contrib import admin
from .models import Movie

# Register your models here.
admin.site.register(Movie)
```

- 관리자 페이지에서 데이터 조회 생성 등이 가능해야 하므로 `python manage.py createsuperuser` 로 id와 pw를 설정해야하고 위에 코드를 작성해야지만 admin 페이지를 들어갈 수 있다.

### urls.py

```python
from django.urls import path
from .  import views

app_name = 'movies'
urlpatterns=[
    path('', views.index, name='index'),
    path('create/', views.create, name='create'),
    path('<int:pk>/', views.detail, name='detail'),
    path('<int:pk>/update', views.update, name='update'),
    path('<int:pk>/delete', views.delete, name='delete'),
]
```

- views 와 django.urls path 를 import 해주고 app_name 도 app 이름으로 설정해준다.
- 경로는 명세서 대로 설정해줬다.

### views.py

- import 3가지

  - ```python
    from django.shortcuts import render,redirect
    from .models import Movie
    from .forms import MovieForm
    ```

  - redirect 추가 / models.py 에 Movie 클래스 추가 / forms.py MovieForm class 추가

- index

  - ```python
    def index(request):
        movies = Movie.objects.order_by('-pk')
        context={
            'movies' : movies,
        }
        return render(request, 'movies/index.html', context)
    ```

  - Movie 쿼리셋을 pk역순 으로 받아와 index.html에 넘겨준다. (1,2,3 으로 보여주기 위하여)

- create

  - ```python
    def create(request):
        if request.method=='POST':
            form=MovieForm(data=request.POST, files=request.FILES)
            if form.is_valid():
                movie = form.save()
                return redirect('movies:detail',movie.pk)
        else:
            form = MovieForm()
        context={
            'form' : form,
        }
        return render(request, 'movies/create.html',context)
    ```

  - POST 방식으로 들어왔을땐 MovieForm에 들어온 데이터와 첨부파일을 넣어준다.
    그 form이 잘 작성되어졌다면 저장하고 세부사항 page로 pk와 redirect 시킨다.

  - 그 외 방식으로 들어온다면 빈 form 을 create.html 파일로 보낸다.

- detail

  - ```python
    def detail(request,pk):
        movie = Movie.objects.get(pk=pk)
        context={
            'movie':movie,
        }
        return render(request, 'movies/detail.html', context)
    ```

  - 세부사항 페이지를 보여주는 함수로 요청받은 pk에 해당하는 쿼리셋을 가져오고 detail 페이지로 넘겨준다.

- update

  - ```python
    def update(request,pk):
        movie = Movie.objects.get(pk=pk)
        if request.method=='POST':
            form = MovieForm(data=request.POST,instance=movie,files=request.FILES)
            if form.is_valid():
                form.save()
                return redirect('movies:detail', movie.pk)
        else:
            form = MovieForm(instance=movie)
        context = {
            'form' : form,
            'movie' : movie,
        }    
        return render(request, 'movies/update.html', context)
    ```

  - 요청받은 pk 에 데이터를 movie에 넣어주고 POST 방식으로 들어왔을때 Form 형태로 데이터를 만들고 그 형태가 옳은 형태면 저장해주고 detail page로 redirect

  - 그 외 방식으로 들어온다면 form에 요청받은 pk데이터만 넣어주고 update.html 로 랜더링.

- delete

  - ```python
    def delete(request,pk):
        movie = Movie.objects.get(pk=pk)
        movie.delete()
        return redirect('movies:index')
    ```

  - 요청받은 pk 데이터 쿼리셋을 movie에 저장후 delete() 시키고 main페이지인 index 페이지로 redirect

# html

### index.html

```html
{% extends 'base.html' %}

{% block content %}
  <h1 class="text-center"><strong>MOVIE</strong></h1>
  <div class="m-3 d-flex justify-content-center">
  <a href="{% url 'movies:create' %}">CREATE</a>
  </div>
  <div class="container">
    <div class="row">
  {% for movie in movies %}
    <article style="width: auto;">
      <div class="card col" style="width: 18rem;">
        <img src="{{ movie.poster_path }}" class="card-img-top" alt="영화포스터">
        <div class="card-body">
          <a href="{% url 'movies:detail' movie.pk %}">
          <h5 class="card-title">{{ movie.title }}</h5>
          </a>
          <p class="card-text">{{ movie.overview|slice:":100" }}...</p>
        </div>
      </div>
    </article>
  {% endfor %}
    </div>
  </div>
{% endblock content %}
```

- html 은 지안님께서 지난 주에 했던 파일로 많이 도움을 주셨다 (Good Navigator)
- 내가 작성했던 html 과는 달리 div 로 잘 감싸지고 정리되어 있어서 보기 편리했다.
- slice:":100" 은 100자 까지만 나타내기 위한 코드

### create.html

```html
{% extends 'base.html' %}

{% block content %}
<h1 class="text-center"><strong>CREATE</strong></h1>

<form action="" method="POST" enctype="multipart/form-data">
  {% csrf_token %}
  {{ form.as_p }}
  <input type="submit">
</form>
<div class="m-3 d-flex justify-content-center">
{% if movie.image %}
  <img
    src="{{ movie.image.url }}"
    alt="{{ movie.image }}"
    width="200"
    heigh="200">
{% endif %}
</div>
{% endblock content %}
```

- model form 을 이용하였으니 `form.as_p` 를 사용했으며 `POST` 방식으로 입력하므로 `csfr_token` 을 추가.

- user가 첨부한 image를 보여줄 코드 추가 if 문으로 image 가 있다면 사진을 보여주고 첨부를 안했다면 안보여준다. 단❗ form 태그 안에 `enctype="multipart/form-data"`을 꼭 입력해줘야한다.

### detail.html

```html
{% extends 'base.html' %}
{% load static %}

{% block content %}
<h1 class="text-center"><strong>MOVIE DETAIL</strong></h1>
<hr>
<div class="d-flex justify-content-center">
  <div class="card" style="width: 30rem;">
    <img src="{{ movie.poster_path }}" class="card-img-top" alt="포스터">
    <div class="card-body">
      <h5 class="card-title">제목: {{ movie.title }}</h5>
      <p class="card-text">줄거리: {{ movie.overview }}</p>
      <div class="d-flex justify-content-center">
        <a href="{% url 'movies:update' movie.pk %}" class="m-1 btn btn-primary">수정하기</a>
        <form action="{% url 'movies:delete' movie.pk %}" method="POST">
          {% csrf_token %}  
          <input type='submit' class="m-1 btn btn-primary" value='삭제하기'></input>
        </form>
      </div>
    </div>
  </div>
</div>
<div class="m-3 d-flex justify-content-center">
{% if movie.image %}
  <img
    src="{{ movie.image.url }}"
    alt="{{ movie.image }}"
    width="200"
    heigh="200">
{% endif %}
</div>
<div class="d-flex justify-content-center">
  <a href="{% url 'movies:index' %}">메인페이지로 가기</a>
</div>
{% endblock content %}
```

- html 코드를 보다가 문득 든 생각인데 modelform을 이용하여 코드작성은 할 수 없나? 라는 생각이 들었다.

- 크게 다른건 없고 마찬가지로 image를 띄어줄 if 문만 추가되었다.

### update.html

```html
{% extends 'base.html' %}
{% load bootstrap5 %}

{% block content %}
<h1 class="text-center"><strong>MOVIE UPDATE</strong></h1>
<div class="container d-flex justify-content-center">
  <form action="{% url 'movies:update' movie.pk %}" method='POST' enctype="multipart/form-data">
    {% csrf_token %}
      {% bootstrap_form form %}
      {% buttons submit='OK' reset="Cancel" %}{% endbuttons %}
      <input class='mt-3' type="submit" value="수정하기">
  </form>
</div>
<div class="d-flex justify-content-center">
  <a href="{% url 'movies:index' %}">BACK</a>
</div>
<hr>
{% endblock content %}
```

- create 와 크게 다른건 없고 bootstrap_form 을 이용하여 OK 버튼과 reset 버튼을 생성하였다. reset 버튼은 안에 내용을 수정 전으로 돌려놓는 기능이다.

# 완성화면

- index

![image-20210319170912284](README.assets/image-20210319170912284.png)

- create

![image-20210319170940653](README.assets/image-20210319170940653.png)

- detail

![image-20210319171015128](README.assets/image-20210319171015128.png)

- update

![image-20210319171030465](README.assets/image-20210319171030465.png)

- image 첨부 후 detail

![image-20210319171100232](README.assets/image-20210319171100232.png)