## django templates 관리

- django는 app의 templates를 한곳에 모아서 관리함
- 따라서 app이 달라도 templates의 이름이 같으면 문제가 발생함
- 이름이 같으면 settings의 installed_app기준으로 상위에 등록되어있는 app의 url만 가져옴
- 예를들어
  - index.html(articles)
  - index.html(pages)
  - 이렇게 두개 있으면 상위에 하나만 호출댐

### 해결방법

- app의 templates 폴더 하위에

- app이름의 하위 폴더를 하나 만들어서

- 안에다가 templates를 넣어줌

- views의 함수에서 request를 보낼 url을
- 기존 `html이름` 에서
- `app이름/html이름`로 설정



## django url 관리

- url도 위와같은 문제가 발생함

- app의 urls.py에 app_name을 선언해서 해결 가능

- ```django
  from django.urls import path
  from . import views
  
  app_name = 'pages'
  
  urlpatterns = [
      path('index/', views.index, name='index'),
      path('dinner/', views.dinner, name='dinner')
  ]
  ```

- 그 후 templates의 html파일로 가 해당하는 url을 `app_name:url` 형식으로 바꿔줌

- ```django
  {% extends 'base.html' %}
  
  {% block content %}
    <h1> pages의 dinner </h1>
    <a href="{% url 'pages:index' %}">pages의 인덱스로 가기</a>
  {% endblock %}
  ```

- 

