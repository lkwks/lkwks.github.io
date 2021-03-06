---
title: CS231n - image classification
---


### 1. 이미지 간 유사도 평가 방법

#### 1) L1 distance

$$
d_1 (I_1, I_2) = \sum_P \left| I_1^P - I_2^P \right|
$$

\- 크기가 같은 이미지의 같은 위치에 있는 각 픽셀이 갖는 값의 차이 절대값을 단순 합하여 이를 두 이미지의 L1 distance로 정의한다.

\- L1 distance는 좌표계가 달라지면 값도 달라진다는 문제가 있는데, 좌표계가 달라질 일 없는 문제에서라면 L1 distance가 L2 distance보다 좋은 결과를 낼 수도 있다.


#### 2) L2 distance

$$
d_2 (I_1, I_2) = \sqrt{\sum_P ( I_1^P - I_2^P )^2}
$$

\- 크기가 같은 이미지의 같은 위치에 있는 각 픽셀이 갖는 값의 차이 제곱을 단순 합하여 이를 두 이미지의 L1 distance로 정의한다.

\- L2 distance는 L1 distance와 달리 좌표계가 달라져도 값이 달라지지 않는다. 요소들 간 의미를 잘 모르는 상황이라면 L2 distance가 더 유용할 수 있다. 



### 2. k-NN(nearest neighbor) classifier


#### 1) NN classifier

\- 다음 두 단계로 동작한다.

(1) 훈련 단계: 특별한 작업을 수행하지 않고, 입력된 이미지의 클래스를 단지 기억하기만 한다. 단순히 메모리 위치 지정만 해도 되므로 시간복잡도는 O(1)로 구현할 수도 있다.

(2) 예측 단계: 방금 입력된 이미지를 기존 기억하던 이미지 중 가장 유사한 이미지의 클래스로 분류한다. L1 distance로 유사도를 평가할 경우 시간복잡도는 O(n)이다.

\- 실제 분류문제에서는 훈련하는 데 오래 걸려도 예측은 빠른 시간 내에 해결할 것을 요하며, 실제로 CNN 등은 그러한 구조를 갖고 있다. 그러나 NN classfier는 정반대다. 


#### 2) k-NN classifier 개요

\- 입력 이미지와 유사도가 낮은 이웃 이미지를 k개 찾아서 이들끼리 투표를 해 분류를 결정하는 알고리즘. 투표 방식은 여러 방식이 있을 수 있으나, 실제로는 최다득표로 분류하는 게 가장 잘 동작한다.

\- 컴퓨터 비전에서는 이미지를 여러 관점에서 볼 수 있어야 한다. 이미지 자체를 공간상의 한 점으로 볼 수도 있고, 이미지의 각 픽셀을 공간상의 한 점으로 보고 이미지를 이들의 집합으로 볼 수도 있다. 

\- 분류문제에서 k-NN classifier는 정답률이 높지 않아 아주 좋은 알고리즘은 아니다. 

\- k-NN classifier는 적절한 유사도 평가 방법을 사용하면 컴퓨터 비전뿐 아니라 어떤 분야에서도 사용할 수 있어, 새로운 분야에서 간단히 시도해보기 좋은 알고리즘이다. 그러나 사실 성능이 그렇게 뛰어난 방법은 아니며, 특히 컴퓨터 비전 분야에서는 잘 쓰이지 않는 방법인데 다음 이유 때문이다.

(1) 예측에 너무 오랜 시간이 걸린다.

(2) 이미지에 대해 k-NN을 사용하려면 L1/L2 distance를 사용해야만 하는데, L1/L2 distance는 이미지의 '지각적' 유사도를 평가하는 데 그닥 적절치 못하다. (약간의 변형으로 완전히 다른 이미지가 되기도 하고, 큰 변형에도 불구하고 이미지의 내용은 거의 달라지지 않는 경우도 있는데 L1/L2 distance로 보면 이 둘이 거의 같은 유사도를 갖는 것으로 평가된다.)

(3) 훈련 단계에서 쓸 데이터셋이 전체 공간을 모두 조밀하게 커버할 정도로 충분해야 하는데, 차원수가 커지면서 필요로 하는 데이터셋의 크기도 기하급수적으로 커진다. 이러한 데이터셋을 확보하는 건 비현실적이다. 


#### 3) hyperparameter 결정 방법

