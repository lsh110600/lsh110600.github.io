---
layout: post
title: '[Hadoop] 시작하세요 하둡 프로그래밍 CH 14 얀 애플리케이션 개발 - part 2'
subtitle: 'Hadoop CH 14 얀 애플리케이션 개발 - part 2'
categories: de
tags: hadoop
comments: true
---
`시작하세요 하둡 프로그래밍`을 기반으로 공부한 내용을 정리합니다. - CH 14 얀 애플리케이션 개발 - part 2

![img](/assets/img/post/hadoop/hadoop_book.png)

### 14.3 애플리케이션 마스터 구현

애플리케이션을 관리하는 애플리케이션 마스터를 구현합니다.

#### 14.3.1 주요 변수 선언

##### AppMaster 생성자

  ```java
  MyApplicationMaster.java
  ...

    // Configuration
    private Configuration conf;

    public MyApplicationMaster() {
    // Set up the configuration
      conf = new YarnConfiguration();
    }

  ...
  ```

인스턴스 변수 `Configuration`를 생성합니다.

=> 리소스 매니저 & 노드 매니저와의 통신에 사용됩니다.

##### 인스턴스 변수 선언

  ```java
  MyApplicationMaster.java
  ...

    // Application Attempt Id ( combination of attemptId and fail count )
    protected ApplicationAttemptId appAttemptID;

    // No. of containers to run shell command on
    private int numTotalContainers = 1;

    // Memory to request for the container on which the shell command will run
    private int containerMemory = 10;

    // VirtualCores to request for the container on which the shell command will run
    private int containerVirtualCores = 1;

    // Priority of the request
    private int requestPriority;

    // Location of shell script ( obtained from info set in env )
    // Shell script path in fs
    private String appJarPath = "";
    // Timestamp needed for creating a local resource
    private long appJarTimestamp = 0;
    // File length needed for local resource
    private long appJarPathLen = 0;

  ...
  ```

노드매니저에게 요청하는 컨테이너 스팩 설정에 필요한 파라미터를 적어줍니다.

애플리케이션을 실행할 컨테이너 개수, 메모리, cpu 자원 등

#### 14.3.2 애플리케이션 실행

애플리케이션을 실행하려면 리소스매니저로부터 컨테이너를 할당받고,

노드매니저에게 할당된 컨테이너에서 애플리케이션을 실행할 것을 요청해야합니다.

##### run 메서드

  ```java
  MyApplicationMaster.java
  ...

    @SuppressWarnings({"unchecked"})
    public void run() throws Exception {
      LOG.info("Running MyApplicationMaster");

      // Initialize clients to ResourceManager and NodeManagers
      AMRMClient<ContainerRequest> amRMClient = AMRMClient.createAMRMClient();
      amRMClient.init(conf);
      amRMClient.start();

      // Register with ResourceManager
      amRMClient.registerApplicationMaster("", 0, "");

      // Set up resource type requirements for Container
      Resource capability = Records.newRecord(Resource.class);
      capability.setMemory(containerMemory);
      capability.setVirtualCores(containerVirtualCores);

      // Priority for worker containers - priorities are intra-application
      Priority priority = Records.newRecord(Priority.class);
      priority.setPriority(requestPriority);

      // Make container requests to ResourceManager
      for (int i = 0; i < numTotalContainers; ++i) {
        ContainerRequest containerAsk = new ContainerRequest(capability, null, null, priority);
        amRMClient.addContainerRequest(containerAsk);
      }


      NMClient nmClient = NMClient.createNMClient();
      nmClient.init(conf);
      nmClient.start();

      // Setup CLASSPATH for Container
      Map<String, String> containerEnv = new HashMap<String, String>();
      containerEnv.put("CLASSPATH", "./*");

      // Setup ApplicationMaster jar file for Container
      LocalResource appMasterJar = createAppMasterJar();

      // Obtain allocated containers and launch
      int allocatedContainers = 0;
      // We need to start counting completed containers while still allocating
      // them since initial ones may complete while we're allocating subsequent
      // containers and if we miss those notifications, we'll never see them again
      // and this ApplicationMaster will hang indefinitely.
      int completedContainers = 0;
      while (allocatedContainers < numTotalContainers) {
        AllocateResponse response = amRMClient.allocate(0);
        for (Container container : response.getAllocatedContainers()) {
          allocatedContainers++;

          ContainerLaunchContext appContainer = createContainerLaunchContext(appMasterJar, containerEnv);
          LOG.info("Launching container " + allocatedContainers);

          nmClient.startContainer(container, appContainer);
        }
        for (ContainerStatus status : response.getCompletedContainersStatuses()) {
          ++completedContainers;
          LOG.info("ContainerID:" + status.getContainerId() + ", state:" + status.getState().name());
        }
        Thread.sleep(100);
      }

      // Now wait for the remaining containers to complete
      while (completedContainers < numTotalContainers) {
        AllocateResponse response = amRMClient.allocate(completedContainers / numTotalContainers);
        for (ContainerStatus status : response.getCompletedContainersStatuses()) {
          ++completedContainers;
          LOG.info("ContainerID:" + status.getContainerId() + ", state:" + status.getState().name());
        }
        Thread.sleep(100);
      }

      LOG.info("Completed containers:" + completedContainers);

      // Un-register with ResourceManager
      amRMClient.unregisterApplicationMaster(
          FinalApplicationStatus.SUCCEEDED, "", "");
      LOG.info("Finished MyApplicationMaster");
    }

  ...
  ```

