---
description: 가용할 수 있는 물리 인프라 자원은 워크스테이션 1대 뿐이었습니다.
---

# E2EE(종단 간 암호화)를 달성하기 위한 고민과 해결 과정

### 문제의 배경

제가 사용 가능한 물리적 자원은 **교내 워크스테이션 1대**였습니다.

<figure><img src="../.gitbook/assets/스크린샷 2024-10-02 오전 1.51.49.png" alt="" width="375"><figcaption><p>환경 별로 다른 도메인</p></figcaption></figure>

따라서, 단일 호스트에 운영 및 테스트 환경 구축이 필요하였으며, &#x20;

<figure><img src="../.gitbook/assets/스크린샷 2024-10-02 오전 1.54.49.png" alt="" width="295"><figcaption><p>각 서비스 환경으로 리버스 프록시</p></figcaption></figure>

위처럼 80/443 포트에 바인딩 해놓은 인그레스 Nginx에서 들어오는 트래픽을&#x20;

2개의 가상 호스트의 서비스들로 **리버스 프록싱하는 작업이 필요**하였습니다.



### 단순한 해결방법의 시도

***

초기엔 팀이 개발한 프론트엔드 컨테이너를 서비스 명만 바꾸어(ex. code-place-test, code-place-prod),

서로 다른 두 포트를 할당해 서비스를 생성하고, 인그레스 트래픽을 각 업스트림 서비스 컨테이너로 분배하려고 하였습니다.



### 또 다른 문제의 가능성

***

하지만 이렇게 구현할 시,&#x20;

<figure><img src="../.gitbook/assets/스크린샷 2024-10-02 오전 2.03.27.png" alt="" width="339"><figcaption><p>서버 내 백도어라도 있어서 패킷이 탈취된다면? Oh-no..</p></figcaption></figure>

**https(443)로 들어온 트래픽을 http로 변환**하여 각 업스트림 서버에 전달해야 한다는 문제가 있었고,\
내/외부 통신 모두에 https를 사용하지 못한다면 **보안적 취약점이 발생할 수 있다고 생각**하였습니다.



그래서 각 업스트림 서버 또한 https를 적용하여 **End-To-End 암호화**를 달성하고자 했습니다.



<figure><img src="../.gitbook/assets/스크린샷 2024-10-02 오전 2.09.25.png" alt="" width="563"><figcaption><p>서비스 내부 아키텍처</p></figcaption></figure>

End-To-End 암호화를 달성하기 위해선

위 **서비스 아키텍처의 게이트웨이 역할을 하는 Nginx에도 SSL 설정**을 해주어야 했습니다.

문제는 이 서비스를 2개를 생성해서 하나는 운영 환경, 나머지 하나는 테스트 환경으로 써야했기에,

두 개의 nginx.conf가 필요한 상황이었습니다.

```nginx
# nginx.conf - 운영
server {
      listen 1443 ssl default_server;
      server_name code.pusan.ac.kr;
      http2 on;
      autoindex_localtime on;

      include ssl_config.conf;
      include https_locations.conf;
}

# nginx.conf - 테스트
server {
      listen 1443 ssl default_server;
      server_name copl-dev.site;
      http2 on;
      autoindex_localtime on;

      include ssl_config.conf;
      include https_locations.conf;
}
```

위처럼 nginx 구성파일을 각각 따로 생성해서 넣어주면 되겠지만, 코드의 중복이 많이 일어날 것이라 생각했습니다.

따라서 server\_name의 변수화가 필요한 시점이었습니다.

### 문제 해결 방법

***

따라서 **server\_name 디렉티브의 value값을 변수화**시키는 방법을 알아보던 중,&#x20;

\
“프론트엔드 컨테이너 nginx의 구성에 도메인 명을 변수화해두고, 도메인 이름을 **동적으로 변경**하면 어떨까?”&#x20;

라는 생각을 하게되었습니다.

\
위 생각을 실현하기 위해 먼저 프론트엔드 nginx 구성파일부터 개선해보았습니다.

1\. 프론트엔드 nginx 구성파일에서, **server\_name 디렉티브를 \_\_SERVER\_NAME\_\_으로 설정**(변수화)

```nginx
# nginx.conf
server {
      listen 1443 ssl default_server;
      server_name __SERVER_NAME__; # server_name 변수화
      http2 on;
      
      ssl_certificate /etc/letsencrypt/live/__SERVER_NAME__/fullchain.pem;
      ssl_certificate_key /etc/letsencrypt/live/__SERVER_NAME__/privkey.pem;
      ...
      include https_locations.conf;
}
```

