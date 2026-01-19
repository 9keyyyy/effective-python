# 43 커스텀 컨테이너 타입은 collections.abc를 상속하라
## TL;DR
- **간편하게 사용할 경우**
    - 파이썬 컨테이너 타입(`list`, `dict` 등)을 직접 상속
- **커스텀 컨테이너를 제대로 구현하려는 경우**
    - 커스텀 컨테이너 타입이 **`collections.abc`에 정의된 인터페이스를 상속**하게 할 것
        - 필요한 인터페이스 및 기능을 제대로 구현하도록 보장 가능
        - 그렇지 않으면 수많은 메서드를 구현해야 함

## 간편하게 사용할 경우
### 예시) 커스텀 리스트 타입
- 빈도를 계산하는 메서드가 포함된 커스텀 리스트 타입
    - `FrequencyList`를 `list`의 하위 클래스로 만듦: `list`가 제공하는 모든 표준 함수 사용 가능
    - 필요한 기능을 제공하는 메서드를 자유롭게 추가 가능
```python
class FrequencyList(list):
    def __init__(self, members):
        super().__init__(members)

    def frequency(self):
        counts = {}
        for item in self:
            counts[item] = counts.get(item, 0) + 1
        return counts
```
```python
foo = FrequencyList(['a', 'b', 'a', 'c', 'b', 'a', 'd'])
print('길이: ', len(foo))

foo.pop()
print('pop한 다음:', repr(foo))
print('빈도:', foo.frequency())

>>>
길이: 7
pop한 다음: ['a', 'b', 'a', 'c', 'b', 'a']
빈도: {'a': 3, 'b': 2, 'c': 1}
```

## 커스텀 컨테이너를 제대로 구현하려는 경우
### 부족한 예시) 일부 메서드를 직접 구현
- `list` 처럼 인덱싱 가능한 객체를 제공하면서도 `list`의 하위 클래스로 만들고 싶지는 않은 경우
    - 이진 트리 클래스를 시퀀스(`list`, `tuple`)의 의미 구조를 사용해 다룰 수 있는 클래스
```python
class BinaryNode:
    def __init__(self, value, left=None, right=None):
        self.value = value
        self.left = left
        self.right = right
```
```python
bar = [1, 2, 3]
bar[0] # bar.__getitem__(0) 로 해석됨
```
- `BinaryNode` 클래스가 시퀀스처럼 작동하게 하려면?
    - 트리 노드를 깊이 우선 순회(depth first traverse)하는 커스텀 `__getitem__`[^1] 메서드 구현을 제공하면 됨
```python
class IndexableNode(BinaryNode):
    def _traverse(self):
        if self.left is not None:
            yield from self.left._traverse()
        yield self
        if self.right is not None:
            yield from self.right._traverse()

    def __getitem__(self, index):
        for i, item in enumerate(self._traverse()):
            if i == index:
                return item.value
        raise IndexError(f'인덱스 범위 초과: {index}')
```
```python
tree = IndexableNode(
    10,
    left=IndexableNode(
        5,
        left=IndexableNode(2),
        right=IndexableNode(
            6,
            right=IndexableNode(7))),
    right=IndexableNode(
        15,
        left=IndexableNode(11)))
```
- 위 트리를 `left`, `right` 애트리뷰트를 사용해 순회할 수도 있지만, 리스트처럼 접근도 가능
```python
print('LRR:', tree.left.right.right.value)
print('인덱스 0:', tree[0])
print('인덱스 1:', tree[1])
print('11이 트리 안에 있나?', 11 in tree)
print('17이 트리 안에 있나?', 17 in tree)
print('트리:', list(tree))

>>>
LRR: 7
인덱스 0: 2
인덱스 1: 5
11이 트리 안에 있나? True
17이 트리 안에 있나? False
트리: [2, 5, 6, 7, 10, 11, 15]
```
### 문제: `__getitem__`을 구현하는 것만으로는 리스트 인스턴스에서 기대할 수 있는 모든 시퀀스 의미 구조를 제공할 수 없음
`len` 내장 함수는 특별 메서드 `__len__`을 구현해야 제대로 작동함
```python
len(tree)

>>>
Traceback ...
TypeError: object of type 'IndexableNode' has no len()
```
```python
class SequenceNode(IndexableNode):
    def __len__(self):
        for count, _ in enumerate(self._traverse(), 1):
            pass
        return count

tree = SequenceNode(
    10,
    left=SequenceNode(
        5,
        left=SequenceNode(2),
        right=SequenceNode(
            6,
            right=SequenceNode(7))),
    right=SequenceNode(
        15,
        left=SequenceNode(11))
)

print('트리 길이:', len(tree))

>>>
트리 길이: 7
```
- 클래스가 올바른 시퀀스가 되려면 `__getitem__`, `__len__`을 구현하는 것만으로는 부족
    - 예: `count`, `index` 등

### 좋은 예시) `collections.abc`에 정의된 인터페이스를 상속해 구현
- 내장 `collections.abc` 모듈 내에 컨테이너 타입에 정의해야 하는 전형적인 메서드를 모두 제공하는 추상 기반 클래스 정의가 여럿 존재
    - 위와 같은 어려움을 덜기 위함
    - 추상 기반 클래스의 하위 클래스를 만들고 필요한 메서드 구현을 누락하면 모듈이 실수를 알려줌
    - `Set`, `MutableMapping`[^2] 등 구현해야 하는 특별 메서드가 훨씬 많은 복잡한 컨테이너 타입을 구현할 때 특히 유용 
```python
from collections.abc import Sequence

class BadType(Sequence):
    pass

foo = BadType()

>>>
Traceback ...
TypeError: Can't instantiate abstract class BadType with
    abstract methods __getitem__, __len__
```
- `collections.abc`에서 가져온 추상 기반 클래스가 요구하는 모든 메서드를 구현하면 `index`, `count`와 같은 추가 메서드 구현을 그냥 얻을 수 있음
```python
class BetterNode(SequenceNode, Sequence):
    pass

tree = BetterNode(
    10,
    left=BetterNode(
        5,
        left=BetterNode(2),
        right=BetterNode(
            6,
            right=BetterNode(7))),
    right=BetterNode(
        15,
        left=BetterNode(11))
)

print('7의 인덱스:', tree.index(7))
print('10의 개수:', tree.count(10))

>>>
7의 인덱스: 3
10의 개수: 1
```
- 파이썬에는 `collections.abc` 모듈 이외에도 객체 비교 및 정렬을 위해 사용하는 다양한 특별 메서드가 있음
    - 컨테이너/비컨테이너 클래스 모두에서 이런 특별 메서드를 구현할 수 있음(Better Way 73: 우선순위 큐로 heapq 사용 참조)

[^1]: double underscore: 줄여서 dunder라고 부르는 경우가 많음.
[^2]: `MutableMapping`:  읽기 전용과 가변 매핑의 ABC. `MutableMapping`의 서브클래스를 만들어 활용하면 `__getitem__`, `__setitem__` 등을 원하는 대로 동작하도록 커스터마이즈할 수 있음.
