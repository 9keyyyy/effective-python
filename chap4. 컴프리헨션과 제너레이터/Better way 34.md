# 34 send로 제너레이터에 데이터를 주입하지 말라
## TL;DR

- send 메서드를 사용해 데이터 제너레이터에 주입 가능
  - 제너레이터는 send로 주입된 값을 yield 식이 반환하는 값을 통해 받음 -> 이 값을 변수에 저장해 활용
- send와 yield from을 함께 사용하는 경우 제너레이터 출력에 None이 나타나는 의외의 결과가 있을 수 있음
- **합성할 제너레이터들의 입력으로 이터레이터를 전달하는 방식이 send 방식보다 더 나음**



## 예시 1) 소프트웨어 라디오를 통해 파형 신호를 송신
wave 제너레이터를 이터레이션 하면서 진폭과 step이 고정된 파형 신호 송신
```python
import math

def wave(amplitude, steps): # 진폭, 신호를 내보내는 단계 수
    step_size = 2 * math.pi / steps   # 2라디안/단계 수
    for step in range(steps):
        radians = step * step_size
        fraction = math.sin(radians)
        output = amplitude * fraction
        yield output

def transmit(output):
    if output is None:
        print(f'출력: None')
    else:
        print(f'출력: {output:>5.1f}')

def run(it):
    for output in it:
        transmit(output)

run(wave(3.0, 8))

>>>
출력:   0.0
출력:   2.1
출력:   3.0
출력:   2.1
출력:   0.0
출력:  -2.1
출력:  -3.0
출력:  -2.1

```

## 예시 2) 진폭에 지속적인 변화를 줘가며 신호를 송신 (ex. AM 라디오)

제너레이터를 이터레이션할 때마다 진폭을 변조해야 함 → send 활용

```python
import math

def wave(amplitude, steps): # 진폭, 신호를 내보내는 단계 수
    step_size = 2 * math.pi / steps   # 2라디안/단계 수
    for step in range(steps):
        radians = step * step_size
        fraction = math.sin(radians)
        output = amplitude * fraction
        yield output

def wave_modulating(steps):
    step_size = 2 * math.pi / steps
    amplitude = yield                # 초기 진폭
    for step in range(steps):
        radians = step * step_size
        fraction = math.sin(radians)
        output = amplitude * fraction
        amplitude = yield output     # 다음 진폭

def run_modulating(it):
    amplitudes = [
        None, 7, 7, 7, 2, 2, 2, 2, 10, 10, 10, 10, 10
    ]
    for amplitude in amplitudes:
        output = it.send(amplitude)
        transmit(output)

run_modulating(wave_modulating(12))

>>>
출력: None
출력:   0.0
출력:   3.5
출력:   6.1
출력:   2.0
출력:   1.7
출력:   1.0
출력:   0.0
출력:  -5.0
출력:  -8.7
출력: -10.0
출력:  -8.7
출력:  -5.0

```
- send: 제너레이터가 재개될 때 yield가 send에 전달된 파라미터 값을 반환
- 방금 생성된 제너레이터는 아직 yield에 도달하지 못했기 때문에 send로 최초로 전달해야 하는 값은 None
- amplitude를 parameter로 받지 않고, send를 활용하여 yield 식의 반환값으로 받음
- 코드가 이해하기 어려움
    - 대입문에 yield를 사용하여 yield의 반환값을 받는 것이 직관적이지 X
    - send와 yield 사이의 연결을 알아보기 어려움

## 예시 3) 진폭과 step을 함께 변조

```python
def wave_modulating(steps):
    step_size = 2 * math.pi / steps
    amplitude = yield                # 초기 진폭
    for step in range(steps):
        radians = step * step_size
        fraction = math.sin(radians)
        output = amplitude * fraction
        amplitude = yield output     # 다음 진폭

def run_modulating(it):
    amplitudes = [
        None, 7, 7, 7, 2, 2, 2, 2, 10, 10, 10, 10, 10]
    for amplitude in amplitudes:
        output = it.send(amplitude)
        transmit(output)

def complex_wave_modulating():
    yield from wave_modulating(3)
    yield from wave_modulating(4)
    yield from wave_modulating(5)

run_modulating(complex_wave_modulating())

>>>
출력: None
출력:   0.0
출력:   6.1
출력:  -6.1
출력: None
출력:   0.0
출력:   2.0
출력:   0.0
출력: -10.0
출력: None
출력:   0.0
출력:   9.5
출력:   5.9

```

- yield from 식이 끝날 때마다 다음 yield from 식이 실행
- 제너레이터를 옮겨갈 때마다 send 로부터 값을 받기 위해 아무 값도 만들어내지 않는 단순한 yield 식으로 시작 → 제너레이터를 옮겨갈 때마다 None 출력



## 해결책) 이전 제너레이터를 다음 제너레이터의 입력으로 연쇄

```python
def wave(amplitude, steps): # 진폭, 신호를 내보내는 단계 수
    step_size = 2 * math.pi / steps   # 2라디안/단계 수
    for step in range(steps):
        radians = step * step_size
        fraction = math.sin(radians)
        output = amplitude * fraction
        yield output

def wave_cascading(amplitude_it, steps):
    step_size = 2 * math.pi / steps
    for step in range(steps):
        radians = step * step_size
        fraction = math.sin(radians)
        amplitude = next(amplitude_it) # 다음 입력 받기
        output = amplitude * fraction
        yield output

def complex_wave_cascading(amplitude_it):
    yield from wave_cascading(amplitude_it, 3)
    yield from wave_cascading(amplitude_it, 4)
    yield from wave_cascading(amplitude_it, 5)

def run_cascading():
    amplitudes = [7, 7, 7, 2, 2, 2, 2, 10, 10, 10, 10, 10]
    it = complex_wave_cascading(iter(amplitudes))
    for amplitude in amplitudes:
        output = next(it)
        transmit(output)

run_cascading()

>>>
출력:   0.0
출력:   6.1
출력:  -6.1
출력:   0.0
출력:   2.0
출력:   0.0
출력:  -2.0
출력:   0.0
출력:   9.5
출력:   5.9
출력:  -5.9
출력:  -9.5
```
- 아무데서나(amplitudes, steps) 이터레이터(제너레이터)를 사용해도 잘 작동
- 제너레이터는 본질적으로 thread-safe 하지 않음
- thread를 넘나들며 제너레이터를 사용해야 하는 경우엔 async 함수를 활용
