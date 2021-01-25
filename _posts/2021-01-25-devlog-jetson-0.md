---
layout: post
title: '[Jetson Nano] Jetson Nano 시작하기'
subtitle: '젯슨 나노 설치'
categories: devlog
tags: jetsonnano
comments: true
---


다시 젯슨 OS 설치할 때 까먹지 않도록 설치 및 환경 설정 방법을 기록합니다.

## 준비물

1. 젯슨 보드

2. micro SD card(16GB이상)

3. USB 무선 랜카드(다이소 및 쿠팡 구매 가능)

4. 5V 4A 젯슨 나노 충전기

5. Hdmi 케이블

6. usb로 연결가능한 키모드 & 마우스

7. Win32 Disk Imager 프로그램 => 메모리카드에 이미지 구울 때 사용

8. Jetson Nano img 파일

## STEP 1 img 파일 다운로드 & 굽기

- [Nvidia 사이트](https://developer.nvidia.com/embedded/downloads)에서 Jetson Nano SD Card image를 다운로드 받습니다.(6GB)

![img](/assets/img/post/jetson/2021-1-25-jetson-0-0.png)

- 다음과 같은 방법으로 포멧을 진행합니다.

![img](/assets/img/post/jetson/2021-1-25-jetson-0-1.png)

![img](/assets/img/post/jetson/2021-1-25-jetson-0-2.png)

![img](/assets/img/post/jetson/2021-1-25-jetson-0-3.png)

- 경고가 뜨면 확인을 눌러서 포멧을 진행합니다.

![img](/assets/img/post/jetson/2021-1-25-jetson-0-4.png)

- 포멧 후 win32 Disk imager를 사용해서 img 파일을 write해줍니다.

- 굽는게 끝나면 파티션이 나누어져서 D,E,F ... 등등을 포멧할 꺼냐고 물어볼텐데, 다 취소하시면 됩니다.
- 아래 그림처럼 나눠지면 성공

![img](/assets/img/post/jetson/2021-1-25-jetson-0-5.png)

## STEP 2 JetSon Nano 전원 부팅

- 먼저 SD 카드를 끼워줍니다.

- J48 핀에 점퍼를 끼워주고 5V 4A 어뎁터를 사용해 전원을 인가해줍니다.
- 저는 점퍼를 연구실에서 구했지만, 없으신 분들은 어뎁터 구매하실 때 같이 구매하세요!
- [예시 사이트](https://www.devicemart.co.kr/goods/view?no=1077104)

- Hdmi 케이블, 키보드, 마우스 usb로 연결해주세요. 있으면, usb 랜카드도...

## STEP 3 부팅 후 세팅

- 입력하라고 하는 것만 입력하고 `다음` 누르면 됩니다. accept 누르고 `다음`

![img](/assets/img/post/jetson/2021-1-25-jetson-0-7.jpg)
![img](/assets/img/post/jetson/2021-1-25-jetson-0-8.jpg)
![img](/assets/img/post/jetson/2021-1-25-jetson-0-9.jpg)
![img](/assets/img/post/jetson/2021-1-25-jetson-0-10.jpg)
![img](/assets/img/post/jetson/2021-1-25-jetson-0-11.jpg)
![img](/assets/img/post/jetson/2021-1-25-jetson-0-12.jpg)
![img](/assets/img/post/jetson/2021-1-25-jetson-0-13.jpg)
![img](/assets/img/post/jetson/2021-1-25-jetson-0-14.jpg)

- 최종화면입니다.
![img](/assets/img/post/jetson/2021-1-25-jetson-0-15.jpg)
