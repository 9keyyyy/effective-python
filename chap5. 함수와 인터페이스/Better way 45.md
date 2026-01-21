# 45 애트리뷰트를 리팩터링하는 대신 @property를 사용하라 

## TL;DR

* `@property` 데코레이터를 사용하면 단순한 숫자 속성에 대해 실시간(on-the-fly) 계산 로직을 주입할 수 있음
* 기존 인터페이스나 함수명을 변경하지 않고도 속성 참조 및 설정 시 필요한 연산을 수행 가능
* 초기 설계가 미흡한 인터페이스를 유지하면서도 시스템을 점진적으로 개선할 때 매우 유효함


## 예시: 물통(Bucket) 할당량 관리 시스템

#### 1. 초기 모델 및 문제점

* **기능**: 물통 채우기, 시간 기반 초기화, 할당량 소비
* **문제**: 현재 남은 양(`quota`)만 기록하므로, 원래 채웠던 양이나 총 사용량을 알 수 없음

```python
from datetime import datetime, timedelta

class Bucket:
    def __init__(self, period):
        self.period_delta = timedelta(seconds=period)
        self.reset_time = datetime.now()
        self.quota = 0

    def fill(self, amount):
        now = datetime.now()
        if (now - self.reset_time) > self.period_delta:
            self.quota = 0
            self.reset_time = now
        self.quota += amount

    def deduct(self, amount):
        if self.quota - amount < 0:
            return False
        self.quota -= amount
        return True

```

#### 2. @property를 활용한 개선 모델

* **변경 사항**: 내부적으로 `max_quota`와 `quota_consumed`를 관리하여 사용 이력을 보존
* **인터페이스**: 외부 사용자는 기존과 동일하게 `.quota` 속성을 통해 접근

```python
class NewBucket:
    def __init__(self, period):
        self.period_delta = timedelta(seconds=period)
        self.reset_time = datetime.now()
        self.max_quota = 0
        self.quota_consumed = 0

    @property
    def quota(self):
        # 남은 양을 실시간으로 계산
        return self.max_quota - self.quota_consumed

    @quota.setter
    def quota(self, amount):
        delta = self.max_quota - amount
        if amount == 0:
            # 리셋 시나리오
            self.quota_consumed = 0
            self.max_quota = 0
        elif delta < 0:
            # 새로운 할당량 충전 시나리오
            assert self.quota_consumed == 0
            self.max_quota = amount
        else:
            # 할당량 소비 시나리오
            assert self.max_quota >= self.quota_consumed
            self.quota_consumed += delta

```


## 개선 결과 비교

| 항목 | 초기 모델 (Bucket) | 개선 모델 (NewBucket) |
| --- | --- | --- |
| **속성 관리** | `self.quota` 단일 변수 | `max_quota`, `quota_consumed` 분리 |
| **정보 가용성** | 잔량만 확인 가능 | 최초 할당량 및 누적 소모량 확인 가능 |
| **데이터 무결성** | 단순 대입으로 오류 발생 가능 | Setter 내부 `assert`를 통한 로직 검증 |


## 기억해야할 내용

* `@property`는 기존 인스턴스 속성에 새로운 기능을 추가하거나 제어 로직을 넣을 때 유용함
* 데이터 모델이 빈약한 상태에서 시작된 실제 프로젝트의 코드를 점진적으로 고도화하는 데 효과적임
* 다만, 클래스의 거의 모든 속성을 `@property`로 감싸기 시작한다면, 이는 속성 관리가 아닌 클래스 설계 자체를 리팩토링해야 한다는 신호임

