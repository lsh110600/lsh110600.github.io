---
layout: post
title: 'Multomodal deep learning - CMU 강의 4.1 Multimodal Representations'
subtitle: '1.1Intro'
categories: deeplearning
tags: multimodal
comments: true
---


강의 4.1 Multimodal Representations에 대한 내용 정리. 


## Graph 
- Graphs are a general language for describing and modeling complex systems
![Capture](/assets/img/post/multimodal/2021-1-10-multimodal-0.JPG)
- Supervised Task Goal : Learn from labels associated with a subset of nodes(or with all nodes) 
- Unsupervised Task Goal : Learn an embedding space where similarity(u,z) ~ Z  => graph에서 connect 되어있는 노드는 embedding space에서도 비슷한 위치에 놓여있어야함. 

- V is the set of vertices
- A is the binary adjacency matrix
- X is a matrix of node features : text, image 등등 
- Y is vector of node labels (optional)

- Key idea : Generate node embeddings based on local neighborhoods in recursive manner 
- 노드 하나에서 다른 노드와 relation이 있는데, 다른 노드들도 자신만의 relation을 가지고 있음. 

