<!-- -
![](https://github.com/hyeonDD/jump_to_django/blob/main/2_Part/2_2_Part/django.png)
-->
# 데이터를 관리하는 모델
장고는 모델로 데이터를 관리한다. 모델로 데이터를 관리한다는 것은 무엇이며 어떤 이점이 있을까? 보통 웹 개발에서는 데이터의 저장,조회를 위해 SQL 쿼리문을 이용한다. 이 말은 데이터 저장,조회를 위해서는 별도의 SQL 쿼리문을 배워야 한다는 말과 같다. 학습 장벽이 하나 더 생기는 셈이다. 하지만 놀랍게도 모델을 사용하면 SQL 쿼리문을 몰라도 데이터를 저장,조회할 수 있다. 모델이 무엇이길래 이렇게 파격적으로 이야기하는지 실습을 통해 알아보자.

## migrate와 테이블 알아보기
1. 장고 개발 서버 구동 시 나오는 경고 메시지살펴보기
모델을 알아보기 위해 python manage.py runserver 명령 실행 시 나오는 경고 메시지를 조금 더 자세히 살펴보자. 중간쯤에 있는 장고 메시지를 보면 '아직 적용되지 않은 18의 migration이 있다'고 한다.
```
Performing system checks...

System check identified no issues (0 silenced).

You have 18 unapplied migration(s). Your project may not work properly until you apply the migrations for app(s): admin, auth, contenttypes, sessions.
Run 'python manage.py migrate' to apply them.
# ... 생략...
```
mirgration이 무엇인지 아직은 잘 모를 것이다. 하지만 적어도 이 경고 메시지가 admin, auth, contenttypes, sessions 앱과 관련된 내용이며, 이 오류를 해력라혀면 python manage.py migrate를 실행해야 한다는 아내는 확인할 수 있따.

## config/settgins.py 열어 기본으로 설치된 앱 확인하기
그러면 경고 메시지에 표시된 앱은 어디서 확인할 수 있고, 왜 경고 메시지에 언급되었을까/ 그 이유는 config/settings.py 파일을 열어 보면 어느 정도 짐작할 수 있다. 파일을 열어 **INSTALLED_APPS** 항목을 찾아보자.
```
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]
```
**INSTALLED_APPS**는 현재 장고 프로젝트에 설치된 앱이다. 경고 메시지에 언급되지 않은 messages, staticfiles 앱도 보일 것이다. 이 앱들은 데이터베이스와 상관 없으므로 경고 메시지에 언급되지 않은 것이다.

## config/settings.py에서 데이터베이스 정보 살펴보기
살펴보는 김에 config/settings.py 파일을 조금 더 살펴보자. config/settings.py 파일에는 설치된 앱뿐만 아니라 사용하는 데이터베이스에 대한 정보도 정의되어 있다. **DATABASES 설정**중 default의 'ENGINE' 항목을 보면 데이터베이스 엔진이 django.db.backends.sqlite3로 정의되어 있음을 알 수 있다. 그리고 'NAME' 항목을 보면 데이터베이스는 BASE_DIR에 있는 db.sqlite3이라는 파일에 저장되는 것도 알 수 있다.
> BASE_DIR은 프로젝트 디렉터리를 의미하며, 현재 우리의 BASE_DIR은 C;\projects\mysite이다.
> 데이터베이스를 여러 개 사용할 때 default에 지정한 데이터베이스 외에 추가로 등록해 사용할 수 있다.
```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db.sqlite3',
    }
}
```

---

**SQLLite는 파일 기반의 아주 작은 데이터베이스이다**

SQLite는 주로 소규모 프로젝트에서 사용되는 파일 기반의 가벼운 데이터베이스이다. 보통 초기 개발 단계에서 SQLite를 사용하여 빠르게 개발하고, 서비스로 제공할 때 운영 환경에 어울리는 데이터베ㅣㅇ스로 바꾼다.

## migrate 명령으로 앱이 필요로 하는 테이블 생성하기
migrate 명령을 실행하여 경고 메시지에 있던 앱들이 필요로 하는 테이블들을 생성하자. 명령 프롬프트에서 python manage.py mirgrate를 입력하면 다음과 같은 메시지가 출력된다.
```
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, sessions
Running migrations:
  Applying contenttypes.0001_initial... OK
  Applying auth.0001_initial... OK
  Applying admin.0001_initial... OK
  Applying admin.0002_logentry_remove_auto_add... OK
  Applying admin.0003_logentry_add_action_flag_choices... OK
  Applying contenttypes.0002_remove_content_type_name... OK
  Applying auth.0002_alter_permission_name_max_length... OK
  Applying auth.0003_alter_user_email_max_length... OK
  Applying auth.0004_alter_user_username_opts... OK
  Applying auth.0005_alter_user_last_login_null... OK
  Applying auth.0006_require_contenttypes_0002... OK
  Applying auth.0007_alter_validators_add_error_messages... OK
  Applying auth.0008_alter_user_username_max_length... OK
  Applying auth.0009_alter_user_last_name_max_length... OK
  Applying auth.0010_alter_group_name_max_length... OK
  Applying auth.0011_update_proxy_permissions... OK
  Applying auth.0012_alter_user_first_name_max_length... OK
  Applying sessions.0001_initial... OK
```
메시지에 대해 간단히 설명하자면 migrate를 통해 admin, auth, contenttypes, sessions 앱이 사용하는 테이블이 생성된다. 아직 여러분은 어떤 테이블이 생성되었는지 깊게 알 필요 없다. 왜냐하면 이 앱들을 사용하더라도 직ㅈ접 테이블을 건드릴 일은 거의 없기 때문이다.

## DB Browser for SQLite로 테이블 살펴보기
테이블의 정체가 궁금하면 migrate 명령을 통해 어떤 테이블이 만을어졌는지 잠깐 확인해보자. SQLite의 GUI 도구(그래픽 도구)인 'DB Browser for SQLite'를 설치하면 데이터베이스의 테이블을 확인할 수 있다. http://sqlitebrowser.org/dl 에 접속하여 'DB Browser for SQLite'를 내려받자. (64비트 standard installer를 내려받음)

여러분의 PC 환경에 맞는 설치 파일을 선택하여 내려받아 설치하자.

## DB Browser for SQLite 살펴보기
설치 후 실행하면 아래 그림과 같은 창이 나타난다. [데이터베이스 열기 -> D:\code\vscode\github\jump_to_django\src\projects\mysite\db.sqlite3 파일 선택]을 수행하면 앞에서 만든 테이블을 눈으로 확인할 수 있다.

![2-02_2.png](https://github.com/hyeonDD/jump_to_django/blob/main/2_Part/2_2_Part/2-02_2.png)

![2-02_3.png](https://github.com/hyeonDD/jump_to_django/blob/main/2_Part/2_2_Part/2-02_3.png)

화면에 뭔가 많이 나타나서 겁먹었을 수도 있겠지만, 여러분은 테이블의 내용을 자세히 볼 필요가 없다. 왜냐하면 장고는 테이블 작업을 위한 쿼리문을 알아서 수행하기 때문이다.

---

**장고의 ORM은 쿼리문을 몰라도 데이터 작업을 할 수 있따**
쿼리문이란 데이터베이스의 테이블을 생성, 수정, 삭제 또는 테이블 데이터의 내용을 생성, 수정 삭제 시 사용하는 데이터베이스 질의(문법)이다. 쉽게 말해 데이터 작업을 위한 문법이다. 데이터를 다루고 싶다면 데이터베이스 종류에 맞는 쿼리문을 공부해야 한다. 하지만 다행스럽게도 여러분은 쿼리문을 전혀 몰라도 된다. 왜냐하면 장고에는 ORM(object relational mpaaing)이라는 기능이 있기 때문이다. ORM은 파이썬으로 데이터 작업을 할 수 있게 해주는 기능이라고 생각하면 된다. 즉, 장고에서는 쿼리문을 몰라도 파이썬을 안다면 데이터를 다룰 수 있다.

---

---

**ORM은 데이터베이스 사용 방식의 단점 3가지를 제거한다**
전통적으로 데이터베이스를 사용하는 프로그램들은 데이터베이스의 데이터를 조회하거나 저장하기 위해 쿼리문을 사용해야 한다. 이 방식은 여전히 많이 사용되고 있는 방식이다. 하지만 이 전통적인 방식에는 몇 가지 단점이 있따. 첫 번째, 쿼리문은 같은 목적으로 작성해도 개발자마다 다양한 쿼리문이 만들어지므로 통일성이 깨진다. 두 번째, 개발자가 쿼리문을 잘못 작성하게 되면 시스템의 성능이 저하될 수도 있다. 세 번째, 데이터베이스를 변경하면(MySQL -> 오라클) 특징 데이터베이스에 의존하는 쿼리문을 모두 수정해야 하므로 유지 보수의 어려움이 생긴다.
하지만 장고의 ORM은 데이터베이스의 테이블을 모델화하여 사용하기 때문에 위에서 열거한 단점이 모두 사라진다. ORM이 쿼리문을 자동으로 생성하므로 통일성이 보장된다. 그리고 쿼리문 자체가 잘못 작성할 가능성도 사라진다. 또한 데이터베이스에 맞게 자동으로 쿼리문을 생성해 주므로 쿼리문 수정 작업이 사라진다. 바로 이것이 ORM을 사용하는 이유이다.

---

## 모델 만들기
이제 파이보에서 사용할 모델을 만들어 보자. 파이보는 질문 답변 게시판이므로 질문과 답변에 해당하는 모델이 있어야 한다.

1. 모델 속성 구상하기
질문과 답변 모델에는 어떤 속성이 있어야 할까? 질문 모델에는 다음 속성이 필요하다.
| 속성명 | 설명 |
| :--- | :--- |
| subject | 질문의 제목 |
| content | 질문의 내용 |
| create_date | 질문을 작성한 일시 |

답변 모델은 다음과 같은 속성이 필요하다.

| 속성명 | 설명 |
| :--- | :--- |
| question | 질문(어떤 질문의 답변인지 알아야 하므로 질문 속성이 필요함) |
| content | 답변의 내용 |
| create_date | 답변을 작성한 일시 |

이와 같이 설계한 모델을 구현해 보자.

2. pybo/models.py에 질문 모델 작성하기
질문 모델을 pybo/models.py에 작성해 보자. 질문 모델은 Question 클래스로 만든다. 앞으로 대부분의 모델은 클래스로 만들 것이며, 이후 책에서는 클래스명으로 모델을 언급하겠다. 이를테면 질문 모델은 Question 모델이라 하겠다.
```
from django.db import models

class Question(models.Model):
    subject = models.CharField(max_length=200)
    content = models.TextField()
    create_date = models.DateTimeField()
```
Question 모델에는 제목(subject), 내용(content), 작성일시(create_date)를 속성으로 추가했다. subject는 최대 200자까지 입력할 수 있도록 max_length=200을 매개변수로 전달하여 설정했다. 이처럼 글자 수를 제한하고 싶은 데이터는 **CharField**를 사용해야 한다. content는 글자 수 제한이 없는 데이터이다. 글자 수 제한이 없는 데이터는 TextField를 사용한다. create_date 같은 날짜,시간 관련 속성은 DateTimeField를 사용한다.

3. pybo/models.py에 답변 모델 작성하기
이어서 같은 파일에 Answer 모델도 작성하자.
```
from django.db import models

class Question(models.Model):
    subject = models.CharField(max_length=200)
    content = models.TextField()
    create_date = models.DateTimeField()

class Answer(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    content = models.TextField()
    create_date = models.DateTimeField()
```
Answer 모델은 어떤 질문에 대한 답변이므로 Question 모델을 속성으로 가져와야한다. 이처럼 어떤 모델이 다른 모델을 속성으로 가지면 **ForeignKey**를 이용한다. ForeignKey는 쉽게 말해 다른 모델과의 연결을 의미하며, on_delete=models.CASCADE는 답변에 연결된 질문이 삭제되면 답변도 함께 삭제하라는 의미이다.
> 장고에서 사용할 수 있는 속성은 아주 많다. 속성이 궁금하다면 장고 공식 문서에 접속하여 어떤 속성이 있는지 읽어 보자.
> 장고 속성 공식 문서 주소: https://docs.djangoproject.com/en/3.0/ref/models/fields/#field-types

4. config/settings.py를 열어 pybo 앱 등록하기
1~3단계에서 마든 모델을 이용하여 테이블을 생성하자. 테이블 생성을 하려면 config/settings.py 파일에서 **INSTALLED_APPS**항목에 pybo 앱을 추가해야 한다.
```
(... 생략 ...)
INSTALLED_APPS = [
    'pybo.apps.PyboConfig',
    'django.contrib.admin',
    'django.contrib.auth',
    (... 생략 ...)
]
(... 생략 ...)
```
**INSTALLED_APPS**에 추가한 pybo.apps.PyboConfig 클래스는 pybo/apps.py 파일에 있는 클래스이다. 잠깐 해당 파일을 열어 확인하고 넘어가자.

5. pybo/apps.py 열어 살펴보기
pybo/apps.py 파일을 열어보자. 이 파일은 pybo 앱을 만들 때 자동으로 생성된 것이므로 생성할 필요 없다. 그저 눈으로 확인하고 넘어가자.
```
from django.apps import AppConfig


class PyboConfig(AppConfig):
    default_auto_field = 'django.db.models.BigAutoField'
    name = 'pybo'
```
여기서 꼭 이해하고 넘어가야 할 점은 이 파일에 정의된 **PyboConfig** 클래스가 config/settings.py 파일의 **INSTALLED_APPS** 항목에 추가되지 않으면 장고는 pybo 앱을 인식하지 못하고 데이터베이스 관련 작업도 할 수 없다는 사실이다. 좀 더 자세히 설명하자면 장고는 모델을 이용하여 데이터베이스의 실체가 될 테이블을 만드는데, 모델은 앱에 종속되어 있으므로 반드시 장고에 앱을 등록해야 테이블 작업을 진행할 수 있따. pybo 앱 등록이 어떤 의미인지 알았으니 이제 모델로 실제 테이블을 만들어 보자.

6. migrate로 테이블 생성하기
테이블을 생성을 위해 migrate 명령을 실행하자.
```
(mysite) D:\code\vscode\github\jump_to_django\src\projects\mysite>python manage.py migrate
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, sessions
Running migrations:
  No migrations to apply.
  Your models have changes that are not yet reflected in a migration, and so won't be applied.
  Run 'manage.py makemigrations' to make new migrations, and then re-run 'manage.py migrate' to apply them. # <- manage.py makemigrations 명령을 실행 후 manae.py migrate 명령을 실행하세요
```

7. makemigrations로 테이블 작업 파일 생성하기
그런데 migrate 명령이 제대로 수행되지 않는다! 왜냐하면 모델이 생성되거나 변경된 경우 migrate 명령을 실행하려면 테이블 작업 파일이 필요하고, 테이블 작업 파일을 만들려면 makemigrations 명령을 실행해야 하기 때문이다. makemigrations 명령을 먼저 실행하자.
```
(mysite) D:\code\vscode\github\jump_to_django\src\projects\mysite>python manage.py makemigrations
Migrations for 'pybo':
  pybo\migrations\0001_initial.py
    - Create model Question      
    - Create model Answer   
```
makemigrations 명령은 장고가 테이블 작업을 수행하기 위한 파일을들을 생성한다. 많은 장고 학습자가 makemigrations 명령을 실제 테이블 생성 명령으로 오해한다. 하지만 실제 테이블 생성 명령은 migrate이다.

8. makemigrations로 생성된 파일 위치 살펴보기
makemigrations 명령을 수행하면 pybo/migrations/0001_initial.py이라는 파일이 자동으로 생성된다. 파일 위치를 확인해 보자.

![2-02_4.png](https://github.com/hyeonDD/jump_to_django/blob/main/2_Part/2_2_Part/2-02_4.png)

9. makemigrations 한 번 더 실행해 보기
makemigrations 명령을 한 번 더 실행해도 'No changes detected'라는 메시지만 뜬다. 모델의 변경사항이 없다면 '모델 변경 사항 없음'이라고 알려 주는 것이다.
```
(mysite) D:\code\vscode\github\jump_to_django\src\projects\mysite>python manage.py makemigrations
No changes detected
```

10. migrate 실행하기
이제 드디어 migrate 명령을 실행할 때가 되었다. 이 명령을 실행하면 장고는 등록된 앱에 있는 모델을 참조하여 실제 테이블을 생성한다.
```
(mysite) D:\code\vscode\github\jump_to_django\src\projects\mysite>python manage.py migrate       
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, pybo, sessions
Running migrations:
  Applying pybo.0001_initial... OK
```

11. DB Browser for SQLite로 생성된 테이블 확인하기
migrate 명령은 정말로 실제 테이블을 생성했을까? DB Browser for SQLite를 이용하여 테이블이 잘 생성되었는지 확인해 보자.

![migrate_chk.png](https://github.com/hyeonDD/jump_to_django/blob/main/2_Part/2_2_Part/migrate_chk.png)

실제 테이블명을 보면 Question 모델은 pybo_question으로, Answer 모델은 pybo_answer로 테이블 이름이 지어졌음을 확인할 수 있다. 혹시 이 실습 과정 때문에 '실제 테이블명을 알아야 개발을 진행할 수 있나?'라고 오해하는 독자가 없길 바란다. 실제로 장고 개발을 진행할 때는 실제 테이블명을 전혀 몰라도 된다. 그저 우리는 Question, Answer 모델을 사용하면 된다.

---

sqlmigrate를 실행하면 실제로 어떤 쿼리문이 실행되는지 볼 수 있다.

migrate 명령을 실행할 때 실제로 어떤 쿼리문이 실행될까? 혹시 궁금한 독자가 있다면 sqlmigrate 명령을 실행해 보자. 다음은 python manage.py sqlmigrate pybo 0001 명령을 실행한 결과이다.

```
(mysite) D:\code\vscode\github\jump_to_django\src\projects\mysite>python manage.py sqlmigrate pybo 0001
BEGIN;
--
-- Create model Question
--
CREATE TABLE "pybo_question" ("id" integer NOT NULL PRIMARY KEY AUTOINCREMENT, "subject" varchar(200) NOT NULL, "content" text NOT NULL, "create_date" datetime NOT NULL);
--
-- Create model Answer
--
CREATE TABLE "pybo_answer" ("id" integer NOT NULL PRIMARY KEY AUTOINCREMENT, "content" text NOT NULL, "create_date" datetime NOT NULL, "question_id" integer NOT NULL REFERENCES "pybo_question" ("id") DEFERRABLE INITIALLY DEFERRED);
CREATE INDEX "pybo_answer_question_id_e174c39f" ON "pybo_answer" ("question_id");
COMMIT;
```

명령어의 의미를 하나씩 설명하자면 'pybo'는 makemigrations 명령을 실행할 때 생성된 pybo/migrations/0001_initial.py의 마이그레이션명 'pybo'를 의미하고, '0001'은 생성된 파일의 일련번호를 의미한다. 실행 결과 메시지에는 SQL 문이 있다. 혹시 SQL 문에 익숙한 독자라면 이 메시지로 데이터베이스에서 어떤 일이 벌여졌는지 유추할 수 있을 것이다.

---

## 데이터 만들고 저장하고 조회하기

지금까지 모델을 이용하여 실제 테이블을 만들었다. 그러면 장고에서는 모델을 어떻게 사용할까? 장고 셸을 사용하면 모델 사용법을 쉽게 익힐 수 있다.

1. 장고 셸 실행하기
다음을 입력하여 장고 셸을 실행하자. 장고 셸을 실행해 보면 '파이썬 셸과 비슷한 것 같은데?'라는 생각이 들 수 있다. 하지만 장고 셸은 장고에 필요한 환경들이 자동으로 설정되어 실행되므로 파이썬 셸과는 약간의 차이가 있다.
```
(mysite) D:\code\vscode\github\jump_to_django\src\projects\mysite>python manage.py shell
Python 3.8.5 (tags/v3.8.5:580fbb0, Jul 20 2020, 15:57:54) [MSC v.1924 64 bit (AMD64)] on win32
Type "help", "copyright", "credits" or "license" for more information.
(InteractiveConsole)
```

2. Question, Answer 모델 임포트하기
장고 셸을 실행했으면 다음 명령으로 Question과 Answer 모델을 장고 셸에 임포트하자.
```
>>> from pybo.models import Question, Answer
```

3. Question 모델로 Question 모델 데이터 만들기
이어서 Question 모델을 이용하여 Question 모델 데이터를 하나만 만들어 보자. Question 모델의 subject 속성에 제목을, content 속성에 문장열로 질문 내용을 입력한다. create_date속성은 DateTimeField이므로 timezone.now()로 현재 일시를 입력한다.
```
>>> from pybo.models import Question, Answer
>>> from django.utils import timezone
>>> q = Question(subject='pybo가 무엇인가요?', content='pybo에 대해서 알고 싶습니다.', create_date=timezone.now())
>>> q.save()
```
이 과정을 통해 객체 q가 생성된다. 객체가 생성된 다음 q.save()를 입력하면 Question 모델 데이터 1건이 데이터베이스에 저장된다.

4. Question 모델 데이터의 id값 확인하기
Question 모델 데이터가 잘 생성되었는지 확인해 보자. 장고는 데이터 생성 시 데이터에 id값을 자동으로 넣어준다.
```
>>> q.id
1
```
id는 데이터의 유일한 값이며, 프라이머리 키(PK)라고 부르기도 한다. id값은 여러분이 데이터를 생성할 때마다 1씩 증가한 값으로 자동으로 입력될 것이다.

5. Question 모델로 Question 모델 데이터 1개 더 만들기
이를 확인하기 위해 두 번째 Question 모델 데이터를 만들어 저장해 보자.
```
>>> q = Question(subject='장고 모델 질문입니다.', content='id는 자동으로 생성되나요?', create_date=timezone.now())
>>> q.save()
>>> q.id
2
```
결과를 보면 두 번째로 생성한 Question 모델 데이터의 id는 예상대로 2이다.

6. Question 모델 데이터 모두 조회하기
지금까지의 실습 과정은 모두 Question 모델 데이터를 저장하기 위한 것이었다. 이번에는 저장된 데이터를 조회해 보자. 여기서는 특별히 모든 Question 모델 데이터를 조회하기 위해 Question.objects.all()을 입력한다.
```
>>> Question.objects.all()
<QuerySet [<Question: Question object (1)>, <Question: Question object (2)>]>
```
장고에서 저장된 모델 데이터는 Question.objects를 사용하여 조회할 수 있다. 그리고 Question.objects.all()은 Question에 저장된 모든 데이터를 조회하는 함수이다. 결과를 보면 QuerySet 객체가 반환됨을 알 수 있다. 그리고 QuerySEt에는 2개의 Question 객체가 포함되어 있따. 이때 <Question object (1)>, <Question object (2)>의 1,2가 바로 장고에서 Question 모델 데이터에 자동으로 입력해 준 id이다.

7. Question 모델 데이터 조회 결과에 속성값 보여 주기
그런데 6단계의 결과는 데이터 유형을 출력한 것이므로 사람이 보기 불편하다. 이때 Question 모델에 __str__ 메서드를 추가하면 된다.
```
(... 생략 ...)

class Question(models.Model):
    subject = models.CharField(max_length=200)
    content = models.TextField()
    create_date = models.DateTimeField()

    def __str__(self):
        return self.subject

(... 생략 ...)
```
이렇게 수정하면 데이터 조회 시 id가 아닌 제목을 표시해 준다.

8. Question 모델 데이터 다시 조회해 보기
모델이 수정되었으므로 장고 셸을 다시 시작하자. 장고 셸에서 quit() 명령을 실행해 종료한 후 다시 장고 셸을 시작한다. 그다음, Question 모델을 다시 임포트한 후 Question 모델 데이터를 조회해 보자.
```
(mysite) D:\code\vscode\github\jump_to_django\src\projects\mysite>python manage.py shell
Python 3.8.5 (tags/v3.8.5:580fbb0, Jul 20 2020, 15:57:54) [MSC v.1924 64 bit (AMD64)] on win32
Type "help", "copyright", "credits" or "license" for more information.
(InteractiveConsole)
>>> from pybo.models import Question, Answer
>>> Question.objects.all()
<QuerySet [<Question: pybo가 무엇인가요?>, <Question: 장고 모델 질문입니다.>]>
```
거려면 Question 모델 데이터에 id가 아니라 제목이 표시된다. 여기서 '모델이 수정되었는데 왜 makemigrations, migrate 명령을 실행하지 않지?'라는 의문을 가진 독자가 있을 수 있다. 좋은 질문이다. **makemigrations, migrate 명령은 모델의 속성이 추가되거나 변경된 경우에 실행하는 명령이다. 지금은 메서드가 추가된 것이므로 이 과정은 하지 않아도 된다.**

9. 조건으로 Question 모델 데이터 조회하기
7~8단계에서는 Question 모델 데이터를 모두 조회했다. 조건을 주어 Question 모델 데이터를 조회하고 싶다면 filter함수를 사용하면 된다.
```
>>> Question.objects.filter(id=1) 
<QuerySet [<Question: pybo가 무엇인가요?>]>
```
filter 함수는 조건에 해당하는 데이터를 모두 찾아준다. 지금은 유일한 값인 id를 조건에 사용했으므로 Question 모델 데이터 하나만 나왔다. 하지만 filter 함수는 1개 이상의 데이터를 반환한다. 다만 filter 함수는 반환값인 리스트 형태인 QuerySet이므로 정말로 1개의 데이터만 조회하고 싶다면 filter 함수 대신 get 함수를 쓰는 것이 좋다.

10. Question 모델 데이터 하나만 조회하기
get 함수를 사용하면 리스트가 아닌 데이터 하나만 조회할 수 있다.
```
>>> Question.objects.get(id=1)
<Question: pybo가 무엇인가요?>
```
반환값을 보면 QuerySet이 아니라 Question이다. 여기서 filter 함수와 get 함수의 차이점을 알 수 있다. filter 함수는 여러 건의 데이터를 반환하지만, get 함수는 단 한 건의 데이터를 반환한다. 두 함수의 차이점을 꼭 알아 두기 바란다.

11. get으로 조건에 맞지 않는 데이터 조회하기
조건에 맞지 않는 데이터를 get 함수로 조회하면 어떻게 될까? id가 3인 데이터를 조회해보자.
```
>>> Question.objects.get(id=3) 
Traceback (most recent call last):
  File "<console>", line 1, in <module>
  File "C:\venvs\mysite\lib\site-packages\django\db\models\manager.py", line 85, in manager_method
    return getattr(self.get_queryset(), name)(*args, **kwargs)
  File "C:\venvs\mysite\lib\site-packages\django\db\models\query.py", line 429, in get
    raise self.model.DoesNotExist(
pybo.models.Question.DoesNotExist: Question matching query does not exist.
```
그러면 이와 같은 오류 메시지를 볼 수 있다. 조건에 맞는 데이터 1개를 반환해야 하는데 조건에 맞는 데이터가 없으니 오류가 발생한 것이다.

12. filter로 조건에 맞지 않는 데이터 조회하기
그러면 filter 함수는 어떨까? filter 함수로 조건에 맞지 않는 데이터를 조회해 보자.
```
>>> Question.objects.filter(id=3) 
<QuerySet []>
```
filter 함수는 조건에 맞는 데이터가 없으면 그저 빈 QuerySet을 반환한다. get 함수는 반드시 1건의 데이터를 반환해야 한다는 특징이 있으므로 오류가 발생할 것이다. 이번에는 조금더 유용한 데이터 조회 방법을 알아보자. 만약 제목에 '장고'라는 글자가 포함된 데이터를 조회하려면 어떻게 해야 할까/

13. 제목의 일부를 이용하여 데이터 조회하기
subject에 "장고"라는 문자열이 포함된 데이터를 조회하려면 조건에 subject__contains를 이용하면 된다. **이때 subject와 contains 사이의 언더스코어는 1개가 아니라 2개이다.** 장고셸에서 다음 코드를 입력해 보자.
```
>>> Question.objects.filter(subject__contains='장고') 
<QuerySet [<Question: 장고 모델 질문입니다.>]>
```
**subject__contains='장고'**의 이미는 'subject 속성에 '장고'라는 문자열이 포함되어 있는가?'이다. 이 밖에도 filter 함수의 사용 방법은 무궁무진하다. 자세한 filter 함수의 사용 방법은 장고 공식 문서를 참조하자.
> 장고는 외워서 사용할 수 있는 프레임워크가 아니므로 장고 공식 문서를 자주 참고하는 습관을 들이는 것이 좋다.
> 장고 공식 문서(데이터 조회 관련): http://docs.djangoproject.com/en/3.0/topics/db/queries

## 데이터 수정하기
이번에는 지금까지 저장했던 Question 모델 데이터를 수정하자.

1. Question 모델 데이터 수정하기
Question 모델 데이터를 수정하려면 우선 수정할 데이터를 조회해야 한다. 다음은 id가 2인 데이터를 조회한 것이다. 이 데이터를 수정할 것이다.
```
>>> q = Question.objects.get(id=2)
>>> q
<Question: 장고 모델 질문입니다.>
```

2. subject 속성 수정하기
subject 속성을 수정하자.
```
>>> q.subject = 'Django Model Question'
```

3. 수정한 Question 모델 데이터베이스에 저장하기
2단계만으로는 변경된 Question 모델 데이터가 데이터베이스에 적용되지 않는다. **반드시 다음처럼 save 함수를 실행해야 변경된 Question 모델 데이터가 데이터베이스에 반영된다.**
```
>>> q.save()
>>> q
<Question: Django Model Question>
```

## 데이터 삭제하기
이번에는 Question 모델 데이터를 데이터베이스에서 삭제해 보자.

1. Question 모델 데이터 삭제하기
데이터 삭제는 데이터 수정과 비슷한 과정으로 진행된다. 여기서는 id가 1인 Question 모델 데이터를 삭제한다.
```
>>> q = Question.objects.get(id=1)
>>> q.delete()
(1, {'pybo.Question': 1})
```
delete 함수를 수행하면 해당 데이터가 데이터베이스에서 즉시 삭제되며, 삭제된 데이터의 추가 정보가 반환된다. (1, {'pybo.Question': 1})에서 앞의 1은 삭제된 Question 모델 데이터의 id를 의미하고 {'pybo.Question': 1}은 삭제된 모델 데이터의 개수를 의미한다.
> Answer 모델을 만들 때 ForeignKey로 Question 모델과 연결한 것이 기억나는가? 만약 삭제한 Question 모델 데이터에 2개의 Answer 모델 데이터가 등록된 상태라면 (1, {'pybo.Answer': 2}, 'pybo.Question': 1)와 같이 삭제된 답변 개수도 함께 반환될 것이다.

2. 삭제 확인하기
Question 모델 데이터가 정말로 삭제되었는지 확인해 보자.
```
>>> Question.objects.all()
<QuerySet [<Question: Django Model Question>]>
```
결과를 보면 첫 번째 질문은 삭제되고, 두 번째 질문만 남아 있다.

## 연결된 데이터 알아보기
앞에서 Answer 모델을 만들 때 ForeignKey로 Question 모델과 연결한 내용이 기억날 것이다. Answer 모델은 Question 모델과 연결되어 있으므로 데이터를 만들 때 Question 모델 데이터가 필요하다.

1. Answer 모델 데이터 만들기
id가 2인 Question 모델 데이터를 얻은 다음, 이를 이용하여 Answer 모델 데이터를 만들어 보자.
```
>>> q = Question.objects.get(id=2)
>>> q
<Question: Django Model Question>
>>> from django.utils import timezone
>>> a = Answer(question=q, content='네 자동으로 생성됩니다.', create_date=timezone.now())
>>> a.save()
```

2. id 확인하기
Answer 모델 데이터에도 id가 있다.
```
>>> a.id
1
```

3. Answer 모델 데이터 조회하기
Answer 모델 데이터를 get 함수로 조회해 보자. 조건은 id를 사용한다.
```
>>> a = Answer.objects.get(id=1)
>>> a
<Answer: Answer object (1)>
```

4. 연결된 데이터로 조회하기: 답변에 있는 질문 조회하기
Answer 모델 데이터에는 Question 모델 데이터가 연결되어 있으므로 Answer 모델 데이터에 연결된 Question 모델 데이터를 조회할 수 있다.
```
>>> a.question
<Question: Django Model Question>
```
Answer 모델 객체인 a에는 question 속성이 있으므로 a를 통해 질문을 찾는 것은 매우 쉽다. 그렇다면 반대로 질문을 통해 답변을 찾을 수 있을까? Question 모델에는 답변 속성이 없어서 불가능할 것 같지만 실제로는 가능하다.

5. 연결된 데이터로 조회하기: 질문을 통해 답변 찾기
다음처럼 answer_set을 사용하면 된다.
```
>>> q.answer_set.all()
<QuerySet [<Answer: Answer object (1)>]>
```
Question 모델과 Answer 모델처럼 서로 연결되어 있으면 **연결모델명_set**과 같은 방법으로 연결된 데이터를 조회할 수 있다. 그리고 아마 여러분은 **연결모델명_set**을 써야 하는 경우와 그렇지 않은 경우가 헷갈릴 것이다.

이때는 질문과 답변이 달리는 게시판을 상식적으로 생각해 보자. 질문 1개에는 1개 이상의 답변이 달릴 수 있으므로 질문에 달린 답변은 **q.answer_set**으로 조회해야 한다(답변 세트를 조회). 답변 질문 1개에 대한 것이므로 애초에 여러 개의 질문을 조회할 수 없다. 다시 말해 답변 1개 입장에서는 질문 1개만 연결되어 있으므로 **a.question**만 실행할 수 있다. 1개의 답변으로 여러 개의 질문을 **a.question_set**으로 조회하는 것은 불가능하며, 상식적으로 생각해보아도 이상하다. **연결모델명_set**은 정말 신통방통한 장고의 기능이 아닐 수 없다. **연결모델명_set**은 자주 사용할 기능이니 꼭 기억하자.