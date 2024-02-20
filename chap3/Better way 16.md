## TL;DR
- 파이썬 decorator는 **실행 시점**에 함수가 **다른 함수를 변경**할 수 있게 해주는 구문
- decorator를 사용하면 디버거 등 인트로스펙션을 사용하는 도구가 잘못 작동할 수 있음
- decorator를 구현할 때 인트로스펙션에서 문제가 생기지 않길 바란다면 functools 내장 모듈의 wraps 데코레이터(`functools.wraps()`)를 사용해라

## python decorator
- 함수가 호출되기 전/후 코드를 추가로 실행해줌
- 함수를 수정하지 않은 상태에서 추가 기능을 구현할 때 사용
- 감싸고 있는 함수의 입력 인자, 반환 값, 함수에서 발생한 오류에 접근할 수 있음 → 디버깅/함수 의미 강화에 유용

### 사용 방법
```python
@데코레이터
def 함수이름():
    코드
 ```

### 예시1
```python
def trace(func):                             # 호출할 함수를 매개변수로 받음
    def wrapper():
        print(func.__name__, '함수 시작')    # 함수 이름 출력
        func()                               # 매개변수로 받은 함수를 호출
        print(func.__name__, '함수 끝')
    return wrapper                           # wrapper 함수 반환
```

```python
# [사용법1]
hello = trace(hello)

# [사용법2]
@trace    
def hello():
    print('hello')

>>>
hello 함수 시작
hello
hello 함수 끝
```

### 예시2 - 재귀 함수에서 디버깅 시 유용
```python
def trace(func):
    def wrapper(*args, **kwargs):
        # 변수 위치 인자/키워드 인자(Better way 22, 23)를 사용해 감싸진 함수의 모든 파라미터 전달받음
        result = func(*args, **kwargs) 
        print(f'{func.__name__}({args!r}, {kwargs!r}) '
              f'-> {result!r}')
        return result
    return wrapper

@trace
def fibonacci(n):
    """Return n 번째 피보나치 수"""
    if n in (0, 1):
        return n
    return (fibonacci(n - 2) + fibonacci(n - 1))

fibonacci(4)

>>>
fibonacci((0,), {}) -> 0
fibonacci((1,), {}) -> 1
fibonacci((2,), {}) -> 1
fibonacci((1,), {}) -> 1
fibonacci((0,), {}) -> 0
fibonacci((1,), {}) -> 1
fibonacci((2,), {}) -> 1
fibonacci((3,), {}) -> 2
fibonacci((4,), {}) -> 3
```
- decorater로 감싸진 fibonacci 함수는 wrapper 함수로 인해 기존 fibonacci 함수 호출 후 result 값 출력
- 즉, 재귀 스택의 매 단계마다 함수의 인자와 반환 값 출력

## decorator 부작용 : 디버거와 같이 인트로스펙션을 하는 도구에서 문제가 됨
**인트로스펙션: 실행 시점에 프로그램이 어떻게 실행되는지 관찰하는 것 (Better way 80 pdb 참고)

### 1. decorator가 반환하는 함수의 이름이 fibonacci(감싸진 함수)가 아님
```python
print(fibonacci)

>>>
<function trace.<locals>.wrapper at 0x10361b9c0>
```

### 2. 독스트링이 출력되는 help 함수를 호출했을 때 아래와 같은 내용이 출력됨
```python
help(fibonacci)

>>>
Help on function wrapper in module __main__:

wrapper(*args, **kwargs)
```
- trace 함수에서 wrapper 함수를 반환하기 때문
- decorator로 인해 해당 wrapper 함수가 모듈에 fibonacci라는 이름으로 등록됨

### 3. 데코레이터가 감싸고 있는 원래 함수의 위치를 찾을 수 없기 때문에 객체 직렬화가 깨짐
(Better way 68: copyreg를 사용해 pickle을 더 신뢰성 있게 만들라 참고)
```python
import pickle
pickle.dumps(fibonacci)
```
```
>>>
Traceback ...
AttributeError: Can't pickle local object 'trace.<locals>.wrapper'
```

## 해결책
### functools 내장 모듈에 정의된 wraps 도우미 함수를 사용하는 것
```python
from functools import wraps

def trace(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        result = func(*args, **kwargs)
        print(f'{func.__name__}({args!r}, {kwargs!r}) '
              f'-> {result!r}')
        return result
    return wrapper


@trace
def fibonacci(n):
    """Return n 번째 피보나치 수"""
    if n in (0, 1):
        return n
    return (fibonacci(n - 2) + fibonacci(n - 1))
```
- wraps를 wrapper 함수에 적용하면, **wraps가 데코레이터 내부에 들어가는 함수에서 중요한 메타데이터를 복사해 wrapper 함수에 적용**해줌
- `__name__`, `__module__`, `__annotations__` 와 같은 표준 애트리뷰트들도 보존

### wraps 적용 후 실행 결과
```python
print(fibonacci)

>>>
<function fibonacci at 0x10580f740>
```

```python
help(fibonacci)

>>>
Help on function fibonacci in module __main__:

fibonacci(n)
    Return n 번째 피보나치 수
```

```python
import pickle
print(pickle.dumps(fibonacci))

>>>
b'\x80\x04\x95\x1a\x00\x00\x00\x00\x00\x00\x00\x8c\x08__main__\x94\x8c\tfibonacci\x94\x93\x94.'
```
