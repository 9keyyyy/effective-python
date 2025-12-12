# 15 딕셔너리 삽입 순서에 의존할 때는 조심하라
## TL;DR

- 딕셔너리 형태의 데이터에 대한 이터레이션을 수행할 때 키 삽입 순서가 유지되는지에 유의할 것
- 이터레이션 수행 시의 순서
    - **`dict`**
        - **Python 3.5 이전**: 키를 임의의 순서로 돌려줌
        - **Python 3.6**: 키를 삽입 순서대로 돌려줌
        - **Python 3.7 이후**: 키를 삽입 순서대로 돌려줌, 이 내용이 파이썬 언어 명세에 포함됨
    - **`collections.OrderedDict`**
        - 파이썬 3.7 이후 표준 `dict`의 동작과 비슷하게 동작
        - 키 삽입, `popitem` 호출을 매우 자주 처리해야 하는 경우 `dict`보다 성능상 나음
    - **딕셔너리와 유사한 객체(`collections.abc.MutableMapping` 등)**
        - 키 삽입 순서가 그대로 보존된다고 가정할 수 없음
        - 주의해서 다루기 위한 방법
            - (1) 삽입 순서 보존에 의존하지 않고 코드 작성
            - (2) 실행 시점에 명시적으로 `dict` 타입 검사
            - (3) 타입 애너테이션(annotation)과 정적 분석(static analysis)을 사용해 `dict` 값 요구

## 1. `dict`
### 이터레이션 수행 시의 순서
- **Python 3.5 이전**: 키를 임의의 순서로 돌려줌
    - `dict`가 내장 `hash` 함수 및 파이썬 인터프리터 시작시 초기화되는 난수 seed 값을 사용하는 해시 테이블 알고리즘으로 구현되었기 때문
- **Python 3.6**: 키를 삽입 순서대로 돌려줌
- **Python 3.7 이후**: 키를 삽입 순서대로 돌려줌, 이 내용이 파이썬 언어 명세에 포함됨
    - 변경 후의 작동 방식이 코드에서 항상 적용된다고 가정해도 안전

### 변경으로 인한 차이 (1): 이터레이션

- **Python 3.5 이전**: 키를 임의의 순서로 돌려줌
  ```python
  # Python 3.5
  baby_names = {
      'cat': 'kitten',
      'dog': 'puppy',
  }
  print(baby_names)
  
  >>>
  {'dog': 'puppy', 'cat': 'kitten'}
  
  ```
- **Python 3.6 이후**: 프로그래머가 생성한 순서대로 내용을 표시
  ```python
  baby_names = {
      'cat': 'kitten',
      'dog': 'puppy',
  }
  print(baby_names)
  
  >>>
  {'cat: 'kitten', 'dog': 'puppy'}
  
  ```

### 변경으로 인한 차이 (2): 딕셔너리 제공 메서드 실행 결과

- **Python 3.5 이전**: `keys`, `values`, `items`, `popitem` 등 딕셔너리가 제공하는 모든 메서드가 임의의 이터레이션 순서에 의존
  ```python
  # Python 3.5
  print(list(baby_names.keys()))
  print(list(baby_names.values()))
  print(list(baby_names.items()))
  print(list(baby_names.popitem())) # 임의로 원소를 하나 선택
  
  >>>
  ['dog', 'cat']
  ['puppy', 'kitten']
  [('dog', 'puppy'), ('cat', 'kitten')]
  ('dog', 'puppy')

  ```
- **Python 3.6 이후**:
  ```python
  print(list(baby_names.keys()))
  print(list(baby_names.values()))
  print(list(baby_names.items()))
  print(list(baby_names.popitem())) # 마지막에 삽입된 원소
  
  >>>
  ['cat', 'dog']
  ['kitten', 'puppy']
  [('cat', 'kitten'), ('dog', 'puppy')]
  ('dog', 'puppy')
  
  ```

### 변경으로 인한 차이 (3): 함수에 대한 키워드 인자(`*kwargs`) 순서

- **Python 3.5 이전**: 함수에 대한 키워드 인자가 순서가 뒤죽박죽인 것처럼 보여 함수 호출 디버깅이 어려웠음
  ```python
  # Python 3.5
  def my_func(**kwargs):
      for key, value in kwargs.items():
          print('%s = %s' % (key, value))
  
  my_func(goose='gosling', kangaroo='joey')
  
  >>>
  kangaroo = joey
  goose = gosling
  
  ```
- **Python 3.6 이후**: 키워드 인자의 순서는 프로그래머가 함수를 호출할 때 사용한 인자 순서와 항상 일치
  ```python
  def my_func(**kwargs):
      for key, value in kwargs.items():
          print(f'{key} = {value}')
  
  my_func(goose='gosling', kangaroo='joey')
  
  >>>
  goose = gosling
  kangaroo = joey
  
  ```