\- 어떤 k를 사용할지, 어떤 유사도 평가 방법을 사용할지 같은 요소는 **훈련 단계에 들어가기 전에 결정해야 하는 요소**로서, 이러한 요소를 hyperparameter라 한다. 이들을 결정하는 문제는 **어떤 문제를 풀 것인가**에 달렸다. 간단하게 생각하면, 같은 데이터를 두고 여러 hyperparameter를 적용해 보아 최적 결과를 내는 hyperparameter를 선택하면 된다. 

(1) 훈련 단계 때 사용한 데이터를 가장 잘 분류하는 hyperparameter를 선택

- 훈련 단계 때 사용된 데이터를 입력으로 넣는다면 k=1이 k>1보다 정확도가 높아 k=1이 선택될 것. 그러나 훈련 단계 때 사용된 적 없는 새로운 데이터가 입력으로 들어온다면 k=1의 정확도는 떨어질 것이고, ML에선 훈련 단계 때 사용되지 않은 데이터가 입력으로 들어오는 상황이 더 중요.

(2) 전체 데이터셋 중 일부만 훈련 단계에서 사용하고, 나머지는 빼두었다가 최적 hyperparameter를 결정하기 위한 테스트용 이미지로 활용

- hyperparameter 결정 단계에서 사용한 데이터셋이 실제로 앞으로 입력할 이미지와 비슷한 구성을 갖는다는 보장이 없기 때문에 이 역시 좋은 방법이라 할 수 없다. hyperparameter 결정 단계에서 사용한 데이터셋에만 최적의 결과를 내는 hyperparameter에 불과할 수 있다.

(3) 전체 데이터셋을 셋으로 나누어, 하나는 훈련 단계에서 사용하고 하나는 hyperparameter 결정 단계(검증 단계)에서 사용하고 마지막 하나는 알고리즘의 성능을 딱 1번 테스트하는 목적으로만 사용

- 분류 알고리즘 문제에서는 검증 단계와 테스트 단계를 구분하는 게 중요.

(4) 먼저 테스트용 데이터를 따로 빼두고, 나머지 데이터를 n개 그룹으로 나누어 한번은 1번 그룹을 검증 데이터로 사용하고 나머지를 훈련 데이터로 사용, 한번은 2번 그룹을 검증 데이터로 사용하고 나머지를 훈련 데이터로 사용, ... 해서 각각이 내는 최적 결과를 비교('교차검증')

- 훈련 데이터로 적은 데이터를 사용하는 경우에는 흔히 쓰이는 방식이지만, 대량의 데이터로 학습을 하는 딥러닝 분야에서는 잘 쓰이지 않는 방법.



### 3. linear classification


NN(neural network)은 다양한 컴포넌트로 구축된 것으로서 CNN을 구성한다. NN을 레고 블럭으로, CNN을 그 레고 블럭을 조합해 만든 결과물로 각각 비유할 수 있다. (예를 들어, image captioning은 단순히 CNN에 RNN이라는 언어학습 알고리즘을 결합해 학습시켜 문제를 해결한다.) 그리고 linear classification은 CNN의 기반 알고리즘으로서, 간단하지만 중요한, 가장 기본이 되는 블럭 중 하나다. 따라서 linear classification의 동작 원리를 이해하는 것은 매우 중요하다.


#### 1) perceptron

\- 동물의 뉴런은 외부로부터 여러 자극을 동시에 받으며, 받아들이는 자극이 어떠한가에 따라 외부로 전기신호를 방출할지 여부가 결정된다. 머신러닝 분야에서는 여기서 영감을 얻어, 여러 개의 실수가 입력으로 주어지면 그 실수들을 모두 사용하는 연산식을 계산하여 그 결과값으로 +1 또는 -1을 출력하는 함수를 정의하고 이를 perceptron이라 이름붙였다. 

$$

f(\mathbf{x}, \mathbf{W}) = \mathbf{W} \, \mathbf{x} = 
\begin{bmatrix}
w_{1} & \cdots & w_{n} \end{bmatrix}

\begin{bmatrix}

x_1 \\
\vdots \\
x_n

\end{bmatrix}

= 
\sum_{i=1}^n w_{i} \, x_i 

$$




\- 더 구체적으로, 머신러닝에서 perceptron은 다음과 같은 특성을 갖는 것으로 정의된다. 

