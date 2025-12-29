# 31 인자에 대해 이터레이션할 때는 방어적이 돼라
## TL;DR

- 입력 인자를 여러 번 이터레이션하는 함수나 메서드를 조심할 것 (함수가 이상하게 작동하거나 결과가 없을 수 있음)
- 파이썬의 **이터레이터 프로토콜**은 컨테이너와 이터레이터가 `iter`, `next` 내장 함수나 `for` 루프 등의 관련 식과 상호작용하는 절차를 정의
  - `__iter__` 메서드를 제너레이터로 정의하면 쉽게 이터러블 컨테이너 타입을 정의 가능
- 어떤 값이 (컨테이너가 아닌) 이터레이터인지 감지하려면, 이 값을 `iter` 내장 함수에 넘겨서 반환되는 값이 원래 값과 같은지 확인
  - 다른 방법으로 `collections.abc.Iterator` 클래스를 `isinstance`와 함께 사용할 수도 있음
 
## 제너레이터 사용 시 발생하는 문제


### 기본 normalize 함수

방문자 데이터를 백분율로 변환하는 함수

```python
def normalize(numbers):
    total = sum(numbers)
    result = []
    for value in numbers:
        percent = 100 * value / total
        result.append(percent)
    return result
```

### 1. 리스트: 정상 동작

```python
visits = [15, 35, 80]
percentages = normalize(visits)
print(percentages)
# [11.538461538461538, 26.923076932076923, 61.53846153846154]
```

### 2. 제너레이터: 빈 결과 반환

파일에서 데이터를 읽는 제너레이터

```python
def read_visits(data_path):
    with open(data_path) as f:
        for line in f:
            yield int(line)

it = read_visits('my_numbers.txt')
percentages = normalize(it)
print(percentages)
# []  ← 빈 리스트!
```

---

## 원인 분석

### 제너레이터의 일회성

- 이터레이터는 결과를 **단 한 번만** 생성함
- 이미 `StopIteration` 예외가 발생한 이터레이터는 재사용 불가

```python
it = read_visits('my_numbers.txt')
print(list(it))  # [15, 35, 80]
print(list(it))  # []  ← 이미 소진됨
```

### 소진된 이터레이터의 특징

- 소진된 이터레이터를 순회해도 **오류가 발생하지 않음**
- `for`, 리스트 생성자 등이 `StopIteration`을 정상 동작으로 간주
- 따라서 소진된 이터레이터를 구분하기 어려움

---

## 해결방법

### 방법 1: 이터레이터를 리스트로 복사

```python
def normalize_copy(numbers):
    numbers_copy = list(numbers)  # 이터레이터 복사
    total = sum(numbers_copy)
    result = []
    for value in numbers_copy:
        percent = 100 * value / total
        result.append(percent)
    return result
```

**장점**
- 간단하고 명확한 구현

**단점**
- 전체 데이터를 메모리에 로드해야 함
- 큰 파일의 경우 메모리 부족 가능

---

### 방법 2: 매 호출 시마다 새 이터레이터 반환하는 함수 전달

```python
def normalize_func(get_iter):
    total = sum(get_iter())   # 새 이터레이터
    result = []
    for value in get_iter():  # 새 이터레이터
        percent = 100 * value / total
        result.append(percent)
    return result

# 사용
path = 'my_numbers.txt'
percentages = normalize_func(lambda: read_visits(path))
```

**장점**
- 메모리 효율적

**단점**
- 람다 함수를 넘기는 코드가 보기 좋지 않음
- 의도가 명확하지 않음

---

### 방법 3: 이터레이터 프로토콜을 구현한 컨테이너 클래스 (권장)

```python
class ReadVisits:
    def __init__(self, data_path):
        self.data_path = data_path
    
    def __iter__(self):
        with open(self.data_path) as f:
            for line in f:
                yield int(line)

# 사용
visits = ReadVisits(path)
percentages = normalize(visits)  # 정상 동작
```

**동작 원리**
- `for` 루프나 `sum()` 등이 내부적으로 `iter()` 호출
- `iter()`는 `__iter__()` 메서드를 실행
- 매번 **새로운 이터레이터 객체**가 생성됨
- `normalize` 함수 내부에서:
  - `sum()`이 `__iter__()`를 호출 → 첫 번째 이터레이터
  - `for` 루프가 `__iter__()`를 호출 → 두 번째 이터레이터
  - 두 이터레이터는 **독립적으로 동작**

**장점**
- 메모리 효율적
- 코드가 깔끔하고 의도가 명확
- 표준 파이썬 프로토콜 사용

**단점**
- 입력 데이터를 여러 번 읽음 (파일 I/O 발생)

---

## 방어적 프로그래밍

### 이터레이터와 컨테이너 구분하기

**핵심 원리**
- 이터레이터: `iter(x) is x` → `True` (자기 자신 반환)
- 컨테이너: `iter(x) is x` → `False` (새 이터레이터 반환)

### 방법 1: `iter()` 결과 비교

```python
def normalize_defensive(numbers):
    if iter(numbers) is numbers:  # 이터레이터인 경우
        raise TypeError('컨테이너를 제공해야 합니다')
    
    total = sum(numbers)
    result = []
    for value in numbers:
        percent = 100 * value / total
        result.append(percent)
    return result
```

### 방법 2: `collections.abc.Iterator` 사용

```python
from collections.abc import Iterator

def normalize_defensive(numbers):
    if isinstance(numbers, Iterator):  # 이터레이터인지 검사
        raise TypeError('컨테이너를 제공해야 합니다')
    
    total = sum(numbers)
    result = []
    for value in numbers:
        percent = 100 * value / total
        result.append(percent)
    return result
```

### 사용 예시

```python
# ✅ 리스트 - 정상 동작
visits = [15, 35, 80]
percentages = normalize_defensive(visits)

# ✅ 이터러블 클래스 - 정상 동작
visits = ReadVisits(path)
percentages = normalize_defensive(visits)

# ❌ 이터레이터 - 예외 발생
visits = [15, 35, 80]
it = iter(visits)
normalize_defensive(it)
# TypeError: 컨테이너를 제공해야 합니다
```

---

## 요약

| 방법 | 메모리 | 코드 품질 | 권장도 |
|------|--------|----------|--------|
| 리스트 복사 | 많이 사용 | 보통 | 작은 데이터 |
| 함수 전달 | 효율적 | 나쁨 | 비권장 |
| 이터러블 클래스 | 효율적 | 좋음 | **권장** |

**Best Practice**
- 재사용이 필요한 경우: 이터레이터 프로토콜(`__iter__`)을 구현한 클래스 사용
- 방어적 프로그래밍: `collections.abc.Iterator`로 입력 검증
- 작은 데이터: 리스트로 변환해도 무방
