### `python manage.py collectstatic`

static 파일들 모으는 코드 단, settings 밑에 `STATIC_ROOT=BASE_DIR / 'static' `  설정해야함

나중에 배포할때 필요함. 



blank : 빈문자열이 들어가는지 안들어가는지

null : 그 부분을 비워도 될지 안될지



### 이미지파일추가하기

1. imageField 추가
   1. db엔 이미지가 저장 x 경로 저장 O
2. pip install Pillow
3. makemigrations , migrate
4. create.html의 form에 enctype="multipart/form-data"속성추가
5. views.py의 create에서 ArticleForm 만들때 files=request.FILES 추가
6. settings.py
   1. MEDIA_ROOT = BASE_DIR ~
   2. MEDIA_URL = ~
7. urls.py에서 url 설정
8. detail.html에서 img태그 추가

