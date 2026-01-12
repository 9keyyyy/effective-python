# 42 비공개 애트리뷰트보다는 공개 애트리뷰트를 사용하라
    
## TL;DR

- 파이썬 컴파일러는 비공개 애트리뷰트에 자식 클래스나 클래스 외부에서 접근하는 것을 엄격하게 금지하지 않음
- 비공개 애트리뷰트로 외부나 하위 클래스의 접근을 막기보다는 **보호된 필드 + 문서 상 가이드** 추천
- 하위 클래스에서 이름 충돌을 막기 위해서만 비공개 애트리뷰트 사용 권장 (ex) `value` 등 흔한 이름)

## 파이썬 클래스 애트리뷰트

### 공개(public) vs 비공개(private)
- 공개는 객체 뒤에 점 연산자(.)를 통해 접근
- 애트리뷰트 앞에 밑줄 두 개(__) 붙일 시 비공개 필드
- 비공개 필드는 해당 클래스 안의 메서드에서만 접근 가능
- 클래스 메서드 또한 class 블록 내부이기에 비공개 필드 접근 가능

```python
class MyObject:
    def __init__(self):
        self.public_field = 5
        self.__private_field = 10

    def get_private_field(self):
        return self.__private_field

    @classmethod
    def get_private_field_of_instance(cls, instance):
        return instance.__private_field

foo = MyObject()

# 공개 애트리뷰트 접근
foo.public_field == 5

# 비공개 애트리뷰트를 메서드를 통해 접근
foo.get_private_field() == 10

### 오류 발생 - 비공개 애트리뷰트를 클래스 외부에서 접근
foo.__private_field

# 비공개 애트리뷰틀르 클래스 메서드에서 접근
MyObject.get_private_field_of_instance(foo) == 10

```

- 하위 클래스에서는 부모 클래스의 비공개 필드에 접근 불가

```python
class MyParentObject:
    def __init__(self):
        self.__private_field = 71

class MyChildObject(MyParentObject):
    def get_private_field(self):
        return self.__private_field

baz = MyChildObject()

### 오류 발생
baz.get_private_field()

```

- 비공개 애트리뷰트의 동작은 애트리뷰트의 이름을 바꾸는 방식으로 구현됨 (`_(선언된 클래스명)__(변수명)` 형식) -> 이름을 적절히 변경하여 접근 가능
- 위의 오류는 `baz.MyChildeObject__private_field`로 접근하기 때문에 생기는 오류
- 접근하는 것을 명확히 막지 않음(`우리는 모두 책임질 줄 아는 성인이다`)

```
assert baz._MyParentObject__private_field == 71

print(baz.__dict__)

>>>
{'_MyParentObject__private_field': 71}

```



## 보호 필드(`_protected_field`)

- 명명 규약을 따라 관례적으로 보호되는 필드
- 비공개 필드 사용 시, 클래스 상속이나 새로운 동작 추가 등 확장 등이 어려움

```python
class MyBaseClass:
    def __init__(self, value):
        self.__value = value

    def get_value(self):
        return self.__value

class MyStringClass(MyBaseClass):
    def get_value(self):
        return str(super().get_value())  # 변경됨

class MyIntegerSubclass(MyStringClass):
    def get_value(self):
        return int(self._MyStringClass__value)  # 변경되지 않음

foo = MyIntegerSubclass(5)

### 오류 발생
foo.get_value()

print(foo.__dict__)
>>>
{'_MyBaseClass__value': 5}

```

- 상속을 허용하는 부모 클래스에서 보호 애트리뷰트를 사용하고, 하위 클래스에서 변경가능한 필드를 명시할 것

```python
class MyStringClass:
    def __init__(self, value):
        # 보호 필드 설명 (문자열 타입 변환이 가능한 필드이며, 불변 값 등등)
        self._value = value

```

### 비공개 애트리뷰트 사용 조건
- 애트리뷰트의 이름이 흔한 이름
- 공개 API로 제공하는 등 하위 클래스 작성을 통제할 수 없을 때

```python
class ApiClass:
    def __init__(self):
        self._value = 5

    def get(self):
        return self._value

class Child(ApiClass):
    def __init__(self):
        super().__init__()
        self._value = "hello"  # 충돌

a = Child()
print(f"{a.get()} 와 {a._value} 는 달라야 합니다.")
print(a.__dict__)

>>>
hello 와 hello 는 달라야 합니다.
{'_value': 'hello'}

```

#### 비공개 필드와 보호 필드의 구분

```python
class ApiClass:
    def __init__(self):
        self.__value = 5  # 밑줄 2개!

    def get(self):
        return self.__value  # 밑줄 2개!

class Child(ApiClass):
    def __init__(self):
        super().__init__()
        self._value = "hello"  # OK!

a = Child()
print(f"{a.get()} 와 {a._value} 는 다릅니다.")

print(a.__dict__)
>>>
5 와 hello 는 다릅니다.
{'_ApiClass__value': 5, '_value': 'hello'}

```
