### 1. 문제 정의

\- classification 문제가 어떤 사진에 어떤 물체가 있는지 여부를 판별하는 문제라 하면, localiazation 문제는 그 사진에서 그 물체가 어떤 위치에 있는지, 얼만큼의 영역을 차지하는지를 구하는 regression 문제의 하나라고 할 수 있다. object detection은 사진 안에서 찾고자 하는 모든 물체의 존재 여부와 있을 경우 그 위치와 차지하는 영역의 크기를 모두 구하는 문제라 할 수 있다.

\- supervisied learning을 할 때 주어지는 정답 결과 벡터는 다음과 같이 구성된다.

- 물체의 존재 여부

- 물체의 중심점의 좌표(x, y)

- 물체가 차지하는 영역의 가로, 세로 길이

- 물체의 정답 클래스가 표시돼 있는 one-hot code



### 2. object detection 분야의 발전사

\- 2012년 deep learning이 성과를 내기 이전만 해도 object detection 분야는 연구 역사가 길었음에도 불구하고 발전에 정체가 있어 기술적 한계를 넘기 어려울 거라는 예상이 지배적이었으나, deep learning 기술이 발전하면서 급속도로 발전했다. 

\- object detection 관련하여 가장 먼저 등장한 deep learning 알고리즘은 RCNN(Regions with Convolutional Neuron Networks features)로, 2014년 발표되었다. RCNN은 먼저 물체가 있을 법한 위치를 파악하고(region proposal) 그 다음에 그 물체의 클래스를 분류하는 'two-stage detection' 구조를 갖는다. 이후 단계를 나누지 않고 한번에 물체를 detection하는 YOLO(You only look once), SSD(single shot multibox detector) 등의 'one-stage detection' 알고리즘이 등장했는데, two-stage detection은 정확도가 높지만 연산 속도가 느리고 one-stage detection은 서로 정반대 장단점을 갖는 것으로 여겨졌다. 그러나 최근에는 one-stage detection 알고리즘들이 연산속도가 빠르면서도 정확도도 높은 성과를 보여 평가가 바뀌고 있다. (다만, segmentation 같은 문제에서는 mask RCNN 같은 two-stage detection 알고리즘이 더 성능이 좋다.)


### 3. RCNN의 구조

\- 간단히 생각하면 사진을 여러 개의 격자로 쪼갠 후 각 격자에 대해 classification을 하는 알고리즘을 생각할 수 있으나 이는 연산이 너무 오래 걸린다. object detection은 연산하는 시간을 줄이는 방향으로 발전해 왔으며, RCNN 같은 two-stage detection 알고리즘도 이를 위해 region proposal, classification으로 나뉜 단계를 갖는 것이다. 

\- RCNN 계열 알고리즘인 Faster RCNN의 단계를 다음과 같이 세분할 수 있다.

(1) region proposal

- 사진을 base network(VGG16)를 통과시켜, object가 있을 것으로 보이는 영역을 모두 찾는다. classification을 수행하는 pre-trained된 모델들은 입력으로 받은 사진을 파악하는 데 사용되는 feature map을 가지고 있으므로, 이를 base network로 이용하여 object가 있을 것으로 보이는 영역을 찾을 수 있다.

(2) non-max suppression: 선택된 영역 중 겹치는 부분을 제거한다. 

(3) classification & bounding box regression

- 최종 선택된 영역들에 대하여 분류작업을 수행하고, 물체가 있을 것 같은 위치를 보다 정확히 찾는다. 최종 선택된 영역들을 원본 사진에서 잘라내 크기를 조정하고(ROI pooling) FC layer에 각각 통과시키면 각각에 대한 결과를 얻을 수 있다.

### 4. RPN(regional proposal network)

#### 1) 개요

\- \\(224 \times 224 \times 3\\) 크기의 사진을 입력으로 받아 VGG16 모델을 통과시키면, FC layer를 통과하기 전에 최종적으로 \\(7 \times 7\\) 크기의 feature map 512개를 얻는다. (\\(7 \times 7\\) 크기의 feature map의 한 픽셀은 원본 입력 사진에서 \\(32 \times 32\\) 크기의 영역에 관한 정보를 담고 있다.) 이 512개의 feature map이 RPN의 입력이 된다.

\- 512개의 feature map이 RPN을 거치면, 최종적으로 \\(7 \times 7\\)개의 픽셀 하나당 2k개, 4k개(단, k는 그 픽셀이 갖는 anchor box의 개수)의 채널을 갖는 벡터 하나씩이 출력된다. (이때 추천되는 region의 개수는 총 49k개가 된다. 이 region들에 대해 non-max suppression을 수행한다.) 이들 벡터에는 입력으로 들어온 사진에서 그 위치에 물체가 있는지, 있다면 그 위치와 영역은 어떤 값을 갖는지에 관한 정보가 담기게 된다.



#### 2) anchor box

\- 어떤 픽셀의 objectness를 판단할 때 물체가 실제 차지하는 영역(ground truth)과 그 픽셀이 차지하는 영역 사이의 일치율을 측정하는 지표로 두 영역의 교집합의 크기를 이용할 수 있는데, 이때 분모로 합집합을 사용하는 방법이 있고(이를 IoU, intersection over union라 한다) 그 픽셀만을 사용하는 방법이 있다. 

