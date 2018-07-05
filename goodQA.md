# Perceptron

weight은 입력의 영향도를 결정, bias는 활성화 허들을 조절하는 

# 좋은 feature란 무엇인가요. 이 feature의 성능을 판단하기 위한 방법에는 어떤 것이 있나요

가장 좋은 분석 결과를 만들어내는 데이터의 컬럼, Filter와 Wrapper, Regulation가 있음

Filter와 Wrapper의 차이점
- Filter는 종속변수와의 상관 관계를 통해 변수의 관련성을 측정하지만 Wrappe는 실제로 모델을 만들어 변수의 집합의 유용성을 측정한다.

- Filter는 모델을 학습하는 것을 포함하지 않기 때문에 훨신 속도가 빠르다

- Filter는 기능의 하위 집합을 평가하는데 통계적인 방법을 사용하고, Wrapper는 교차 유효성 검사를 사용한다. 

- Wrapper의 일부 기능을 사용하면 모델을 Filter의 하위 집합을 사용하는 경우와 비교할 때 과적합이 발생하기 쉽다 

Regulation의 경우 피처 선택이 아니라 패널티를 부과하여 결과에 작은 영향을 주는 피처요소를 0으로 만듬

# “상관관계는 인과관계를 의미하지 않는다”

까마귀가 울면 손님이 찾아온다

까마귀를 울리면 손님을 얻는다

인과관계를 증명하기 위한 방법

# A/B 테스트의 장점과 단점, 그리고 단점의 경우 이를 해결하기 위한 방안에는 어떤 것이 있나요?

상관관계를 알수 있다

단점
- 지역최적점에 머물수 있다, 추가적인 트리형태의 탐색
- 

# 데이터 간의 유사도를 계산할 때, feature의 수가 많다면(예: 100개 이상), 이러한 high-dimensional clustering을 어떻게 풀어야할까요?

# Cross Validation은 무엇이고 어떻게 해야하나요?

((훈련데이터, 검증데이터), 테스트데이터) 나눠서 학습/검증

k구역으로 나눠서 트레인과 테스트셋을 정하고 지속적으로 트레인과 테스트셋을 바꿔서 학습/검증

# 회귀 / 분류시 알맞은 metric은 무엇일까요?

회귀

Root Mean Square Error (RMSE) : 편차 제곱의 평균에 루트를 씌운 값.

이걸 기준으로 성능을 올리면, 이는 표준편차를 기준으로 하기때문에, 큰 에러를 최대한 줄이는 방향으로 학습을 함.

-> ex) 정답이 9인 경우
9, 9, 6, 9 보다 8, 8, 8 ,8 를 좋게 평가


mean absolute error (MAE) : 편차에 절대값을 씌운것의 평균

단순 편차의 절대값의 평균임. 그러므로 RMSE와 달리 작은 에러에 더 민감함.

-> ex) 정답이 9인 경우
8, 8, 8 ,8 보다 9, 9, 6, 9 를 좋게 평가

분류

Accuracy

(TP+TN)/(TP + TN + FP +FN)

Precision

TP/(TP+FP)

Recall

TP / (TP + FN)

F1

2*(Precision*Recall)/(Precision+Recall), Precision과 Recall 계산 분모에 FP와 FN이 있으므로 오탐이 작아야 값이 커진다

# 정규화를 왜 해야할까요? 정규화의 방법은 무엇이 있나요?

# Local Minima와 Global Minima에 대해 설명해주세요.

미분에 의존하는 Gradient Descent 방식으로 업데이트의 방향을 찾았을때 미분값이 0이 되는곳을 만나면 방향이 없어지므로 탐색이 멈춤 따라서 다른 최적의 해(Global Minima)가 있더라도 Local Minima에 빠지게됨

Local Minima의 새로운 시각 http://darkpgmr.tistory.com/148

# 차원의 저주에 대해 설명해주세요

데이터가 세밀(차원)이 증가할 수록 학습에 더 많은 데이터가 필요하여 학습 성능을 떨어뜨린다.

