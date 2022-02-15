<!-- -
![](https://github.com/hyeonDD/jump_to_django/blob/main/2_Part/2_3_Part/django.png)
-->
# 개발 편의를 제공하는 장고 Admin

필자가 장고를 처음 접하고 가장 깜짝 놀란 기능이 바로 장고 Admin이다. 장고 ADmin은 한문장으로 표현하기 어려울 정도로 개발자에게 마법 같은 기능을 제공한다. 여기서는 장고 ADmin에 대해 알아보자.

## 장고 ADmin 사용하기

장고 ADmin을 사용하려면 슈퍼 유저를 먼저 생성해야 한다. 슈퍼 유저는 쉽게 말해 장고 운영자 계정이라 생각하면 된다.

1. 슈퍼 유저 생성하기
명령 프롬프트에서 python manage.py createsuperuser 명령을 실행하여 슈퍼 유저를 생성하자. 사용자 이름에는 admin을 입력하고(다른 것을 입력해도 된다), 이메일 주소는 가상의 이메일 주소를 적는다. 비밀번호는 여러분이 기억하기 쉬운 것으로 입력하자. 이때 단순한 구성의 비밀번호를 입력하면 경고 메시지가 나올 텐데, 이는 무시하는 옵션으로 'Bypass password validation and create user anyway?'의 질문에 y를 입력해 답하자.
```
(mysite) D:\code\vscode\github\jump_to_django\src\projects\mysite>python manage.py createsuperuser
Username (leave blank to use 'selwh'): admin
Email address: admin@mysite.com
Password: 
Password (again): 
The password is too similar to the username.
This password is too short. It must contain at least 8 characters.
This password is too common.
Bypass password validation and create user anyway? [y/N]: y
Superuser created successfully.
```
여기서는 다음과 같은 정보로 슈퍼 유저를 생성했다.
| 항목 | 값 |
| :--- | :--- |
| 사용자명 | admin |
| 이메일 주소 | admin@mysite.com |
| 비밀번호 | admin |

2. 장고 Admin에 접속해 로그인하기
1단계를 통해 슈퍼 유저가 생성되었으니 장고 개발 서버를 구동한 후 localhost:8000/admin에 접속해 보자. 그리고 앞에서 입력한 사용자명과 비밀번호를 입력해 로그인까지 진행하면 다음과 같은 화면이 나타난다.

![2-03_br1.png](https://github.com/hyeonDD/jump_to_django/blob/main/2_Part/2_3_Part/2-03_br1.png)

![2-03_br2.png](https://github.com/hyeonDD/jump_to_django/blob/main/2_Part/2_3_Part/2-03_br2.png)

장고 Admin에서는 현재 등록된 그룹 및 사용자에 대한 정보 확인과 수정을 할 수 있다. 물론 그룹은 아직 등록하지 않았으므로 클릭해서 조회해 보아도 아무것도 표시되지 안흔ㄴ다. 그러면 본격적으로 장고 Admin의 재미있는 기능을 알아보자.

3. 장고 Admin에서 모델 관리하기
우리는 Question, Answer 모델을 만들었다. 이 모델들을 장고 Admin에 등록하면 손쉽게 모델을 관리할 수 있다. 쉽게 말해 장고 셸로 수행했던 데이터 저장, 수정, 삭제 등의 작업을 장고 Admin에서 할 수 있다. 장고 Admin에서 어떤 마법이 벌어지는지 살펴보자. pybo/admin.py 파일을 열고 다음과 같이 코드를 입력하여 Question 모델을 장고 Admin에 등록하자.
```
from django.contrib import admin
from .models import Question

admin.site.register(Question)
```

4. 장고 Admin 새로고침 하기
장고 Admin으로 돌아가 새로고침 하면 다음처럼 Question 모델이 추가되었다.

![2-03_br3.png](https://github.com/hyeonDD/jump_to_django/blob/main/2_Part/2_3_Part/2-03_br3.png)

장고 Admin에 Question 모델을 등록했으니 이제 장고 셸이 아닌 장고 Admin 화면에서 Question 모델 데이터를 직관적으로 관리할 수 있다. Question 모델 데이터를 추가하고 수정하고 삭제하는 작업을 좀 더 쉽게 할 수 있게 되었다. 정말 그럴까?

5. Question 모델 데이터 추가하기
화면에서 Question 모델의 <+ 추가> 버튼을 누르자. 그러면 Question 모델의 데이터 등록 화면이 나타난다. 이어서 Question 모델의 속성에 맞는 값을 입력하고 <저장> 버튼을 누르자. 그러면 Question 모델 데이터가 추가된다.

![2-03_br4.png](https://github.com/hyeonDD/jump_to_django/blob/main/2_Part/2_3_Part/2-03_br4.png)

![2-03_br5.png](https://github.com/hyeonDD/jump_to_django/blob/main/2_Part/2_3_Part/2-03_br5.png)

6. 장고 Admin에 데이터 검색 기능 추가하기
장고 Admin에서 제목으로 질문을 검색할 수 있도록 검색 항목을 추가하자. pybo/admin.py 파일에 QuestionAdmin 클래스를 추가하고 search_fields에 'subject'를 추가하자.
```
from django.contrib import admin
from .models import Question


class QuestionAdmin(admin.ModelAdmin):
    search_fields = ['subject']


admin.site.register(Question, QuestionAdmin)
```

7. 장고 Admin에서 데이터 검색해 보기
장고 Admin으로 돌아가서 새로고침을 하면 검색 기능이 추가되었음을 알 수 있다. 검색어로 '장고'를 입력하고 <검색>을 눌러보자.
> 장고 ADmin의 기능이 궁금하다면 장고 공식 문서를 참고하자. 장고 공식 문서 주소 (장고 Admin 기능): https://docs.djangoproject.com/en/3.0/ref/contrib/admin

![2-03_br7.png](https://github.com/hyeonDD/jump_to_django/blob/main/2_Part/2_3_Part/2-03_br7.png)

그러면 제목에 '장고'가 포함된 Question 모델 데이터만 조회한다. 장고 ADmin에는 이런 마법 같은 기능이 무궁무진하다.