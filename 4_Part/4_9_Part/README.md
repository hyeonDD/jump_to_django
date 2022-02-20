<!-- -
![](https://github.com/hyeonDD/jump_to_django/blob/main/4_Part/4_9_Part/django.png)
-->
# Gunicon
이전 장에서 웹 서버에서 파이썬 장고 애플리케이션이 구현되어 있는 wsgi.py 파일을 호출하려면 WSGI 서버가 필요하다고 했다. 이번에는 파이보에서 사용할 WSGI 서버인 Gunicorn을 설치하고 사용해 보자.

> Gunicorn은 구니콘이라고 읽는다.

WSGI 서버의 양대산맥으로 Gunicorn과 uwsgi가 있다. 과거에는 "성능은 uwsgi가 앞서고 편의성은 Gunicorn이 좋다" 라는 의견들이 많았는데 요새는 Gunicorn의 성능이 매우 좋아졌기 때문에 Gunicorn을 사용하는 사람들이 점점 더 많아지고 있다.

## Gunicorn 설치
Gunicorn은 개발이 아니라 운영을 위한 도구이므로 로컬 환경에 설치할 필요가 없다. 서버환경에 Gunicorn을 설치하자. MobaXterm으로 AWS 서버에 접속한 뒤 가상 환경에서 pip 을 이용하여 Gunicorn을 설치하자.
```
(mysite) ubuntu@ip-172-26-12-247:~/projects/mysite$ pip install gunicorn
Collecting gunicorn
  Downloading https://files.pythonhosted.org/packages/69/ca/926f7cd3a2014b16870086b2d0fdc84a9e49473c68a8dff8b57f7c156f43/gunicorn-20.0.4-py2.py3-none-any.whl (77kB)
    (... 생략 ...)
Requirement already satisfied: setuptools>=3.0 in /home/ubuntu/venvs/mysite/lib/python3.6/site-packages (from gunicorn)
Installing collected packages: gunicorn
Successfully installed gunicorn-20.0.4
```

## Gunicorn 테스트
Gunicorn이 정상으로 실행되는지 간단하게 실행해 보자.

```
(mysite) ubuntu@ip-172-26-12-247:~$ cd ~/projects/mysite/
(mysite) ubuntu@ip-172-26-12-247:~/projects/mysite$ gunicorn --bind 0:8000 config.wsgi:application
[2020-04-17 00:59:12 +0000] [32356] [INFO] Starting gunicorn 20.0.4
[2020-04-17 00:59:12 +0000] [32356] [INFO] Listening at: http://0.0.0.0:8000 (32356)
[2020-04-17 00:59:12 +0000] [32356] [INFO] Using worker: sync
[2020-04-17 00:59:12 +0000] [32359] [INFO] Booting worker with pid: 32359
```
/home/ubuntu/projects/mysite 디렉터리로 이동한 뒤 gunicorn --bind 0:8000 config.wsgi:application 명령을 수행했다. 명령을 쪼개서 설명하면 --bind 0:8000은 8000번 포트로 WSGI 서버를 수행한다는 의미이고, config.wsgi:application은 WSGI 서버가 호출하는 WSGI 애플리케이션이 config/wsgi.py 파일의 application이라는 의미이다.

config.wsgi:application 명령에서 config.wsgi 와 application을 구분하는 구분자는 '.'이 아니라 ':'임을 주의하자.

서버가 오류없이 잘 시작되는 것을 확인할 수 있을 것이다.

하지만 웹 브라우저로 다음 URL에 접속해 보면 다음과 같이 보인다.
```
http://3.35.153.92:8000
```
> 3.35.153.92는 필자의 고정 IP이므로 여러분의 서버 고정 IP로 바꿔서 접속하자.

![4-09_1.png](https://github.com/hyeonDD/jump_to_django/blob/main/4_Part/4_9_Part/4-09_1.png)

이렇게 보이는 이유는 Gunicorn이 정적 파일들을 해석하지 못했기 때문이다. 파이보는 bootstrap.min.css, bootstrap.min.js, style.css 등의 정적 파일을 필요로 한다. 하지만 Gunicorn은 동적 페이지 요청만 처리할 수 있기 때문에 위와 같이 표시되는 것이다.

정적 파일을 효과적으로 처리하는 웹 서버는 04-10 장에서 알아볼 것이다. 아무튼 Gunicorn이 정상으로 작동하는 것을 확인했으므로 <Ctrl+C>를 눌러 Gunicorn을 종료하자.

## Gunicorn 소켓
Gunicorn은 앞에서 본 것처럼 포트(8000)를 이용하여 서버를 띄울수 있다. 하지만 Unix 계열 시스템에서는 포트로 서비스하기보다는 유닉스 소켓(Unix socket)을 사용하는 것이 빠르고 효율적이다. 이번에는 Gunicorn을 유닉스 소켓으로 서비스하는 방법을 알아보자.

> 현재 여러분의 AWS 서버가 Unix 계열 시스템인 우분투이다

다음과 같이 Gunicorn을 실행하자.
```
(mysite) ubuntu@ip-172-26-12-247:~/projects/mysite$ gunicorn --bind unix:/tmp/gunicorn.sock config.wsgi:application
[2020-04-17 01:14:51 +0000] [32392] [INFO] Starting gunicorn 20.0.4
[2020-04-17 01:14:51 +0000] [32392] [INFO] Listening at: unix:/tmp/gunicorn.sock (32392)
[2020-04-17 01:14:51 +0000] [32392] [INFO] Using worker: sync
[2020-04-17 01:14:51 +0000] [32395] [INFO] Booting worker with pid: 32395
```
명령을 유심히 살펴보면 포트 방식으로 Gunicorn을 실행했을 때와 다르다. 즉, --bind unix:/tmp/gunicorn.sock 부분이 다르다. 기존에는 --bind 0:8000와 같이 입력했지만 유닉스 소켓 방식은 --bind unix:/tmp/gunicorn.sock와 같이 입력했다.

> 유닉스 소켓 방식으로 Gunicorn 서버를 실행하면 단독으로 Gunicorn 서버에 접속하여 실행할 수 없다. 유닉스 소켓 방식으로 실행한 Gunicorn 서버는 Nginx와 같은 웹 서버에서 유닉스 소켓으로 WSGI 서버에 접속하도록 설정해야 한다.

## Gunicorn 서비스
이번에는 AWS 서버에 Gunicorn을 서비스로 등록해 보자. 그 이유는 Gunicorn의 시작, 중지를 쉽게 하고, 또 AWS 서버를 다시 시작할 때 Gunicorn을 자동으로 실행하기 위해서이다. Gunicorn을 서비스로 등록하려면 환경 변수 파일과 서비스 파일을 작성해야 한다.

환경 변수 파일
Gunicorn이 서비스로 실행될 경우에는 alias나 mysite.sh에 의해 생성되는 DJANGO_SETTINGS_MODULE 환경변수가 생성되지 않기 때문에 환경변수 파일을 통해 생성해 주어야 한다.

다음과 같이 Gunicorn의 환경 변수 파일을 작성하자.

> 서버에서 nano 편집기를 이용하여 작성하자.

[파일명: /home/ubuntu/venvs/mysite.env]

```
DJANGO_SETTINGS_MODULE=config.settings.prod
```

파일의 내용은 위처럼 한줄만 작성하면 된다.

## 서비스 파일
그리고 /etc/systemd/system/ 디렉터리에 다음과 같은 내용의 mysite.service라는 이름의 서비스 파일을 생성하자. 다만 서비스 파일은 시스템 디렉터리에 저장해야 하므로 sudo nano mysite.service와 같이 관리자 권한으로 파일을 생성해야 한다.

[파일명: /etc/systemd/system/mysite.service]

```
[Unit]
Description=gunicorn daemon
After=network.target

[Service]
User=ubuntu
Group=ubuntu
WorkingDirectory=/home/ubuntu/projects/mysite
EnvironmentFile=/home/ubuntu/venvs/mysite.env
ExecStart=/home/ubuntu/venvs/mysite/bin/gunicorn \
        --workers 2 \
        --bind unix:/tmp/gunicorn.sock \
        config.wsgi:application

[Install]
WantedBy=multi-user.target
```
서비스 파일의 EnvironmentFile 항목이 우리가 작성한 환경 변수 파일을 불러오는 설정이다. --worker 2는 Gunicorn 프로세스를 2개 사용하라는 의미이다.

> 프로세스를 2개로 지정한 이유는 여러분이 선택한 월 사용료가 3.5달러인 사양의 서버에는 프로세스 개수가 이 정도이면 적당하기 때문이다

## 서비스 실행과 등록
서비스 파일을 생성한 후 다음 명령으로 서비스를 실행해 보자. 서비스 파일이 관리자 디렉터리에 있으므로 실행 역시 관리자 권한으로 실행해야 한다.
```
(mysite) ubuntu@ip-172-26-14-223:/etc/systemd/system$ sudo systemctl start mysite.service
```
서비스가 잘 실행됐는지 확인하려면 sudo systemctl status mysite.service 명령을 실행하면 된다. 만약 다음과 같은 메시지가 나타나지 않으면 /var/log/syslog 파일에서 오류 원인을 확인하고 수정해야 한다.
```
(mysite) ubuntu@ip-172-26-12-247:/etc/systemd/system$ sudo systemctl status mysite.service
● mysite.service - gunicorn daemon
   Loaded: loaded (/etc/systemd/system/mysite.service; disabled; vendor preset: enabled)
   Active: active (running) since Thu 2020-04-23 12:12:27 UTC; 1s ago
 Main PID: 26513 (gunicorn)
    Tasks: 3 (limit: 547)
   CGroup: /system.slice/mysite.service
           ├─26513 /home/ubuntu/venvs/mysite/bin/python3 /home/ubuntu/venvs/mysite/bin/gunicorn --workers 2 --bind unix:/tmp/gunicorn.sock config.wsgi:application
           ├─26534 /home/ubuntu/venvs/mysite/bin/python3 /home/ubuntu/venvs/mysite/bin/gunicorn --workers 2 --bind unix:/tmp/gunicorn.sock config.wsgi:application
           └─26536 /home/ubuntu/venvs/mysite/bin/python3 /home/ubuntu/venvs/mysite/bin/gunicorn --workers 2 --bind unix:/tmp/gunicorn.sock config.wsgi:application

Apr 23 12:12:27 ip-172-26-12-247 systemd[1]: Started gunicorn daemon.
Apr 23 12:12:28 ip-172-26-12-247 gunicorn[26513]: [2020-04-23 12:12:28 +0000] [26513] [INFO] Starting gunicorn 20.0.4
Apr 23 12:12:28 ip-172-26-12-247 gunicorn[26513]: [2020-04-23 12:12:28 +0000] [26513] [INFO] Listening at: unix:/tmp/gunicorn.sock (26513)
Apr 23 12:12:28 ip-172-26-12-247 gunicorn[26513]: [2020-04-23 12:12:28 +0000] [26513] [INFO] Using worker: sync
Apr 23 12:12:28 ip-172-26-12-247 gunicorn[26513]: [2020-04-23 12:12:28 +0000] [26534] [INFO] Booting worker with pid: 26534
Apr 23 12:12:28 ip-172-26-12-247 gunicorn[26513]: [2020-04-23 12:12:28 +0000] [26536] [INFO] Booting worker with pid: 26536
```
마지막으로 AWS 서버가 다시 시작될 때 Gunicorn을 자동으로 실행하도록 enable 옵션을 이용하여 서비스로 등록하자.
```
(mysite) ubuntu@ip-172-26-12-247:/etc/systemd/system$ sudo systemctl enable mysite.service
```

---

**서비스 종료와 재시작**

만약 서비스를 종료하려면 다음 명령을 수행하면 된다.
```
$ sudo systemctl stop mysite.service
```
서비스를 다시 시작하려면 다음 명령을 수행하면 된다.
```
$ sudo systemctl restart mysite.service
```

---