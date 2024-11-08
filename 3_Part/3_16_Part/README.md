<!-- -
![](https://github.com/hyeonDD/jump_to_django/blob/main/3_Part/3_16_Part/django.png)
-->
# 도전! 저자 추천 파이보 추가 기능

아쉽지만 이 책에서 구현할 파이보의 기능은 여기까지다. 여러분과 함께 더 많은 기능을 추가하고 싶지만 이런 경험은 여러분 스스로 파이보를 직접 성장시키며 체험해 보길 바란다.
다만 이 책에서는 다루지 않았지만 구현하면 좋은 기능을 소개한다. 다음 기능을 스스로 구현하다 보면 장고를 더 깊이 이해할수 있을 것이다.

## 답변 페이징과 정렬
현재 하나의 질문에 무수히 많은 답변이 달릴 수 있는 구조이다. 만약 답변이 100개가 된다고 상상해 보자. 성능을 위해서라도 답변의 페이징은 반드시 필요할 것이다.

그리고 답변을 보여줄 때에도 최신순, 추천순 등으로 정렬하여 보여줄 수 있는 기능도 필요할 것이다. 유명한 질문 답변 사이트인 스택오버플로우(stackoverflow.com)나 레딧(reddit.com)을 보아도 항상 추천수가 많은 답변을 먼저 보여주고 있다.

여러분은 이미 질문 목록에 페이징과 정렬을 적용한 경험이 있기 때문에 답변에 페이징과 정렬을 적용하는 것이 그리 어렵지는 않을 것이다.

## 카테고리
현재는 '질문답변' 이라는 하나의 카테고리로만 게시판이 구성되지만 여기에 '강좌'나 '자유게시판'과 같은 게시판을 추가로 더 만들고 싶을 수도 있을 것이다. 이런 경우에 Question 모델에 카테고리를 추가하여 게시판을 분류할 수 있을 것이다.

## 비밀번호 찾기와 변경
현재 사용자가 비밀번호를 분실했을때 조치할수 있는 방법이 없다. 비밀번호 분실 시 임시비밀번호를 가입할 때 등록한 이메일 주소로 발송하여 로그인 할 수 있도록 조치하는 간단한 방법을 구현해 보자.

그리고 비밀번호 변경 프로그램도 필요하다. 로그인 후 기존 비밀번호와 새 비밀번호를 입력받아 비밀번호를 변경할 수 있는 프로그램을 만들어 보자.

## 프로필
로그인 한 사용자의 프로필 화면을 만들어 보자. 이 화면에는 사용자에 대한 기본정보와 작성한 질문, 답변, 댓글 등을 확인할 수 있도록 하면 좋을 것이다.

최근 답변과 최근 댓글
현재 파이보는 질문글 위주로 목록이 보여진다. 하지만 최근에 작성된 답변이나 최근에 작성된 댓글이 궁금할 수도 있을 것이다. 최근 답변과 최근 댓글을 확인할 수 있는 기능을 추가해 보자.

## 조회 수
현재 파이보는 답변 수와 추천 수를 표시하고 있지만 조회 수는 표시하지 않는다. 조회 수를 표시해 보자.

## 소셜 로그인
파이보에 구글이나 페이스북, 트위터 등을 경유하여 로그인하는 소셜 로그인 기능을 구현해 보자.

참고: django-allauth

## 마크다운 에디터
마크다운 문법을 더 쉽게 입력할 수 있는 마크다운 에디터를 적용해 보자. 인터넷을 찾아보면 추천하는 마크다운 에디터가 몇 가지 있는데, 필자는 그중에서 simpleMDE(simplemde.com)를 추천한다. simpleMDE를 파이보에 적용해 보자.