### 변경으로 인한 차이 (4): 클래스 인스턴스 딕셔너리 동작 방식

- **Python 3.5 이전**: `object` 필드가 난수 같이 동작
  ```python
  # Python 3.5
  class MyClass:
      def __init__(self):
          self.alligator = 'hatchling'
          self.elephant = 'calf'
  
  a = MyClass()
  for key, value in a.__dict__.items():
      print('%s = %s' % (key, value))
  
  >>>
  elephant = calf
  alligator = hatchling
  
  ```
- **Python 3.6 이후**: 키워드 인자의 순서는 프로그래머가 함수를 호출할 때 사용한 인자 순서와 항상 일치
  ```python
  class MyClass:
      def __init__(self):
          self.alligator = 'hatchling'
          self.elephant = 'calf'
  
  a = MyClass()
  for key, value in a.__dict__.items():
      print(f'{key} = {value}')
  
  >>>
  alligator = hatchling
  elephant = calf

  ```

## 2. `collections.OrderedDict`

### `OrderedDict`: `collections` 내장 모듈에 포함된 삽입 순서를 유지해 주는 클래스
- 파이썬 3.7 이후 표준 `dict`의 동작과 비슷하게 동작
- 성능 특성은 많이 다름
    - 키 삽입, `popitem` 호출을 매우 자주 처리해야 하는 경우
        - `dict`보다 `OrderedDict`가 나음(Better Way 70 참조)
        - 예: 최소 최근 사용(least-recently-used) 캐시[^1] 구현

## 3. 딕셔너리와 유사한 객체

- 딕셔너리를 처리할 때 삽입 순서 관련 동작이 항상 성립한다고 가정해서는 안 됨
    - 파이썬에서는 프로그래머가 `list`, `dict` 등의 표준 프로토콜(protocol)을 흉내 내는 커스텀 컨테이너 타입을 쉽게 정의할 수 있음
- **덕 타이핑(duck typing)**
    - 엄격한 클래스 계층보다는 객체의 동작이 객체의 실질적인 타입을 결정하는 것을 의미
    - 파이썬은 정적 타입 지정 언어가 아니므로 대부분의 경우 덕 타이핑에 의존

### 예시: `dict`

- 각 동물의 득표 수를 저장하는 딕셔너리
  ```python
  votes = {
      'otter': 1281,
      'polar bear': 587,
      'fox': 863,
  }
  
  ```
- 득표 데이터를 처리하고 각 동물의 이름과 순위를 빈 딕셔너리에 저장하는 함수
  ```python
  def populate_ranks(votes, ranks):
      names = list(votes.keys())
      names.sort(key=votes.get, reverse=True)
      for i, name in enumerate(names, 1):
          ranks[name] = i
  
  ```
- 콘테스트에서 어떤 동물이 우승했는지 보여줄 함수
    - `populate_ranks`가 `ranks` 딕셔너리에 내용을 등수 오름차순으로 등록한다고 가정하고 동작
        - 즉, 첫 번째 키가 우승자
    - `next(iter(ranks))`[^2]
  ```python
  def get_winner(ranks):
      return next(iter(ranks))
  
  ```
- 동작 확인
  ```python
  ranks = {}
  populate_ranks(votes, ranks)
  print(ranks)
  winner = get_winner(ranks)
  print(winner)
  
  >>>
  {'otter': 1, 'fox': 2, 'polar bear': 3}
  otter
  
  ```

### 예시: `collections.abc` 기반으로 새로 정의한 클래스

- 프로그램 요구사항 변경
    - UI 요소에서 결과를 보여줄 때 등수가 아니라 알파벳 순으로 표시해야 하게 됨
- 딕셔너리와 비슷하지만 내용을 알파벳 순서대로 이터레이션해주는 클래스 `SortedDict`를 새로 정의
    - `collections.abc` 모듈[^3]의 `MutableMapping`[^4]을 활용

```python
from collections.abc import MutableMapping

class SortedDict(MutableMapping):
    def __init__(self):
        self.data = {}

	def __getitem__(self, key):
	    return self.data[key]

	def __setitem__(self, key, value):
	    self.data[key] = value

    def __delitem__(self, key):
        del self.data[key]

    def __iter__(self):
        keys = list(self.data.keys())
        keys.sort()
        for key in keys:
            yield key

    def __len__(self):
        return len(self.data)

```

