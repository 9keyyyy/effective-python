# 40 super로 부모 클래스를 초기화하라

## TL;DR

- 표준 메서드 결정 순서(MRO)를 활용해 상위 클래스 초기화 순서 및 다이아몬드 상속 문제를 해결할 수 있음
- 부모 클래스를 초기화할 때는 `super` 내장함수를 아무 인자 없이 호출 가능 (파이썬 컴파일러가 자동으로 올바른 파라미터 넣어줌)

</br>

## 자식 클래스에서 부모 클래스를 초기화하는 가장 기본적인 방법

자식 인스턴스에서 부모 클래스의 `__init__`메서드 직접 호출

```python
class MyBaseClass:
    def __init__(self, value):
        self.value = value

class MyChildClass(MyBaseClass):
    def __init__(self):
        MyBaseClass.__init__(self, 5)

```

</br>

## 문제점?

클래스가 다중 상속을 받는 상황에서 상위 클래스의 `__init__` 메서드를 직접 호출하는 경우, 예측할 수 없는 방식으로 작동할 수 있음

### 문제1) 다중 상속의 경우 모든 하위 클래스에서 `__init__` 호출의 순서가 정해져 있지 않음

```python
class TimesTwo:
    def __init__(self):
        self.value *= 2

class PlusFive:
    def __init__(self):
        self.value += 5

class OneWay(MyBaseClass, TimesTwo, PlusFive):
    def __init__(self, value):
        MyBaseClass.__init__(self, value)
        TimesTwo.__init__(self)
        PlusFive.__init__(self)

foo = OneWay(5)
print('첫 번째 부모 클래스 순서에 따른 값은 (5 * 2) + 5 =', foo.value)

>>>
첫 번째 부모 클래스 순서에 따른 값은 (5 * 2) + 5 = 15

```

위 `OneWay` class에서는 부모 클래스의 순서에 따라 `__init__`을 호출해 초기화가 실행됨

```python
class AnotherWay(MyBaseClass, PlusFive, TimesTwo):
    def __init__(self, value):
        MyBaseClass.__init__(self, value)
        TimesTwo.__init__(self)
        PlusFive.__init__(self)

bar = AnotherWay(5)
print('두 번째 부모 클래스 순서에 따른 값은', foo.value)

>>>
두 번째 부모 클래스 순서에 따른 값은 15

```

- `AnotherWay` class에서는 `OneWay`에서와 같은 부모 클래스를 사용하지만 부모 클래스의 나열 순서가 다름
- 하지만, `__init__`의 순서는 그대로 두었기 때문에 `OneWay` 인스턴스에서와 같은 결과를 반환


</br>

### 문제2) 다이아몬드 상속으로 인한 문제 발생할 수 있음

- 어떤 클래스가 두 가지 서로 다른 클래스를 상속하는데, 두 상위 클래스의 상속 계층을 거슬러 올라가면 같은 조상 클래스가 존재하는 경우
- 공통 조상 클래스의 `__init__` 메서드가 여러 번 호출될 수 있어 예기치 않은 방식으로 코드가 작동할 수 있음


```python
class TimesSeven(MyBaseClass):
    def __init__(self, value):
        MyBaseClass.__init__(self, value)
        self.value *= 7

class PlusNine(MyBaseClass):
    def __init__(self, value):
        MyBaseClass.__init__(self, value)
        self.value += 9

class ThisWay(TimesSeven, PlusNine):
    def __init__(self, value):
        TimesSeven.__init__(self, value)
        PlusNine.__init__(self, value)

foo = ThisWay(5)
print('(5 * 7) + 9 = 44가 나와야 하지만 실제로는', foo.value)

>>>
(5 * 7) + 9 = 44가 나와야 하지만 실제로는 14

```

- 예상: `MyBaseClass`에서 value=5 → `TimesSeven`에서 value=35 → `PlusNine`에서 value=44
- 실제: `MyBaseClass`에서 value=5 → `TimesSeven`에서 value=35 → `MyBaseClass`에서 value=5 → `PlusNine`에서 value=44
   혼란 초래 + 복잡한 경우 디버깅이 어려움

## 해결법? super 내장 함수 + 표준 메서드 결정 순서(Method Resolution Order, MRO)

### super 내장함수

다이아몬드 계층의 공통 상위 클래스를 단 한번만 호출하도록 보장 (Better way 48 참고)

### MRO

상위 클래스를 초기화 하는 순서 정의 ([C3 선형화 알고리즘](https://www.python.org/download/releases/2.3/mro/) 사용)

```python
class TimesSevenCorrect(MyBaseClass):
    def __init__(self, value):
        super().__init__(value)
        self.value *= 7

class PlusNineCorrect(MyBaseClass):
    def __init__(self, value):
        super().__init__(value)
        self.value += 9

class GoodWay(TimesSevenCorrect, PlusNineCorrect):
    def __init__(self, value):
        super().__init__(value)

foo = GoodWay(5)
print('7 * (5 + 9) = 98이 나와야 하고 실제로도', foo.value)

>>>
7 * (5 + 9) = 98이 나와야 하고 실제로도 98

```

예상 ) `TimesSevenCorrect.__init__`이 먼저 호출되어 (5 * 7) + 9 = 44
실제 ) 7 * (5 + 9) = 98

### 호출 순서는 클래스에 대한 MRO 정의를 따름

`mro`라는 클래스 메서드를 통해 순서를 확인할 수 있음

```python
mro_str = '\\n'.join(repr(cls) for cls in GoodWay.mro())
print(mro_str)

>>>
<class '__main__.GoodWay'>
<class '__main__.TimesSevenCorrect'>
<class '__main__.PlusNineCorrect'>
<class '__main__.MyBaseClass'>
<class 'object'>

```

- 호출 순서 : `GoodWay.__init__` `TimesSevenCorrect.__init__` `PlusNineCorrect.__init__` `MyBaseClass.__init__`
- 상속 다이아몬드의 정점에 도달하면 각 초기화 메서드는 `__init__`이 호출된 순서의 역순으로 작업을 수행 (스택)
- `MyBaseClass` value=5 → `PlusNineCorrect` value=14 → `TimesSevenCorrect` value 98

즉, `super().__init__`의 사용은

- 다중 상속을 튼튼하게 함
- 직접 `__init__`을 하는 것보다 유지 보수가 더 편함 (상위 클래스 이름을 변경해도 `__init__`메서드 정의 바꿀 필요 X)

### super 함수의 파라미터

- 1번째 파라미터 : 접근하고 싶은 MRO 뷰를 제공할 부모 타입
- 2번째 파라미터 : 1번째 파라미터로 지정한 타입의 MRO 뷰에 접근할 때 사용할 인스턴스
- 하지만, 두 파라미터를 지정할 필요는 없음 → **컴파일러가 자동으로 `__class__`와 `self`를 넣어주기** 때문

```python
class ExplicitTrisect(MyBaseClass):
    def __init__(self, value):
        super(ExplicitTrisect, self).__init__(value)
        self.value /= 3

class AutomaticTrisect(MyBaseClass):
    def __init__(self, value):
        super(__class__, self).__init__(value)
        self.value /= 3

class ImplicitTrisect(MyBaseClass):
    def __init__(self, value):
        super().__init__(value)
        self.value /= 3

assert ExplicitTrisect(9).value == 3
assert AutomaticTrisect(9).value == 3
assert ImplicitTrisect(9).value == 3

```

위 세 클래스에서 super는 동일하게 작용
