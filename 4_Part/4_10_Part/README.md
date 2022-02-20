<!-- -
![](https://github.com/hyeonDD/jump_to_django/blob/main/4_Part/4_10_Part/django.png)
-->
# Nginx
웹서버(Web Server)는 브라우저의 정적 페이지 요청을 처리하고 동적 페이지 요청인 경우 WSGI 서버를 호출하여 응답한다. 이번 장에서는 파이보가 사용할 웹 서버인 엔진엑스(Nginx)를 설치하고 적용해 보자.

---

**Nginx**

Nginx는 높은 성능을 위해서 개발된 웹 서버로 점점 사용자가 증가하는 추세이며 특히 파이썬 웹 프레임워크인 장고나 플라스크등에서 주로 사용되는 서버이다. 또한 Nginx를 사용하기 위한 설정도 무척 간단하여 쉽게 사용할수 있다.

---

## Nginx 설치
다음과 같이 관리자 권한으로 Nginx를 설치하자.
```
(mysite) ubuntu@ip-172-26-12-247:~/projects/mysite$ sudo apt install nginx
```
대략 10초 내외로 설치가 될 것이다.

## Nginx 설정
먼저 다음처럼 /etc/nginx/sites-available 디렉터리로 이동하자.
```
(mysite) ubuntu@ip-172-26-12-247:~/projects/mysite$ cd /etc/nginx/sites-available/
```
/etc/nginx/sites-available 디렉터리는 Nginx의 설정 파일들이 위치한 디렉터리이다. 최초 설치시에는 deafult라는 설정 파일만 존재한다.

그리고 파이보 서비스에 대한 Nginx의 설정파일을 다음과 같이 관리자 권한으로 작성한다.
```
(mysite) ubuntu@ip-172-26-12-247:/etc/nginx/sites-available$ sudo nano mysite
```
그리고 mysite 파일의 내용은 다음과 같이 작성하자.

[파일명: /etc/nginx/sites-available/mysite]
```
server {
        listen 80;
        server_name 3.35.153.92;

        location = /favicon.ico { access_log off; log_not_found off; }

        location /static {
                alias /home/ubuntu/projects/mysite/static;
        }

        location / {
                include proxy_params;
                proxy_pass http://unix:/tmp/gunicorn.sock;
        }
}
```
listen 80 은 웹 서버를 80 포트로 서비스 한다는 의미이다. HTTP 프로토콜의 기본포트는 80이다. 따라서 이제 http://3.35.153.92:8000/ 대신 포트를 생략하여 http://3.35.153.92 처럼 웹 브라우저에서 접속 할 수 있다.

server_name 에는 여러분의 고정IP를 등록한다.

location /static 은 정적 파일에 대한 설정으로 /static으로 시작되는 URL 요청은 Nginx가 /home/ubuntu/projects/mysite/static 디렉터리의 파일을 읽어서 처리한다는 설정이다.

location /은 location /static 에서 설정한 것 이외의 모든 요청은 Gunicorn이 처리하도록 하는 설정이다. proxy_pass 는 이전 장에서 설정했던 Gunicorn의 유닉스 소켓 경로이다.

이와 같은 설정을 통해 /static 으로 시작되는 URL은 Nginx가 처리하고 나머지 URL에 대해서는 Gunicorn이 처리하게 된다.

이제 작성한 mysite 파일을 Nginx가 환경 파일로 읽을 수 있도록 설정해야 한다.

다음처럼 /etc/nginx/sites-enabled 디렉터리로 이동하자.
```
(mysite) ubuntu@ip-172-26-12-247:/etc/nginx/sites-available$ cd /etc/nginx/sites-enabled/
(mysite) ubuntu@ip-172-26-12-247:/etc/nginx/sites-enabled$
```
sites-enabled 디렉터리는 site-available 디렉터리에 있는 설정 파일 중에서 활성화하고 싶은 것을 링크로 관리하는 디렉터리이다.

ls 명령을 수행하면 현재 default 설정 파일만 링크됨을 확인할 수 있다.
```
(mysite) ubuntu@ip-172-26-12-247:/etc/nginx/sites-enabled$ ls
```
default
이제 default 링크는 삭제하고 mysite 파일을 링크하도록 변경해야 한다.

먼저 다음처럼 default 링크를 삭제하자.
```
(mysite) ubuntu@ip-172-26-12-247:/etc/nginx/sites-enabled$ sudo rm default
```
그리고 다음처럼 mysite 파일을 링크하자.
```
(mysite) ubuntu@ip-172-26-12-247:/etc/nginx/sites-enabled$ sudo ln -s /etc/nginx/sites-available/mysite
```
ls 명령을 수행하면 default는 사라지고 mysite 링크만 남은 것을 확인할 수 있다.
```
(mysite) ubuntu@ip-172-26-12-247:/etc/nginx/sites-enabled$ ls
mysite
```

## Nginx 실행
Nginx는 설치할 때 자동으로 실행되므로 앞에서 작성한 Nginx 설정을 적용하려면 Nginx를 다음처럼 다시 시작해야 한다.
```
(mysite) ubuntu@ip-172-26-12-247:/etc/nginx/sites-enabled$ sudo systemctl restart nginx
```

---

**혹시 Nginx 설정 파일에 오류가 발생했다면?**

Nginx의 설정 파일에 오류가 있는지 확인하는 방법은 다음과 같다.
```
(mysite) ubuntu@ip-172-26-12-247:/etc/nginx/sites-enabled$ sudo nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```
nginx -t 명령 수행 시 오류가 발생한다면 설정이 올바르지 않은 경우이므로 Nginx서버가 정상적으로 실행되지 않을 것이다.

Nginx를 중지하는 명령은 다음과 같다.
```
(mysite) ubuntu@ip-172-26-12-247:/etc/nginx/sites-enabled$ sudo systemctl stop nginx
```
Nginx를 시작하는 명령은 다음과 같다.
```
(mysite) ubuntu@ip-172-26-12-247:/etc/nginx/sites-enabled$ sudo systemctl start nginx
```

---

## 파이보 작동 확인하기
Gunicorn과 Nginx를 모두 적용했으니 웹 브라우저로 다음 URL에 접속해 보자.

> http://3.35.153.92/ (여러분의 고정아이피를 사용하도록 하자.)

웹서버(Nginx)를 사용하기 때문에 이전과 달리 :8000과 같은 포트번호를 사용할 필요가 없어졌다. Nginx를 사용하면 HTTP 기본 포트인 80 포트를 사용할 수 있기 때문이다. 80 포트를 사용할 경우에는 http://3.35.153.92:80 처럼 사용해도 되지만 :80 을 생략할 수 있다.

다음처럼 완벽하게 실행되는 파이보를 확인할 수 있을 것이다.

![4-10_1.png](https://github.com/hyeonDD/jump_to_django/blob/main/4_Part/4_10_Part/4-10_1.png)
