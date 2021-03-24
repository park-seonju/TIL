# Model Relationship 1



## Relationship fields

- 1 : N 관계
  - ForeignKey()
- M : N 관계
  - ManytoManyField()
- 1 : 1 관계
  - onetooneField()



# Many to one

- Foreign Key
  - 한 테이블의 필드 중 다른 테이블의 행을 식별할 수 있는 키
  - 참조하는 테이블에서 1개의 키
  - 참조 되는 테이블 측의 PK
  - 하나의 테이블이 여러개의 Foreign Key를 가질 수 있음
    - 각각 다른 테이블 참조 가능
  - 참조하는 테이블에 Foreign Key가 여러개 존재 가능
    - 이것은 참조 되는 테이블에서 하나의 PK
  - 참조하는 테이블과 참조되는 테이블이 동일할 수 있음(재귀적 외래 키)



## Foreign Key 특징

- 참조 무결성
  - 키를 사용해 부모 테이블의 유일한 값을 참조
  - 데이터의 정확성과 일관성을 유지하고 보증하는 것
- 외래키의  값이 꼭 부모 테이블의 PK일 필요는 없음, 다만 Unique한 값이어야 함 (후보키 중 하나)



## `ForeignKey()`

- 장고에서 1 : N 관계를 표현하기 위한 model Field
- 필수 인자 2개
  - 참조하는 모델 클래스
  - on_delete 옵션



## On_delete 옵션

- Foreign Key가 참조하는 개체가 사라졌을 때 Foreign Key를 가진 객체를 어떻게 처리할 지 정의
- __데이터 무결성__을 위해 필요
  - 데이터의 정확성과 일관성을 유지하고 보증하는 것

- __CASCADE : 부모 객체 삭제 시 이를 참조하는 객체도 삭제__
- PROTECT : 삭제 못하게 막음
- SET_NULL : 부모 객체 삭제 시 값을 NULL
- SET_DEFAULT : 모든 값이 DEFAULT값으로 치환
- SET() : 특정 함수 호출
- DO_NOTHING : 아무것도 안함
  - 단, 데이터베이스 필드에 대한 SQL ON DELETE 제한 조건 설정해야 함
- RESTRICT



## 코드

- 게시글에 댓글을 작성하는 경우

```python
#models.py

class Comment(models.Model):
    article = models.ForeignKey(Article, on_delete=models.CASCADE)
    content = models.CharField(max_length=200)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    
    #해당 코드가 있어야 댓글이 Comment object (1)가 아닌 정상적으로 출력
    def __str__(self):
        return self.content
```



- 댓글 저장 (in shell_plus)

```shell
#shell plus
#객체 생성
comment = Comment()

#객체에 내용 저장
comment.content = '댓글'

#참조하는 테이블의 데이터를 객체로 생성
article = Article.objects.get(pk=1)

#댓글 객체의 fk테이블에 pk를 가진 객체를 객체자체로 넣음
comment.article = article

#이런식으로 한번에 객체를 생성할 수도 있음
comment = Comment(content='댓글2', article=article)

comment.save() #저장

#근데 그냥 adminsite 사용하는게 편함...
```



- 어드민 사이트에 등록

```python
#admin.py

from .models import Comment

admin.site.register(Comment)
```



---



## 역참조

- Article(1) 이 Comment(N)을 참조(역참조)
  - __comment_set__
  - 장고에서는 역참조 시 모델이름 set 형식의 manager 생성



- 역참조 코드

```shell
#shell_plus
article = Article.objects.get(pk=1)

#역참조
article.comment_set.all()
#쿼리셋을 반환함

#따라서 이렇게 객체에 저장해 사용 가능
comments = article.comment_set.all()
```



## ForeignKey's Arguments - related_name

- 장고가 기본적으로 만들어 주는 _set manager(역참조 manager)를 변경할 이름 설정
- 1 : N 에선 거의 사용 안함
- M : N 관계에선 반드시 사용해야 하는 상황 발생

