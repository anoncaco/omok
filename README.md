# omok AI

## overview

- 기본적인 수들 및 VCT, VCF (연속공격)를 탐색하고, 해당하는 수가 없으면 신경망을 통해 착수한다.
- 신경망을 통하여 착수할 때에는 몬테카를로 트리 탐색을 활용한다.
- 신경망의 학습은 알파제로 알고리즘과 유사한 방식으로 강화학습을 통해 이루어진다.


### state

- 흑돌과 백돌이 놓여진 상태가 각각 15x15 크기의 이차원 리스트에 표현된다.
- 현재 누구의 차례인지를 뜻하는 bool 값을 포함한다. (True:흑, False:백)
- 렌주룰 구현 및 연속공격 탐지를 위하여 돌들의 모양을 인식하는 메소드들이 포함된다.


### network

- Resnet으로 구성되어있다.
- 입력형태는 15x15x3이다.
- 입력정보는 흑돌과 백돌의 위치정보, 누구의 차례인지 (흑이면 모두 1, 백이면 모두 0)이다.
- 출력 형태는 15x15의 확률분포 (위치별 정책값)와 0~1의 스칼라값 (현재 상태에서의 예상 승률)이다.


### mcts

- mcts에서 자식 노드를 선택할 때 신경망의 출력값을 이용한다.
- 별도로 탐색너비를 줄여 약 5개 정도의 후보군 중에서 트리탐색을 진행한다.


### 티칭 기능

- AI를 통하여 오목 데이터를 생산한 후, 이를 분석하여 티칭기능을 구현한다.
- 오목의 26주형에 대한 통계 및 예상 승률 등을 분석할 예정이다.