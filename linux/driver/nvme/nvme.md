# NVMe란 무엇일까?

## NVMe란?

***

* **호스트 소프트웨어**가 **비휘발성 메모리 서브시스템과 통신**할 수 있도록 하는 **레지스터 레벨 인터페이스**
* 주로 기업용 또는 Client용 SSD에 최적화되어있음
* **PCI Express 인터페이스**에 주로 연결됨
* 최적화된 submission and completion paths
* 병렬 I/O Queue(큐 당 64K개의 명령) 에 대한 연산 지원
* 최대 65,535개의 I/O 큐를 지원

## NVMe 규격

***

* 비휘발성 메모리 서브시스템과의 통신을 위한 레지스터 인터페이스 정의
* NVM 하위 시스템과 함께 사용하기 위한 표준 명령 세트도 정의

### 인터페이스 주요 특성

* 다중 경로 I/O 및 네임스페이스 공유를 지원
* Command Submit 또는 Complete 시 캐시 불가능
* 명령 제출 경로에는 최대 1개의 MMIO 레지스터 쓰기가 필요
  * MMIO란?
* 효율적이고 능률적인 명령 세트

### NVMe 매커니즘

* NVMe 컨트롤러는 단일 PCI 함수와 연결
* SQ(Submission Queue), CQ(Completion Queue) 존재
* Command는 **Host SW에 의해 SQ에 배치**
* Completion은 **NVMe Controller에 의해 CQ에 배치**
* 여러 SQ는 한개의 CQ를 사용가능
* SQ 및 CQ는 호스트 메모리에 할당

#### Admin

Admin Cmd 및 관련 CQ는 컨트롤러 관리 및 제어 목적으로 존재

Admin Cmd Set 일부만 Admin SQ에 submit 가능
