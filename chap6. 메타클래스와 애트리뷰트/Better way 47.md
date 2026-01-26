## TL;DR


- **`__getattr__(self, name)`** : 객체의 attribute에 접근하려고 할 때 해당 attribute이 객체의 dictionary에 존재하지 않으면 자동으로 호출
- **`__getattribute__(self, name)`** : 객체의 attribute에 접근할 때마다 항상 호출
- **`__setattr__(self, name, value)`** : 객체에 attribute을 설정할 때마다 항상 호출

**지연 계산 애트리뷰트(lazy evaluation attributes)**: 어떤 attribute의 값이 실제로 필요할 때까지 계산을 미룸
파이썬 객체를 이용해 데이터베이스를 표현하는 경우, **지연 계산** 으로 스키마를 표현하는 클래스를 일반화할 수 있음

**`__getattr__`**
---

- 객체의 dictionary에서 찾을 수 없는 attribute에 접근할 때마다 **`__getattr__`** 이 자동 호출
- 해당 attribute이 처음 요청될 때만 값을 계산하고 저장할 수 있고, 이후에는 저장된 값이 반환됨

```python
class LazyRecord:
    def __init__(self):
        self.exists = 5

    def __getattr__(self, name):
        value = f'{name}를 위한 값'
        setattr(self, name, value)
        return value

class LoggingLazyRecord(LazyRecord):
    def __getattr__(self, name):
        print(f'* 호출: __getattr__({name!r}), '
              f'인스턴스 딕셔너리 채워 넣음')
        result = super().__getattr__(name) # super(): 무한 재귀 방지
        print(f'* 반환: {result!r}')
        return result

data = LoggingLazyRecord()
print('exists: ', data.exists)
print('이전:', data.__dict__)
print('첫 번째 foo: ', data.foo)
print('이후:', data.__dict__)
print('두 번째 foo: ', data.foo)
```

```python
>>>
exists:  5
이전: {'exists': 5}
* 호출: __getattr__('foo'), 인스턴스 딕셔너리 채워 넣음
* 반환: 'foo를 위한 값'
첫 번째 foo:  foo를 위한 값
이후: {'exists': 5, 'foo': 'foo를 위한 값'}
두 번째 foo:  foo를 위한 값
```

**`__getattribute__`** 
---

- 객체의 attribute에 접근할 때마다 **`__getattribute__`** 이 항상 호출 (이미 존재하는 attribute에 접근할 때도 호출됨)

```python
class ValidatingRecord:
    def __init__(self):
        self.exists = 5
    def __getattribute__(self, name):
        print(f'* 호출: __getattribute__({name!r})') # 책 오타 (__getattr__)
        try:
            value = super().__getattribute__(name)
            print(f'* {name!r} 찾음, {value!r} 반환')
            return value
        except AttributeError:
            value = f'{name}를 위한 값'
            print(f'* {name!r}를 {value!r}로 설정')
            setattr(self, name, value)
            return value

data = ValidatingRecord()
print('exists: ', data.exists)
print('첫 번째 foo: ', data.foo)
print('두 번째 foo: ', data.foo)
```

```python
>>>
* 호출: __getattribute__('exists')
* 'exists' 찾음, 5 반환
exists:  5
* 호출: __getattribute__('foo')
* 'foo'를 'foo를 위한 값'로 설정
첫 번째 foo:  foo를 위한 값
* 호출: __getattribute__('foo')
* 'foo' 찾음, 'foo를 위한 값' 반환
두 번째 foo:  foo를 위한 값
```

**`__setattr__`** 
---

- 계산된 값을 객체의 attribute으로 저장할 때 사용
- 객체에 attribute을 설정할 때마다 호출

```python
class SavingRecord:
    def __setattr__(self, name, value):
        # 데이터를 데이터베이스 레코드에 저장한다
        super().__setattr__(name, value)

class LoggingSavingRecord(SavingRecord):
    def __setattr__(self, name, value):
        print(f'* 호출: __setattr__({name!r}, {value!r})')
        super().__setattr__(name, value)

data = LoggingSavingRecord()
print('이전: ', data.__dict__)
data.foo = 5
print('이후: ', data.__dict__)
data.foo = 7
print('최후:', data.__dict__)
```

```python
이전:  {}
* 호출: __setattr__('foo', 5)
이후:  {'foo': 5}
* 호출: __setattr__('foo', 7)
최후: {'foo': 7}
```

Example: 객체의 dictionary에 key가 있을 때만 attribute에 접근하고 싶을 때
---

- **`__getattribute__`** 와 **`__setattr__`** 의 단점: 원하든 원하지 않든 어떤 attribute에 접근할 때마다 함수가 호출

```python
class BrokenDictionaryRecord:
    def __init__(self, data):
        self._data = data
    def __getattr__(self, name):
        print(f'* 호출: __getattr__({name!r})')
        return self._data[name]

data = Brokedata = BrokenDictionaryRecord({'foo': 3})
data.foo

>>>
* 호출: __getattr__('foo')
3
```

```python
class BrokenDictionaryRecord:
    def __init__(self, data):
        self._data = data
    def __getattribute__(self, name):
        print(f'* 호출: __getattribute__({name!r})')
        return self._data[name]

data = Brokedata = BrokenDictionaryRecord({'foo': 3})
data.foo

>>>
* 호출: __getattribute__('foo')
* 호출: __getattribute__('_data')
* 호출: __getattribute__('_data')
* 호출: __getattribute__('_data')
...
RecursionError: maximum recursion depth exceeded while calling a Python object
```

- self._data에 접근하려 할 때마다 **`__getattribute__`** 을 다시 호출 → 무한 재귀

```python
class DictionaryRecord:
    def __init__(self, data):
        self._data = data

    def __getattribute__(self, name):
        print(f'* 호출: __getattribute__({name!r})')
        data_dict = super().__getattribute__('_data')
        return data_dict[name]

data = DictionaryRecord({'foo': 3})
print('foo: ', data.foo)

>>>
* 호출: __getattr__('foo')
foo:  3
```

* **`super().__getattribute__('_data')`** 을 호출해 **`_data`** attribute의 값을 직접 가져오면 무한 재귀를 피할 수 있음
* **`self._data`**: "내가 커스텀한(재귀가 발생하는) 규칙으로 가져와!"
* **`super().__getattribute__('_data')`**: "내가 만든 규칙 다 건너뛰고, **파이썬이 원래 하던 방식(object의 방식)**으로 내 안에 있는 `_data`를 가져와!"

즉, `super()`는 "나의 오버라이딩된 함수"를 무시하고 **부모의 원래 함수**를 호출하라는 뜻


| 호출 방식 | 호출되는 대상 | 결과 |
| --- | --- | --- |
| `self._data` | **현재 클래스**의 `__getattribute__` | 다시 `self._data`를 만나 무한 루프 |
| `super().__getattribute__('_data')` | 부모(**object**)의 `__getattribute__` | 재귀 없이 실제 메모리에 있는 `_data` 값을 반환 |

