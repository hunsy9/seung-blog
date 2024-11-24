---
cover: ../../.gitbook/assets/leetcode.png
coverY: 0
layout:
  cover:
    visible: true
    size: hero
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# \[LeetCode] Counting Elements

**문제 설명**: Given an integer array `arr`, count how many elements `x` there are, such that `x + 1` is also in `arr`. If there are duplicates in `arr`, count them separately.

**제약 사항**:

* `1 <= arr.length <= 1000`
* `0 <= arr[i] <= 1000`

**난이도**: `EASY`



## 문제를 보며 처음 한 생각

***

```python
def countElements(self, arr: List[int]) -> int:
    count = 0
    for x in arr:
        if x + 1 in arr:
            count += 1
    return count
```

처음 봤을 땐 단순히 in 메서드를 이용해서 풀면 바로 풀릴거라고 생각했다. 물론 이렇게 간단한 방법으로도 풀 수 있지만, in 메서드는 주어진 arr를 한번 순회하므로, arr의 length가 N이라고 했을 때 **시간 복잡도는 O(N^2)**&#xAC00; 걸리게 된다.

그래서 투 포인터를 이용해서 시간복잡도를 개선하였다.

## 문제를 해결한 방법

***

```python
def countElements(self, arr: List[int]) -> int:
        answer = 0
        arr.sort()
        i=0 # slow pointer
        j=1 # fast pointer
        while j < len(arr):
            if arr[j] - arr[i] == 0:
                j += 1
            else:
                if arr[j]-arr[i] == 1:
                    answer += 1
                i+=1
                          
        return answer
```

문제는 특이하게도 x에게 x+1이 있는지 찾는 문제이기 때문에, 두 개의 Slow, Fast 포인터를 두고,

1. 두 포인터의 arr 값이 같으면 Fast Pointer + 1
2. **두 포인터의 arr 값의 차이가 1(우리가 찾는 값)이라면**, answer + 1, Slow Pointer +1
3. 두 포인터의 arr 값의 차이가 0도, 1도 아니라면, 단순히 Slow Pointer

이 방식으로 문제를 해결하면, x 와 x + 1이 arr안에 각각 어떤 위치에 존재하던지 간에, x+1을 가진 x의 개수를 셀 수 있다.

그리고 _If there are duplicates in `arr`, count them separately. 이 조건 또한 만족시킬 수 있다._

또한 while문을 통한 순회 시, 각각의 원소에 포인터가 머무를 수 있는 횟수는 최대 2번이기에, 최악의 경우더라도 **시간복잡도 O(2N)**&#xB85C; 끝낼 수 있다.
