# 35 제너레이터 안에서 throw로 상태를 변화시키지 말라

## TL;DR

- `thorw` 메서드를 사용하면 제너레이터가 마지막으로 실행한 yield 식의 위치에서 예외를 다시 발생시킬 수 있음
- `thorw` 를 사용하면 가독성이 나빠짐 (예외를 잡아내고 다시 발생시키는 데 준비 코드가 필요 + 내포 단계가 깊어지기 때문)
- 제너레이터에서 예외적인 동작을 제공하는 더 나은 방법은 **`__iter__` 메서드를 구현하는 클래스를 사용하면서 예외적인 경우에 상태를 전이**시키는 것임

## Exception을 다시 던질 수 있는 throw 메서드

- yield from 식과 send 메서드 외에, 제너레이터 안에서 Exception을 다시 던질 수 있는 `thorw` 메서드가 있음
- 어떤 제너레이터에 대해 `thorw`가 호출되면 이 제너레이터는 값을 내놓은 `yield`로부터 평소처럼 제너레이터 실행을 계속하는 대신, thorw가 제공한 Exception을 다시 던짐
  ```python
  class MyError(Exception):
      pass
  
  def my_generator():
      yield 1
      yield 2
      yield 3
  
  it = my_generator()
  print(next(it))  # 1을 내놓음
  print(next(it))  # 2를 내놓음
  print(it.throw(MyError('test error')))
  
  >>>
  1
  2
  Traceback ...
  MyError: test error
  
  ```
- thorw를 호출해 제너레이터에 예외를 주입해도, 제너레이터는 `try/except` 복합문을 사용해 마지막으로 실행된 yield 문을 둘러쌈으로써 이 예외를 잡아 낼 수 있음 
  ```python
  def my_generator():
      yield 1
      try:
          yield 2
      except MyError:
          print('MyError 발생!')
      else:
          yield 3
      yield 4
  
  it = my_generator()
  print(next(it))  # 1을 내놓음
  print(next(it))  # 2를 내놓음
  print(it.throw(MyError('test error'))
  >>>
  1
  2
  MyError 발생!
  4
  
  ```
## throw를 활용한 양방향 통신
- throw은 제너레이터와 제너레이터를 호출하는 쪽 사이에 양방향 통신 수단을 제공
- 다음은 throw 메서드에 의존하는 제너레이터를 통해 타이머를 구현하는 코드 (프로그램에 간헐적으로 재설정할 수 있는 타이머가 필요할 때)

```python
class Reset(Exception):
    pass

def timer(period):
    current = period
    while current:
        current -= 1
        try:
            yield current
        except Reset:
            current = period

```

- yield 식에서 `Reset` 예외가 발생할 때마다 카운터가 `period`로 재설정 됨
- 매 초 한 번 `폴링(polling)`되는 외부 입력과 이 재설정 이벤트를 연결할 수도 있음
- 그 후 timer 제너레이터를 구동시키는 `run` 함수를 정의할 수 있음
- `run` 함수는 throw를 사용해 타이머를 재설정하는 예외를 주입하거나, 제너레이터 출력에 대해 `announce` 함수를 호출함

```python
def check_for_reset():
    # 외부 이벤트를 폴링한다
    ...

def announce(remaining):
    print(f'{remaining} 틱 남음')

def run():
    it = timer(4)
    while True:
        try:
            if check_for_reset():
                current = it.throw(Reset())
            else:
                current = next(it)
        except StopIteration:
            break
        else:
            announce(current)
run()

>>>
3 틱 남음
2 틱 남음
1 틱 남음
3 틱 남음
2 틱 남음
3 틱 남음
2 틱 남음
1 틱 남음
0 틱 남음

```

### 문제점
- 코드는 잘 작동하지만 필요 이상으로 읽기 어려움
- 각 내포 단계마다 `StopIteration` 예외를 잡아내거나 throw를 할지, next나 announce를 호출할지 결정하는데, 이로 인해 코드에 잡음이 많음

### 해결법: 이터러블 컨테이너 객체
이 기능을 구현하는 더 단순한 접근 방법은 이터러블 컨테이너 객체를 사용해 상태가 있는 클로저를 정의하는 것

```python
class Timer:
    def __init__(self, period):
        self.current = period
        self.period = period

    def reset(self):
        self.current = self.period

    def __iter__(self):
        while self.current:
            self.current -= 1
            yield self.current

```

run 메서드에서는 for를 사용해 훨씬 단순하게 이터레이션을 수행할 수 있고, 내포 수준이 줄어들어 코드가 훨씬 읽기 쉬워짐

```python
def run():
    timer = Timer(4)
    for current in timer:
        if check_for_reset():
            timer.reset()
        announce(current)

run()

>>>
3 틱 남음
2 틱 남음
1 틱 남음
3 틱 남음
2 틱 남음
3 틱 남음
2 틱 남음
1 틱 남음
0 틱 남음

```

- 출력은 throw를 사용하던 예전 버전과 똑같지만, 훨씬 더 이해하기 쉽게 구현됨
- 제너레이터와 예외를 섞어서 만들어야 하는 작업이 있다면, 비동기 기능를 사용하면 더 좋게 구현할 수 있는 경우가 많음 (Better way 60 참고)
