---
description: 변경사항이 감지된 컴포넌트만 조건부로 빌드하고 싶다.
---

# 더 나은 CI 파이프라인을 위한 고민과 해결 과정

### 문제의 배경

저희 팀은 개발 초기엔 프론트엔드, 백엔드를 각각 레포지토리를 따로 만들어 개발 중이었습니다.

관리의 불편함을 느끼고 이를 r이 있습니다.

따라서 지금 현재 프로젝트의 구조는 아래와 같은데요.

```sh
Code Place 프로젝트
├── frontend # 프론트엔드 디렉토리
├── backend # Django API 서버 디렉토리
├── scheduler # Django APScheduler 디렉토리
├── deployment # 배포 환경 구성 관련 Yaml 파일 디렉토리
└── scripts # 배포와 관련된 각종 스크립트 디렉토리
```

각각 별도의 레포지토리로 관리할땐, 커밋이 push 되었을때 각각 CI 파이프라인을 실행하면 문제가 없었죠.

하지만 모노레포는 커밋이 어떤 디렉토리의 변경사항을 반영하는지 바로 알 수가 없기 때문에,

CI 파이프라인을 짤 때, 커밋을 분류하여 조건부로 CI 파이프라인을 실행시키는 것이 중요하였습니다.

### 추상적으로 생각한 해결방법

***

<figure><img src="../.gitbook/assets/스크린샷 2024-10-02 오후 3.54.09.png" alt=""><figcaption><p>ㅊ</p></figcaption></figure>



### Path-filter Action의 발견

***

{% code title="ci2production-and-release.yml" lineNumbers="true" %}
```yaml
detect-changes-by-component:
    needs: [check-release-branch]
    if: ${{ needs.check-release-branch.outputs.is_release == 'true' }}
    runs-on: ubuntu-latest
    outputs:
      backend: ${{ steps.filter.outputs.backend }}
      scheduler: ${{ steps.filter.outputs.scheduler }}
      frontend: ${{ steps.filter.outputs.frontend }}
    steps:
      - uses: actions/checkout@v2
        with:
          fetch-depth: 0
      - uses: dorny/paths-filter@v2
        id: filter
        with:
          base: 'main'
          filters: |
            backend:
              - 'backend/**'
            scheduler:
              - 'scheduler/**'
            frontend:
              - 'frontend/**'
```
{% endcode %}

### R lJaoabs에서 발생할 수 있는 문제

***





### 문제 해결 방법

***