- 코드

```python
class Comment(models.Model):
    article = models.ForeignKey(Article, on_delete=models.CASCADE, related_name='comments')
```

- 단 이렇게 바꿀 시 기존에 사용하던 manager의 이름 (ex.`article.comment_set.all()`)은 더이상 사용불가능



---



## 댓글 create 코드

- form 생성

```python
#forms.py
from .models import Comment

class CommentForm(forms.ModelForm):

    class Meta:
        model = Comment
        fields = '__all__'
```



- 게시글 detail의 view에 댓글을 작성하는 기능 달아줌

```python
#views.py
from .forms import CommentForm

@require_safe
def detail(request, pk):
    article = get_object_or_404(Article, pk=pk)
    comment_form = CommentForm()
    context = {
        'article': article,
        'comment_form' : comment_form,
    }
    return render(request, 'articles/detail.html', context)
```



- detail.html에 나타내줌

```django
  <form action="" method="POST">
    {% csrf_token %}
    {{ comment_form }}
    <input type="submit">
  </form>
```



- 이렇게 하고 실제로 서버를 돌려서 확인해보면 문제가 있음
- 댓글의 내용만 받아야 하는데 어떤 글에다가 댓글을 달건지도 선택할 수 있게 돼있음
- forms.py를 수정해야함

```python
#forms.py
class CommentForm(forms.ModelForm):

    class Meta:
        model = Comment
        #fields = '__all__'
        exclude = ('article',)
```



- url 생성
  - content는 form에서 넘겨주나 foreign key는 넘겨주는 곳이 없음
  - 따라서 url로 foreign key를 넘겨줌

```python
#urls.py

path('<int:pk>/comments/', views.comments_create, name='comments_create'),
```



- detail.html 의 form 태그 수정

```django
<form action="{% url 'articles:comments_create' article.pk %}" method="POST">
```



- views.py에 댓글을 다는 기능 추가
  - POST요청만 처리하면 됌

```python
#views.py

@require_POST          
def comments_create(request, pk):
    article = get_object_or_404(Article, pk=pk)
    comment_form = CommentForm(request.POST)
    if comment_form.is_valid():
        #commit=False를 통해 인스턴스는 만들지만 데이터베이스에 입력은 하지 않고 대기
        #이게 없으면 NOT NULL constraint failed 발생
        comment = comment_form.save(commit=False)
        comment.article = article
        comment.save()
        return redirect('articles:detail', article.pk)
    context = {
        'comment_form' : comment_form,
        'article' : article,
    }
    return render(request, 'aritcles/detail.html', context)
```

- 만약 `comment_form.save()`로 객체를 만들고 바로 저장하면 NOT NULL 에러 발생
  - 왜냐하면 url로 넘겨준 외래키가 comment_form에 들어가지 않았기 때문
- 이때 `commit=False`로 객체를 만들되 데이터베이스에 저장하지 않고 대기시킴
  - 만들어진 객체를 변수에 저장
  - `comment = comment_form.save(commit=False)`
- 그 후 외래키 값을 넘겨줌 
  - `comment.article = article`
- 최종 저장



## 댓글 read 코드

- 글에 달린 모든 댓글 다 가져와야함
- 그러나 글은 댓글을 직접적으로 참조할수 없음
  - 참조할 키가 물리적으로 존재하지 않음
- 따라서 역참조 모델 매니저를 사용해야함

```python
#views.py

@require_safe
def detail(request, pk):
    article = get_object_or_404(Article, pk=pk)
    comment_form = CommentForm()
    comments = article.comment_set.all() #역참조 모델 매니저
    context = {
        'article': article,
        'comment_form' : comment_form,
        'comments' : comments,
    }
    return render(request, 'articles/detail.html', context)
```



- detail.html에 댓글을 보여줌

