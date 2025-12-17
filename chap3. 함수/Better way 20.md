# 20 None을 반환하기보다는 예외를 발생시켜라
## TL;DR

- None을 반환하는 함수를 사용하면 None과 다른 값(ex. 0 혹은 빈 문자열)이 조건문에서 False로 평가되기 때문에 실수하기 쉬움
- None을 반환하기보다는 예외를 발생시키자.
  - docstring으로 예외 정보를 기록해 호출자가 예외 처리를 제대로 하도록 하면 됨
  - 타입 애너테이션으로 함수가 None을 반환하지 않는다는 사실 명시

<br/>

### 예시: 한 수를 다른 수로 나누는 도우미 함수

- 0으로 나누는 경우 결과가 정해져 있지 않음

```python
def careful_divide(a, b):
    try:
        return a / b
    except ZeroDivisionError:
        return None

x, y = 1, 0
result = careful_divide(x, y)
if result is None:
    print('잘못된 입력')

```

- None인지 검사하는 대신, 빈 값을 False로 취급하는 검사
⇒ 도우미 함수의 반환 값이 0일 경우, 오류 발생

```python
x, y = 0, 5
result = careful_divide(x, y)
if not result:
    print('잘못된 입력') # 이 코드가 실행되는데, 사실 이 코드가 실행되면 안된다!

>>>
잘못된 입력

```

<br/>

### 해결책 1: 반환 값을 (성공 여부, 결괏값) 튜플로 분리

```python
def careful_divide(a, b):
    try:
        return True, a / b
    except ZeroDivisionError:
        return False, None

success, result = careful_divide(x, y)
if not success:
    print('잘못된 입력')

```

- 사용하지 않는 변수 밑줄(_)로 표시
⇒ 만약 success 변수가 위에서 정의됐더라면…

```python
_, result = careful_divide(x, y)
if not success:
    print('잘못된 입력')

```

<br/>

### 해결책 2: 에러가 발생한 경우 Error로 바꿔 던져 입력 값이 잘못됐음을 알리기

- 반환 값에 대한 조건문 필요 X
- 값이 항상 올바르다 가정하고, else 블록에서 반환 값 사용

```python
def careful_divide(a, b):
    try:
        return a / b
    except ZeroDivisionError as e:
        raise ValueError('잘못된 입력')

x, y = 5, 2
try:
    result = careful_divide(x, y)
except ValueError:
    print('잘못된 입력')
else:
    print('결과는 %.1f 입니다' % result)

```

<br/>

### Best way

- Type annotation으로 None이 결코 반환되지 않음을 명시
- 예외가 포함되어 있음을 주석으로 명시
```python
def careful_divide(a: float, b: float) -> float:
    """a를 b로 나눈다.

    Raises:
        ValueError: b가 0이어서 나눗셈을 할 수 없을 때
    """
    try:
        return a / b
    except ZeroDivisionError as e:
        raise ValueError('잘못된 입력')

```
