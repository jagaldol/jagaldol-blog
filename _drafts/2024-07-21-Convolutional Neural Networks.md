---
layout: single
title: "[DLS]Convolutional Neural Networks"
date: 2024-07-20 13:04:00 +0900
categories: google-ml-bootcamp
---

Andrew Ng 교수님의 Coursera - Deep Learning Specialization 네번째 강의입니다.

## Computer Vision

CNN은 주로 이미지 처리에 사용된다. 따라서 Computer Vision 알고리즘도 함께 사용된다.

> 2023년에 수강했던 [computer vision 강의를 정리한 github repo](https://github.com/jagaldol/computer-vision)를 참고하자

## Convolution

(M x N)의 행렬에 대해 어떤 작은 (f x f)행렬(filter 혹은 kernel)이 적용되어 (M-f+1 x N-f+1)의 행렬을 만드는 연산이다.

만들어진 행렬의 (i,j)요소는 기존 행렬의 (i,j)요소부터 filter 크기의 행렬을 추출하여 filter와 elementwise product를 수행한 결과이다.

> CNN에서의 convolution 연산은 엄밀히 말하면 cross-correlation 연산이다. 하지만 Neural Network에서는 filter를 뒤집든 안뒤집든 결국 filter의 parameter들이 이전 layer의 출력에 일괄 적용된다는 것이 중요하기 때문에 뒤집지 않더라도 Convolution 연산이라고 부른다.

### Padding

filter를 적용하면 이미지 출력의 크기가 줄어들 수 밖에 없다. 또한 테두리나 모서리의 픽셀은 중간의 픽셀에 비해 반영이 적게 되는 문제도 존재한다.

가장자리에 padding을 추가하여 크기를 키우고 convolution을 적용하여 원래 크기를 보존할 수 있다.

- Valid convolution: padding 없는 연산
- Same convolution: output size와 input size를 같게 만드는 padding 추가한 연산

> 보통 filter의 크기는 홀수를 사용한다.

### Stride

기존 연산은 한번에 1칸씩 filter가 움직이지만 stride를 적용하면 stride 간격만큼 건너뛰어 계산한다.

$$output\,size = \left\lfloor \frac{n + 2p - f}{s} \right\rfloor + 1$$

### Channels

RGB 같이 이미지의 픽셀에 여러개의 값이 겹칠 수 있다. 이 경우 input의 채널이 3개라고 할 수 있다.

이전 layer의 채널이 c라면 필터 또한 c의 채널을 가져야한다. 즉, (f x f x c)가 된다. 이 3차원 필터가 여러개가 될 수 있고 필터가 여러개가 되면 다음 레이어에 필터의 개수만큼 채널이 생긴다.

### Bias

앞서 배운 Neural Network와 마찬가지로 각 filter에는 bias라는 단일 scalar 값이 하나씩 존재한다. filter로 convolution 연산을 적용하고 bias 값을 더해준다.

### Notation

![notation](/assets/images/2024/07/20/notation.png)

복잡해 보이지만 하나씩 읽어보면 당연한 형태이다.

## ConvNet

Conv1 -> Pooling -> Conv2 -> Conv3 -> fully-connected -> logistic/softmax

위와 같은 형태로 layer를 쌓아 Convolutional Neural Network 혹은 ConvNet을 구성한다.

### Conv layer

위에서 설명한 Convolution 연산을 하는 filter와 bias를 파라미터로 가지는 레이어이다.

### pool layer

데이터를 좀 더 간결하게 표현할 수 있도록 압축하는 레이어이다.

filter 사이즈에 맞춰 해당 공간에 max를 적용(Max pooling)하거나, Average를 적용(Average pooling)하는 등의 역할을 한다.

정해진 pooling 연산을 수행하므로 파라미터는 존재하지 않는다.

> Max Pooling이 많이 쓰인다.

### FC layer

Fully connected layer로 일반적인 Neural Network처럼 이전 레이어의 모든 출력을 flatten하여 W weight 행렬을 적용해 다음 레이어를 만든다.

많은 채널로 이루어진 2차원 데이터들을 최종적으로 하나로 모아 하나의 결과값을 얻어내기 위해 필요하다.
