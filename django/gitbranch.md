### A

git branch feature/__(A)

git switch feature/__ (A)

위에 두개 먼저하고 코드작성해야함 그리고나서

add -> commit 해야함



다했으면 git switch master

git merge feature/__(A)

그러면 내가 구현한 __ 가 master랑 합쳐지고

git push origin master  push를 함

### B

이렇게 하면 다른 작업자인 B 가 git pull origin master 로 땡겨옴 (A가 작업한 것)

B : 둘다 받아주는거 클릭

git switch master 에서 풀 받아오고

merge feature/__(B) 해줌

git push origin master

### A

git pull 해서 B 랑 싱크 맞춰줌

