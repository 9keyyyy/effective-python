# 38 간단한 인터페이스의 경우 클래스 대신 함수를 받아라
## **TL;DR**

- 파이썬에서는 함수가 일급 시민 객체이기 때문에 특정 인터페이스를 만족시키기 위해 함수를 인자로 전달할 수 있음
  - 여러 컴포넌트 사이 간단한 인터페이스가 필요할 경우 간단한 함수 사용 가능
- 도우미 클래스와 `__call__` 특별 메서드를 통해 클래스 인스턴스 객체를 일반 파이썬 함수처럼 사용하며 인자로 전달하면 코드의 가독성, 전달성이 좋아짐

### 훅(hook)

- 파이썬에서 API가 실행되는 과정에서 전달한 함수가 실행되는 경우
- 아래의 경우에는 sort를 실행하는 과정에서 전달한 함수 len을 실행
  ```python
  names = ['소크라테스', '아르키메데스', '플라톤', '아리스토텔레스']
  names.sort(key=len)
  # 길이에 따른 정렬
  
  ```
- **java, C계열 언어 :** 훅을 추상클래스를 통해 정의 해야함
- **파이썬 :** 함수를 **일급 시민 객체(first-class-citizen)**로 취급하기 때문에 인자 반환값이 잘 정의된 함수라면 훅으로 사용할 수 있음
    - **일급 시민 객체(first-class-citizen) :** 다른 타입의 값과 변수에 할당 될 수 있고, 인자, 반환값으로도 사용될 수 있음

## 예시: `defaultdict` 클래스의 동작을 사용자 정의
key가 없을 때 기본값을 설정함과 동시에 키가 추가되었다는 로그를 남기도록 하려는 경우

### 1. 함수를 외부에 정의하여 인자로 사용

```python
def log_missing():
    print('키 추가됨')
    return 0

from collections import defaultdict

current = {'초록': 12, '파랑': 3}
increments = [
    ('빨강', 5),
    ('파랑', 17),
    ('주황', 9),
]

result = defaultdict(log_missing, current)
print('이전:', dict(result))
for key, amount in increments:
    result[key] += amount
print('이후:', dict(result))

>>
이전: {'초록': 12, '파랑': 3}
키 추가됨
키 추가됨
이후: {'초록': 12, '파랑': 20, '빨강': 5, '주황': 9}

```

- `defaultdict`의 첫번째 인자는 factory 함수라고도 불리며 키가 없을 때 자동으로 호출
    - **factory함수 :** 객체를 생성하고 반환하는 함수
    - 주로 사용하는 `defaultdict(list)`, `defaultdict(int)`같은 경우도 같은 방식으로 해당 타입의 기본값을 생성하는 경우
- 두번째 인자는 초기값으로 사용
    - 위 예시의 경우에는 `current`라는 dict가 초기값

→ log_missing과 같은 함수를 사용해서 정해진 동작(기본값)과 부수효과(로깅)를 분리할 수 있음 → API 만들기 용이

### 2. 추상 클래스를 정의하는 경우 (BAD)

```python
from collections.abc import MutableMapping

class LoggingDict(MutableMapping):
    def __init__(self, original=None):
        self.store = dict()
        if original is not None:
            self.update(original)  # 기존 딕셔너리로부터 값 복사

    def __getitem__(self, key):
        try:
            return self.store[key]
        except KeyError:
            self.store[key] = 0
            print('키 추가됨')
            return 0

    def __setitem__(self, key, value):
        self.store[key] = value

    def __delitem__(self, key):
        del self.store[key]

    def __iter__(self):
        return iter(self.store)

    def __len__(self):
        return len(self.store)

current = {'초록': 12, '파랑': 3}
increments = [
    ('빨강', 5),
    ('파랑', 17),
    ('주황', 9),
]

result = LoggingDict(current)
print('이전:', dict(result))
for key, amount in increments:
    result[key] += amount
print('이후:', dict(result))

>>
이전: {'초록': 12, '파랑': 3}
키 추가됨
키 추가됨
이후: {'초록': 12, '파랑': 20, '빨강': 5, '주황': 9}

```

- `increment_with_report`라는 함수가 실행될 때 `defaultdict`에서 디폴트값이 사용되는 횟수를 count하고 싶을 때

### 3. 훅으로 클로저 함수를 사용

```python
def increment_with_report(current, increments):
    added_count = 0

    def missing():
        nonlocal added_count  # 상태가 있는 클로저
        added_count += 1
        return 0

    result = defaultdict(missing, current)
    for key, amount in increments:
        result[key] += amount

    return result, added_count

result, count = increment_with_report(current, increments)
assert count == 2

```

- 가독성이 떨어질 수 있음
- 앞서 든 예시와 같이 추상클래스를 정의하지않고 함수를 정의하여 디폴트값을 가져올 때의 동작을 임의로 수정한 것
- 훅으로 클로저를 사용하면 “상태가 있는” 함수이기 때문에 이해하기 어려움

### 4. 추척하기 쉽도록 작은 클래스를 하나 정의 (Better way)

```python
class CountMissing:
    def __init__(self):
        self.added = 0

    def missing(self):
        self.added += 1
        return 0

counter = CountMissing()
result = defaultdict(counter.missing, current)  # 메서드 참조
for key, amount in increments:
    result[key] += amount
assert counter.added == 2

```

- 추가 도우미 클래스를 하나 정의한뒤 내부적으로 변수를 관리하고 매서드를 통해 함수를 인자로 전달
- 하지만 클래스 자체만 놓고보면 역할을 잘알기 어려움 (defaultdict에 counter.missing이 들어가는 것을 봐야함)

### 5. __call__을 통해 callable 객체로 만들고 인스턴스를 함수로 사용 (Best way)
```python
class BetterCountMissing:
    def __init__(self):
        self.added = 0

    def __call__(self):
        self.added += 1
        return 0

counter = BetterCountMissing()
assert counter() == 0
assert callable(counter)

counter = BetterCountMissing()
result = defaultdict(counter, current)  # __call__에 의존
for key, amount in increments:
    result[key] += amount
assert counter.added == 2

```
