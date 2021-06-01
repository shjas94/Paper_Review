# <center>**RARE**</center>

## Overview
* Real World의 text는 irregular한 형태인 경우가 많음. 하지만 text recognizer는 tightly bounded regular text image가 input으로 주어졌을 때 최고의 성능을 발휘함.

* **RARE**는 이러한 문제를 극복하기 위한 architecture
* **RARE**는 크게 **Spatial Transformer Network**, **Sequence Recognition Network**의 두 부분으로 구성되어 있음
* **STN**을 통하여 이미지를 더 readable한 형태로 rectify하고 **SRN**으로 전달
* **SRN**은 Sequence Recognition approach를 통하여 텍스트를 인식한다.

## Method

### Spatial Transformer Network(STN)
* 자세한 내용은 [Spatial Transformer Networks](https://arxiv.org/pdf/1506.02025v3.pdf) 논문 참조
* STN은 input image $I$를 rectified image $I'$로 변환시킴
* **Localization Network**
    * $K$개의 fiducial point의 좌표를 regression task를 통해서 잡아준다.
    * 상수 $K$는 짝수이고 각 좌표는 $\mathbf{C} = [\bold{c}_1, \dots,\bold{c}_K] \in \mathbf{R}^{2 \times K}$로 표현한다.
    * $\mathbf{C}$의 $k$번째 column $\mathbf{c}_k = [x_k, y_k]^\top$이다.
    * 또한 각 좌표들은 $[1,-1]$로 normalized되어 있다.
* **Grid Generator**
    * Grid Generator에서는 변환 파라미터들을 계산하고, sampling grid를 생성한다.
    * 파라미터들은 행렬 $\mathbf{T}$로 계산.(수식 생략)
    * $I'$의 grid pixel $\mathcal{P}' = \{\mathbf{p}'_i\}_{i=1,\dots,N}$를 통하여 $I$의 grid pixel $\mathcal{P} = \{\mathbf{p}_i\}_{i=1,\dots,N}$를 생성한다. 
* **Sampler** 
    * Sampler에서 픽셀 값 $\mathbf{p}_i'$는 원본 이미지의 픽셀 값 $\mathbf{p}_i$의 인접한 값들로부터 bilinearly interpolated된다.
    * 최종 rectified image는 $I' = V(\mathcal{P}, I)$로 나타낸다. 이 때, $V$는 bilinear sampler를 의미하며 역시 미분가능한 모듈이다.

### Sequence Recognition Network(SRN)
* target words는 **순서를 이루는** 글자들이라고 볼 수 있음
* 따라서 저자들은 recognition problem을 Sequence recognition problem으로 인지하고 모델링함.
* SRN은 rectified image $I'$를 입력으로 받고 sequential representation을 추출한다. 최종적으로 이로부터 단어를 추출한다.
* SRN은 Attention-Based Model이며 Encoder와 Decoder로 구성되어 있다.
* Encoder에서는 input image $I'$로부터 Sequential Representation을 추출하며 Decoder에서는 재귀적으로 sequence를 생성한다.  
*  **Encoder**
   * convolutional layer와 recurrent network를 결합함.
   * 우선 CNN layer를 통과시켜 $D_{conv} \times{H_{conv}} \times{W_{conv}}$ 크기의 feature map을 생성한다. 
   * 다음으로, map_to_sequence operation을 통하여 이러한 feature map을 각각의 element가 $D_{conv}W_{conv}$의 dimension을 갖는 $W_{conv}$크기의 벡터로 변환한다.
   * 최종적으로 변환된 feature vector를 2-layer의 Bidirectional LSTM에 투입하여 $\mathbf{h} = (\mathbf{h_1, \dots, h_L}),\;\;\;L = W_{conv}$의 output을 얻는다.
*  **Decoder**
    * Decoder는 Encoder에서 생성한 feature vector를 바탕으로 글자를 순차적으로 생성함.
    * 네트워크는 GRU로 구성되며 Attention mechanism을 활용한다.