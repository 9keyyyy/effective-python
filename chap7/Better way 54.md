## TL;DR
- 파이썬에는 GIL이 있지만, 여러 스레드 사이에 일어나는 데이터 경합은 피할 수 없음
- 코드에서 여러 스레드가 상호 배제 락(뮤텍스) 없이 같은 객체를 변경하도록 허용하면 코드가 데이터 구조를 오염시킬 수 있음
- 여러 스레드 사이에서 프로그램의 불변 조건을 유지하려면 `threading` 내장 모듈의 `Lock` 클래스를 활용해야 함

</br>

## GIL (Global Interpreter Lock)
- 한 번에 하나의 스레드만 Python 코드를 실행할 수 있도록 함
- 다중 CPU에서 파이썬 스레드들이 병렬적으로 실행될 수 없게 막음

그렇다면 파이썬 스레드들이 프로그램의 데이터 구조에 동시에 접근할 수 없도록 막는 Lock 역할도 해줄까? **No‼️**

</br>

## GIL과 Lock
- GIL이 동시 접근을 보장해주는 Lock 역할을 해주는 것처럼 보이지만, 실제로는 전혀 그렇지 않음
- 파이썬 스레드는 **한번에 단 하나만 실행**되지만, 어떤 스레드가 데이터 구조에 대해 수행하는 연산은 **연속된 두 바이트 코드 사이에서 언제든 인터럽트 가능**
- 이러한 인터럽트로 인해 실질적으로는 언제든지 데이터 구조에 대한 불변 조건 위반 가능 → 프로그램 상태 오염 초래

### 예시
- 광센서를 통해 빛이 들어온 횟수 카운트
- 블로킹 I/O를 수행하므로 센서마다 작업자 스레드 할당

```python
class Counter:
    def __init__(self):
        self.count = 0

    def increment(self, offset):
        self.count += offset

def worker(sensor_index, how_many, counter):
    for _ in range(how_many):
        # 센서를 읽는다
        counter.increment(1)
```

아래 코드는 병렬로 worker 스레드를 실행하고, 모든 스레드가 값을 다 읽을 때까지 기다림
```python
how_many = 10**5
counter = Counter()

threads = []
for i in range(5):
    thread = Thread(target=worker,
                    args=(i, how_many, counter))
    threads.append(thread)
    thread.start()

for thread in threads:
    thread.join()

expected = how_many * 5
found = counter.count
print(f'카운터 값은 {expected}여야 하는데, 실제로는 {found} 입니다')

>>>
카운터 값은 500000여야 하는데, 실제로는 481384 입니다
```

### GIL에 의해 파이썬 스레드는 한 순간에 단 하나씩만 실행되는데, 위와 같은 결과가 나온 이유?
- 파이썬 인터프리터는 실행되는 모든 스레드를 강제로 공평하게 취급해서 **각 스레드의 실행 시간을 거의 비슷**하게 만듬
- 이를 위해 실행 중인 스레드를 일시 중단시키고 다른 스레드를 실행시키는 일 반복
- 이때, **파이썬이 스레드를 언제 일시 중단 시킬지는 알 수 없음❗️**

</br>

## 문제) 원자적(atomic)인 것처럼 보이는 연산을 수행하는 도중에도 스레드는 일시 중단될 수 있음
**Counter 객체의 increment 메서드는 작업자 스레드 입장에서보면 아래와 같음**
```python
counter.count += 1
```

</br>

**하지만 객체 애트리뷰트에 대한 += 연산은 실제로 아래 세가지 연산 과정으로 이루어짐**
```python
value = getattr(counter, 'count')
result = value + 1
setattr(counter, 'count', result)
```

</br>

**카운터를 증가시키는 파이썬 스레드는 세 연산 사이에서 일시 중단될 수 있음**
```python
# 스레드 A에서 실행
value_a = getattr(counter, 'count')

# 스레드 B로 컨텍스트 전환
value_b = getattr(counter, 'count')
result_b = value_b + 1
setattr(counter, 'count', result_b)

# 다시 스레드 A로 컨텍스트 전환
result_a = value_a + 1
setattr(counter, 'count', result_a)
```
일시 중단으로 인해 스레드 간 연산 순서가 뒤섞이면서 **value의 이전 값을 카운터에 대입**
1. 스레드 A가 완전히 끝나기 전 인터럽트가 일어나 스레드 B 실행
2. 스레드 B의 실행이 끝남
3. 스레드 A가 중간부터 실행을 재개

→ `value_a`에 값이 더해지므로 중간에 스레드 B가 카운터를 증가시키킨 결과는 무시됨

</br>

## 해결법) `threading` 내장 모듈의 `Lock` 클래스를 사용하자
- Lock 클래스가 상호 배제 락(뮤텍스) 역할을 함 → 한번에 단 하나의 스레드만 Lock 획득
- 데이터의 동시성 문제나 race condition을 방지함으로써 데이터 구조 오염 해결

### 예시
- Lock을 사용하여 Counter 클래스가 여러 스레드의 동시 접근으로부터 자신의 현재 값 보호
- `with` 문을 사용해 Lock 획득/해제 (with 문 관련해서는 Better way 66 참고)
```python
from threading import Thread
from threading import Lock

class LockingCounter:
    def __init__(self):
        self.lock = Lock()
        self.count = 0

    def increment(self, offset):
        with self.lock:
            self.count += offset

def worker(sensor_index, how_many, counter):
    for _ in range(how_many):
        # 센서를 읽는다
        counter.increment(1)
```
```python
how_many = 10**5
counter = LockingCounter()

threads = []
for i in range(5):
    thread = Thread(target=worker,
                    args=(i, how_many, counter))
    threads.append(thread)
    thread.start()

for thread in threads:
    thread.join()

expected = how_many * 5
found = counter.count
print(f'카운터 값은 {expected}여야 하는데, 실제로는 {found} 입니다')

>>>
카운터 값은 500000여야 하는데, 실제로는 500000 입니다
```
