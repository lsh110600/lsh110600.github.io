---
layout: post
title: '[Hadoop] 시작하세요 하둡 프로그래밍 CH 5 맵리듀스 기초 다지기'
subtitle: 'Hadoop CH 5 맵리듀스 기초 다지기'
categories: de
tags: hadoop
comments: true
---
`시작하세요 하둡 프로그래밍`을 기반으로 공부한 내용을 정리합니다. - CH5

![img](/assets/img/post/hadoop/hadoop_book.png)

## 맵리듀스 기초 다지기

### 맵리듀스 잡의 실행 단계

![img](/assets/img/post/hadoop/2021-1-28-hadoop-5-0.jpg)

1. 클라이언트가 잡 실행을 요청하는 단계 (1,2,3,4)

    - 맵리듀스 프로그램이 잡 클라이언트(JobClient)에 잡 실행을 요청합니다.

    - 잡 트래커(JobTracker)는 잡의 출력 파일 경로가 정상적인지 확인한 후 잡 ID를 발급합니다.(RPC 통신 사용)

    - 클라이언트는 잡을 실행하는 데 필요한 정보를 잡 트래커(JobTracker)와 테스크트래커(TaskTracker)에게 공유합니다.

    - 따라서 잡 클라이언트(JobClient)는 입력 스플릿 정보, JAR파일 등을 HDFS에 저장합니다.

    - 이후 잡 클라이언트(JobClient)는 잡 트래커(JobTracker)의 sumitjob 메서드를 호출해 잡 실행을 요청합니다.

2. 해당 잡이 초기화되는 단계 (5,6)

    - 잡 트래커(JobTracker)는 잡의 상태와 진행 과정을 모니터링 할 수 있는 JobinProgress를 생성합니다.

    - JobinProgress는 HDFS에 등록한 잡 공통 파일을 로컬로 복사, 스플릿 정보를 이용해 맵 & 리듀스 테스크 개수를 계산합니다. 또한 잡의 실행 상태를 RUNNING으로 설정합니다.

    - JobinProgress 객체를 내부 큐인 jobs에 등록합니다.

3. 잡을 실행하기 위한 테스크를 할당하는 단계 (7)

    - 테스크트래커(TaskTracker)는 3초에 한 번씩 하트비트(heartbeat) 메시지를 전송하여 실행 중이라는 것과 새로운 테스크를 실행할 준비가 됐다는 것을 알려줍니다.

    - 스케줄러는 하트비트를 확인한 후 테스크를 할당할 잡을 선택합니다.(여러 개의 스케줄러 알고리즘이 존재함)

4. 할당된 테스크가 실행되는 단계 (8, 9. 10)

    - 테스크트래커(TaskTracker)는  HDFS에 저장된 잡 공통 파일들을 로컬로 복사합니다.

    - 이후 테스크 실행 결과를 저장할 로컬 디렉터리를 생성한 후 잡 JAR 파일을 이 디텍터리에 복사합니다.

    - 테스크트래커(TaskTracker)는 할당받은 테스크를 새로운 JVM에서 실행하며, 맵리듀스는 이를 Child JVM이라고 합니다.

    - 사용자가 정의한 매퍼 & 라듀서 클래스가 실행됩니다.

5. 잡이 완료되는 단계

    - 테스크트래커(TaskTracker)가 잡 트래커(JobTracker)에게 전송하는 하트비트에 완료된 테스크의 정보가 포합됩니다.

    - JobinProgress는 잡의 상태를 SUCCEEDED로 변경합니다. (실패하면 FAILED)

    - 클라이언트는 최종 결과를 출력하고 잡 실행을 완료합니다.

### 분석용 데이터 준비

- 책에서 안내하는 데이터 제공 사이트 주소가 바뀌었습니다.

- [항공데이터 출처](https://packages.revolutionanalytics.com/datasets/AirOnTime87to12)

- `AirOnTimeCSV.zip`을 다운로드 받았습니다. 

### Reference

1. 시작하세요 하둡 프로그래밍
2. Hadoop The Definitive Guide(eng, 3rd) p.189