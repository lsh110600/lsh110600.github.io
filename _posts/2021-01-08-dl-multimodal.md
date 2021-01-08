---
layout: post
title: 'Multomodal deep learning - CMU 강의1'
subtitle: '1.1Intro'
categories: deeplearning
tags: multimodal
comments: true
---


강의 1.1 Intro에 대한 내용 정리. 


## What is Multimodal
![Capture](/assets/img/post/multimodal/2021-1-8-multimodal-1.JPG)
- 수학적인 정의보다 '감각'과 관련됨.

- 인간은 3가지 언어로 '소통'한다. 
1. '내용언어'(Verbal) = 전달하고자 내용
2. '시각언어'(Visual) = 보이는 언어로서 표정이나 태도 
3. '청각언어'(Vocal) = 들리는 언어로서 음성, 강약, 어투, 어속, 발음 
=> 이 3가지 중 1개만 사용하는 것이 아니라, 복합적으로 사용함. 
![Capture](/assets/img/post/multimodal/2021-1-8-multimodal-2.JPG)

## Modality
- 정의 : The way in which something happens or is experienced.
- (논문과 강의에서는 주로 감각적인 Modality(촉감, 시각, 청각 등)를 말한다.)
- 예시 : 1. Text로 주어진 자연어, 2. 음성으로 주어진 자연어 or 음악, 3. 이미지 or 비디오, 4. MRI or 적외선 사진 등 
- 이 강의에서는 Natural language, Visual, Auditory에 초점을 맞춘다. 

## Historical view
- 심리학 분야에서 early work가 있었고, 딥러닝 분야에서는 2010년부터 연구되었다. 
- 'Representation learning' (a.k.a deep learning) 에서 가능했던 이유
1. New large-scale multimodal datasets
2. Faster computer and GPUS
3. High-level visual features => CNN의 등장으로 High dimensional representation을 take 할 수 있음. 
4. “Dimensional” linguistic features => 단어를 vector로 표현하는 등, useful한 representation 가능
+) 모달리티들은 서로가 상당히 비슷한 특징을 가지고 있음. noise를 예로 들면, 이미지에서의 noise는 pixcel 단위로 튀는? 느낌이고, 자연어처리에서의 noise는 오탈자라고 볼 수 있음. <br>
   정확하게 '같은' space에 놓여있지는 않지만 싱딩히 비슷함. 따라서 heterogeneous한 관계임. <br>
   이러한 특징은 '딥러닝'을 적용시키기에 적합함. <br>

- 관련 논문
1. Multimodal deep learning [ICML 2011] <br>
2. Multimodal Learning with Deep Boltzmann Machines [NIPS 2012] <br>
3. Visual attention: Show, Attend and Tell: Neural Image Caption Generation with Visual Attention [ICML 2015]



## Core Technical Challenges
- 이에 대한 근거는 Tadas Baltrusaitis, Chaitanya Ahuja, and Louis-Philippe Morency 이 작성한 [Multimodal Machine Learning: A Survey and Taxonomy](https://arxiv.org/abs/1705.09406).를 바탕으로 함. <br>
1. Representation  <br>
2. Alignment  <br>
3. Fusion <br>
4. Translation <br>
5. Co-Learning <br>

+) 추가 설명 사진 
![Capture](/assets/img/post/multimodal/2021-1-8-multimodal-3.JPG)
- Representation과 Alignment라는 용어가 어색할 때 위 그림을 참고해서 이해하시길...
![Capture](/assets/img/post/multimodal/2021-1-8-multimodal-3-1.JPG)
- Fusion과 Translation이라는 용어가 어색할 때 위 그림을 참고해서 이해하시길...

## Core Challenge 1 - Representation
![Capture](/assets/img/post/multimodal/2021-1-8-multimodal-4.JPG)
- "I like it!", 행복한 표정과 말투 등은 어떤 공간에 유사하게 Encode될 것임. 
- 이러한 Multimodal Vector Space에서의 연산을 잘 다루는 것이 중요함. 
- 정의 : : Learning how to represent and summarize multimodal data in away that exploits the complementarity and redundancy.
- 이러한 Representation은 2 가지 종류가 있음. 하나는 Joint representations이고 나머지 하나는 Coordinated representations임.
![Capture](/assets/img/post/multimodal/2021-1-8-multimodal-5.JPG)
- 많은 사람들은 멀티모달에 대한 설명을 들으면, 왼쪽 그림의 Joint representations을 생각했을 것. 
- 두 개의 Modality가 있을때, 이 둘을 합쳐서 한 개의 vector space를 갖는 Representation으로 나타내는 것을 의미
- 두 개의 Modality 합치는 것이 아니라 그들 각각의 space를 갖는 Representation을 구해서 sub space를 만든 후 스팩트럼처럼 조정할 수 있음. 
- (Ex, 언어적 표현과 시각적 표현을 따로 구해서 동일선상의 space로 조정해줌. CCA(canonical correlation analysis를 )