```django
  <ul>
    {% for comment in comments %}
      <li>{{ comment }}</li>
    {% endfor %}
  </ul>
```



## 댓글 delete 코드

- urls.py
  - article의 pk도 넘겨줌
  - 왜나햐면 댓글 삭제 후 detail로 redirect 할 때 아티클의 pk가 필요함

```python
#urls.py
path('<int:pk>/comments/<int:comment_pk>/delete/', views.comments_delete, name='comments_delete'),
```



- views.py
  - 삭제 후 article의 pk를 사용해서 redirect

```python
from .models import Comment

@require_POST
def comments_delete(request, pk, comment_pk):
    comment = get_object_or_404(Comment, pk=comment_pk)
    comment.delete()
    return redirect('articles:detail', pk)
```



- detail.html
  - url에 인자가 2개 필요함에 주의

```django
<ul>
  {% for comment in comments %}
    <li>{{ comment }}</li>
    <form action="{% url 'articles:comments_delete' article.pk comment.pk %}" method="POST">
      {% csrf_token %}
      <input type="submit" value="삭제">
    </form>
  {% endfor %}
</ul>
```



---



## 좀더 조건을 세세하게 줘보자

- 댓글을 로그인한 사용자만 쓰고 삭제할 수 있게끔

```python
# views.py

@require_POST          
def comments_create(request, pk):
    if request.user.is_authenticated: #로그인 조건
        article = get_object_or_404(Article, pk=pk)
        comment_form = CommentForm(request.POST)
        if comment_form.is_valid():
            comment = comment_form.save(commit=False)
            comment.article = article
            comment.save()
            return redirect('articles:detail', article.pk)
        context = {
            'comment_form' : comment_form,
            'article' : article,
        }
        return render(request, 'aritcles/detail.html', context)
    return redirect('accounts:login') #로그인 안되어 있으면 로그인 페이지로

@require_POST
def comments_delete(request, pk, comment_pk):
    if request.user.is_authenticated: #로그인 조건
        comment = get_object_or_404(Comment, pk=comment_pk)
        comment.delete()
        return redirect('articles:detail', pk)
    return redirect('accounts:login') #로그인 안되어 있으면 로그인 페이지로
```

- `@login_required`를 사용하지 않는 이유

  - 만약 비로그인 사용자가 시도할 경우
  - 로그인 페이지로 이동
  - 단, url에 next 파라미터와 함께 이동 (login-required의 특징)
  - 사용자 로그인 성공 하면
  - 다시 next파라미터에 담긴 url로 시도
  - 단, 이때의 method는 "GET" 임
  - 그러면 `require_POST`에 걸림
  - 405에러 발생 (method not allowed)
  - 따라서 "GET" method로 사용 가능한 함수만 사용 가능

  

## 상태코드 보내주기

`from django.http import HttpResponse`

- 위에코드에서 그냥 로그인 페이지로 보내는 것 보단
- 어떤 상태인지 사용자에게 알려주는것이 더 사용자 경험이 좋음

- [자세한 내용은 여기를 참조](https://docs.djangoproject.com/en/3.1/ref/request-response/)



## 댓글목록 아래쪽에 댓글의 갯수 출력

```django
#detail.html

{{ comments|length }} 개
{{ aritlce.comment_set.all|length }} 개
{{ comments.count }} 개
```

- 장고템플릿언어에서 함수 호출 시 `()` 붙이지 않음에 주의

- 셋 다 동일한 작업을 수행



- for-empty 구문을 사용하여 좀더 꾸며보자

```django
#detail.html

<ul>
  {% for comment in comments %}
    <li>{{ comment }}</li>
    <form action="{% url 'articles:comments_delete' article.pk comment.pk %}" method="POST">
      {% csrf_token %}
      <input type="submit" value="삭제">
    </form>
  {% empty %}
    아직 댓글이 없네요..
  {% endfor %}
</ul>
```

