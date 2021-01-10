---
layout: post
title: 'Multomodal deep learning - CMU 강의 1.2 Datasets'
subtitle: '1.2 Datasets'
categories: deeplearning
tags: multimodal
comments: true
---


강의 1.2 Datasets에 대한 내용 정리. 

## History

### 2010년 이전
- 2010년 이전에는 psychology and philosophy theories에 대한 연구가 주로 진행됨. 
- 또한 Humans 간의 interaction을 관심있게 연구함. 
- 주요 task = Audio-visual speech recognition, Content-based video retrieval, Multimodal sentiment analysis
- 2010년을 기점으로 `딥러닝 붐`이 일어남. 

### 2010년 이후
- 딥러닝으로 접근하는 방법이 좋은 performance를 보임. language와 vision을 합친 Multimodal deep learning(CNN + RNN or LSTM)
- 주요 task = Image captioning, Video captioning, Visual question answering(VQA) 등등 


## Multimodal tasks
Affect recognition
  ▪ Emotion
  ▪ Personality traits
  ▪ Sentiment

Media description
  ▪ Image captioning
  ▪ Video captioning
  ▪ Visual Question Answering
  
Event recognition
  ▪ Action recognition
  ▪ Segmentation

Multimedia information retrieval
  ▪ Content based/Cross-media

## dataset
1. Affet recognition
- AVEC(Audio Visual Emotion Challenge) datasets = Audio content(verbal modality), gestures(non-verbal modality) 등의 멀티모달리티에서 emotion을 추론.
- RECOLA dataset = 2015/2016 AVEC 데이터셋이며 fine-grained annotation이 제공됨. 추가적으로, 비디오 말고도 Electrical activity(심장 박동이나 어떠한 센서로 측정될 수 있는 데이터)도 포함됨.
- MELD dataset = 드라마 friends에서 single person to multi-party emotional recognition을 위한 annotation이 포함됨. 

여러 분야의 데이터셋을 알려주실 줄 알았지만 대표적인 것들만 알려주심. 
이후 설명해주신 것은 이 수업을 들었던 학생들의 프로젝트 내용. (감정분석, Fusion에 대한 연구 등)

2. Media Description
- Given a piece of media (image, video, audiovisual clips) provide a free form text description.(Ex. 이미지 캡셔닝)
- MS COCO dataset, MPII Movie Description dataset, Montreal Video Annotation dataset, Charades Dataset 
- 이 분야에서 어려운 첼린지는 이 결과가 좋은 캡션인지, 아닌지를 판단하기가 힘들다는 것이다. (사람이 관여하는 부분)
- 따라서 M1 ~ M5까지 기준을 정해두고 점수를 매김.
![Capture](/assets/img/post/multimodal/2021-1-9-multimodal-1.JPG)
- 사람이 라벨을 다는 것은 expensive하기 때문에, Automatic evaluation에 관한 방법론도 존재함. 
- CIDEr-D, Meteor, ROUGE, BLEU 등이 있는데 전부는 설명하진 않음. n-grams을 사용하여 reference caption과 모델이 만들어낸 caption이 얼마나 겹치는지를 봄. 

3. Multimodal QA
- 이미지 캡셔닝과 유사. Question에 해당하는 것을 여러 image에서 찾아내는 것(localize) 
- VQA dataset = MS COCO 기반 https://visualqa.org/ , DAQUAR, COCO-QA, Visual7W, Referring Expressions(RefClef, RefCOCO, GRef), GuessWaht?!, VCR, Flicker30k Entities 


또 여러가지 진행했었던 프로젝트를 소개해주심. 이게 학부생이 듣는지 석사가 듣는지 모르겠지만, 수준이 상당히 높다.(AAAI 등 학회에 제출했으니...)

## Multimodal Research에 대한 조언

1. Aim for generalizable models across several datasets.
2. Aim for models inspired by exisiting research. (Ex. psychology)
3. Some areas to Consider beyond performance: 
   - Robustness to missing/noisy modalities, adversarial attacks
   - Strudying social biases and creating fairer models
   - Interpretable models
   - Faster models for training/storage/inference

오늘까지가 첫 주 강의 소개 내용(오리엔테이션) 

