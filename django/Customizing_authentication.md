# Customizing authentication



## Substituting a custom User model (커스텀 유저 모델로 대체하기)

- 일부 프로젝트에선 `built-in User model`이 제공하는 인증 요구사항이 적절하지 않을 수 있음
- 장고는 `custom model`을 참조하는 `AUTH_USER_MODEL` 설정을 제공해서 기본 user model을 재정의(override) 할 수 있도록 함

- 새 프로젝트를 시작하는 경우 기본 사용자 모델이 충분하더라도 커스텀 유저 모델을 설정하는 것을 __강력하게 권장__
- 커스텀 유저 모델은 기본 사용자 모델과 동일하게 작동하면서
- 필요한 경우 나중에 맞춤 설정할 수 있기 때문
- __단, 프로젝트의 모든 `migrations` 또는 `migrate`실행 이전에 이 작업을 마쳐야 함__
  - 안되는건 아닌데.. 진짜 너무너무 힘듦 전부 수동으로 해야함

- [자세한 사항은 이곳을 참고](https://docs.djangoproject.com/en/3.1/topics/auth/customizing/)



## 코드

- `settings.py`에 해당 코드 입력

```python
#settings.py

AUTH_USER_MODEL = 'accounts.User'
```



- `accounts/models.py`에 모델 정의

```python
#models.py

from django.contrib.auth.models import AbstractUser

class User(AbstractUser):
    pass
```

- 왜 `AbstractBaseUser`가 아닌 `AbstractUser`를 상속받아 사용하는가?
  - `AbstractBaseUser`에 구현된 것이 너무 적어서
    - 기본적으로 password와 last_login만 제공
    - 자유도는 높지만 직접 구현해야 할것이 많음
- `AbstractUser`
  - 관리자 권한과 함께 완전한 기능을 갖춘 사용자 모델을 구현하는 기본 클래스
- Abstract class 들은 테이블을 만들지 않고 하위 클래스에 기능만 제공
  - meta class에 `Abstract = True`로 되어있음
  - 따라서 직접사용하지 않고 상속을 통해서만 사용



## AUTH_USER_MODEL

- User를 나타내는데 사용하는 모델
- 기본 값은 `'auth.User'`
- 주의
  - 프로젝트가 진행되는 동안 변경할 수 없음 (변경 하려면 많은 노력 필요)
  - 모든 마이그레이션 전에 실행되어야함



## 추가 설정해줘야 할 것

- 여기까지 진행하고 실행시켜보면
- `AttributeError` 발생
  - `Manager isn't available; 'auth.User' has been swapped for 'accounts.User'`

- `UserCreationForm`, `UserChangeForm`이 기본적으로 `auth.User`에 묶여있다.
- 따라서 이 두개의 form을 재정의 해야한다.



## form 재정의

- Custom Form 작성 시
- User를 직접참조 하지 말고
- get_user_model을 사용해야함
  - 그래야 커스텀 유저 모델 사용 시 자동으로 활성화된 유저 모델에서 데이터를 가져오기 때문

```python
# forms.py

from django.contrib.auth.forms import UserChangeForm, UserCreationForm
from django.contrib.auth import get_user_model


class CustomUserChangeForm(UserChangeForm):

    class Meta:
        model = get_user_model()
        fields = ('email', 'first_name', 'last_name',)

class CustomUserCreationForm(UserCreationForm):

    class Meta:
        model = get_user_model()
        fields = UserCreationForm.Meta.fields + ('email',) #이런식으로 추가하고 싶은 필드를 튜플로 추가가능
```

 

---



## User - Article 을 1 : N의 관계로 연결

- User는 작성할게 없음



```python
# articles/models.py

from django.db import models
from django.conf import settings #세팅즈를 임포트 해야함

# Create your models here.
class Article(models.Model):
    #models.py 에서 유저 모델을 참조하는 방식
    user = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
    title = models.CharField(max_length=10)
    content = models.TextField()
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
```



## 유저 모델 참조

- `settings.AUTH_USER_MODEL`

  - 유저 모델에 대한 외래 키 또는 M : N 관계를 정의 할 때 사용
  - __`models.py`에서 유저 모델을 참조할 때 사용__
  - 문자열을 리턴 `'accounts.User'`

  

- `get_user_model()`

  - 장고는 User모델을 직접 참조하는 대신 `get_user_model()` 을 사용하여 사용자 모델을 참조하라고 권장
  - 현재 활성화(activate)된 유저 모델을 반환
    - 활성화된 유저 모델 = 지정된 커스텀 유저 모델
    - 없응 경우 User
  - __`models.py`가 아닌 다른 모든 곳에서 유저 모델을 참조할 때 사용__
  - 유저 객체를 리턴



## ArticleForm 수정

- 이대로 실행하면 글 작성시 회원을 정하는 문제 발생
  - 댓글작성 만들 때 처럼

```python
# articles/forms.py

from django import forms
from .models import Article


class ArticleForm(forms.ModelForm):

    class Meta:
        model = Article
        fields = ('title', 'content')
```

