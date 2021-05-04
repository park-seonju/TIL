![image-20210504134128395](0504_ws.assets/image-20210504134128395.png)

-  제일 먼저 base.html에서 script 부분의 해당하는 block을 미리 생성해준다!

- 그 다음으로 좋아요가 표시되는 index 페이지의 base.html 에서 만든 script block을 만들어줘서 그 안에 코드들을 입력한다.

```javascript
// 좋아요 요청을 보내는 Form을 변경하는 것이므로 그 form 을 먼저 가져온다.
// 편히 가져오기 위해 class명을 like-form으로 주었다.
const likeForms = document.querySelectorAll('.like-form')
// 그냥 가져와서 작업을 하려하면 에러가 난다 이유는 csfr 토큰 때문!
// 고로 밑에처럼 input 태그에 name이 csrf 인 것을 가져와서
// axios header 에 같이 넣어줘야 한다.
const csrfToken = document.querySelector(
    'input[name="csrfmiddlewaretoken"]'
    // input태그에서 name이 csrf~ 인 것을 가져오겠다.
).value
```

```javascript
// Form들이 게시글 별로 있으므로 하나씩 가져와서 작업해야한다.
likeForms.forEach((likeForm)=>{
    // form 하나의=(likeForm) dataset/id 를 가져와서 어떤 게시글인지 알아야
    // URL 안의 articleId를 넘겨줘야 함
    const articleId = likeForm.dataset.id
    const URL = `/articles/${articleId}/likes/`
    // likeForm이 제출이 된다면
    likeForm.addEventListener('submit', (event)=>{
      // 기존 submit 방식을 막아주고 axios를 사용한다.
      event.preventDefault()
```

```javascript
// 밑 코드들 모두 addEventListener 안에 있는걸로 axios 부분이다.
// 선언한 URL 과 csrfToken 그리고 POST 방식으로 넣어준다.
axios({
        method:'POST',
        url:URL,
        headers:{
          'X-CSRFToken':csrfToken,
        },
      })
		// 응답이 잘 도착한다면
        .then((response)=>{
          // const likes = response.data.likes
          // const count = response.data.count
    	  // 응답 데이터 안의 views.py / likes function 에서 만들어준
          // likes와 count를 받아온다.
          const { likes, count } = response.data
          // 좋아요 버튼을 선택
          const likeButton = document.querySelector(`#like-btn-${articleId}`)
          // 몇명이 좋아요를 눌렀는지를 span 태그로 감싸서 선택
          const likeCount = document.querySelector(`#like-count-${articleId}`)
          // span 태그 안의 text로 몇명인지 count를 넣어준다.
          likeCount.innerText = count
    	 // 만약 likes가 참이면 좋아요를 누른 것 이므로
          if (likes) {
            // 버튼에 적힌 좋아요 -> 좋아요 취소
            likeButton.innerText = '좋아요 취소'
          }else {
            likeButton.innerText = '좋아요'
          }

        })
		// 에러가 났을 땐 콘솔에 에러를 띄워준다.
        .catch((error)=>{
          console.log(error)
        })
```

```python
# views.py
from django.http import JsonResponse
@require_POST
def likes(request, article_pk):
    # Json 형태로 보낼 dict 를 선언해준다.
    context={}
    if request.user.is_authenticated:
        article = get_object_or_404(Article, pk=article_pk)

        if article.like_users.filter(pk=request.user.pk).exists():
        # if request.user in article.like_users.all():
            # 좋아요 취소
            article.like_users.remove(request.user)
            # 좋아요를 취소할땐 False를 넣어주고
            context['likes']=False
        else:
            # 좋아요 누름
            article.like_users.add(request.user)
            # 좋아요를 누른다면 True를 넣어줌
            context['likes']=True
        # 좋아요를 누른 유저가 몇명인지 count에 넣어줌
        context['count']=article.like_users.count()
    #     return redirect('articles:index')
    # return redirect('accounts:login')
    # 이 유저가 현재 게시글을 좋아하는 지 정보
    # 좋아요 카운트
    # JSONResponse를 통해 반환
    return JsonResponse(context)
```

![image-20210504141524944](0504_ws.assets/image-20210504141524944.png)

- network 에는 likes 부분 만 주고 받는 것을 볼 수 있다.