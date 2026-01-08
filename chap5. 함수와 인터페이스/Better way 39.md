# 39 객체를 제너릭하게 구성하려면 @classmethod를 통한 다형성을 활용하라

## TL;DR

1. **파이썬 기본 생성자는 `__init__` 하나뿐**
2. **`@classmethod`로 추가 생성자 정의 가능**
3. **`cls` 파라미터 = 호출한 클래스 자체**
4. **다형성 + 클래스 메소드 = 제네릭하고 확장 가능한 설계**


## 다형성(Polymorphism)
같은 인터페이스를 가진 여러 클래스가 각각 다른 방식으로 동작하는 것

**장점:**
- 코드 재사용성 향상
- 유지보수 용이
- 확장성 증가

## 문제 상황: 제네릭하지 않은 코드

### Before: 특정 클래스에 종속된 코드

```python
import os
from threading import Thread

# 도우미 함수들이 특정 클래스에 하드코딩됨
def generate_inputs(data_dir):
    for name in os.listdir(data_dir):
        yield PathInputData(os.path.join(data_dir, name))  # PathInputData로 고정

def create_workers(input_list):
    workers = []
    for input_data in input_list:
        workers.append(LineCountWorker(input_data))  # LineCountWorker로 고정
    return workers

def mapreduce(data_dir):
    inputs = generate_inputs(data_dir)
    workers = create_workers(inputs)
    return execute(workers)
```

**문제점:**
- 새로운 InputData 타입 추가 시 → `generate_inputs` 함수를 새로 작성해야 함
- 새로운 Worker 타입 추가 시 → `create_workers` 함수를 새로 작성해야 함
- 조합이 늘어날수록 함수가 기하급수적으로 증가



## 해결책: @classmethod 활용

### After: 제네릭한 코드

```python
class GenericInputData:
    def read(self):
        raise NotImplementedError

    @classmethod
    def generate_inputs(cls, config):
        """각 하위 클래스가 자신의 방식으로 객체 생성"""
        raise NotImplementedError


class PathInputData(GenericInputData):
    def __init__(self, path):
        super().__init__()
        self.path = path

    def read(self):
        with open(self.path) as f:
            return f.read()

    @classmethod
    def generate_inputs(cls, config):
        data_dir = config['data_dir']
        for name in os.listdir(data_dir):
            yield cls(os.path.join(data_dir, name))  # cls = PathInputData


class GenericWorker:
    def __init__(self, input_data):
        self.input_data = input_data
        self.result = None

    def map(self):
        raise NotImplementedError

    def reduce(self, other):
        raise NotImplementedError

    @classmethod
    def create_workers(cls, input_class, config):
        """어떤 InputData, Worker든 처리 가능"""
        workers = []
        for input_data in input_class.generate_inputs(config):
            workers.append(cls(input_data))  # cls = 호출한 Worker 클래스
        return workers


class LineCountWorker(GenericWorker):
    def map(self):
        data = self.input_data.read()
        self.result = data.count('\n')

    def reduce(self, other):
        self.result += other.result


# 제네릭 mapreduce 함수
def mapreduce(worker_class, input_class, config):
    workers = worker_class.create_workers(input_class, config)
    return execute(workers)


# 사용 예시
config = {'data_dir': 'test_inputs'}
result = mapreduce(LineCountWorker, PathInputData, config)
```

**장점:**
- 새로운 클래스 추가 시 기존 코드 수정 불필요
- 모든 조합을 단일 `mapreduce` 함수로 처리
- 확장성 극대화



## 실전 활용 예시

### 새로운 기능 추가하기

```python
# 1. URL에서 데이터 읽기
class URLInputData(GenericInputData):
    def __init__(self, url):
        super().__init__()
        self.url = url
    
    def read(self):
        import requests
        return requests.get(self.url).text
    
    @classmethod
    def generate_inputs(cls, config):
        for url in config['urls']:
            yield cls(url)


# 2. 단어 세기 Worker
class WordCountWorker(GenericWorker):
    def map(self):
        data = self.input_data.read()
        self.result = len(data.split())
    
    def reduce(self, other):
        self.result += other.result


# 3. 모든 조합 사용 가능 (함수 추가 없이!)
result = mapreduce(LineCountWorker, PathInputData, {'data_dir': 'files'})
result = mapreduce(LineCountWorker, URLInputData, {'urls': ['http://...']})
result = mapreduce(WordCountWorker, PathInputData, {'data_dir': 'files'})
result = mapreduce(WordCountWorker, URLInputData, {'urls': ['http://...']})
```



## Instance Method vs Class Method

| | Instance Method | Class Method |
|---|---|---|
| **데코레이터** | 없음 | `@classmethod` |
| **첫 번째 인자** | `self` (인스턴스) | `cls` (클래스) |
| **접근 가능** | 인스턴스 속성/메소드 | 클래스 속성/메소드 |
| **호출 방법** | `객체.method()` | `클래스.method()` |
| **용도** | 객체별 동작 | 대체 생성자, 팩토리 메소드 |



