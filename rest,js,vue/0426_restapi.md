python manage.py seed articles --number=20

20개의 랜덤 데이터를 생성함 articles 안에!

rest_framework 랑은 상관없음 rest는 백엔드 장소를 만드는것이고

seed는 안에 재료들을 넣는다 생각하면 편할 듯

- settings.py

```python
INSTALLED_APPS = [
    'articles',
    'django_seed',
    'django_extensions',
    'rest_framework',
    ...
]
```

- views.py

```python
from rest_framework.response import Response
from rest_framework.decorators import api_view
from rest_framework import status

from django.shortcuts import render, get_list_or_404, get_object_or_404


from .models import Article,Comment
from .serializers import (
    ArticleSerializer,
    ArticleDetailSerializer,
    CommentSerializer,
)
# Create your views here.

# rest_framework 쓰려면 꼭 api_view를 사용해야함
@api_view(['GET','POST'])
def article_list(request):
    if request.method=='GET':
        # 1.article을 모두 가져와서
        # aritlces=Article.objects.all()
        articles=get_list_or_404(Article) #Article.objects.filter(title_contains='hello') 이런식으로 필터가능
        # 2.json화 할 수 있게 변환해서
        serializer = ArticleSerializer(articles,many=True) # many=True 원래 하나만 받아야하는데 지금은 쿼리셋이므로 many=True 필요
        # 3.응답해준다.
        return Response(serializer.data)
    elif request.method=='POST':
        # 사용자가 보낸 data를 serializer에 넣어서 검증!
        # 통과하면 저장
        serializer = ArticleDetailSerializer(data=request.data)
        if serializer.is_valid(raise_exception=True): # raise_exception 주면 bad request 없어도 알아서 에러발생해줌
            serializer.save()
            return Response(serializer.data,status=status.HTTP_201_CREATED) #의미(status)
        # return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

@api_view(['GET','PUT','DELETE'])
def article_detail(request,article_pk):
    article = get_object_or_404(Article, pk=article_pk)
    if request.method == 'GET':
        # article_pk에 맞는 모델인스턴스 가져와서
        # 변환하고,
        # 응답한다.
        # article=Article.objects.all(pk=article_pk)
        serializer = ArticleDetailSerializer(article)
        return Response(serializer.data)
    elif request.method == 'PUT':
        # 사용자의 데이터와, 기존 모델 인스턴스를 통해서
        # serializer를 만들고
        # 검증하고
        # 저장하고
        # 응답한다
        serializer = ArticleDetailSerializer(instance=article, data=request.data)
        if serializer.is_valid(raise_exception=True):
            serializer.save()
            return Response(serializer.data)
    elif request.method=='DELETE':
        # article을 삭제하고
        # 응답한다.
        article.delete()
        data={
            'delete':f'article {article_pk}이(가) 삭제 되었습니다.'
        }
        return Response(data,status=status.HTTP_204_NO_CONTENT)

@api_view(['GET'])
def comment_list(request):
    # 모든 comment를 불러와서 적절히 변환하여 응답
    # comments=Comment.objects.all() # 데이터 비면 그냥 빈 배열로 보여줌
    comments = get_list_or_404(Comment) # 데이터가 비었을때 404에러 나타나게하기
    serializer=CommentSerializer(comments, many=True)
    return Response(serializer.data)
    
@api_view(['GET','PUT','DELETE'])
def comment_detail(request, comment_pk):
    comment = get_object_or_404(Comment, pk=comment_pk)
    if request.method=='GET':
        serializer=CommentSerializer(comment)
        return Response(serializer.data)
    elif request.method == 'PUT':
        serializer=CommentSerializer(comment, request.data)
        if serializer.is_valid(raise_exception=True):
            serializer.save()
            return Response(serializer.data)
    elif request.method == 'DELETE':
        comment.delete()
        data={
            'delete':f'comment {comment_pk}이(가) 삭제 되었습니다.'
        }
        return Response(data,status=status.HTTP_204_NO_CONTENT)
        
@api_view(['POST'])
def comment_create(request, article_pk):
    # 첫 번째 방법!
    # url도 article_pk 가 필요 없음.
    # request.data에 article정보가 들어있음.
    # 시리얼라이저 만들고, 검증하고, 저장하고 응답!
    # serializer=CommentSerializer(data=request.data)
    # if serializer.is_valid(raise_exception=True):
    #     serializer.save()
    #     return Response(serializer.data)

    # 두 번째 방법!
    # url에 작성된 article_pk를 이용하는 방법!
    article = get_object_or_404(Article,pk=article_pk)
    serializer=CommentSerializer(data=request.data)
    if serializer.is_valid(raise_exception=True):
        serializer.save(article=article) # commit=False 가 아닌 아예 article을 지정함
        return Response(serializer.data)


```

- serializers.py

```python
from rest_framework import serializers
from .models import Article,Comment

class CommentSerializer(serializers.ModelSerializer):

    class Meta:
        model=Comment
        fields='__all__'
        read_only_fields = ('article',) #

class ArticleSerializer(serializers.ModelSerializer):
  
    class Meta:
        model=Article
        fields=('id','title','comment_set')
        # depth = 1 이 존재하면 한단계더 보여줌
        # ex) "article" : 1 => "article"=["title":~~ ]

class ArticleDetailSerializer(serializers.ModelSerializer):
    # valid 검사할때 오로지 읽기만을 위한 것이므로 지나가야함
    comment_set = CommentSerializer(many=True, read_only=True)
    comment_count = serializers.IntegerField(source='comment_set.count',read_only=True)

    class Meta:
        model=Article
        fields='__all__'
```

