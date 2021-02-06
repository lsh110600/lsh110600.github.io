---
layout: post
title: '[Hadoop] 시작하세요 하둡 프로그래밍 CH 14 얀 애플리케이션 개발 - part 3'
subtitle: 'Hadoop CH 14 얀 애플리케이션 개발 - part 3'
categories: de
tags: hadoop
comments: true
---
`시작하세요 하둡 프로그래밍`을 기반으로 공부한 내용을 정리합니다. - CH 14 얀 애플리케이션 개발 - part 3

![img](/assets/img/post/hadoop/hadoop_book.png)

### 14.5 애플리케이션 실행

1. 코드 다운로드

    `git clone https://github.com/blrunner/yarn-beginners-examples.git` 으로 코드를 다운받습니다.

    [img1](/assets/img/post/hadoop/2021-2-04-hadoop-14-1.png)

2. mvn으로 예제파일을 빌드합니다.

    저같은 경우는 하둡 버전이 3.2.1 이므로 pom.xml 파일을 수정해줍니다.

      [img2](/assets/img/post/hadoop/2021-2-06-hadoop-14-2.png)

      [img3](/assets/img/post/hadoop/2021-2-06-hadoop-14-3.png)

    이처럼 에러가 발생했는데, pom.xml에서 다음 코드를 추가해주세요.

    ```xml
    <properties>
      <maven.compiler.source>6</maven.compiler.source>
      <maven.compiler.target>1.6</maven.compiler.target>
    </properties>
    ```

      [img4](/assets/img/post/hadoop/2021-2-06-hadoop-14-4.png)

    빌드가 완료되면 target 디랙토리가 생성됩니다.

      [img5](/assets/img/post/hadoop/2021-2-06-hadoop-14-5.png)

    `yarn-examples-1.0-SNAPSHOT.jar` 파일이 생성됨을 확인합니다.

3. 생성된 JAR 파일을 하둡으로 옮겨줍니다.

    `docker cp yarn-examples-1.0-SNAPSHOT.jar namenode:/tmp/`

4. 파일 확인

    `docker exec -it namenode /bin/bash`
    로 네임노드로 이동합니다.

      [img6](/assets/img/post/hadoop/2021-2-06-hadoop-14-6.png)

    네임노드 /tmp/ 디렉토리에 방금 만든 JAR 파일이 있는 것을 확인합니다.

5. 하둡 실행

    `hadoop jar yarn-examples-1.0-SNAPSHOT.jar com.wikibooks.hadoop.yarn.examples.MyClient`

    명렁어를 사용하면 아래와 같이 파라미터 안내문이 출력됩니다.

    파라미터를 넣어주지 않았기 때문입니다.

      [img7](/assets/img/post/hadoop/2021-2-06-hadoop-14-7.png)

    `hadoop jar yarn-examples-1.0-SNAPSHOT.jar com.wikibooks.hadoop.yarn.examples.MyClient -jar yarn-examples-1.0-SNAPSHOT.jar -num_containers=1`

      [img8](/assets/img/post/hadoop/2021-2-06-hadoop-14-8.png)

    로그도 안쌓이고 Web에 접근할수가 없어서 다시 확인해야함.

    => Docker-hadoop 에서 포트포워딩이 안된 것으로 추측

### Reference

1. 시작하세요 하둡 프로그래밍