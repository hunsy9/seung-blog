---
description: 여러가지 이유로 일단 Harbor를 사용해보기로 하였다.
---

# HTTPS Harbor 사설 레지스트리 구축하기

## 설치 전제 조건

***

공식 문서를 찾아보니, Harbor는 몇 가지 설치 전제 조건이 있다.

하드웨어 전제조건은 CPU코어 2개, 4GB MEM, 40GB Disk면 충분하다고하며\
소프트웨어 전제조건은 컨테이너 기반이라 그런지, 도커 엔진과 도커 컴포즈, Openssl이 설치되어 있어야 한다.

<figure><img src="../../.gitbook/assets/스크린샷 2024-08-27 오후 5.10.58.png" alt="" width="308"><figcaption><p>소프트웨어 전제조건</p></figcaption></figure>



## 설치 프로그램 다운로드

***

설치 프로그램은 온라인 설치 프로그램(도커 허브에서 하버 이미지를 다운로드한다.)과, 오프라인 설치프로그램이 있다.

네트워크에 연결된 상태이므로 온라인 설치 프로그램을 다운받았다.

```sh
sudo wget https://github.com/goharbor/harbor/releases/download/v2.11.1/harbor-online-installer-v2.11.1.tgz
```

tar 파일을 압축해제한다.

```sh
tar -xzvf harbor-online-installer-v2.11.1.tgz
```



## HTTPS 구성

***

#### HTTPS 인증서 발급

https 연결을 위해 CA에서 발급한 인증서가 필요하다. openssl을 통해 자체 서명 인증서를 발급할 수도 있으며,

나는 기존에 letsencrypt에서 발급한 ssl 인증서를 사용하였다.

운영환경에선 letsencrypt와 같은 제 3자 CA로부터 인증서를 받는 것을 권장한다.

```bash
sudo ls /etc/letsencrypt/live/[발급받은 도메인 주소]
```

certbot을 이용해 인증서를 발급받았다면 아래와 같은 네 가지 키가 존재할 것이다.

1. **cert.pem**&#x20;
2. **chain.pem**&#x20;
3. **fullchain.pem**&#x20;
4. **privkey.pem**

서버의 공개 인증서인 **cert.pem**, 서버의 개인 키인 **privkey.pem**을 harbor에 등록할 것이다.



#### Harbor 구성에 인증서 설정

```bash
cd ./harbor
```

압축을 푼 디렉토리에 들어가면, harbor.yml.tmpl이 존재한다.

이는 harbor의 구성파일(docker-compose.yml)을 만들기 위한 사전 구성 파일이다.

```bash
cp harbor.yml.tmpl harbor.yml
vi harbor.yml
```

harbor.yml의 내용을 봐보자

```bash
# Configuration file of Harbor

# The IP address or hostname to access admin UI and registry service.
# DO NOT use localhost or 127.0.0.1, because Harbor needs to be accessed by external clients.
hostname: [레지스트리 도메인 주소]

# http related config
http:
  # port for http, default is 80. If https enabled, this port will redirect to https port
  port: [http 포트]

# https related config
https:
  # https port for harbor, default is 443
  port: [https 포트]
  # The path of cert and key files for nginx
  certificate: /etc/letsencrypt/live/[도메인 주소]/cert.pem
  private_key: /etc/letsencrypt/live/[도메인 주소]/privkey.pem
  # enable strong ssl ciphers (default: false)
  # strong_ssl_ciphers: false
```

파일의 위쪽 내용을 보면 `hostname` 에 인증서를 발급받은 도메인 주소,&#x20;

http port에 커스텀할 http 포트 주소, https port에 https 포트 주소를 넣어준다.

마지막으로 서버 공개 인증서키의 경로와 서버 개인 키의 경로를 입력해주면 된다.

```bash
./prepare
sudo./install.sh
```

이제 prepare 스크립트로 readonly docker compose.yml 구성파일을 만들고,&#x20;

install 스크립트를 실행시키면 자동으로 서버 내에 배포된다.



## 번외: 리버스 프록시 구성

***

사실 이 부분 때문에 애를 먹었다.

나는 Harbor 앞에 nginx를 리버스 프록시로 두고 있다.

Harbor를 443포트로 두고 https로 접속해서 사용하지 않고, 업스트림 서버로 두고 프록시를 거쳐 갈것이라면,

```bash
# Uncomment external_url if you want to enable external proxy
# And when it enabled the hostname will no longer used
external_url: [리버스 프록시 서버 네임]
```

harbor.yml 구성에 external\_url에 본인의 프록시 서버 네임을 반드시 입력해야 사설 레지스트리에 정상적으로 로그인이 가능하다.

<figure><img src="../../.gitbook/assets/스크린샷 2024-08-27 오후 11.19.12.png" alt=""><figcaption><p>구축된 사설 레지스트리</p></figcaption></figure>



글은 짧았지만 공식문서랑 여러가지 레퍼런스를 뒤지면서 구성을 하느라 4시간은 족히 쓴 것 같다..

그래도 굉장히 뿌듯하다.



이제 pull rate 제한이 없는 나만의 레지스트리를 가지고 놀아보겠다. ㅎㅎ!!
