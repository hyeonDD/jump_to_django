<!-- -
![](https://github.com/hyeonDD/jump_to_django/blob/main/1_Part/1_3_Part/django.png)
-->
# 장고 개발 환경 준비하기

이제 본격적으로 장고 개발 환경을 준비해 보자. 그전에 여러분이 알아야 할 중요한 개념이 하나 있다. 바로 파이썬 가상 환경이다. 우리는 장고를 파이썬 가상 환경에 설치할 것이다.

## 파이썬 가상 환경 알아보기

파이썬 가상 환경은 파이썬 프로젝트를 진행할 때 독립된 환경을 만들어 주는 고마운 도구다. 예를 들어 파이썬 개발자 A가 2개의 파이썬 프로젝트를 개발하고 관리한다고 가정하자. 파이썬 프로젝트를 각각 P-1, P-2라고 브르겠다. 이때 P-1, P-2에 필요한 파이썬 또는 파이썬 라이브러리의 버전이 다를 수 있다. 이를테면 P-1에는 파이썬2.7 버전이, P-2에는 파이썬 3.8 버전이 필요할 수 있다. 이때 하나의 데스크톱에 서로 다른 버전의 파이썬을 설치해야 하는 문제가 발생한다.

![1-03_1.png](https://github.com/hyeonDD/jump_to_django/blob/main/1_Part/1_3_Part/1-03_1.png)

이러한 개발 환경은 구축하기도 어렵고 사용하기도 힘들다. 가상 환경이 없던 예전에는 그런 고생을 감수할 수밖에 없었다. 하지만 파이썬 가상 환경을 이용하면 하나의 데스크톱 안에 독립된 가상 환경을 여러 개 만들 수 있다. 즉, 프로젝트 P-1을 위해 가상 환경 V-1을 만들어 파이썬 2.7버전을 설치하고, 프로젝트 P-2를 위해 가상 환경 V-2를 만들어 파이썬 3.8버전을 설치해서 사용할 수 있다.

![1-03_2 (1).png](https://github.com/hyeonDD/jump_to_django/blob/main/1_Part/1_3_Part/1-03_2 (1).png)

이처럼 가상 환경을 이용하면 하나의 데스크톱에 서로 다른 버전의 파이썬과 라이브러리를 쉽게 설치해 사용할 수 있다. 물론 이 책에서는 '파이보'라는 하나의 프로젝트만 진행할 것이므로 가상 환경이 필수는 아니다. 하지만 앞으로 웹 프로그래밍을 계속하고 싶다면 가상 환경의 개념을 익히고 실제로 사용해 보는 것이 좋다.

## 파이썬 가상 환경 사용해 보기

**가상 환경 디렉터리 생성하기**
윈도우에서 명령 프롬프트를 실행하고 다음 명령어를 입력해 C:\venvs라는 디렉터리를 만들자.

```
C:\Users\pahkey> cd \
C:\> mkdir venvs
C:\> cd venvs
```

**가상 환경 만들기**
파이썬 가상 환경을 만드는 다음 명령어를 입력해 실행하자.
```
C:\venvs> python -m venv mysite
```

명령에서 python -m venv는 파이썬 모듈 중 venv라는 모듈을 사용한다는 의미다. 그 뒤의 mysite는 여러분이 생성할 가상 환경의 이름이다. 가상 환경의 이름을 반드시 mysite로 지을 필요는 없다. 만약 가상 환경의 이름을 awesomesite와 같이 지정했다면 책에 등장하는 mystie라는 가상 환경 이름을 awesomesite로 대체하여 읽으면 된다. 명령을 잘 수행했다면 C:\venvs 디렉터리 아래에 mysite라는 디렉터리가 생성되었을 것이다. 이 디렉터리를 가상환경이라 생각하면 된다. 그런데 가상 환경을 만들었다 해서 바로 가상 환경을 사용할 수는 없다. 가상 환경을 사용하려면 가상 환경에 진입해야 한다.
> 하지만 실습 진행의 편의를 위해 가상 환경 이름을 동이랗게 하기를 권장한다.

**가상 환경에 진입하기**
가상 환경에 진입하려면 우리가 생성한 mysite 가상 환경에 있는 Scripts 디렉터리의 **activate** 명령을 수행해야 한다. 다음 명령을 입력하여 mysite 가상 환경에 진입해 보자.
```
C:\venvs>cd C:\venvs\mysite\Scripts
C:\venvs\mysite\Scripts> activate
(mysite) C:\venvs\mysite\Scripts>
```

그러면 C:\ 왼쪽에 **(mysite)**라는 프롬프트를 확인할 수 있다. 이름에서 볼 수 있듯 현재 여러분이 진입한 가상 환경을 의미한다.

**가상 환경에서 벗어나기**
현재 진입한 가상 환경에서 벗어나려면 **deactivate**라는 명령을 실행하면 된다. 이 명령은 어느 위치에서 실행해도 상관없다.
```
(mysite) C:\venvs\mysite\Scripts> deactivate
```

가상 환경에서 벗어났다면 C;\ 왼쪽에 있던 **(mystie)**라는 프롬프트가 사라졌을 것이다. 지금까지 가상 환경의 개념과 실습을 진행해 보았다. 가상 환경이라는 개념이 조금은 생소하겠지만 익혀 두면 여러분의 웹 프로그래밍 경험에 도움이 될 것이다.

## 장고 설치하기

드디어 장고를 설치할 차례가 왔다. 앞에서 만든 mysite 가상 환경에 장고를 설치해 보자.

**가상 환경인지 확인하기**

명령 프롬프트 왼쪽에 (mysite) 프롬프트가 보이는지 확인하자. 만약 명령 프롬프트 왼쪽에 (mystie) 프롬프트가 보이지 않는다면 바로 이전의 실습을 참고하여 가상 환경에 진입한 상태에서 장고 설치를 진행하자.
```
C:\venvs\mysite\Scripts> activate
(mysite) C:\venvs\mysite\Scripts>
```

**가상 환경에서 장고 설치하기**
mysite 가상 환경에 진입한 상태에서 pip install django==3.1.3 명령을 입력하자. pip은 파이썬 라이브러리를 설치하고 관리해 주는 파이썬 도구다. pip install django==3.1.3는 pip으로 장고 3.1.3 버전을 설치하는 명령이라고 생각하면된다. 다음 화면이 나오면 장고 설치가 잘 된 것이다. 그런데 마지막에 다음과 같은 경고 문구가 보인다. pip이 최신 버전이 아니라는 내용이다.
```
(mysite) C:\venvs\mysite\Scripts> pip install django==3.1.3
Collecting django
  Using cached https://files.pythonhosted.org/packages/01/a5/fb3dad18422fcd4241d18460a1fe17542bfdeadcf74e3861d1a2dfc9e459/Django-3.1.3-py3-none-any.whl
Collecting asgiref~=3.2.10 (from django)
  Using cached https://files.pythonhosted.org/packages/d5/eb/64725b25f991010307fd18a9e0c1f0e6dff2f03622fc4bcbcdb2244f60d6/asgiref-3.2.10-py3-none-any.whl
Collecting sqlparse>=0.2.2 (from django)
  Using cached https://files.pythonhosted.org/packages/85/ee/6e821932f413a5c4b76be9c5936e313e4fc626b33f16e027866e1d60f588/sqlparse-0.3.1-py2.py3-none-any.whl
Collecting pytz (from django)
  Using cached https://files.pythonhosted.org/packages/4f/a4/879454d49688e2fad93e59d7d4efda580b783c745fd2ec2a3adf87b0808d/pytz-2020.1-py2.py3-none-any.whl
Installing collected packages: asgiref, sqlparse, pytz, django
Successfully installed asgiref-3.2.10 django-3.1.3 pytz-2020.1 sqlparse-0.3.1

WARNING: You are using pip version 19.2.3, however version 20.2.3 is available.
You should consider upgrading via the 'python -m pip install --upgrade pip' command.
```

**pip 최신 버전으로 설치하기**
경고 메시지에 따라 python -m pip install --upgrade pip 명령을 입력해 pip을 최신 버전으로 설치하자.
```
(mysite) C:\venvs\mysite\Scripts> python -m pip install --upgrade pip
Collecting pip
  Downloading https://files.pythonhosted.org/packages/54/0c/d01aa759fdc501a58f431eb594a17495f15b88da142ce14b5845662c13f3/pip-20.0.2-py2.py3-none-any.whl (1.4MB)
     |================================| 1.4MB 226kB/s
Installing collected packages: pip
  Found existing installation: pip 19.2.3
    Uninstalling pip-19.2.3:
      Successfully uninstalled pip-19.2.3
Successfully installed pip-20.0.2
```