# Nginx

## Nginx란?

간단하게 말하면 **트래픽이 많은** 웹사이트의 **확장성을 위해 개발된 경량 웹 서버**이다.

Nginx가 등장하기 전에는 Apache를 웹서버로 많이 사용하였지만 1999년 이후 컴퓨터 보급이 증가하면서 하나의 서버에 동시에 몰리는 커넥션이 많아졌다. 이 때 하나의 서버에 몰린 커넥션이 너무 많아 더 이상 커넥션을 형성하지 못하는 문제가 발생(=**C10K 문제**)했는데 이것을 해결하기 위해 등장했다.

Nginx는 요청에 응답하기 위해 **비동기 이벤트 기반**으로 더 **적은 자원의 사용**으로 **가벼움과 높은 성능**을 목표로 한다. 또한, Apache보다 동작이 단순하고 전달자 역할만 하기 때문에 **동시접속 처리에 특화**되어 있다. 

클라이언트로부터 요청을 받았을 때 요청에 맞는 정적 파일을 응답해주는 HTTP Web Server로 활용되기도 하고, Reverse Proxy Server로 활용하여 WAS 서버의 부하를 줄일 수 있는 로드 밸런서로 활용되기도 한다.

> 🔎 **비동기 처리 방식 vs 동기 처리 방식**
>
> - 동기(Synchronous) : A가 B에게 데이터를 요청했을 때, 이 요청에 따른 응답을 주어야만 A가 다시 작업 처리가 가능 (하나의 요청, 하나의 작업에 충실)
> - 비동기(Asynchronous) : A의 요청을 B가 즉시 주지 않아도, A의 유휴시간으로 또 다른 작업 처리가 가능한 방식

> 💡 **비동기 이벤트 기반(Event-Driven) 방식의 특징**
>
> - **요청을 하나의 Event**라 보고 **Event Handler로 관리**를 하기 때문에 **메모리의 낭비가 적다**
> - 부하에 대한 예측 가능성을 제공한다

## Reverse Proxy

리버스 프록시(Reverse Proxy)란 중계 기능을 하는 서버로, 클라이언트와 웹서버 사이에 존재하는 서버이다.

클라이언트가 서버를 호출할 때 직접 서버에 접근하는 것이 아니라 리버스 프록시 서버를 호출하게 되고, 리버스 프록시 서버가 웹서버에게 요청을 하고 응답을 받아 클라이언트에 전달한다. 

웹서버 앞에서 클라이언트 요청을 대신 받는 역할을 수행하다 보니  클라이언트는 직접적으로 실제 서버에는 통신할 수 없다는 특징을 가지게 된다. 

그렇다보니 몇가지의 큰 이점이 주어지게 된다.

### 1. 로드밸런싱

서버에 가해지는 **부하를 분산해주는 역할**로, 이용자가 많아서 발생하는 요청이 많을 때 기존 하나의 서버에서 이를 모두 처리하도록 성능을 높이는 것(`Scale up`)이 아니라 **여러대의 서버를 이용하여 요청을 처리**(`Scale out`)한다. 이때 서버의 로드율과 부하량 등을 고려하여 **적절하게 서버들에게 분산 처리하는 것**을 **로드 밸러싱**이라고 한다.

하나의 서버가 멈추더라도 서비스 중단 없이 다른 서버가 서비스를 계속 유지할 수 있는 **무중단 배포가 가능하다**는 장점이 있다.

### 2. 보안

외부 사용자로부터 내부망에 있는 서버의 존재를 숨길 수 있다. 

모든 요청은 리버스 프록시 서버에서 받기 때문에 실제 서버의 IP 주소를 필요로 하지 않으며, DDos와 같은 공격이 들어와도 Nginx를 공격할 뿐 실제 서버에는 공격이 들어온는 것을 막을 수 있다.

### 3. 캐싱

NGINX는 콘텐츠를 캐싱할 수 있어 결과를 더 빠르게 응답하여 성능을 높일 수 있다.

## Apache와의 비교

nginx와는 다르게 **프로세스 기반 접근 방식**으로 하나의 스레드가 하나의 요청을 처리하는 구조를 가진다. (환경에 따라 Prefork 방식과 Worker 방식 선택 가능)

이 구조는 클라이언트 하나당 스레드 하나를 사용하기 때문에 클라이언트가 많아질수록 계속해서 스레드가 생성되게 된다. 즉, 메모리 낭비가 심하고 문맥 교환 시 비용이 든다. 또한, 프로세스가 blocking 되면 요청을 처리하지 못하고 이전 요청을 처리하기 전까지 대기상태가 된다. 

그러나 Nginx는 비동식 구조이기 때문에 모든 클라이언트의 요청을 **병렬로 처리**한다. **싱글 프로세스**이며 event는 쓰레드가 아닌 event Handler가 처리한다. <br/>따라서 쓰레드의 사용이 적으므로 Nginx의 서버 자원 활용 능력이 더 좋다고 할 수 있다.

## Nginx 기본 환경 설정

Nginx는 환경 설정 텍스트 파일로 여러 가지 값을 지정해 Nginx 설정을 할 수 있도록 지원한다.

> Nginx의 메인 설정 파일 경로는 `/etc/nginx/nginx.conf`

```nginx
user nginx;
worker_processes 1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;

events {
    worker_connections 1024;
}
http {
    include mime.types;
    server {
        listen 80;
        location / {
            root html;
            index index.html index.htm;
        }
    }
}
```

1. Core 모듈

   코어 모듈은 대부분 환경 설정 파일의 최상단에 위치하며 한번만 사용할 수 있다. nginx의 기본적인 동작 방식을 정의한다.

   - `user` : Nginx 프로세스가 실행되는 권한
   - `worker_processes` : NGINX 프로세스 실행 가능 수

   - `pid` : NGINX 마스터 프로세스 ID 정보가 저장

2. events

   주로 네트워크 동작에 관련된 설정하는 영역으로, 비동기 이벤트 처리 방식에 대한 옵션을 설정한다.

   - `worker_connections` : 하나의 프로세스가 처리할 수 있는 커넥션의 수

3. http 블록

   웹서버에 대한 동작을 설정하는 영역으로, server 블록과 location 블록의 루트 블록이다.

4. server 블록

   하나의 웹사이트를 설정하는 데 사용된다. 가상 호스팅의 개념으로 http 블록 안에서만 사용할 수 있다.

5. location 블록

   server 블록 내에서 특정 URL을 처리하는 방법을 정의한다.

### Reverse Proxy 설정

```nginx
http {
    server {
        listen 80;
        location / {
            proxy_pass http://127.0.0.1:8081;
        }
    }
}
```

/ 경로로 요청이 들어왔을 때 (하위 경로도 포함) `proxy_pass`로 프록시 서버로 전달하도록 한다.

## 참고

- https://velog.io/@kimjiwonpg98/Nginx-%EB%A1%9C%EB%93%9C%EB%B0%B8%EB%9F%B0%EC%8B%B1-%EA%B0%9C%EB%85%90-%EB%B0%8F-%EA%B5%AC%EC%B6%95#-%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98-%EC%A2%85%EB%A5%98

- https://www.youtube.com/watch?v=6FAwAXXj5N0
- https://kanoos-stu.tistory.com/entry/Nginx
- https://prohannah.tistory.com/136