컨테이너 할당 요청을 진행하려면, 리소스매니저와 통신하기 위한 클라이언트가 필요합니다.

`createAMRMClient();`

=> Yarn이 제공하는 AMRMClient를 사용합니다.

`registerApplicationMaster("", 0, "");`

=> 애플리케이션마스터를 등록합니다.

`addContainerRequest(containerAsk);`

=> 큐에 컨테이너 할당 요청을 저장합니다.

`createNMClient();`

=> 노드매니저와 통신을 위해 `NMClient` 클라이언트를 생성합니다.

`createAppMasterJar();`

=> 컨테이너가 JAR 파일 경로를 알 수 있도록 합니다.

##### createAppMasterJar 메서드

<details markdown="1">
<summary>접기/펼치기</summary>

  ```java
  MyApplicationMaster.java - createAppMasterJar method

    private LocalResource createAppMasterJar() throws IOException {
    LocalResource appMasterJar = Records.newRecord(LocalResource.class);
    if (!appJarPath.isEmpty()) {
      appMasterJar.setType(LocalResourceType.FILE);
      Path jarPath = new Path(appJarPath);
      jarPath = FileSystem.get(conf).makeQualified(jarPath);
       appMasterJar  .setResource(ConverterUtils.getYarnUrlFromPath(jarPath));
      appMasterJar.setTimestamp(appJarTimestamp);
      appMasterJar.setSize(appJarPathLen);
      appMasterJar.setVisibility(LocalResourceVisibility.PUBLIC);
    }
    return appMasterJar;
  }

  ```

MyClient가 전달한 정보를 바탕으로 애플리케이션 클래스가 포함된 JAR 파일 경로를 LocalResource로 생성합니다.

</details>

`amRMClient.allocate(0);`

=> 노드매니저 클라이언트를 실행할 준비가 완료되면 할당된 컨테이너를 실행합니다.

`createContainerLaunchContext()';`

=> 컨테이너가 실행될 때 필요한 환경설정 정보를 ContainerLaunchContext에 담아서 전달합니다.

##### createContainerLaunchContext 메서드

