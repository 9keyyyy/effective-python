# 37 내장 타입을 여러 단계로 내포시키기보다는 클래스를 합성하라


## TL;DR

- 딕셔너리, 긴 튜플, 다른 내장 타입이 복잡하게 내포된 데이터를 값으로 사용하는 딕셔너리는 만들면 안됨
- 완전한 클래스가 제공하는 유연성이 필요하지 않고 가벼운 불변 데이터 컨테이너가 필요할 땐 `namedtuple` 사용
- 내부 상태를 표현하는 딕셔나리가 복잡해지면 이 **데이터를 관리하는 코드를 여러 클래스로 나눠**서 재작성해야 함

## 1단계: 단순한 딕셔너리

### 요구사항
학생의 성적 리스트를 저장하고 평균을 계산

### 데이터 구조
```python
{
  "아이작 뉴턴": [90, 95, 85],
  # ...
}
```

### 구현
```python
class SimpleGradebook:
    def __init__(self):
        self._grades = {}

    def add_student(self, name):
        self._grades[name] = []

    def report_grade(self, name, score):
        self._grades[name].append(score)

    def average_grade(self, name):
        grades = self._grades[name]
        return sum(grades) / len(grades)
```

### 사용 예시
```python
book = SimpleGradebook()
book.add_student('아이작 뉴턴')
book.report_grade('아이작 뉴턴', 90)
book.report_grade('아이작 뉴턴', 95)
book.report_grade('아이작 뉴턴', 85)

print(book.average_grade('아이작 뉴턴'))
# 출력: 90.0
```

**평가:** ✅ 간단하고 명확함

---

## 2단계: 중첩된 딕셔너리

### 요구사항
학생의 과목별 성적 리스트 저장

### 데이터 구조
```python
{
  "알버트 아인슈타인": {
    "수학": [75, 65],
    "체육": [90, 95]
  },
  # ...
}
```

### 구현
```python
from collections import defaultdict

class BySubjectGradebook:
    def __init__(self):
        self._grades = {}  # 외부 dict

    def add_student(self, name):
        self._grades[name] = defaultdict(list)  # 내부 dict

    def report_grade(self, name, subject, grade):
        by_subject = self._grades[name]
        grade_list = by_subject[subject]
        grade_list.append(grade)

    def average_grade(self, name):
        by_subject = self._grades[name]
        total, count = 0, 0
        for grades in by_subject.values():
            total += sum(grades)
            count += len(grades)
        return total / count
```

### 사용 예시
```python
book = BySubjectGradebook()
book.add_student('알버트 아인슈타인')
book.report_grade('알버트 아인슈타인', '수학', 75)
book.report_grade('알버트 아인슈타인', '수학', 65)
book.report_grade('알버트 아인슈타인', '체육', 90)
book.report_grade('알버트 아인슈타인', '체육', 95)

print(book.average_grade('알버트 아인슈타인'))
# 출력: 81.25
```

**평가:** ⚠️ 아직은 관리 가능하지만 복잡도 증가

---

## 3단계: 과도하게 중첩된 구조

### 요구사항
각 점수에 가중치 추가 (중간고사, 기말고사 > 쪽지 시험, 수행 평가)

### 데이터 구조
```python
{
  "알버트 아인슈타인": {
    "수학": [(75, 0.05), (65, 0.15), (80, 0.80)],
    "체육": [(100, 0.40), (85, 0.60)]
  },
  # ...
}
```

### 구현
```python
class WeightedGradebook:
    def __init__(self):
        self._grades = {}

    def add_student(self, name):
        self._grades[name] = defaultdict(list)

    def report_grade(self, name, subject, score, weight):
        by_subject = self._grades[name]
        grade_list = by_subject[subject]
        grade_list.append((score, weight))

    def average_grade(self, name):
        by_subject = self._grades[name]
        score_sum, score_count = 0, 0

        for subject, scores in by_subject.items():
            subject_avg, total_weight = 0, 0

            for score, weight in scores:
                subject_avg += score * weight
                total_weight += weight

            score_sum += subject_avg / total_weight
            score_count += 1

        return score_sum / score_count
```

