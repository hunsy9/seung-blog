---
description: 변경사항이 감지된 컴포넌트만 조건부로 빌드하고 싶다.
---

# 더 나은 CI 파이프라인을 위한 고민과 해결 과정

### 문제의 배경

저희 팀은 개발 초기엔 프론트엔드, 백엔드를 각각 레포지토리를 따로 만들어 개발 중이었습니다.

관리의 불편함을 느끼고 이를 **모노레포로 전환**한 경험이 있습니다.

따라서 지금 현재 프로젝트의 구조는 아래와 같은데요.

```sh
├── frontend # 프론트엔드 디렉토리
├── backend # Django API 서버 디렉토리
├── scheduler # Django APScheduler 디렉토리
├── deployment # 배포 환경 구성 관련 Yaml 파일 디렉토리
└── scripts # 배포와 관련된 각종 스크립트 디렉토리
```

각각 별도의 레포지토리로 관리할땐, 커밋이 push 되었을때 각각 CI 파이프라인을 실행하면 문제가 없었죠.

하지만 모노레포는 커밋이 어떤 디렉토리의 변경사항을 반영하는지 바로 알 수가 없기 때문에,

CI 파이프라인을 짤 때, 커밋을 분류하여 조건부로 CI 파이프라인을 실행시키는 것이 중요하였습니다.

### 추상적으로 생각한 해결 방법

***

<figure><img src="../../.gitbook/assets/스크린샷 2024-10-02 오후 4.33.06.png" alt=""><figcaption><p>머리 속의 사고 과정</p></figcaption></figure>

"아, 그러면 커밋이 어떤 디렉토리의 변경사항인지만 파악하면, 원하는 컴포넌트만 빌드할 수 있겠다!"

라는 생각이 들었고,&#x20;

커스텀 Action을 만들만한 실력은 안됬기에.. Marketplace에서 이를 지원하는 Action이 없는지 찾아보았습니다.

### Path-filter Action의 발견

***

<figure><img src="../../.gitbook/assets/스크린샷 2024-10-02 오후 4.39.37.png" alt="" width="563"><figcaption><p>고마운 Dorny 씨의 path-filter</p></figcaption></figure>

검색 중 dorny/path-filter action을 발견하였고, Star가 굉장히 많은 것을 보아 검증된 액션인 것을 확인하였습니다

```yaml
- uses: dorny/paths-filter@v3
  id: changes
  with:
    filters: |
      src:
        - 'src/**'

  # run only if some file in 'src' folder was changed
- if: steps.changes.outputs.src == 'true'
  run: ...
```

사용법은 굉장히 간단하였는데요.

위처럼 step의 with에 변경을 감지할 경로만 지정해주면, 경로 내 커밋이 감지 여부를 boolean 형식으로 출력해줍니다.



저는 **세 가지 디렉토리에 대해서 커밋을 감지**해야 하기에, 조금 응용해서 아래와 같은 형태로 만들어주었습니다!

{% code title="ci2production-and-release.yml" lineNumbers="true" %}
```yaml
detect-changes-by-component: # 변경사항 감지 Job
    ...
    outputs: # 파이프라인의 결과를 true/false로 출력한다.
      backend: ${{ steps.filter.outputs.backend }}
      scheduler: ${{ steps.filter.outputs.scheduler }}
      frontend: ${{ steps.filter.outputs.frontend }}
    steps:
      ...
      - uses: dorny/paths-filter@v3
        id: filter
        with:
          base: 'main'
          filters: | # 디렉토리 필터 설정
            backend:
              - 'backend/**'
            scheduler:
              - 'scheduler/**'
            frontend:
              - 'frontend/**'
              
ci-backend:
...
ci-frontend:
...
```
{% endcode %}

### Release Job에서 발생할 수 있는 문제

***

저희 팀은 **Git Flow**를 적용하여 **develop 브랜치를 분기한 release 브랜치**를 만들고,&#x20;

이를 **main에 병합 시 CI 파이프라인과, 릴리스가 자동으로 실행**되는 게 목표였습니다.



아래 코드를 보면, create-release라는 새로운 job을 만들어 각 컴포넌트별 CI 파이프라인에 의존하도록 하였는데요.

```yaml
create-release:
    needs: [ci-frontend, ci-backend, ci-scheduler]
```

여기서 심각한 문제가 있었습니다.

컴포넌트별 변경사항을 감지하기에, 변경이 감지되지 않는 컴포넌트의 경우는 skip됩니다.

하지만 github action의 `needs`에 나열된 모든 job이 성공적으로 완료되어야만, 해당 job이 시작하는 특성이 있었습니다.

즉, `needs`는 AND로 동작하여 사용하기가 까다로웠습니다.



따라서 위 코드를 사용할 경우

**모든 컴포넌트에 변경사항이 있지 않는 한,** CI 파이프라인 이후 **릴리스 액션이 Skip되는 문제의 가능성**이 있었습니다.

### always()를 이용해 문제 해결

***

위 문제를 해결하기 위해 **각 컴포넌트별 ci가 수행여부와 관계 없이 일단 release job을 실행**시켜야만 했습니다.&#x20;

&#x20;따라서 방법을 찾던 중, **이전 단계의 성패 여부와 관계 없이 항상 작업을 수행하는 조건인** `always()`를 발견했습니다.

{% code title="ci-and-release.yml" %}
```yaml
create-release:
  needs: [check-release-branch, ci-frontend, ci-backend, ci-scheduler]
  if: |
    always() &&
    needs.check-release-branch.outputs.is_release == 'true' &&
    (needs.ci-frontend.result == 'success' || needs.ci-frontend.result == 'skipped') &&
    (needs.ci-backend.result == 'success' || needs.ci-backend.result == 'skipped') &&
    (needs.ci-scheduler.result == 'success' || needs.ci-scheduler.result == 'skipped')
```
{% endcode %}

`always()`를 job의 if 내에 넣게 되면, `needs`의 충족여부에 관계없이 일단 실행되게 됩니다.

따라서 아래처럼 코드를 짜면, 이전 컴포넌트들의 CI가 실패하지만 않는다면, 릴리스 액션을 수행할 수 있었습니다.

```
(needs.ci-frontend.result == 'success' || needs.ci-frontend.result == 'skipped') &&
(needs.ci-backend.result == 'success' || needs.ci-backend.result == 'skipped') &&
(needs.ci-scheduler.result == 'success' || needs.ci-scheduler.result == 'skipped')
```

### 결과 및 느낀 점

***

path-filter를 활용한 조건부 빌드에 성공하고, `always()`를 통해 릴리스 액션까지 잘 실행되는 것을 확인할 수 있습니다.

<figure><img src="../../.gitbook/assets/스크린샷 2024-10-02 오후 4.01.21.png" alt="" width="375"><figcaption><p>조건부 빌드 성공!</p></figcaption></figure>

<figure><img src="../../.gitbook/assets/스크린샷 2024-10-02 오후 3.54.09.png" alt=""><figcaption><p>Release 1.0.0 성공!</p></figcaption></figure>

고민을 해결하고 나니, 만약 모노레포가 아니었다면 위와 같은 과정들이 모두 생략될 수 있는 부분이었습니다.

\
따라서 좋은 아키텍처라고 해서 단점이 없는 것은 아니며, 역시 모든 것엔 trade off가 있다는 것을 깨달았습니다.
