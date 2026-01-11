# 41 기능을 합성할 때는 믹스인 클래스를 사용하라


## TL;DR

- 믹스인은 자식 클래스가 사용할 메소드만 정의한 클래스
- 범용적으로 활용 가능한 제너릭 메소드로 구성
- 복잡한 기능 구현 시 다중 상속 대신 믹스인 합성 사용



## 믹스인이란?

- 파이썬은 다중 상속을 지원하지만, 이는 지양하는 것이 좋음
- 다중 상속의 편의성과 캡슐화가 필요할 때 믹스인을 사용하면 됨

### 특징

- 자식 클래스가 사용할 기능 메소드만 정의
- 자체 어트리뷰트가 없어 `__init__` 메소드 불필요
- `isinstance()`, `hasattr()` 등으로 다양한 타입에 범용적 활용 가능
- 여러 하위 클래스에서 재사용 가능한 제네릭 메소드로 구성



## 기본 예시: ToDictMixin

파이썬 객체를 딕셔너리로 직렬화하는 믹스인 클래스

```python
class ToDictMixin:
    def to_dict(self):
        return self._traverse_dict(self.__dict__)

    def _traverse_dict(self, instance_dict):
        output = {}
        for key, value in instance_dict.items():
            output[key] = self._traverse(key, value)
        return output

    def _traverse(self, key, value):
        if isinstance(value, ToDictMixin):
            return value.to_dict()
        elif isinstance(value, dict):
            return self._traverse_dict(value)
        elif isinstance(value, list):
            return [self._traverse(key, i) for i in value]
        elif hasattr(value, '__dict__'):
            return self._traverse_dict(value.__dict__)
        else:
            return value
```

### 사용 예시

```python
class BinaryTree(ToDictMixin):
    def __init__(self, value, left=None, right=None):
        self.value = value
        self.left = left
        self.right = right

tree = BinaryTree(10)
print(tree.to_dict())
# 출력: {'value': 10, 'left': None, 'right': None}
```



## 문제 상황: 순환 참조

부모 노드를 참조하는 트리 구조에서 순환 참조 문제가 발생할 수 있음

```python
class BinaryTreeWithParent(BinaryTree):
    def __init__(self, value, left=None, right=None, parent=None):
        super().__init__(value, left=left, right=right)
        self.parent = parent

root = BinaryTreeWithParent(10)
root.left = BinaryTreeWithParent(7, parent=root)
root.right = BinaryTreeWithParent(9, parent=root)
print(root.to_dict())
# RecursionError: maximum recursion depth exceeded
```

### 원인

에러가 발생하는 '무한 굴레' 단계: root.to_dict()를 호출하면 아래와 같은 일이 벌어짐
- 1단계 (root 처리): root의 속성 중 left가 있기 때문에 left.to_dict()를 호출
- 2단계 (left 자식 처리): left 노드의 속성에는 parent가 있음 (이 parent는 아까 본 root -> root.to_dict()를 호출)
- 3단계 (다시 root 처리): root 속성을 보니 또 left가 있음 -> left.to_dict()를 호출
- 4단계... (무한 반복): root → left → root → left... 이 과정이 컴퓨터 메모리가 꽉 찰 때까지 반복

### 해결 방법

메소드 오버라이드로 순환 참조를 방지

```python
class BinaryTreeWithParent(BinaryTree):
    def __init__(self, value, left=None, right=None, parent=None):
        super().__init__(value, left=left, right=right)
        self.parent = parent

    def _traverse(self, key, value):
        if isinstance(value, BinaryTreeWithParent) and key == 'parent':
            return value.value  # 전체 인스턴스 대신 value만 반환
        else:
            return super()._traverse(key, value)
```

### 결과

```python
root = BinaryTreeWithParent(10)
root.left = BinaryTreeWithParent(7, parent=root)
root.right = BinaryTreeWithParent(9, parent=root)
print(root.to_dict())
# 출력:
# {
#   "value": 10,
#   "left": {"value": 7, "left": None, "right": None, "parent": 10},
#   "right": {"value": 9, "left": None, "right": None, "parent": 10},
#   "parent": None
# }
```

---

## 믹스인 합성

단순한 동작으로부터 복잡한 기능을 만들 때 여러 믹스인을 합성하여 사용

### JSON 직렬화 믹스인

```python
import json

class JsonMixin:
    @classmethod
    def from_json(cls, data):
        kwargs = json.loads(data)
        return cls(**kwargs)

    def to_json(self):
        return json.dumps(self.to_dict())
```

- 일반 메서드 (to_json): 이미 만들어진 객체(인스턴스)가 가진 데이터를 밖으로 내보낼 때 사용 (이미 존재해야 함)
- 클래스메서드 (from_json): 아직 객체가 없는데, 외부 데이터(JSON)를 바탕으로 새로운 객체를 찍어낼 때 사용
   - from_json의 cls는 메서드를 호출한 클래스 자신을 가리킴
### 믹스인 합성 예시

`ToDictMixin`과 `JsonMixin`을 합성하여 JSON 직렬화 기능을 추가합니다.

```python
class Switch(ToDictMixin, JsonMixin):
    def __init__(self, ports=None, speed=None):
        self.ports = ports
        self.speed = speed

# 사용 예시
serialized = '{"ports": 10, "speed": 10}'
deserialized = Switch.from_json(serialized)
assert json.loads(serialized) == json.loads(deserialized.to_json())
```

- Switch.from_json(...)이라고 호출하면 cls는 Switch 클래스가 됨
- return cls(**kwargs)는 내부적으로 `return Switch(ports=10, speed=10)`과 똑같이 동작하게 됨


## 핵심 포인트

**`__dict__` 어트리뷰트**
- 파이썬 객체의 내장 특수 어트리뷰트
- 객체의 모든 어트리뷰트를 딕셔너리로 반환

**믹스인 설계 원칙**
- 자체 상태를 갖지 않음
- 범용적이고 재사용 가능한 메소드 제공
- 필요시 메소드 오버라이드로 커스터마이징 가능
- 여러 믹스인을 조합하여 복잡한 기능 구현
