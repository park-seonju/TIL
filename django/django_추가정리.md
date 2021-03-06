## django 추가정리

### 목적

- 어떤 웹페이지를 제공하는 프로그램을 만드는 것 (서버)

### 요약

- url, view(어떤 동작을 거쳐서), template(어떻게 보여줄 것인지)
- urls.py 
  - 어떤 url로 왔을 때 어떤 동작을 해야할지 정의
- views.py 
  - 어떤 동작을 해야하는지, 함수를 통해 코드 작성
- templates
  - 정보를 어떻게 표현할지를 dtl로 작성



### url

- 관리폴더의 urls.py 한페이지에 전부다 넣으면 관리 힘듬

  - 따라서 앱별로 urls.py를 만들어서 관리

  - ```django
    from django.urls import path, include
    ```

  - 를 통해 관리 가능

- name을 사용하는 이유

  - template에 쓰인 여러개의 url을 직접 입력하지 않기 위해
  - 유지보수를 위함



### templates

- 템플릿의 상속

  - 페이지마다 동일하게 들어가는 부분에 대해서

  - 일일이 템플릿 마다 적용하지 않고 한번에 적용하기 위해

  - 즉, 반복을 줄이고 유지보수하기 좋게  만들기 위함

  - ```django
    {% blcok content %}
    {% endblock %}
    ```

  - 관리폴더의 settings.py 의 TEMPLATES의 DIRS에 경로를 추가해줘야 사용 가능하다

  - 사용하려는 템플릿에서는 최상단에

  - ```django
    {% extends 파일명 %}
    ```



### ETL

- 템플릿에 작성하는 ETL의 경우엔 인덱스 접근이 `[]`가 아닌 파이썬의 클래스 사용처럼 `.`을 통해 접근



### Throw / catch

- 목적

  - 내가 만든 웹페이지에서 사용자의 정보를 받아서 처리해야 할때 필요함
  - html에서 서버로 뭔가 정보를 보낼 떄
    - form태그 사용
  - 예를들어서 로그인을 가능하게 만드려면 2가지가 필요
    - login을 할 수 있도록 하는 페이지를 보여주는 url / 로그인 페이지를 주는 로직
    - 로그인 요청을 받아 실제로 로그인을 시켜주는 url / 아이디와 비밀번호를 받아서 db에 존재하는지 확인하고 로그인을 처리하는 로직
  - 예를들어서 검색을 하려면 2가지가 필요
    - 검색어를 입력할 수 있게 마련해놓은 페이지를 보여주는 로직
    - 검색어를 받아서 내 db에서 검색을 해서 결과를 출력해주는 로직

  

- Throw

  - 사용자가 메세지를 입력할 수 있게끔 하는 페이지를 보여줌
  - url - view - templates 필요
  - form - submit -> action에 적힌 url로 요청을 보내는 것 ( catch로 )

- Catch

  - 적절한 처리를 해서 응답
  - url - view - templates 필요
  - `request.Get.get('변수명')`으로 메시지를 받을 수 있음



### 동적 라우팅

- 동적 url
- 목적
  - 비슷한 종류의 너무 많은 라우팅을 작성하는 것을 방지하기 위함
  - 즉, 코드의 중복을 방지하기 위함
- url의 값을 변수로 받아서 해당 값을 view로 넘겨줌

- 예시

  - ```django
    path('hello/<str:name>/', view.hello, name='hello')
    ```

- view에선 해당 변수가 들어갈 파라미터를 만들어줌

