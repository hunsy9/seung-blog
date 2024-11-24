---
description: 배열과 문자열, 불변/가변 타입을 알아본다
---

# 배열과 문자열

## 배열(Array)은 무엇일까?

***

알고리즘 문제 측면에선 1차원 배열과 문자열의 의미는 매우 유사하다. 둘 다 **정렬된 요소들의 그룹**을 나타낸다.

하지만 언어 측면에선 배열의 의미는 모두 다르다.

예를 들어, **Python의 경우**는 매우 추상적인 배열(Array) 대신 **리스트(List)**&#xB97C; 사용하며, 들어가는 데이터 타입이나 리스트 크기에 대해 신경쓰지 않아도 된다.

하지만 C++과 같은 언어의 경우, 초기화 중에 배열의 크기와 데이터 타입을 지정해야 한다.

~~물론 std::vector를 사용하면 리스트를 사용할 수 있다.~~

> 배열은 크기를 조정할 수 없다. 동적 배열 또는 리스트는 조정이 가능하다.
>
> 알고리즘 문제에서의 배열의 의미는 일반적으로 동적 배열을 의미한다.

## 문자열(String)의 언어별 구현은 어떻게 될까?

***

문자열도 배열과 같이 언어별로 다르게 구현된다.

Python, Java에선 문자열은 **Immutable** 타입이며, C++에선 **Mutable** 타입이다.

## 왜 Immutable, Mutable이 중요하지?

***

만약 아래와 같은 동적 배열과, Immutable한 문자열이 있다고 치자(Python)

```python
arr = ["a", "b", "c"] # 동적 배열(리스트)
s = "abc" # 문자열
```

동적 배열에 새로운 문자를 추가하거나 변경하고 싶다면, 아래와 같이 변경이 가능하며, 이의 시간복잡도는 O(1)이 된다.

```python
arr.append("d")
# 또는
arr[2] = "d"
```

하지만, s="abc"라는 문자열을 내부 요소를 변경하거나, 뒤에 문자를 추가하고 싶다면, 아래와 같은 코드를 쓰면 될까?

```python
s[2] = "d"
```

아니다. **TypeError: 'str' object does not support item assignment** 이러한 오류가 날 것이다.

그 이유는 Python 에서 문자열(str)은 Immutable 타입으로 설정되기 때문이며,  문자열을 내부 요소를 변경하거나, 뒤에 문자를 추가하고 싶다면, 아예 새로운 문자열을 생성해야 한다.

따라서 문자열과 관련된 연산의 시간복잡도는 O(n)이 되며, 여기서 n은 문자열의 길이이다.

아래는 리스트와 문자열의 각종 연산 시간복잡도를 표로 나타낸 것이다.

<table><thead><tr><th width="168">연산</th><th>배열 또는 리스트</th><th>문자열(불변)</th></tr></thead><tbody><tr><td>Append</td><td>O(1)</td><td>O(n)</td></tr><tr><td>Pop</td><td>O(1)</td><td>O(n)</td></tr><tr><td>Insert</td><td>O(n)</td><td>O(n)</td></tr><tr><td>Delete</td><td>O(n)</td><td>O(n)</td></tr><tr><td>Modify</td><td>O(1)</td><td>O(n)</td></tr><tr><td>Random Access</td><td>O(1)</td><td>O(1)</td></tr></tbody></table>