(1) 그 perceptron이 갖는 고유의 가중치 벡터(\\(\mathbf{W}\\))와 그 perceptron이 갖는 고유의 역치값(\\(T\\))을 갖는다. 

(2) 입력으로 주어지는 벡터(\\(\mathbf{x}\\))와 \\(\mathbf{W}\\)를 내적하여, 그 내적값이 \\(T\\)보다 크면 +1, 작으면 -1을 출력하도록 한다. (여기서는 설명을 위해 출력값이 +1/-1인 경우를 기준으로 설명했으나, 실제로는 여러 형태의 비선형 함수가 쓰인다. ReLU(rectified linear unit)라 불리는 \\(max(0, f)\\) 함수가 실제 뉴런이 동작하는 것과 가장 비슷하다 하며 가장 많이 쓰이기도 한다.)

- 만약 어떤 perceptron의 \\(T=0\\)이라면, 그 perceptron은 \\(\mathbf{x}\\)가 \\(\mathbf{W}\\)와 이루는 각도가 \\(90^{\circ}\\)보다 작을 때에 +1을 출력하고 클 때 -1을 출력한다.

- 만약 어떤 perceptron의 \\(\mathbf{W}\\)의 크기가 1이라면, 그 perceptron은 \\(\mathbf{x}\\)가 \\(\mathbf{W}\\)가 갖는 방향을 축으로 하는 좌표계에서 \\(\mathbf{W}\\) 방향 성분의 크기가 \\(T\\)보다 크면(즉, \\(\mathbf{W}\\) 벡터와 수직인 linear한 경계선/경계면의 위쪽이면) +1을 출력하고 작으면(경계선/경계면의 아래쪽이면) -1을 출력한다.

\- \\(\mathbf{W}\\)가 linear한 경계선/경계면을 의미하는 것은 내적이 단순히 linear한 연산이기 때문이다. 이 경계선/경계면을 기준으로 입력 벡터가 +1 또는 -1로 분류되기 때문에, perceptron을 입력 벡터가 이 linear한 경계선/경계면의 위쪽에 있는지 아래쪽에 있는지를 알려주는 일종의 classifier라고 할 수 있다. 이처럼 linear한 연산을 통해 입력 데이터를 여러 개의 클래스 중 어느 하나로 분류하는 것을 linear classification이라 한다.

\- perceptron이 뉴런의 특성을 따와 제안된 개념이기 때문에 뉴런도 그러할 것이라고 생각하기 쉬우나, 이런 생각은 조심할 필요가 있다. 실제 뉴런은 구조가 훨씬 복잡하고, 전기신호도 단순히 linear한 계산을 통해 방출여부를 결정하는 것이 아니기 때문이다. 



#### 2) MLP(multi-layer perceptron)

$$

f(\mathbf{x}, W) = W \, \mathbf{x} = 
\begin{bmatrix}
w_{1, 1} & \cdots & w_{1, n} \\
\vdots & \ddots & \vdots \\
w_{m, 1} & \cdots & w_{m, n} \end{bmatrix}

\begin{bmatrix}

x_1 \\
\vdots \\
x_n

\end{bmatrix}

= \begin{bmatrix}

\sum_{i=1}^n w_{1, i} \, x_i \\

\vdots \\

\sum_{i=1}^n w_{m, i} \, x_i \end{bmatrix}

$$


\- 똑같은 하나의 벡터를 여러 개의 perceptron에 동시에 대입하여 각 perceptron의 가중지 벡터와 내적한 결과값으로 새로운 하나의 벡터를 얻는 모델을 생각할 수 있다. 이는 입력 벡터와 각 perceptron 사이의 유사도를 계산하여 각 perceptron과의 유사도로 이루어진 벡터를 구한 것이다. single-layer perceptorn이 단순히 +1 또는 -1을 출력하는 문제였다면, MLP는 주어진 입력 벡터가 수개의 클래스 중 어느 클래스에 가장 유사한지를 판별하는 linear classifer라고 생각할 수 있다.

