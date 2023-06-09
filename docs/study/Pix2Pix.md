---
layout: default
title: Pix2Pix
nav_order: 4
parent: 논문 구현(Deep Learning)
---

# Pix2Pix
{: .no_toc}

## Table of contents
{: .no_toc .text-delta}

Generative model인 GAN을 기반으로 한 Pixel-to-Pixel Colorizaing

참고 논문: [Image-to-Image Translation with Conditional Adversarial Networks.pdf](https://github.com/KimSungHeon/KimSungHeon.github.io/files/11052529/Image-to-Image.Translation.with.Conditional.Adversarial.Networks.pdf)

Ubuntu,Jupyter-lab 에서 구현.

<script src="https://gist.github.com/KimSungHeon/8ea4e05b76b01f46f1635cbce51f9dac.js"></script>
( 사전에 dataset으로 활용할 coco sample을 미리 다운받아 두었다.)

<script src="https://gist.github.com/KimSungHeon/2779627fd7ac16ccea3399946ac7eca2.js"></script>

<script src="https://gist.github.com/KimSungHeon/b4b9b85ca07f2f5d14a203c5f634c9ab.js"></script>

Dataloader에 입력으로 들어갈 Custom Dataset을 만들기 위한 작업.

<script src="https://gist.github.com/KimSungHeon/73660ad753ecbccfc18a145d1979d2f0.js"></script>

(사실 처음에 전체 데이터를 가지고 model을 훈련 시켜봤더니, 너무 오래걸렸다... 따라서 4000개만 쓰기로 하자.)

<script src="https://gist.github.com/KimSungHeon/da597b0140d77f2c3f9b6c75fce20b5f.js"></script>

원래 coco data에서 4000개를 무작위로 추출 해 dataset을 만들고, 그 중에서 80%는 train dataset으로 쓰고, 20%는 validation dataset으로 쓰기로 하였다.

<script src="https://gist.github.com/KimSungHeon/6964c06929c70f8b47339a61312a2283.js"></script>

Custom Dataset을 보면 Lab 채널에서 L채널과 나머지 ab채널을 따로 분리하여 L채널로부터 model이 Fake color image를 출력하도록 설계되어있다.
따라서 입력으로 들어가는 채널의 dimension은 1이다.
  
  cf) What is 'Lab' stands for? : https://www.xrite.com/blog/lab-color-space


U-net의 skip connection을 모방하기 위해 encoder와 decoder 그리고 middel layer을 각각 만들어 줬으며, forward시 decoder의 입력으로 이전 decoder의 출력과 거울상으로 상응하는 encoder의 출력을 concatenate한 data가 들어간다.

<script src="https://gist.github.com/KimSungHeon/118d5a3db41dd3ea9b37dbab25cc9227.js"></script>
<script src="https://gist.github.com/KimSungHeon/95df80b24e51f1f62b70d5f8679dc8d3.js"></script>

Generator의 parameter만 봐도 약 5천4백만개이다.

<script src="https://gist.github.com/KimSungHeon/f1631c8ca146391844b9b3649b1c3560.js"></script>

Discriminator는 입력 채널은 3이다. Why? Generator로 생성한 ab채널을 가진 이미지를 원래의 L채널과 concatenate하여 3채널의 가짜 이미지를 입력으로 받아 판별할 것이기 때문.

<script src="https://gist.github.com/KimSungHeon/14717bff9d11c61accac11b6f369c2c6.js"></script>
<script src="https://gist.github.com/KimSungHeon/e853e1607b708db0dc141441fefe434e.js"></script>

Discriminator의 parameters는 약 270만개.

<script src="https://gist.github.com/KimSungHeon/40599f4b4959f27f52e7619425ded4a1.js"></script>

가중치 초기화를 해주기 위한 함수를 정의하였다.
Generator와 Discriminator의 architecture을 보면, learnable parameter를 가진 layer는 Convolution layer, DeConvolution (TransposeConvolution) layer, Batchnorm layer 밖에 없어서, 3개의 층에서 parameter을 초기화 시켜주었다. gaussian 분포를 따르게 초기화 시켜주었는데, mean과 deviation 값은 논문을 참조하였다.


<script src="https://gist.github.com/KimSungHeon/4e5b2a6b0b2c199dd3bef6371619be6b.js"></script>

학습을 위해 RGB채널의 이미지를 Lab채널로 바꾸었으므로, 다시 RGB채널로 바꿔주기 위한 함수를 정의하였다.


<script src="https://gist.github.com/KimSungHeon/29a5cf5392956d4adae5e0968ce937ff.js"></script>

Loss function을 정의하기 위한 class를 선언하였다.

Discriminator가 pixel 별로 True or False를 구별하기 때문에, pixel별로 True(1)/False(0)의 Label을 가지는 임의의 tensor를 ground truth로 삼아 loss function(criterion)에 주입시켜 주어야한다, model의 output과 BinaryCrossEntropy를 통한 차이를 손실로 return 시키도록 하였다.

<script src="https://gist.github.com/KimSungHeon/0c30d6d822c61ed0f7ba1aed322c1a55.js"></script>

Discriminator와 Generator의 optimize을 정의하였다. optimizer의 설정은 논문을 따랐다.

<script src="https://gist.github.com/KimSungHeon/5f9064723ce938d3c89e519eda6c17cc.js"></script>
전체적인 실행코드. 
DataLoader의 50번째 mini batch마다 batch의 첫 번째 그림을 출력하여 평가하도록 설계했다.

epoch 1)
![epoch 1 pix2pix](https://user-images.githubusercontent.com/103099516/229021790-28aae567-3603-43f8-aa17-ce612c4575bf.png)

epoch 25)
![epoch 25 pix2pix](https://user-images.githubusercontent.com/103099516/229021894-51e8b5b5-4a02-4c95-9a0a-f5b412a41aca.png)

epoch 50)
![epoch 50 pix2pix](https://user-images.githubusercontent.com/103099516/229021912-b66f06bf-5237-4c7f-8b76-50d1a6507b72.png)


<script src="https://gist.github.com/KimSungHeon/1e2f1d47417386f192d43fd8dd7640cc.js"></script>

학습한 모델의 파라미터를 저장하기 위한 함수를 정의하고 geneartor와 discriminator 저장.

<script src="https://gist.github.com/KimSungHeon/f0e17d69623600085815206bb80e3a75.js"></script>
![download](https://user-images.githubusercontent.com/103099516/229034478-9cf915bc-5e35-4db8-974c-f7e75ecda928.png)

학습된 Generator를 실제로 평가해 보았음.