- `SortedDict`는 표준 딕셔너리 프로토콜을 준수
    - 앞에서 정의한 함수를 호출하면서 `SortedDict` 인스턴스를 표준 `dict` 위치에 사용해도 오류가 발생하지 않음
- 하지만 실행 결과는 요구 사항에 맞지 않음

```python
sorted_ranks = SortedDict()
populate_ranks(votes, sorted_ranks)
print(sorted_ranks.data)
winner = get_winner(sorted_ranks)
print(winner)

>>>
{'otter': 1, 'fox': 2, 'polar bear': 3}
fox

```

- `get_winner`의 구현이 `populate_ranks`의 삽입 순서에 맞게 딕셔너리를 이터레이션한다고 가정한 것이 문제
    - `dict` 대신 `SortedDict`를 사용하므로 해당 가정이 성립하지 않게 됨
    - 따라서 우승 동물로는 득표수가 1등인 동물이 아니라 알파벳 순서로 1등인 동물이 반환됨

### 해결책 (1): 삽입 순서 보존에 의존하지 않고 코드 작성
`ranks` 딕셔너리가 특정 순서로 이터레이션된다고 가정하지 않고 `get_winner` 함수 구현

```python
def get_winner(ranks):
    for name, rank in ranks.items():
        if rank == 1:
            return name

winner = get_winner(sorted_ranks)
print(winner)

>>>
otter

```

### 해결책 (2): 실행 시점에 명시적으로 `dict` 타입 검사
함수 맨 앞에 `ranks`의 타입이 원하는 타입인지 검사하는 코드 추가
- 원하는 타입이 아니면 예외 발생
- 보수적인 접근 방법보다 나은 실행 성능

```python
def get_winner(ranks):
    if not isinstance(ranks, dict):
        raise TypeError('dict 인스턴스가 필요합니다')
    return next(iter(ranks))

get_winner(sorted_ranks)

>>>
Traceback …
TypeError: dict 인스턴스가 필요합니다

```

### 해결책 (3): 타입 애너테이션(annotation)과 정적 분석(static analysis)을 사용해 `dict` 값 요구
타입 애너테이션 및 정적 분석 사용
- `get_winner`에 전달되는 값이 `MutableMapping` 인스턴스가 아니라 `dict` 인스턴스가 되도록 강제(Better Way 90 참조)
- 코드에 타입 애너테이션을 붙이고 `mypy` 도구를 엄격한 모드로 사용
- `dict`와 `MutableMapping` 타입의 차이를 올바로 감지해서 적절한 타입의 객체를 사용하지 않았을 때 오류를 발생시킴
- 정적 타입 안정성과 런타임 성능을 가장 잘 조합

```python
from typing import Dict, MutableMapping

def populate_ranks(votes: Dict[str, int],
                   ranks: Dict[str, int]) -> None:
    names = list(votes.keys())
    names.sort(key=votes.get, reverse=True)
    for i, name in enumerate(names, 1):
    ranks[name] = i

def get_winner(ranks: Dict[str, int]) -> str:
    return next(iter(ranks))

Class SortedDict(MutableMapping[str, int]):
    …

votes = {
    'otter': 1281,
    'polar bear': 587,
    'fox': 863,
}

sorted_ranks = SortedDict()
populate_ranks(votes, sorted_ranks)
print(sorted_ranks.data)
winner = get_winner(sorted_ranks)
print(winner)

>>>
$ python3 -m mypy —strict example.py
…/example.py:48: error: Argument 2 to "populate_ranks" has
incompatible type "SortedDict"; expected "Dict[str, int]"
…/example.py:50: error: Argument 1 to "get_winner" has
incompatible type "SortedDict"; expected "Dict[str, int]"

```

[^1]: 최소 최근 사용(least-recently-used) 캐시: 메모리에 남아 있는 캐시 중 가장 오래 전에 참조된 캐시를 새로운 캐시로 교체하는 알고리즘.
[^2]: `next(iter())`:  `iter`는 객체의 `__iter__` 메서드를 호출하고(즉, 반복 가능한 객체에서 이터레이터를 반환하고), `next`는 객체의 `__next__` 메서드를 호출함(즉, 이터레이터에서 값을 차례대로 꺼냄).
[^3]: `collections.abc`: 클래스가 특정 인터페이스를 제공하는지를 검사할 수 있는 추상 베이스 클래스(abstract base class, ABC)를 제공하는 모듈. ABC는 다른 클래스들의 청사진으로 사용됨.
[^4]: `MutableMapping`:  읽기 전용과 가변 매핑의 ABC. `MutableMapping`의 서브클래스를 만들어 활용하면 __getitem__, __setitem__ 등을 원하는 대로 동작하도록 커스터마이즈할 수 있음.
