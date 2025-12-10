# 13 슬라이싱보다는 나머지를 모두 잡아내는 언패킹을 사용하라
## TL;DR
- 별표식(*) 사용하면 언패킹 패턴에 대입되지 않는 모든 부분을 리스트로 받음
- 별표식은 앞, 중간, 뒤 어디든 가능하며 0개 또는 그 이상이 들어감 (0개일 때는 빈 list)
- 리스트를 나눌 경우, 슬라이싱 및 인덱싱보다는 별표식으로 불필요한 부분들을 잡아내는 것이 실수 여지를 줄일 수 있음

### 기본 언패킹
- 시퀀스 길이를 미리 알아야 함
- 때문에 제일 앞 원소 한개와 그 외로 나누고자 할 시에 주로 슬라이싱 이용

```python
oldest = car_ages_descending[0]
second_oldest = car_ages_descending[1]
others = car_ages_descending[2:]
print(oldest, second_oldest, others)

>>>
20 19 [15, 9, 8, 7, 6, 4, 1, 0]

```
### 문제점?
- 이런 경우 off-by-one-error(인덱스에서 1 차이 나는 오류) 위험 높음
- 인덱스가 하나 변경되면 동시에 여러 군데에서 연쇄적으로 범위를 바꿔야 함

### 해결책: 별표식

```python
oldest, second_oldest, *others = car_ages_descending
print(oldest, second_oldest, others)

>>>
20 19 [15, 9, 8, 7, 6, 4, 1, 0]

```

- 중간이나 앞에서도 쓸 수 있음
  ```python
  oldest, *others, youngest = car_ages_descending
  print(oldest, youngest, others)
  
  *others, second_youngest, youngest = car_ages_descending
  print(youngest, second_youngest, others)
  
  >>>
  20 0 [19, 15, 9, 8, 7, 6, 4, 1]
  0 1 [20, 19, 15, 9, 8, 7, 6, 4]
  
  ```
- 별표만 쓰는 것은 불가
  ```python
  *others = car_ages_descending
  
  >>>
  Traceback ...
  SyntaxError: starred assignment target must be in a list or tuple
  ```
- 두 개 이상 쓸 수 없음
  ```python
  first, *middle, *second_middle, last = [1, 2, 3, 4]
  
  >>>
  Traceback ...
  SyntaxError: two starred expressions in assignment
  ```
- 계층 구조로 쓸 때에는 서로 다른 부분에서 동시에 별표를 쓸 수 있음
  ```python
  car_inventory = {
      '시내': ('그랜저', '아반떼', '티코'),
      '공항': ('제네시스 쿠페', '소나타', 'K5', '엑센트'),
  }
  ((loc1, (best1, *rest1)),
   (loc2, (best2, *rest2))) = car_inventory.items()
  print(f'{loc1} 최고는 {best1}, 나머지는 {len(rest1)} 종')
  print(f'{loc2} 최고는 {best2}, 나머지는 {len(rest2)} 종')
  
  >>>
  시내 최고는 그랜저, 나머지는 2 종
  공항 최고는 제네시스 쿠페, 나머지는 3 종
  ```

### 별표 이용한 테이블 처리

```python
def generate_csv():
   yield('날짜, '제조사, '모델', '연식', '가격)
   ...

```

- 언패킹 이용시
  ```python
  all_csv_rows = list(generate_csv())
  header = all_csv_rows[0]
  rows = all_csv_rows[1:]
  print('CSV 헤더:', header)
  print('행 수:', len(rows))
  
  >>>
  CSV 헤더: ('날짜', '제조사', '모델', '연식', '가격')
  행 수: 200
  ```
- 별표식 이용시
  ```python
  it = generate_csv()
  header, *rows = it
  print('CSV 헤더:', header)
  print('행 수:', len(rows))
  
  >>>
  CSV 헤더: ('날짜', '제조사', '모델', '연식', '가격')
  행 수: 200
  ```
