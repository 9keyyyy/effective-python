# 33 yield from을 사용해 여러 제너레이터를 합성하라
## TL;DR

- yield from 식을 사용하면 여러 내장 제너레이터를 모아서 제너레이터 하나로 합성 가능
- 직접 내포된 제너레이터를 이터레이션 하면서 각 제너레이터의 출력을 내보내는 것 보다 yield from을 사용하는 것이 성능적으로 더 좋음

### 예시1) yield만 사용

제너레이터를 사용해 한 루프마다 이미지의 이동 변위(delta)를 만들어내는 동작을 하게하는 코드

```python
def move(period, speed):
    for _ in range(period):
        yield speed

def pause(delay):
    for _ in range(delay):
        yield 0

def render(delta):
    print(f'Delta: {delta:.1f}')

def run(func):
    for delta in func():
        render(delta)

def animate():
    for delta in move(4, 5.0):
        yield delta
    for delta in pause(3):
        yield delta
    for delta in move(2, 3.0):
        yield delta

run(animate)

>>>
Delta: 5.0
Delta: 5.0
Delta: 5.0
Delta: 5.0
Delta: 0.0
Delta: 0.0
Delta: 0.0
Delta: 3.0
Delta: 3.0

```

- 위 코드는 animate이 너무 반복적이라는 문제점이 있음
    - 더 복잡한 움직임을 표현하려면 제너레이터가 더 많아질텐데 가독성이 떨어짐

### 예시2) yield from 식 사용

```python

def animate_composed():
    yield from move(4, 5.0)
    yield from pause(3)
    yield from move(2, 3.0)

run(animate_composed)

>>>
Delta: 5.0
Delta: 5.0
Delta: 5.0
Delta: 5.0
Delta: 0.0
Delta: 0.0
Delta: 0.0
Delta: 3.0
Delta: 3.0

```

- for loop을 내포시키고 yield 식을 끝까치 처리
- 동일한 결과이지만 더 명확하고 직관적
- 인터프리터가 우리 대신 for loop을 내포시켜 yield를 처리하도록 동작 + 성능 향상

### 성능 측정
```python
import timeit

def child():
    for i in range(1_000_000):
        yield i

# yield from 사용하지 않은 구현
def slow():
    for i in child():
        yield i

# yield from 사용한 구현
def fast():
    yield from child()

baseline = timeit.timeit(
    stmt='for _ in slow(): pass',
    globals=globals(),
    number=50)
print(f'수동 내포: {baseline:.2f}s')

comparison = timeit.timeit(
    stmt='for _ in fast(): pass',
    globals=globals(),
    number=50)
print(f'합성 사용: {comparison:.2f}s')

reduction = -(comparison - baseline) / baseline
print(f'{reduction:.1%} 시간이 적게 듦')

>>>
수동 내포: 2.78s
합성 사용: 2.55s
8.5% 시간이 적게 듦

```
