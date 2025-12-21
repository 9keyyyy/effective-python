# 23 키워드 인자로 선택적인 기능을 제공하라
## TL;DR

- 함수 호출 시 **1. 위치 기반 인자**와, **2. 키워드 인자**로 전달 가능하다.
- 키워드 인자를 사용하면, **인자의 목적을 파악하기 쉽다**.
- 키워드 인자를 **디폴트 값**과 함께 사용하면, **하위 호환성**을 제공하며 **함수에 새로운 기능을 추가**할 수 있다.
- **선택적 키워드 인자**는 **항상 키워드 인자로 전달**해야 한다. (Convention)

# 키워드 인자

- 모든 일반적인 인자를 **키워드**를 사용해 넘길 수 있다.
- 필요한 위치 기반 인자가 모두 제공되는 한, **키워드 인자를 넘기는 순서는 상관 없다.**

```python
def remainder(number, divisor):
    return number % divisor

# 모두 같은 결과!
remainder(20, 7)
remainder(20, divisor=7)
remainder(number=20, divisor=7)
remainder(divisor=7, number=20)

def example(a, b, c, x, y, z):
  return (a+b+c) - (x+y+z)

# 모두 같은 결과!
example(10, 20, 30, x=1, y=2, z=3)
example(10, 20, 30, y=2, z=3, x=1)

```

### 주의해야 할 규칙

1. 위치 기반 인자는 키워드 인자보다 앞에 지정해야 한다.
  ```python
  remainder(number=20, 7) # ❌
  # >>>
  # Traceback ...
  # SyntaxError: positional argument follows keyword argument
  
  ```
2. 각 인자는 단 한 번만 지정해야 한다.
  ```python
  remainder(20, number=7) # ❌
  # >>>
  # Traceback ...
  # SyntaxError: remainder() got multiple values for argument
  → 'number'
  
  ```

### **(double asterisk) 연산자 활용시
- **연산자: 딕셔너리의 값들을 함수에 전달하되 각 값에 대응하는 키를 키워드로 사용할 수 있다.
    
  ```python
  my_kwargs = {
    'number': 20,
    'divisor': 7,
  }
  assert remainder(**my_kwargs) = 6 ✅  
  ```
- **연산자를 위치 인자 혹은 키워드 인자와 섞어서 함수 호출할 때, 중복되는 인자가 없어야 한다.
  ```python
  my_kwargs = {
    'divisor': 7,
  }
  assert remainder(number=20, **my_kwargs) == 6 ✅
  
  my_kwargs = {
    'divisor': 7,
  }
  assert remainder(divisor=7, **my_kwargs) == 6 ❌
  ```
- **연산자를 여러 번 사용할 때, 중복되는 인자가 없어야 한다.
  ```python
  my_kwargs = {
    'number': 20,
  }
  other_kwargs = {
    'divisor': 7,
  }
  assert remainder(**my_kwargs, **other_kwargs) === 6 ✅
  ```

### 장점

### 1. 함수 호출의 의미를 명확히 알려줄 수 있다.
어떤 파라미터가 어떤 목적을 쓰는지 명확

```python
remainder(20, 7)
remainder(number=20, divisor=7)

```

### 2. 키워드 인자의 경우 함수 정의에서 디폴트 값을 지정할 수 있다.

```python
def flow_rate(weight_diff, time_diff, period=1):
  return (weight_diff / time_diff) * period

weight_diff = 0.5
time_diff = 3

flow_rate(weight_diff, time_diff)
flow_rate(weight_diff, time_diff, period=3600)

```

- 위치 기반 인자의 경우에도 디폴트 값을 지정할 수 있음
- 그러나, 디폴트 값을 사용하는 파라미터는 뒤쪽에 배치돼야 하며, 키워드 인자도 마찬가지

```python
def example(a, b, c, x=1, y=2, z=3): ✅
  return (a+b+c) - (x+y+z)
def example(x=1, y=2, z=3, a, b, c): ❌
  return (a+b+c) - (x+y+z)

example(10, 20, 30, y=2, z=3, x=1)

```

### 3. 호출자에게 하위 호환성을 제공하면서 함수 파라미터를 확장할 수 있다.

함수의 변경사항을 알든 모르든, 기존 호출 방식을 그대로 사용할 수 있다.

```python
def flow_rate(weight_diff, time_diff, period=1, units_per_kg=1):
  return ((weight_diff * units_per_kg) / time_diff) * period

weight_diff = 0.5
time_diff = 3

# 기존 호출자
flow_rate(weight_diff, time_diff, period=3600)
# 추가 기능
flow_rate(weight_diff, time_diff, period=3600, units_per_kg=2.2)

```

선택적인 인자를 여전히 위치 인자로 전달 가능

```python
# 혼동 야기
flow_rate(weight_diff, time_diff, 3600, 2.2)

# 물론 변수 전달 가능
# 그러나, 매번 변수에 할당하는게 번거로움
period = 3600
units_per_kg = 2.2
flow_rate(weight_diff, time_diff, period, units_per_kg)

```

선택적인 인자를 지정하는 최선의 방법 -> 항상 키워드 인자를 사용하자
```python
flow_rate(weight_diff, time_diff, 3600, 2.2) # :(
flow_rate(weight_diff, time_diff, period=3600, units_per_kg=2.2) # :)

```