\- 이 MLP classifier는 머신러닝의 관점에서 볼 때 분류를 위하여 (k-NN classifier처럼) 모델의 학습에 사용된 모든 훈련 데이터와의 직접 비교를 필요로 하지 않으며, 분류를 위해서는 각 perceptron이 갖는 고유의 가중치 벡터를 하나로 이은 가중치 행렬을 입력 벡터와 단순히 곱하기만 한다. (그 결과값에 보정 차원에서 특정 열벡터를 합산하기도 한다.) 분류 성능의 향상을 위하여 훈련에 쓰인 데이터들에 관한 정보는 가중치 행렬에 저장되는데, 이처럼 학습 결과인 기계학습 모델이 일종의 linear function의 매개변수가 되기 때문에 이러한 기계학습 모델을 parametric model이라고도 한다.

\- 가중치 행렬의 행벡터 하나하나를 각 클래스의 주요 특징들을 모아놓은 하나의 대표 모델(흔히 템플릿이라 한다)이라고 생각할 수 있다. 여기서 소개한 linear classification은 이처럼 특정 클래스와의 유사도를 계산하는 데 있어서 딱 하나의 템플릿과만 유사도를 측정하기 때문에 성능에 한계가 있지만, 보다 복잡한 신경망 알고리즘에서는 보다 더 많은 템플릿과 유사도를 측정하므로 이러한 문제는 극복 가능하다.

\- linear classification은 parity 문제나 multi modes 문제를 해결하는 데 어려움이 있다. 이들 문제는 예를 들어 입력 데이터들이 실제로는 같은 클래스로 분류돼야 함에도 좌표공간상에서 서로 다른 영역에 밀집되어 linear한 경계로는 이들을 같은 클래스로 분류하지 못하는 경우에 해당한다. 



#### 3) neural network

\- MLP를 둘 이상 쌓는 것을 neural network라 한다. 

- 예를 들어, MLP인 임의의 선형 함수 \\(f_1 = W_1 \, \mathbf{x}\\), 임의의 비선형 함수 \\(f_2 = W_2 \, max(0, \mathbf{x})\\)가 있다 하자. 

  이때, 함수 \\(f = W_2 \, max(0, W_1 \, \mathbf{x})\\)는 두 함수 \\(f_1, f_2\\)를 결합하여 만든 함수로서 기존 MLP와 마찬가지로 classification 문제를 풀 수 있지만 \\(f_1\\)과 \\(f_2\\)의 정보를 모두 반영한 결과값을 낸다. 
  
  이처럼 여러 개의 MLP를 쌓아 기존 MLP와 마찬가지 기능을 하면서도 각각의 정보를 모두 담고 있는 것을 neural network라 한다. (사용되는 행렬의 개수 \\(n\\)을 기준으로 \\(n\\)-layer neural network라 한다. \\(n\\)-layer neural network는 결과적으로 (\\(n-1\\))개의 층이 숨겨져 있는 셈이므로, (\\(n-1\\))-hidden-layer neural network라고도 한다.)

- 단순히 linear function을 결합한 것을 neural network라 하지는 않는다. 단순히 linear function을 결합하면 그냥 또 하나의 linear function만 나올 뿐이기 때문이다. 


\- 위에서 linear classifier는 각 클래스에 해당하는 가중치 행렬의 행벡터 하나하나를 템플릿으로 볼 수 있는데 각 클래스에 해당하는 템플릿이 하나만 있어 유사도 측정에 한게가 있다고 했는데, neural network에는 여러 층의 network가 있어 각 층에서 계산한 유사도 점수를 활용하여 훨씬 더 많은 범위의 이미지 파일들을 같은 클래스로 분류하는 것이 가능해진다. 딥러닝에서는 이런 식으로 neural network를 훨씬 더 여러 층으로 쌓아나가는 경우가 흔하며, 이렇게 여러 개의 층으로 구성돼 있는 neural network를 흔히 deep MLP 또는 deep neural network라 한다.

\- 분류문제를 푸는 neural network로 들어온 입력 데이터는 각 MLP 층을 거치며 변형되는 특징 공간에서 각 MLP 층마다 고유의 위치를 갖게 된다. 이때 각 MLP 층은 그 MLP 층으로 들어오는 입력 데이터를 분류하는 그 MLP 층 고유의 특징을 의미한다고 볼 수 있다. 보통 neural network의 앞쪽에 위치한 MLP 층이 입력 데이터의 단순한 특징을 분류하며 neural network가 깊어질수록 안쪽의 MLP 층은 점점 더 복잡한 특징을 분류하는 분류기가 된다.
