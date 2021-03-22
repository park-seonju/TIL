### MVC pattern

- 모델 뷰 컨트롤러 (소프트웨어 디자인 패턴)

### DJANGO

- 모델 템플릿 뷰 (MTV 패턴임)
  - Model : DB관리
  - Template : 레이아웃
  - View : 중심 컨트롤러
- 한 줄 주석: {# ... #}
- 두 줄 주석: {% comment %} ... {% endcomment %}

### HTTP GET

- 서버로부터 __정보를 조회__하는 데 사용
- 데이터를 가져올 때만 사용해야함
- 우리가 서버에 요청을 하면 html 문서 파일 한 장을 받는데 이때 사용하는 요청 방식이 GET

### ORM

- object-relational-mapping : Django - SQL 데이터를 변환하는 기술
- DB를 객체로 조작하기 위해 ORM을 사용

## Django가 model에 생긴 변화를 반영하는 방법

- makemigrations
  - model을 변경한 것에 기반한 새로운 마이그레이션(설계도)을 만듬
- migrate
  - 설계도를 DB에 반영
  - 모델에서의 변경사항들과 DB의 스키마가 동기화
- sqlmigrate
  - 마이그레이션에 대한 sql 구문 보기 위해
- showmigrations
  - 프로젝트 전체 마이그레이션 상태 확인
  - migrate 됐는지 안됐는지 여부 확인

