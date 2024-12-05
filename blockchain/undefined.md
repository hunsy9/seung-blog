---
description: 실습실습실습
---

# 이더리움 블록체인 노드를 메인넷에 연결해보자

### 서론

***

지금까지 정보보안 수업이나 네트워크 보안 수업을 들으며 블록체인과 이더리움의 스마트 컨트랙트 개념에 대해서 배우는 시간들이 있었다.

하지만 설명으로만 들어서는 정확히 어떤 방식으로 동작하는지 이해할 수 없었다.

그래서 AWS에서 EC2를 대여하고, 이를 이더리움 노드로 만들어서 메인넷에 참여해보려고 한다.

### 본론

***

이더리움 클라이언트를 설치하는 방법은 두 가지 방법이 있다.

먼저 go-ethreum의 소스코드를 빌드하는 방법과 바이너리를 설치하는 방법이 있는데 나는 바이너리를 직접 설치하였다.

```bash
sudo apt-get install software-properties-common
sudo add-apt-repository -y ppa:ethereum/ethereum
sudo apt-get update
sudo apt-get install ethereum
```

위처럼 바이너리를 설치하고 나면 아래와 같이 이더리움 클라이언트를 실행할 수 있다.

```bash
geth --http --http.addr 0.0.0.0 --http.port <PORT>
```

하지만 위처럼 클라이언트를 실행하면 아래와 같이 비콘 클라이언트가 존재하지 않는다는 로그가 뜰 것이다.

```
WARN Post-merge network, but no beacon client seen. Please launch one to follow the chain!
```

이는 이더리움의 변화 때문인데, 이전의 Proof-of-Work(PoW) 방식에서 Proof-of-Stake(PoS) 방식으로 변화하여,\
\
네트워크 작동 방식이 달라졌기 때문이다.

> ### PoW 방식과 PoS 방식의 차이
>
> PoW는 채굴자(Miner)들이 연산 경쟁을 통해 블록을 생성하던 방식이다.
>
> 따라서 많은 전력과 하드웨어 자원이 필요하며, 네트워크의 안정성을 채굴자들의 참여에 의존하였다
>
> 한편, Pos는 검증자(Validator)가 일정량의 이더리움을 네트워크에 예치(스테이킹)하여 블록을 생성하고 검증한다.
>
> 채굴 대신 스테이킹을 통해 네트워크 참여가 이루어지며, 더 적은 전력으로 네트워크를 운영할 수 있는 장점이 있다.



PoS 방식에서는 기본적으로 `Beacon Chain Client`와 `Execution Client`가 필요하다.

Beacon Chain Client는 검증자(Validator) 관리 및 합의(Consensus) 처리를 하며 핵심 기능을 담당하고,\
Execution Client는 스마트 컨트랙트를 실행하고 트랜잭션을 처리하는 역할을 한다.



[https://docs.prylabs.network/docs/install/install-with-script](https://docs.prylabs.network/docs/install/install-with-script)

두 클라이언트를 실행하기 위해 Prysm을 사용하였으며, 예제는 위 링크를 참고하였다.

Prysm은 이더리움 2.0의 PoS 합의 알고리즘을 구현한 소프트웨어이며 Consensus Layer를 구성한다.

```
📂ethereum
┣ 📂consensus
┣ 📂execution
```

먼저 Prysm을 설치하기 전 위처럼 디렉토리 구조를 만들어주고 아래 스크립트를 consensus 디렉토리 내에서 실행한다.

```bash
curl https://raw.githubusercontent.com/prysmaticlabs/prysm/master/prysm.sh --output prysm.sh && chmod +x prysm.sh
```

그러면 prysm 클라이언트용 쉘스크립트(prysm.sh)가 다운로드된다.

consensus layer와 execution layer간 http 연결은 jwt token을 통해 인증되므로 jwt를 prysm을 통해 생성한다.

```bash
./prysm.sh beacon-chain generate-auth-secret
```

jwt가 생성되었다면 아래 명령어를 통해 execution client를 실행하고, 이후 beacon client를 실행할 수 있다.

```bash
# execution layer client 실행
geth --mainnet --http --http.api eth,net,engine,admin --authrpc.jwtsecret=<PATH_TO_JWT_FILE> 
# beacon client 실행
./prysm.sh beacon-chain --execution-endpoint=http://localhost:8551 --mainnet --jwt-secret=<PATH_TO_JWT_FILE> --checkpoint-sync-url=https://beaconstate.info --genesis-beacon-api-url=https://beaconstate.info
```

#### 실행결과

<figure><img src="../.gitbook/assets/스크린샷 2024-12-05 오후 8.40.20.png" alt=""><figcaption><p>실행된 이더리움 노드(좌: execution client, 우: beacon client)</p></figcaption></figure>

좌측은 execution 클라이언트로, synced new block...이란 로그를 보면 새로 동기화된 블록들의 해시를 알 수 있다.

또한 우측은 beacon 클라이언트로 검증자들이 최종적으로 동의한 블록과 네트워크의 현재 블록체인의 head 블록 상태를 실시간으로 확인할 수 있다.

### 결론

***

이더리움 메인넷에 스마트 컨트랙트를 배포해보면 가장 좋겠지만, 가스비로 이더리움을 지불해야 한다.\
그래서 이제부턴 이더리움 로컬 네트워크를 구성하여 비용 없이 스마트 컨트랙트를 작성하고 배포해보려고 한다.
