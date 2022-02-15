<!-- -
![](https://github.com/hyeonDD/jump_to_django/blob/main/1_Part/1_4_Part/django.png)
-->
# 장고 프로젝트 생성하기

장고에는 프로젝트라는 개념이 있는데, 장고의 프로젝트는 하나의 웹 사이트라고 생각하면된다. 즉, 장고 프로젝트를 생성하면 한 개의 웹 사이트를 생성하는 것과 같다. 프로젝트 안에는 여러 개의 앱이 존재한다. 이 앱들이 모여 웹 사이트를 구성한다. 여기서 앱이란 관리자 앱, 인증 앱 등과 같이 장고가 기본으로 제공하는 앱과 개발자가 직접 만든 앱을 칭한다.
> 장고에서 말하는 앱은 일반적으로 여러분이 알고 있는 안드로이드 앱, iOS 앱과는 성격이 다르다. 안드로이드 앱이 하나의 프로그램이라면, 장고의 앱은 프로젝트를 구성하는 작은 단위의 기능이다.

## 프로젝트 디렉터리 생성하기

1. 프로젝트 루트 디렉터리 생성하기
장고 프로젝트는 여러 개가 될 수 있으므로 프로젝트를 모아 둘 프로젝트 루트 디렉터리 생성은 필수다. 여기서는 프로젝트 루트 디렉터리 이름을 projects로 지었다.
> 여기서는 C 드라이브에 프로젝트를 담을 루트 디렉터리를 생성했다.
```
C:\Users\pahke>cd \
C:\>mkdir projects
C:\>cd projects
C:\projects>
```

2. 프로젝트 루트 디렉터리 안에서 가상 환경에 진입하기
프로젝트 루트 디렉터리 안에서 다음 명령어를 입력해 앞에서 만든 mysite 가상 환경에 진입한다. 이때 반드시 프로젝트 루트 디렉터리에서 명령어를 입력해야 한다. 가상 환경 진입 명령어가 길어서 좀 불편하겠지만, 지금은 이 방법으로 가상환경에 진입하겠다.
> 가상 환경 진입 명령어를 간단하게 만드는 팁은 이 절의 마지막에서 설명한다.
```
C:\projects>C:\venvs\mysite\Scripts\activate
(mysite) C:\projects>
```

3. 장고 프로젝트를 담을 디렉터리 생성하고 이동하기
장고 프로젝트를 담을 mysite 디렉터리를 생성하고 이동하자.
```
(mysite) C:\projects>mkdir mysite
(mysite) C:\projects>cd mysite
(mysite) C:\projects\mysite>
```

4. 장고 프로젝트 생성하기
장고 프로젝트 디렉터리에서 django-admin이라는 도구로 장고 프로젝트를 생성하자. 이때 config 다음에 점 기호(.)가 있음에 주의하자. 점 기호는 '현재 디렉터리를 프로젝트 디렉터리로 만들라'는 의미이다.
```
(mysite) C:\projects\mysite>django-admin startproject config .
```

5. 장고 프로젝트 내용물 확인하기
4 단계가 완료되면 다음과 같은 구조로 디렉터리가 구성되어 있는지, 파일이 잘 생성되어 있는지 확인해 보자.

## 개발 서버 구동하고 웹 사이트에 접속해 보기

이제 웹 사이트를 생성한 것과 다름엇다. 개발 서버를 구동하고 웹 사이트에 접속해 보자.

1. 개발 서버 구동하기.
python manage.py runserver 명령을 실행하면 개발 서버가 구동된다. 맨 아랫줄에는 'Quit the server with CTRL-BREAK'이라는 개발 서버 종료 방법도 안내되어 있다. 개발 서버를 종료하려면 Ctrl + C를 누르란 뜻이다. 개발 서버가 구동된 상태를 유지하고 웹 브라우저를 이용하여 127.0.01:8000에 접속해 보자. 그러면 장고가 기본으로 만든 웹 사이트 화면을 볼 수 있다.
> 개발 서버는 127.0.0.0, 즉 로컬호스트로 실행되므로 로컬 서버라 부르기도 한다. 필자는 개발을 위해 실행된 개발 서버를 '개발 서버'라 부르겠다.
> 127.0.0.1:8000 대신 localhost:8000이라고 입력해도 같은 화면을 볼 수 있다. 127.0.0.1과 localhost는 현재 컴퓨터를 가리키는 아이피 주소다.
```
(mysite) C:\projects\mysite>python manage.py runserver
Watching for file changes with StatReloader
Performing system checks...

System check identified no issues (0 silenced).

You have 18 unapplied migration(s). Your project may not work properly until you apply the migrations for app(s): admin, auth, contenttypes, sessions.
Run 'python manage.py migrate' to apply them.
April 21, 2020 - 21:51:03
Django version 3.1.3, using settings 'config.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CTRL-BREAK.
```

![1-04_1.png](https://github.com/hyeonDD/jump_to_django/blob/main/1_Part/1_4_Part/1-04_1.png)

아쉽게도 아직은 개발 서버 환경에서 사이트가 실행되고 있으므로 개발 서버 환경을 실행한 컴퓨터(여러분의 컴퓨터)에서만 사이트에 접속할 수 있다. 즉, 아직은 다른 사람이 접속할 수 있는 상태가 아니다. 다른 사람이 접속할 수 있는 기능은 파이보를 멋지게 만든 후에 추가할 것이다.

**개발 서버 종료하기**
ctrl + c를 눌러 개발 서버를 종료해 보자. 개발 서버가 종료되면 127.0.0.1:8000으로 mysite에 접속할 수 없다. 웹 브라우저에서 새로고침을 눌러 정말 그런지 확인해 보자.

![no_con.png](https://github.com/hyeonDD/jump_to_django/blob/main/1_Part/1_4_Part/no_con.png)

## mysite 가상 환경에 간단히 진입하기

mysite 가상 환경에 진입하려면 매번 명령 프롬포트를 실행하고 c:\venvs\mysite\Scripts 디렉터리로 이동하여 activate 명령을 수행해야 한다. 이런 일련의 과정을 한 번에 수행할 수 있는 배치 프로그램을 만들어 귀찮음을 덜어 보자.

1. mysite.cmd 배치 파일 생성하기
mysite.cmd 파일을 venvs 디렉터리에 만들고 다음과 같이 작성하여 저장하자. 확장자.cmd가 붙은 파일은 배치파일이라 부르며, 명령어 입력과 실행을 한 번에 해주는 파일이라 생각하면 된다.
```
@echo off
cd c:/projects/mysite
c:/venvs/mysite/scripts/activate
```
배치 파일의 내용은 C:\projects\mystie 디렉터리로 이동한 다음, C:\venvs\mysite\scripts\activate 명령을 수행하라는 뜻이다.

2. 배치 파일 위치를 PATH 환경 변수에 추가하기
이 배치 파일이 명령 프롬프트 어느 곳에서나 수행될 수 있도록 C:\venvs 디렉터리를 시스템의 환경 변수 PATH에 추가해야 한다. 먼저 window + r 키를 입력하여 다음처럼 sysdm.cpl 명령을 입력한 다음 <확인>을 누르자.

![1-04_3.png](https://github.com/hyeonDD/jump_to_django/blob/main/1_Part/1_4_Part/1-04_3.png)

그러면 다음과 같은 '시스템 속성' 창이 나타난다. 여기서 <고급> 탭을 선택하고 <환경 변수> 버튼을 누르자.

![1-04_4.png](https://github.com/hyeonDD/jump_to_django/blob/main/1_Part/1_4_Part/1-04_4.png)

그러면 다음과 같은 '환경 변수' 창이 나타난다. 여기서 사용자 변수 중 <Path>를 선택하고 <편집> 버튼을 누르자.

![1-04_5.png](https://github.com/hyeonDD/jump_to_django/blob/main/1_Part/1_4_Part/1-04_5.png)

그러면 다음과 같은 '환경 변수 편집' 창이 나타난다. 여기서 <새로 만들기> 버튼을 누르자.

![1-04_6.png](https://github.com/hyeonDD/jump_to_django/blob/main/1_Part/1_4_Part/1-04_6.png)

그리고 다음 그림처럼 C:\venvs라는 디렉터리를 추가하고 <확인> 버튼을 누르자.

![1-04_7.png](https://github.com/hyeonDD/jump_to_django/blob/main/1_Part/1_4_Part/1-04_7.png)

마지막으로 다음 '환경 변수' 창에서 <확인> 버튼을 누르자.

3. PATH 환경 변수 확인하기.
이렇게 하면 환경 변수 PATH에 C:\venvs 디렉터리가 추가되어 mysite.cmd명령을 어디서든 실행할 수 있다. 명령 프롬프트를 다시 시작하자(그래야 환경 변수 PATH가 제대로 반영된다). 그리고 set path 명령을 실행하여 변경된 환경 변수 PATh의 내용을 확인해 보자. C:\venvs라는 디렉터리 환경 변수 PATH에 포함되어 있으면 된다.
```
C:\Users\pahkey>set path
Path=C:\Windows\system32; (... 생략 ...) ;C:\venvs
PATHEXT=.COM;.EXE;.BAT;.CMD;.VBS;.VBE;.JS;.JSE;.WSF;.WSH;.MSC
```

4. 배치 파일 실행하여 가상 환경에 진입하기
이제 지금까지 만든 mysite 명령(배치 파일 이름)을 실행하여 가상 환경에 잘 진입하는지 확인해 보자. 참고로 윈도우에서 확장자가 .cmd 파일은 확장자까지 입력하지 않아도 도니다.
```
C:\Users\pahkey> mysite
(mysite) C:\projects\mysite>
```