### 사용 예시
```python
book = WeightedGradebook()
book.add_student('알버트 아인슈타인')
book.report_grade('알버트 아인슈타인', '수학', 75, 0.05)
book.report_grade('알버트 아인슈타인', '수학', 65, 0.15)
book.report_grade('알버트 아인슈타인', '수학', 70, 0.80)
book.report_grade('알버트 아인슈타인', '체육', 100, 0.40)
book.report_grade('알버트 아인슈타인', '체육', 85, 0.60)

print(book.average_grade('알버트 아인슈타인'))
# 출력: 80.25
```

**문제점:**
- ❌ `average_grade` 메서드에 이중 반복문
- ❌ 위치 기반 인자가 많아 사용하기 어려움
- ❌ 코드 가독성과 유지보수성 저하

---

## 튜플의 문제점

### 위치 기반 접근의 한계

```python
grades = []
grades.append((95, 0.45))
grades.append((85, 0.55))
total = sum(score * weight for score, weight in grades)
total_weight = sum(weight for _, weight in grades)
average_grade = total / total_weight
```

### 요구사항 추가 시
점수마다 선생님의 피드백을 추가해야 한다면?

```python
grades = []
grades.append((95, 0.45, '참 잘했어요'))
grades.append((85, 0.55, '조금 만 더 열심히'))
total = sum(score * weight for score, weight, _ in grades)
total_weight = sum(weight for _, weight, _ in grades)
average_grade = total / total_weight
```

**결론:** 튜플 원소가 3개 이상이면 다른 접근 방법 필요

---

## 해결책: namedtuple 사용

### namedtuple의 장점

```python
from collections import namedtuple

Grade = namedtuple('Grade', ('score', 'weight'))
```

- ✅ 위치 기반 인자와 키워드 인자 모두 지원
- ✅ 인덱스로 접근 가능
- ✅ 이터러블
- ✅ 명확한 필드 이름

### 주의사항
- ❌ 디폴트 인자 불가능
- ℹ️ 선택적 프로퍼티가 많은 경우 `dataclasses` 모듈 사용 권장

---

## 클래스로 리팩터링

### 클래스 계층 구조

```
Gradebook
  └─ Student
       └─ Subject
            └─ Grade (namedtuple)
```

### Grade 클래스

```python
from collections import namedtuple

Grade = namedtuple('Grade', ('score', 'weight'))
```

### Subject 클래스

```python
class Subject:
    def __init__(self):
        self._grades = []

    def report_grade(self, score, weight):
        self._grades.append(Grade(score, weight))

    def average_grade(self):
        total, total_weight = 0, 0
        for grade in self._grades:
            total += grade.score * grade.weight
            total_weight += grade.weight
        return total / total_weight
```

### Student 클래스

```python
from collections import defaultdict

class Student:
    def __init__(self):
        self._subjects = defaultdict(Subject)

    def get_subject(self, name):
        return self._subjects[name]

    def average_grade(self):
        total, count = 0, 0
        for subject in self._subjects.values():
            total += subject.average_grade()
            count += 1
        return total / count
```

### Gradebook 클래스

```python
class Gradebook:
    def __init__(self):
        self._students = defaultdict(Student)

    def get_student(self, name):
        return self._students[name]
```

### 사용 예시

```python
book = Gradebook()
albert = book.get_student('알버트 아인슈타인')

math = albert.get_subject('수학')
math.report_grade(75, 0.05)
math.report_grade(65, 0.15)
math.report_grade(70, 0.80)

gym = albert.get_subject('체육')
gym.report_grade(100, 0.40)
gym.report_grade(85, 0.60)

print(albert.average_grade())
# 출력: 80.25
```

---

## 리팩터링 결과 비교

### 장점
- ✅ **캡슐화**: 각 클래스가 자신의 책임만 관리
- ✅ **명확한 인터페이스**: 각 계층의 역할이 분명함
- ✅ **유지보수성**: 코드 수정이 용이함
- ✅ **확장성**: 새로운 기능 추가가 쉬움
- ✅ **가독성**: 코드 이해가 쉬움

### 단점
- ⚠️ **보일러플레이트 증가**: 코드량이 늘어남

---

## 핵심 원칙 정리

1. **내포 1단계**: 딕셔너리와 리스트 사용 OK
2. **내포 2단계**: 신중하게 사용, 복잡도 관리 필요
3. **내포 3단계 이상**: 즉시 클래스로 리팩터링
4. **튜플 원소 3개 이상**: namedtuple 또는 클래스 사용
5. **의존 관계 트리**: 하위 계층부터 클래스로 분리