<details markdown="1">
<summary>접기/펼치기</summary>

  ```java
  MyApplicationMaster.java - createContainerLaunchContext method
    
    // 
    private ContainerLaunchContext createContainerLaunchContext(LocalResource appMasterJar,
            Map<String, String> containerEnv) {
    ContainerLaunchContext appContainer =
        Records.newRecord(ContainerLaunchContext.class);
    appContainer.setLocalResources(
        Collections.singletonMap(Constants.AM_JAR_NAME, appMasterJar));
    appContainer.setEnvironment(containerEnv);
    appContainer.setCommands(
        Collections.singletonList(
            "$JAVA_HOME/bin/java" +
                " -Xmx256M" +
                " com.wikibooks.hadoop.yarn.examples.HelloYarn" +
                " 1>" + ApplicationConstants.LOG_DIR_EXPANSION_VAR + "/stdout" +
                " 2>" + ApplicationConstants.LOG_DIR_EXPANSION_VAR + "/stderr"
        )
    );

    return appContainer;
  }  

  ```

</details>

JAR 파일의 정보를 LocalResources에 추가합니다.

커멘드 라인에 애플리케이션 클래스 실행을 설정합니다.(com.~~~.HelloYarn)

##### 컨테이너 정보 출력

`LOG.info("ContainerID:" + status.getContainerId() + ", state:" + status.getState().name());`

=> 완료된 컨테이너 목록을 출력하고, 마지막에는 실행 중인 컨테이너가 있는지 체크하기 위해 한번 더 컨테이너 상태를 출력합니다.

`unregisterApplicationMaster`

=> 컨테이너 실행이 완료되면 MyApplicationMaster를 리소스매니저에서 제거합니다.

#### 14.3.3 MyApplicationMaster 실행

노드 매니저가 MyApplicationMaster를 실행하면 MyApplicationMaster의 main 메서드가 실행됩니다.

MyApplicationMaster 객체를 생성해 생성자를 초기화하고 run 메서드를 출력합니다.

##### MyApplicationMaster 실행(main code)

사용자가 커맨드 라인에서 MyClient 실행을 요청하면 MyClient의 main 메서드가 실행됩니다.

main 메서드에서 MyClient 객체를 선언하고 실행합니다.

  ```java
  MyApplicationMaster.java
  ...

  public static void main(String[] args) throws Exception {
    try {
      MyApplicationMaster appMaster = new MyApplicationMaster();
      LOG.info("Initializing MyApplicationMaster");
      boolean doRun = appMaster.init(args);
      if (!doRun) {
        System.exit(0);
      }
      appMaster.run();
    } catch (Throwable t) {
      LOG.fatal("Error running MyApplicationMaster", t);
      LogManager.shutdown();
      ExitUtil.terminate(1, t);
    }
  }

  ```

### 14.4 애플리케이션 구현

실제로 하둡에서 실행되는 애플리케이션을 구현합니다.

예제로, JVM 메모리, 전체 메모리 등을 출력합니다.

  ```java
  HelloYarn.java

    public class HelloYarn {
      private static final long MEGABYTE = 1024L * 1024L;

      public HelloYarn() {
        System.out.println("HelloYarn!");
      }

      public static long bytesToMegabytes(long bytes) {
        return bytes / MEGABYTE;
      }

      public void printMemoryStats() {
        long freeMemory = bytesToMegabytes(Runtime.getRuntime().freeMemory());
        long totalMemory = bytesToMegabytes(Runtime.getRuntime().totalMemory());
        long maxMemory = bytesToMegabytes(Runtime.getRuntime().maxMemory());

        System.out.println("The amount of free memory in the Java Virtual Machine: " + freeMemory);
        System.out.println("The total amount of memory in the Java virtual machine: " + totalMemory);
        System.out.println("The maximum amount of memory that the Java virtual machine: " + maxMemory);
      }

      public static void main(String[] args) {
        HelloYarn helloYarn = new HelloYarn();
        helloYarn.printMemoryStats();
      }
    }

  ```

### 14.5 애플리케이션 실행

1. 코드 다운로드

 `git clone https://github.com/blrunner/yarn-beginners-examples.git` 으로 코드를 다운받습니다.

[img1](/assets/img/post/hadoop/2021-2-06-hadoop-14-1.png)

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
2. [Apache Hadoop]](https://hadoop.apache.org/docs/stable/hadoop-yarn/hadoop-yarn-site/YARN.html)