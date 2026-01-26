# 46 재사용 가능한 @property 메서드를 만들려면 디스크립터를 사용하라 


## TL;DR

1. **재사용성:** `@property`의 검증 로직(예: 점수 범위 확인)을 여러 속성에서 반복 사용하려면 **디스크립터 클래스**를 구현하라.
2. **메모리 관리:** 디스크립터 내부에서 인스턴스별 데이터를 저장할 때는 반드시 **`WeakKeyDictionary`**를 사용하여 메모리 누수를 방지하라.



## 1. 문제 상황: @property의 한계

`@property`는 데이터 유효성 검증에 유용하지만, 여러 속성에 동일한 로직을 적용해야 할 경우 코드가 중복되고 비효율적입니다.

### 예시: 반복적인 검증 로직

아래 `Exam` 클래스는 `writing_grade`와 `math_grade` 모두 0~100 사이의 값인지 검증해야 합니다. 이 경우 동일한 getter, setter 구조와 검증 함수(`_check_grade`)를 반복해서 작성해야 합니다.

```python
class Exam:
    def __init__(self):
        self._writing_grade = 0
        self._math_grade = 0

    @staticmethod
    def _check_grade(value):
        if not (0 <= value <= 100):
            raise ValueError('점수는 0과 100 사이입니다')

    @property
    def writing_grade(self):
        return self._writing_grade

    @writing_grade.setter
    def writing_grade(self, value):
        self._check_grade(value)
        self._writing_grade = value

    @property
    def math_grade(self):
        return self._math_grade

    @math_grade.setter
    def math_grade(self, value):
        self._check_grade(value)
        self._math_grade = value

```

---

## 2. 해결책: 디스크립터 프로토콜 도입

디스크립터 프로토콜(`__get__`, `__set__`)을 이용하면 검증 로직을 하나의 클래스로 분리하여 재사용할 수 있습니다.

### 사용 예시

```python
class Exam:
    # Grade 디스크립터를 클래스 애트리뷰트로 선언
    math_grade = Grade()
    writing_grade = Grade()
    science_grade = Grade()

```

### 동작 원리

파이썬은 `exam.writing_grade = 40`과 같은 접근을 다음과 같이 해석하여 처리합니다.

* **설정 (Set):** `Exam.__dict__['writing_grade'].__set__(exam, 40)`
* **조회 (Get):** `Exam.__dict__['writing_grade'].__get__(exam, Exam)`



## 3. 구현 단계별 문제점과 최종 해결책

### 시도 1: 단순 값 저장 (잘못된 구현)

디스크립터 인스턴스 하나(`Grade()`)가 클래스 레벨에서 공유되므로, 모든 `Exam` 인스턴스가 같은 값을 공유하게 되는 치명적인 오류가 발생합니다.

```python
class Grade:
    def __init__(self):
        self._value = 0  # 문제 발생 지점: 모든 인스턴스가 이 변수를 공유함

    def __set__(self, instance, value):
        # ... 검증 로직 ...
        self._value = value 

```

* **결과:** `first_exam`의 점수를 바꾸면 `second_exam`의 점수도 같이 바뀝니다.

### 시도 2: 인스턴스별 딕셔너리 저장 (메모리 누수 발생)

각 인스턴스를 키(Key)로 하여 값을 따로 저장하는 방식입니다. 데이터 공유 문제는 해결되지만 **메모리 누수(Memory Leak)**가 발생합니다.

```python
class Grade:
    def __init__(self):
        self._values = {} # 일반 딕셔너리 사용

    def __set__(self, instance, value):
        # ... 검증 로직 ...
        self._values[instance] = value # 강한 참조 발생

```

* **문제점:** `_values` 딕셔너리가 `instance`에 대한 **강한 참조(Strong Reference)**를 유지합니다. 따라서 `Exam` 인스턴스가 더 이상 필요 없어 삭제되더라도, `Grade` 클래스가 잡고 있어 가비지 컬렉터(Garbage Collector)가 메모리를 회수하지 못합니다.

### 시도 3: WeakKeyDictionary 사용 (최종 해결책)

`weakref` 모듈의 `WeakKeyDictionary`를 사용합니다.

```python
from weakref import WeakKeyDictionary

class Grade:
    def __init__(self):
        # 약한 참조를 사용하는 딕셔너리
        self._values = WeakKeyDictionary()

    def __get__(self, instance, instance_type):
        if instance is None:
            return self
        return self._values.get(instance, 0)

    def __set__(self, instance, value):
        if not (0 <= value <= 100):
            raise ValueError('점수는 0과 100 사이입니다')
        # 인스턴스가 메모리에서 해제되면, 이 딕셔너리 항목도 자동으로 삭제됨
        self._values[instance] = value

```

* **장점:** `WeakKeyDictionary`는 객체에 대해 **약한 참조(Weak Reference)**를 사용합니다. 런타임에 해당 객체를 가리키는 강한 참조가 사라지면, 가비지 컬렉터가 메모리를 회수할 수 있으며 딕셔너리에서도 해당 키가 자동으로 삭제됩니다.



## 4. 요약

1. **디스크립터 사용:** 반복되는 프로퍼티 로직(검증 등)을 캡슐화하여 코드 중복을 제거합니다.
2. **동작 방식:** `__get__`과 `__set__` 메서드를 통해 속성 접근을 제어합니다.
3. **메모리 안전성:** 디스크립터 내부에서 인스턴스 상태를 저장할 때는 `WeakKeyDictionary`를 사용하여 메모리 누수를 방지해야 합니다.
