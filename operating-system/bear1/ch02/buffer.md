# 버퍼
## 버퍼 개요
* 버퍼는 속도에 차이가 있는 두 장치 사이에서 그 차이를 완화하는 역할을 한다.
* 하드디스크에는 메모리 버퍼가 있다. 하드디스크의 사양이 1TB, 7200rpm, 32MB로 표기된다면 32MB가 버퍼의 용량을 의미하는 것이다.
* 버퍼는 하드웨어적으로 사용되는 개념이 아니다. 소프트웨어적으로 사용되는 대표적인 예는 **동영상 스트리밍**이다.
## 스풀
* 버퍼와 유사한 개념이나, CPU와 입출력장치가 독립적으로 동작하도록 고안된 소프트웨어적인 버퍼이다.
* 대표적인 예는 프린터에 사용되는 스풀러이다.
* 기존의 버퍼랑 차이가 있는데, 스풀러는 하나의 인쇄물이 완료될 때까지 다른 인쇄물이 끼어들 수 없는 프로그램 간 배타적인 특성이 있다.
# 캐시
## 캐시 개요
* 캐시는 메모리와 CPU 간의 속도 차이를 완화하기 위해 메모리의 데이터를 미리 가져와 (prefetch) 저장해두는 임시 장소이며 버퍼의 일종이다.
* 캐시는 CPU 안에 있으며, CPU 내부 버스의 속도로 작동한다.
* CPU는 메모리에 접근하기 전 캐시를 먼저 방문하여 원하는 데이터가 있는 지 찾아본다. 캐시에서 원하는 데이터를 찾았다면 **캐시 히트**라 하며, 그 데이터를 바로 사용한다.
* 원하는 데이터가 없으면 메모리로 가서 데이터를 찾는데 이를 **캐시 미스**라 한다.
* 일반적인 컴퓨터의 캐시 적중률은 90% 내외이다.
## 즉시 쓰기와 지연 쓰기
캐시에 있는 데이터가 변경되면 메모리에 있는 원래의 데이터 역시 변경해야 하는데, 이를 반영하는 방법으로 즉시 쓰기와 지연 쓰기가 있다.
### 즉시 쓰기
* 캐시에 있는 데이터가 변경되면 이를 즉시 메모리에 반영한다.
* 장점: 메모리의 최신 값이 항상 유지됨, 정전 등의 상황에서도 데이터를 유실하지 않음
* 단점: 메모리와의 빈번한 데이터 전송으로 성능 저하 발생
### 지연 쓰기
* 캐시에 있는 데이터가 변경되면 변경된 내용을 모아 주기적으로 반영한다.
* 장점: 메모리와의 데이터 전송 횟수가 줄어들어 시스템의 성능을 향상할 수 있음
* 단점: 메모리와 캐시된 데이터 사이의 불일치 발생 가능성 있음
## L1, L2, L3 캐시
* 프로그램의 명령어는 크게 어떤 작업을 할지 나타내는 **명령어 부분**과 작업 대상인 **데이터 부분**으로 나눌 수 있다.
* L1 캐시 (특수 캐시): 명령어와 데이터를 구분하여 가져오는 캐시. CPU 레지스터와 직접 연결된다.
* L2 캐시 (일반 캐시): 명령어와 데이터 구분 없이 모든 자료를 가져오는 캐시
* L3 캐시: 멀티코어일 경우 존재하는 캐시이다. 가장 낮은 수준의 캐시이며, L1, L2가 코어 당 존재한다면 L3은 모든 코어가 공유한다.
# 저장장치의 계층 구조
저장장치의 계층 구조는 속도가 빠르고 값이 비싼 저장 장치를 CPU에 가까운 쪽에 두고, 값이 싸고 용량이 큰 저장 장치를 반대쪽에 배치하여 적당한 가격으로 빠른 속도와 큰 용량을 동시에 얻는 방법이다.
* 레지스터
* 캐시
* 메모리
* 저장장치
# 인터럽트
## 폴링
* CPU가 직접 입출력장치에서 데이터를 가져오거나 내보내는 방식
* CPU가 명령어 해석과 실행이라는 본래 역할 외 모든 입출력까지 관여해야 하므로 작업 효율이 떨어진다.
## 인터럽트
* CPU의 작업과 저장장치의 데이터 이동을 독립적으로 운영하기 위한 방식
* 입출력 관리자가 CPU에 보내는 완료 신호. CPU는 입출력 관리자에게 작업 지시를 내리고 완료 신호를 받으면 하던 일을 중단하고 옮겨진 데이터를 처리한다.
## 인터럽트의 동작 방식
1. CPU가 입출력 관리자에게 입출력 명령을 보낸다. (입출력 요청)
2. 입출력 관리자는 명령 받은 데이터를 메모리에 가져다놓거나 메모리에 있는 데이터를 저장장치로 옮긴다. (데이터 전송)
3. 데이터 전송이 완료되면 입출력 관리자는 완료 신호를 CPU에 보낸다. (인터럽트 발생)
* 인터럽트는 컴퓨터에 연결된 많은 주변장치 중 어떤 것의 작업이 끝났는지를 CPU에 알려주기 위해 인터럽트 번호를 사용한다.
* CPU는 입출력 관리자에게 여러 개의 입출력 작업을 동시에 시킬 수 있는데, 여러 작업이 동시에 완료될 때 이를 하나의 배열로 만든다. 이를 인터럽트 벡터라 한다.
## 직접 메모리 접근
* 메모리 영역은 CPU만 직접 접근이 가능하여 입출력 관리자는 접근할 수가 없다.
* 이를 해결하기 위해 입출력 관리자에게 CPU의 허락 없이 메모리에 접근할 수 있는 권한이 필요한데 이를 직접 메모리 접근(DMA)이라고 한다.
* CPU의 작업과 입출력 작업을 분리하기 위해 메모리의 일정 공간을 입출력에 할당하는데 이를 메모리 매핑 입출력이라고 한다.
* CPU와 DMA의 메모리 접근이 동시에 이루어질 때 CPU의 작업 속도가 느리므로 입출력장치에 메모리 접근을 양보하는 데, 이러한 상황을 사이클 훔치지(Cycle Stealing)이라 한다.