\- ground truth가 그 픽셀보다 훨씬 큰데 그 픽셀이 ground truth 안에 포함되는 경우에 전자와 후자의 방법을 사용하면 각각 다음과 같은 문제가 있을 수 있다.

(1) 분모가 합집합인 경우

- 이 경우 IoU의 값이 너무 작게 나와 그 픽셀에 그 물체에 관한 정보가 없다고 판별해버릴 위험이 있다.


(2) 분모가 그 픽셀인 경우

- 이 경우 objectness는 true가 되나 이렇게 되면 픽셀이 너무 작기 때문에 bounding box regression에서 오차가 커져 이를 수행하는 신경망의 성능이 크게 떨어진다. 

- 이러한 경우의 objectness를 true로 정해 버리면, 픽셀이 너무 작아 물체의 아주 작은 부분에 관한 정보만 담기게 되며, 이렇게 되면 실제로는 전혀 다른 물체라 하더라도 그 부분이 포함되면 그 물체를 실제 클래스와 다른 클래스로 분류하는 게 참이라고 학습하게 될 위험이 있다. 


\- 위와 같은 문제가 발생하기 때문에, 한 픽셀에 하나의 벡터를 구하는 것이 아니라 한 픽셀에 여러 개의 anchor box를 배정하고 각 anchor box마다 벡터를 구한다. 그 픽셀을 중심으로 가로세로 길이가 1:1, 1:2, 2:1 등의 크기를 갖고 면적이 각각 다른 anchor box를 지정하고 이에 대하여 objectness 등을 예측하게 하면 위와 같은 문제를 해결할 수 있다. 



#### 3) non-max suppression

\- 다음과 같은 알고리즘으로 중복 region을 제거한다.

(1) RPN을 거쳐 얻은 49k개의 region 중에서, objectness score가 가장 높은 region을 선택한다.

(2) 그 region을 기준으로, 그 region과 다른 region들 사이에 IoU를 계산한다. 만약 이 값이 특정 값 이상이라면 그 region을 기준 region에 중복되는 region으로 보고 제거한다.

(3) 제거하고 남는 region의 개수가 특정 개수 이하가 될 때까지 1-2단계를 반복 수행한다.

\- 이렇게 얻은 region의 크기는 정사각형도 아니고 제각각이지만, classification과 regression을 수행하는 FC layer의 입력의 크기는 고정돼 있으므로 여기서 얻은 region의 크기를 조정하여 FC layer의 입력으로 전달해야 한다. 크기를 조정할 때 가로축 또는 세로축을 강제로 늘리거나 줄이는 작업을 수행하게 된다(ROI pooling).



### 5. YOLO

\- Faster RCNN이 \\(224 \times 224 \times 3\\) 크기의 사진 입력을 받아 \\(7 \times 7\\)개의 픽셀 하나당 2k개, 4k개의 채널을 갖는 두 개의 벡터를 얻는다면, YOLO v1의 경우 \\(7 \times 7\\)개의 픽셀 하나당 30개의 채널을 갖는 벡터 하나를 얻는다. YOLO v1은 픽셀 하나당 단 두 개의 anchor box만 사용하기 때문이다. 구체적으로, 다음과 같은 방식으로 anchor box를 정하여 각각의 anchor box에 관한 정보를 얻는다.

(1) ground truth의 중심점을 포함하는 픽셀의 anchor box 중 하나만 confidence(물체가 존재할 확률)를 1로 정한다. 일단 그 픽셀이 ground truth의 중심점을 포함한다면, 여기서 세로로 긴 anchor box와 가로로 긴 anchor box 두 개를 갖고 ground truth와의 IoU를 계산하여, 더 큰 쪽의 confidence를 1로 정한다. 그밖의 모든 anchor box는 confidence를 0으로 정한다.

(2) confidence가 1인 anchor box에 대하여 region의 위치와 크기에 관한 regression을 수행한다. (이 과정은 YOLO v1에서는 없었으나 v2에서 추가되었다.)

\- YOLO v2, v3가 나오면서 v1에 비해 약간의 변화가 생기게 되었다. 예를 들어 YOLO v2는 anchor box 5개를, v3는 9개를 사용한다.

\- YOLO v3의 경우 feature map의 해상도를 하나만 쓰는 게 아니라 여러 개를 쓴다(feature pyramid). 해상도의 크기가 너무 작으면 한 픽셀에 지나치게 많은 정보가 들어가 있어 픽셀의 크기보다 ground truth가 더 작은 물체를 감지할 수 없기 때문이다.


### 6. SSD

\- SSD는 YOLO v3의 경우와 같이 여러 해상도의 feature map을 쓰는데, base network를 거친 높은 해상도의 feature map을 더 해상도가 낮은 feature map을 만드는 신경망에 입력으로 차례로 넣는다. 이 신경망 내부를 보면 신경망을 하나씩 거칠 때마다 feature map의 해상도가 점점 낮아진다는 특징이 있다. 한번 해상도가 낮아질 때마다 그 feature map은 해상도를 낮추는 다음 신경망의 입력이 되는 동시에 classification을 수행하는 신경망으로 그대로 입력으로 전달되기도 한다.

