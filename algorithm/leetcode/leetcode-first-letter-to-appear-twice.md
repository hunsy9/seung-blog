# \[LeetCode] First Letter to Appear Twice

**문제 설명**: Given a string `s` consisting of lowercase English letters, return the first letter to appear twice.

**참고사항**:

* A letter `a` appears twice before another letter `b` if the second occurrence of `a` is before the second occurrence of `b`.
* `s` will contain at least one letter that appears twice.

**난이도**: `EASY`



### 문제를 보며 처음 한 생각 <a href="#undefined" id="undefined"></a>

***

* 문자열을 앞에서부터 뒤로 훑으면서 스택(파이썬 리스트)에 넣고, 스택의 head를 확인
* 스택의 head가 처음으로 지금 탐색 중인 character와 같다면, 지금 탐색 중인 character를 바로 리턴 후 종료

위 두 가지 과정을 거쳐 해결하려고 하였다.\


### 문제를 해결한 방법 <a href="#undefined" id="undefined"></a>

***

“문제를 보며 처음 한 생각”을 기반으로

```python
class Solution:
    def repeatedCharacter(self, s: str) -> str:
        stack = [s[0]]
        for i in range(1, len(s)):
            if stack[-1] == s[i]:
                return s[i]
            stack.append(s[i])
```

위와 같은 코드를 짰다.

Run 했을 때 예시 테스트 케이스는 통과하였지만, \


<figure><img src="https://velog.velcdn.com/images/juniper0917/post/7f81312c-f576-4645-8f5d-ac42cd028ece/image.png" alt="" width="375"><figcaption></figcaption></figure>

이처럼 “nwcn”과 같은 문자열은 통과하지 못하였다.

생각을 해보니, 문제 description을 제대로 이해하지 못하고 있었다.

무조건 처음으로 붙어있는 문자를 찾는 것이 아니라, 붙어있지 않더라도, 탐색 중 처음으로 중복되는 문자열이 발견되면 return 하라는 문제인 것이다.

아무래도 영어가 미숙하다보니, 이렇게 문제 이해를 하는데도 오류를 많이 범한다.

스택은 팰린드롬 문제와 같은 문자의 패턴이 일정하거나, 순서에 규칙이 있거나 하는 경우에 쓰일 수 있다.

하지만 이 문제는 언제 어떤 문자열이 중복되서 나올지 모른다.

따라서 딕셔너리를 사용하는 것이 합리적일 것이다.

어쨋든 그럼 다시 문제 해결 과정을 짜면

* 딕셔너리에 Key: (탐색 중인 Character), Value에는 쓰레기 값을 넣어준다.
* in 키워드를 이용하여 이미 키 값이 있는지 확인하고, 있다면 바로 키 값과 함께 바로 리턴한다.

```python
class Solution:
    def repeatedCharacter(self, s: str) -> str:
        dic = {}
        for i in range(len(s)):
            if s[i] in dic:
                return s[i]
            dic[s[i]] = -1
```

### 문제 해결에 필요했던 지식 <a href="#undefined" id="undefined"></a>

***

in 키워드를 사용하는 것이라면 딕셔너리를 사용할 것이 아니라, 리스트를 써도 된다.

하지만 딕셔너리를 사용한 이유는 in 키워드를 사용할 때, 리스트라면 리스트의 모든 요소를 전부 확인하여 O(n)의 시간복잡도가 걸린다.

하지만 딕셔너리는 파이썬 내부적으로 해시 테이블을 이용하여 구현되어 있어, 평균적으로 키를 검색하는데 O(1)의 시간복잡도를 가진다.

최악의 경우(모든 키가 동일한 해시 값을 가지는 경우)에는 O(n)의 시간복잡도를 가질 수 있지만 보통 그렇진 않다.
