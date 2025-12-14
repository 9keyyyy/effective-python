# 17 내부 상태에서 원소가 없는 경우를 처리할 때는 setdefault보다 defaultdict를 사용하라
## TL;DR

- key로 어떤 값이 들어올지 모르는 딕셔너리를 관리해야 한다면 collections 내장 모듈에 있는 defaultdict 사용
- 임의의 key가 들어있는 딕셔너리가 어떻게 생성됐는지 모르는 경우 setdefault를 사용하는 것도 고려

## setdefault
dictionary key가 없을 때 get을 사용하는 방법이 유용하지만 경우에 따라 setdefault가 가장 빠른 지름길일 수 있음

```python
visits = {
    '미국': {'뉴욕', '로스엔젤레스'},
    '일본': {'하코네'},
}

```

dictionary 안에 나라 이름이 들어 있는지 여부와 관계없이 각 집합에 새 도시를 추가하는 경우,

- `visits.setdefault('프랑스', set()).add('칸')` : 짧게 가능
- `get`: 비교적 길어짐
  ```python
  if (Japan := visits.get('일본')) is None:
      visits['일본'] = Japan = set()
  Japan.add('교토')
  
  ```


직접 딕셔너리 생성을 제어한다면

```python
class Visits:
    def __init__(self):
        self.data = {}

    def add(self, country, city):
        city_set = self.data.setdefault(country, set())
        city_set.add(city)

```

이렇게 하면 setdefault의 복잡도를 감춰주며 더 나은 인터페이스를 제공

```python
visits = Visits()
visits.add('러시아', '예카테린부르크')
visits.add('탄자니아', '잔지바르')
print(visits.data)

>>>
{'러시아': {'예카테린부르크'}, '탄자니아': {'잔지바르'}}
```

하지만, 아래의 이유로 Visits.add는 여전히 이상적이지 않음
- setdefault라는 method 이름은 여전히 헷갈려서 코드를 읽는 사람은 이해가 어려움
- 주어진 나라가 data에 있든 없든 관계없이 호출할 때마다 새로 set 인스턴스를 만들어 비효율적

## defaultdict

```python
from collections import defaultdict

class Visits:
    def __init__(self):
        self.data = deafultdict(set)

    def add(self, country, city):
        self.data[country].add(city)

visits = Visits()
visits.add('영국', '바스')
visits.add('영국', '런던')
prints(visits.data)

>>>
defaultdict(<class 'set'>, {'영국': {'바스', '런던'}})

```

구현이 짧고 간결해짐
