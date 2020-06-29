# 오목 AI

![오목](https://user-images.githubusercontent.com/65106267/85946756-99c58580-b981-11ea-9c92-986b8c4c6952.jpg)

## overview

- 예측 네트워크와 평가 네트워크를 이용하여 몬테카를로 트리탐색을 시행한다.
- 예측 네트워크는 오목 데이터를 통해 다음수를 예측하도록 학습되었으며 평가 네트워크는 현재의 상태에 대한 가치를 평가하도록 학습되었다.
- 5목을 두거나 4목을 막거나 43, 44(외통수) 등의 몇몇 특정 상태에서는 실수를 방지하기 위해 자체 알고리즘을 통해 둔다.
- 성능 향상을 위하여 트리탐색에서 43, 44와 같은 외통수 상태의 노드에서는 평가 네트워크의 추론 대신에 상황에 맞는 값이 노드에 메겨진다. (-1 혹은 1)
- 트리탐색 120회를 기준으로 온라인 오목어플에서 1급~2단의 유저들을 상대로 3승2패의 성능을 보였다.

### 진행과정 및 설명

- 우선 유연한 내부처리와 신경망과의 연동을 위해서 직접 오목 프로그램을 구현하였다. (오목에는 렌주룰이 적용되었다.)
- 그 다음으로는 오목 데이터에서 여러가지의 feature들을 추출하는 함수들을 구현하였다.
- 그 후에는 renju.net의 오목 데이터를 이용하여 신경망을 학습시켰다.
- 학습시킨 신경망은 주어진 상태에서 다음 수를 예측하는 네트워크와 주어진 상태의 가치를 평가하는 네트워크이다.
- 렌주룰에서는 금수로 인하여 흑백의 플레이방식 및 전략이 다르므로 흑과 백의 네트워크를 따로 학습시켰다.
- 예측 네트워크의 추론 결과를 오목 프로그램의 gui에 시각화하는 작업과, 예측 네트워크를 기반으로 오목을 두는 ai를 구현하였다.
- 예측 네트워크를 기반으로 오목을 두는 ai는 predict.py 파일의 predict_p() 함수이다.
- 예측 네트워크만으로 착수를 하게 되면 중반 이후 실수할 가능성이 높아 몇몇 특정 상황에서는 자체 알고리즘에 따라 두게 된다. (overview의 내용 참고)
- 이후에는 앞서 학습시킨 두 신경망을 이용하는 트리탐색 알고리즘을 구현하였다.
- 트리탐색 과정에서는 평가값이 높은 노드를 선택하며 leaf 노드까지 내려간 뒤, 해당 노드의 가치를 평가하고 예측 네트워크를 이용하여 자식노드를 생성한다. (PRUNING_COUNT에 따라 가지치기) https://github.com/ssof12/omok/blob/d7fd987d3e7f046f76b6e6f0a061eac21cc93f90/mcts.py#L166
- 노드의 가치평가는 자식노드가 없는 경우 평가 네트워크로 추론한 수치이며 자식노드가 있는 경우는 자식노드들의 가치를 방문 횟수만큼 각각 더한 뒤 자식노드 방문횟수의 합으로 나누어 평균을 낸 값이다. https://github.com/ssof12/omok/blob/d7fd987d3e7f046f76b6e6f0a061eac21cc93f90/mcts.py#L161
- 트리탐색도 마찬가지로 몇몇 특정 상황에서는 자체 알고리즘에 따라 둔다.
- 노드의 가치 평가에서 몇몇 특정 상태의 노드에서는 신경망의 추론 대신 자체적으로 수치를 대입한다. (overview의 내용을 참고)
- 앞서 구현하였던 예측 네트워크를 기반으로 착수하는 predict_p()와 트리 탐색을 기반으로 착수하는 mcts_action()간의 게임에서는 불과 트리탐색 횟수 10회로 테스트 하였음에도 트리 탐색 기반 AI가 26승3패1무로 우수한 성능을 보였다. (test.py 파일을 통해 테스트, mcts.py 파일의 MCTS_COUNT 값을 수정하여 트리탐색 횟수를 조절 가능)
- 사람과의 대결에서는 온라인 오목 어플에서 1급~2단 사이의 유저들을 상대로 3승2패를 거두었다. (트리탐색 횟수 120회 설정)


### state.py

- 오목 상태에 대한 데이터를 담는 클래스이다.
- self.black, self.white는 각각 흑돌과 백돌의 배치인 15x15 행렬이며 self.turn은 누구의 차례인지에 대한 bool 값이다. 흑의 차례면 True, 백의 차례면 False이다.
- 렌주룰에 따라 둘 수 없는 수들과 승패를 판정하며 자체 알고리즘 및 feature 추출에 필요한 다양한 함수들이 포함되어있다. (그 결과 1400줄이나 되어버렸는데 state 클래스의 구조를 세분화해서 분리시키면 더 좋을 것 같다.)


### game.py

- 게임을 진행시키는 클래스이다.
- 착점이 주어지면 next함수는 다음 state로 진행시킨다.


### gui.py

- pygame으로 구현된 오목게임이다.
- 모든 이미지 및 내용물은 자체제작되었다.
- mcts_action()함수가 호출되는 위치를 찾아 predict.py 파일의 predict_p() 함수로 바꾸면 컴퓨터가 트리탐색 대신 예측 네트워크만 이용해서 두게 설정할 수 있다.


### train_policy.py
- 딥 컨볼루션 신경망의 형태이며 자세한 구조는 해당 파일의 코드 상단부를 참조
- 입력형태는 15x15x2의 오목판 상태이며 흑과 백의 규칙 및 전략이 서로 다르므로 각각 따로 학습시켰다.
- 흑의 예측 네트워크는 pb, 백의 예측 네트워크는 pw이다.


### train_value.py
- 구조는 예측 네트워크와 출력부를 제외하고 같다. 자세한 구조는 해당 파일의 코드 상단부를 참조
- 입력형태는 15x15x2의 오목판 상태이며 흑과 백의 규칙 및 전략이 서로 다르므로 각각 따로 학습시켰다.
- 흑의 평가 네트워크는 vb, 백의 평가 네트워크는 vw이다.


### mcts.py
- 몬테카를로 트리탐색을 수행하는 함수를 담고있다.
- mcts_action() 함수를 통해 간단하게 다음 action을 받아올 수 있으며 파라미터는 4개의 신경망 모델(pb, pw, vb, vw)과 state이다.
- MCTS_COUNT의 값을 조절하여 트리탐색의 횟수를 변경할 수 있고 PRUNING_COUNT의 값을 조절하여 트리확장 시의 노드의 갯수를 변경할 수 있다.


### predict.py
- 예측 네트워크와 평가 네트워크 추론에 사용되는 함수와 예측 네트워크를 이용하여 착수하는 함수로 구성되어있다.
- 신경망 시각화, 예상 승률 출력 / 예측 네트워크만으로 착수에 사용된다.


### test.py
- 예측 네트워크만으로 착수하는 경우와 트리탐색을 사용하여 착수하는 경우간의 성능비교를 위한 테스트 파일이다.
- 10회만 트리탐색을 시행해도 26승3패1무의 결과로 트리탐색이 우수한 성능을 보였다.


### feature.py
- 오목판의 상태에서 여러 feature들을 추출하는 함수를 담고있다.
- 눈에 띄는 성능향상을 보이지 않아 신경망의 input에 추가적인 feature들을 사용하지는 않았지만 추후 feature들을 변경하여 테스트해볼 수 있을 것이다.


### Requirements
- pygame, tensorflow (신경망 학습의 경우 케라스 일부가 필요할 수도 있음)
- renju.net의 오목 데이터
