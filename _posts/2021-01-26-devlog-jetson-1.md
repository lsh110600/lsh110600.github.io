---
layout: post
title: '[Jetson Nano] Jetson Nano 센서 연결'
subtitle: '젯슨 나노 센서 연결'
categories: devlog
tags: jetsonnano
comments: true
---


ETRI 취미 업무로 받은 센서 데이터 저장을 위한 post

## ETRI 업무

멀티모달 딥러닝을 사용한 감염병 예방 시스템 연구제안서를 작성한 후 추가 업무로 센서 데이터를 받아서 DB에 저장하는 task를 받았습니다.

온도, 습도, CO2, 대기질 등을 측정해서 전처리 후 DB에 입력해야합니다.

## 센서 정보

센서는 ETRI에서 4 종류의 센서를 통합한 상태였고, 제품이 나왔는데 아직 테스트를 못해본 상황이었습니다.

[BME280 Github](https://github.com/adafruit/Adafruit_CircuitPython_BME280)
[SGP30 GitHub](https://github.com/pimoroni/sgp30-python)
[VEML7700 GitHub](https://github.com/adafruit/Adafruit_CircuitPython_VEML7700)
[SCD30 GitHub](https://github.com/adafruit/Adafruit_CircuitPython_SCD30.git)

## 초기 세팅