또한 서로 다른 도메인을 볼륨으로 넣어주기 위해,

운영/테스트 환경(YAML)에 각각 운영/테스트용 SSL 인증서 경로를 볼륨으로 설정해주었습니다.

<pre class="language-yaml" data-title="docker-compose.yml(test,prod))" data-line-numbers><code class="lang-yaml"><strong># production 환경
</strong>frontend:
    image: registry.copl-dev.site/code-place-prod/frontend:latest
    ...
    volumes:
      - /etc/letsencrypt/live/운영도메인/인증서파일:/etc/letsencrypt/live/운영도메인/인증서파일:ro
    
# test 환경  
frontend:
    image: registry.copl-dev.site/code-place-dev/frontend:latest
    ...
    volumes:
            - /etc/letsencrypt/live/테스트도메인/인증서파일:/etc/letsencrypt/live/테스트도메인/인증서파일:ro
</code></pre>

\
이젠 프론트엔드 이미지를 빌드할 때, 이미지가 사용될 가상호스트 도메인을 지정해주는 작업이 필요하였습니다.

당시 프론트엔드 Dockerfile은 멀티스테이지 빌드를 사용 중이었습니다.\
**첫 번째 스테이지:** npm build 스테이지

**두 번째 스테이지:** nginx로 빌드된 정적파일을 복사하고 실행하는 스테이지

nginx.conf를 구성하는 과정은 두 번째 스테이지에서 이루어졌기에,&#x20;

Dockerfile의 전역 빌드 인수를 SERVER\_NAME으로 설정하고 이를 두번째 스테이지에서 환경변수로 주입하였습니다.

<pre class="language-docker" data-title="Dockerfile(frontend)"><code class="lang-docker"><strong># 전역 빌드 인수
</strong>ARG SERVER_NAME

# 빌드 스테이지
FROM node:16 AS build-stage

...생략

# 프로덕션 스테이지
FROM nginx:stable-alpine AS production-stage

ARG SERVER_NAME # 다음 스테이지에서 전역 빌드 인수 사용
ENV SERVER_NAME=$SERVER_NAME # 전역 빌드 인수를 환경 변수로 넣어줌

...생략
</code></pre>

남은 작업은 초반에 변수화 시켜놓은 nginx.conf의 \_\_SERVER\_NAME\_\_을, 가상 호스트 도메인의 환경변수로 바꿔주는 작업이었습니다.

{% code title="Dockerfile" %}
```docker
ENTRYPOINT ["/app/deploy/entrypoint.sh"] # entrypoint 스크립트 실행
```
{% endcode %}

이는 컨테이너 실행 시  쉘스크립트에서 이루어질 작업이었기 때문에\


{% code title="entrypoint.sh" %}
```sh
#!/bin/sh

APP=/app
DIST=/usr/share/nginx/html

...

echo "Using SERVER_NAME: $SERVER_NAME"

# 동적으로 server_name 변경
sed -i "s/__SERVER_NAME__/$SERVER_NAME/g" /app/deploy/nginx/nginx.conf
```
{% endcode %}

리눅스 명령어 중에 문자열을 교체하는 명령인 sed를 찾게 되었고, 위 과정처럼 동적으로 server\_name을 변경할 수 있었습니다.

> `sed`: 스트림 편집기(Stream Editor)의 약자로, 텍스트 파일을 수정하는 데 사용되는 Unix 유틸리티
>
> `"s/`**`__SERVER_NAME__`**`/$SERVER_NAME/g"`
>
> `__SERVER_NAME__`: 찾을 패턴
>
> `$SERVER_NAME`: 치환할 내용

### 문제 해결 결과

위 과정을 통해 인그레스 트래픽을 암호화된 상태로 리버스 프록싱 할 수 있게 되었고, End to End 암호화를 달성할 수 있었습니다.

```nginx
server {
        listen 443 ssl;
        server_name code.pusan.ac.kr;
        ...
        location / {
                proxy_pass                          https://prod-service; # https 프록시
                proxy_set_header  Host              $http_host;
                proxy_set_header  X-Real-IP         $remote_addr;
                proxy_set_header  X-Forwarded-For   $proxy_add_x_forwarded_for;
                proxy_set_header  X-Forwarded-Proto $scheme;
        }
        ...
```