# dimension reduction기법으로 보통 어떤 것들이 있나요?

LDA

PCA

LDA PCA 차이 설명 https://wikidocs.net/5957

# PCA는 차원 축소 기법이면서, 데이터 압축 기법이기도 하고, 노이즈 제거기법이기도 합니다. 왜 그런지 설명해주실 수 있나요?

http://sherry-data.tistory.com/2

# 딥러닝은 무엇인가요? 딥러닝과 머신러닝의 차이는?

딥러닝도 머신러닝의 일종, 딥러닝은 인공신경망을 바탕으로 깊어진 레이어를 통해 추상화를 시도하는 알고리즘 집합

 “A fast learning algorithm for deep belief nets”
 
 # 왜 갑자기 딥러닝이 부흥했을까요?
 
 데이터저장, 네트워크, 분산처리 이 모든것을 논리적으로 쉽게 다룰수 있게 해주는 클라우드 기술이 핵심
 
 # Cost Function과 Activation Function은 무엇인가요
 
 Activation Function 다음 레이어에 입력값으로 변환하기 위한 
 
 Cost Function 현재와 이전의 차를 비교하여 더 나은값을 찾기 위한 측정함수
 
 # 뉴럴넷의 가장 큰 단점은 무엇인가? 이를 위해 나온 One-Shot Learning은 무엇인가?
 
 ‘Matching Networks for One Shot Learning‘
 
 # Weight Initialization 방법에 대해 말해주세요. 그리고 무엇을 많이 사용하나요?
 
 다음 그래프는 Xavier 초기화 방법과 ReLU 함수를 결합했을 때의 그래프 입니다. 출력값이 0 으로 수렴하고 평균과 표준편차 모두 0으로 수렴하는 모습을 확인할 수 있습니다. ReLU 함수를 사용할 경우에는 Xavier 초기화 방법을 사용할 수 없다는 의미입니다.
 
Sigmoid 나 Tanh 를 사용할 경우에는 Xavier 초기화를 

ReLU 를 사용할 경우에는 He 초기화를 사용하시면 됩니다.

# 요즘 Sigmoid 보다 ReLU를 많이 쓰는데 그 이유는?

선형은 1개의 일정한 기울기를 가지는 반면, 비선형은 1개 이상의 기울기를 가집니다. 즉, 기울기의 변화가 없는 것은 선형, 기울기가 임의의 조건에 의해 변한다면 비선형입니다.

Vanishing gradient problem은 activation function을 선택하는 문제에 의존적으로 일어난다. sigmoid나, tanh 등 요즘 많이들 사용하는 activation function들은 매우 비선형적인 방식으로 그들의 input을 매우 작은 output range로 짓이겨넣는다('squash')

예를 들어서, sigmoid는 실수 범위의 수를 [0, 1]로 맵핑한다. 그 결과로 매우 넓은 input space 지역이 극도로 작은 범위로 맵핑되어버린다.

이렇게 되어 버린 input space에서는 큰 변화가 있다고 하더라도, output에는 작은 변화를 보이게 된다.  gradient(기울기)가 작기 때문이다.



이러한 현상은 우리가 서로(??)의 꼭대기 층에 그러한 비선형성을 여러개 레이어로 쌓을 때 더욱 악화된다.

예를들어, 첫 레이어에서 넓은 input region을 작은 output region으로 맵핑하고, 그것이 2차 3차 레이어로 갈수록 더 심각하게 작은 region으로 맵핑되는 경우이다.

그 결과로 만약 첫 레이어 input에 대해 매우 큰 변화가 있다고 하더라도 output을 크게 변화시키지 못하게 된다.



우리는 이러한 문제를 해결하기 위해 짓이겨 넣는식('squashing')의 특징을 갖지 않는 activation function을 사용할 수 있다.

ReLU(Rectified Linear Unit - max(0, x))가 잘 선택되는 편이다.

# Deeplearning

https://www.youtube.com/watch?v=n7DNueHGkqE&list=PLlMkM4tgfjnLSOjrEJN31gZATbcj_MpUm&index=22
