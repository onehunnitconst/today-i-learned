# 2장: 의미 있는 이름 실습

## 실습 코드

### 예제 1: 백준 10250번 문제인 "ACM 호텔"의 풀이 코드
```python
t = int(input())

while t > 0:
  t = t - 1
  h, w, n = list(map(int, input().split()))
  floor = 1
  room = 1
  if h > 1 and w > 1:
    floor = (n-1) % h + 1
    room = (n-1) // h + 1
  elif h == 1:
    room = n
  elif w == 1:
    floor = n

  result = floor*100 + room
  print(result)
```

### 예제 2: 도서관 프로젝트
* 현재 진행중인 도서관 리포지토리에서도 적절한 예시가 있으면 가져올 예정임

## 의도를 분명히 밝혀라

### 코드에서 짐작하기 어려운 사항
* 예제 1에서의 `t`, `h`, `w`, `n`은 변수의 의도가 불분명하다.
* `floor`와 `room`은 왜 1로 초기화되는가?
* `h`와 `w`가 1보다 클때 floor와 room은 새롭게 연산된다.
* `h`와 `w`가 1일때는 floor와 room에 n 값이 대입된다.

### 변수의 의도
* `t`: 테스트 케이스
* `h`: 호텔의 층수
* `w`: 각 층의 방수
* `n`: 해당 손님의 방문 순서
* `floor`: n번째 손님이 묵을 층
* `room`: n번째 손님이 묵을 방 번호

### 바꾼 이름?
* `t` -> `totalTestCase`
* `h` -> `heightOfHotel`
* `w` -> `widthOfHotel`
* `n` -> `customerArrivalNumber`
* `floor` -> `roomFloor`
* `room` -> `roomNumber`

```python
totalTestCase = int(input())

for _ in range(0, totalTestCase):
  heightOfHotel, widthOfHotel, customerArrivalNumber = list(map(int, input().split()))
  roomFloor = 1
  roomNumber = 1
  if h > 1 and w > 1:
    roomFloor = (customerArrivalNumber-1) % heightOfHotel + 1
    roomNumber = (customerArrivalNumber-1) // heightOfHotel + 1
  elif h == 1:
    roomNumber = customerArrivalNumber
  elif w == 1:
    roomFloor = customerArrivalNumber

  result = floor*100 + room
  print(result)
```

### 값 1의 의미
* `elif h == 1`에서 1은 호텔이 1층일 때
  * `SINGLE_FLOOR` 상수로 정의
* `elif w == 1`에서 1은 각 층의 방이 1호실만 있을 때
  * `SINGLE_ROOM_NUMBER` 상수로 정의
* `floor`와 `room`이 1로 초기화되는 건 출입구와 가장 가까운 방이 101호기 때문!
  * `FIRST_FLOOR`와 `FIRST_ROOM_NUMBER` 정의

```python
SINGLE_FLOOR = 1
SINGLE_ROOM_NUMBER = 1
FIRST_FLOOR = 1
FIRST_ROOM_NUMBER = 1

totalTestCase = int(input())

for _ in range(0, totalTestCase):
  heightOfHotel, widthOfHotel, customerArrivalNumber = list(map(int, input().split()))
  roomFloor = SINGLE_FLOOR
  roomNumber = SINGLE_ROOM_NUMBER
  if h > FIRST_FLOOR and w > FIRST_ROOM_NUMBER:
    roomFloor = (customerArrivalNumber - 1) % heightOfHotel + 1
    roomNumber = (customerArrivalNumber - 1) // heightOfHotel + 1
  elif h == FIRST_FLOOR:
    roomNumber = customerArrivalNumber
  elif w == FIRST_ROOM_NUMBER:
    roomFloor = customerArrivalNumber

  result = floor*100 + room
  print(result)
```

### 맥락 개선
