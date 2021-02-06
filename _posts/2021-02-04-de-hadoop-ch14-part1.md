---
layout: post
title: '[Hadoop] 시작하세요 하둡 프로그래밍 CH 14 얀 애플리케이션 개발 - part 1'
subtitle: 'Hadoop CH 14 얀 애플리케이션 개발 - part 1'
categories: de
tags: hadoop
comments: true
---
`시작하세요 하둡 프로그래밍`을 기반으로 공부한 내용을 정리합니다. - CH 14 얀 애플리케이션 개발 - part 1

![img](/assets/img/post/hadoop/hadoop_book.png)

## 14 얀 어플리케이션 개발

얀 클러스터에서 동작하는 간단한 애플리케이션을 알아봅니다.

### 14.1 Yarn Concepts and Applications(얀 어플리케이션)

애플리케이션 실행을 요청하는 클라이언트, 애플리케이션을 관리하는 애플리케이션 마스터, 노드메니저에서 실행되는 애플리케이션으로 구성됩니다.

![img](/assets/img/post/hadoop/2021-2-04-hadoop-14-0.png)

위 그림은 세 종류의 클래스가 얀 클러스터에서 어떻게 동작하는지 알 수 있습니다.

다음과 같은 순서로 동작합니다.

1. 클라이언트는 리소스 매니저에게 신규 애플리케이션 ID 요청합니다.

2. 클라이언트는 애플리케이션 마스터 및 애플리케이션 실행 시 공통 리소스를 HDFS에 업로드합니다.

3. 클라이언트는 리소스 매니저에게 애플리케이션 실행을 요청 후 모니터링합니다.

4. 리소스 매니저는 노드매니저에게 애플리케이션 마스터 실행을 요청합니다.

5. 노드 매니저는 컨테이너에서 애플리케이션 마스터를 실행합니다.

6. 애플리케이션 마스터는 애플리케이션 실행 시 필요한 리소스를 HDFS 에 업로드 합니다.

7. 애플리케이션 마스터는 노드매니저에게 애플리케이션 실행을 요청합니다.

8. 노드매니저는 컨테이너에서 애플리케이션을 실행합니다.

### 14.2 클라이언트 구현

클라이언트는 리소스 매니저에게 애플리케이션 실행을 요청합니다.

#### 14.2.1 주요 변수 선언

##### YarnClient 객체 생성

  ```java
  MyClient.java

  public class MyClient {
  private static final Log LOG = LogFactory.getLog(MyClient.class);

  ...

    public MyClient() throws Exception  {
      createYarnClient();
      initOptions();
    }

    private void createYarnClient() {
      yarnClient = YarnClient.createYarnClient();
      this.conf = new YarnConfiguration();
      yarnClient.init(conf);
    }

  ...
  ```

- 리소스 매니저와 통신하기 위해서 YarnClient 객체를 사용합니다.
=> YarnClient 객체를 인스턴스 변수로 선언합니다.

##### 애플리케이션 마스터용 변수 선언

  ```java
  MyClient.java
  ...

    // Application master specific info to register a new Application with RM/ASM
    private String appName = "";

    // App master priority
    private int amPriority = 0;

    // Queue for App master
    private String amQueue = "";

    // Amt. of memory resource to request for to run the App Master
    private int amMemory = 10;

    // Amt. of virtual core resource to request for to run the App Master
    private int amVCores = 1;

    // ApplicationMaster jar file
    private String appMasterJarPath = "";

    // Container priority
    private int requestPriority = 0;

    // Amt of memory to request for container in which the HelloYarn will be executed
    private int containerMemory = 10;

    // Amt. of virtual cores to request for container in which the HelloYarn will be executed
    private int containerVirtualCores = 1;

    // No. of containers in which the HelloYarn needs to be executed
    private int numContainers = 1;

    // Timeout threshold for client. Kill app after time interval expires.
    private long clientTimeout = 600000;

  ...
  ```

컨테이너 개수, 컨테이너 사용할 CPU, Memory 정보 등 파라미터를 입력합니다.

#### 14.2.2 애플리케이션 실행 요청

##### run 메서드

  ```java
  MyClient.java
  ...

  public boolean run() throws IOException, YarnException {

    LOG.info("Running Client");
    yarnClient.start();

    // Get a new application id
    YarnClientApplication app = yarnClient.createApplication();
    GetNewApplicationResponse appResponse = app.getNewApplicationResponse();

  ...
  ```

애플리케이션 요청 기능은 run 메소드에 구현되어 있습니다.

`yarnClient.start();`
=> YarnClient를 실행합니다.

`YarnClientApplication app = yarnClient.createApplication();`
`GetNewApplicationResponse appResponse = app.getNewApplicationResponse();`

=> createApplication 실행 시 YarnClientApplication이 반환되는데, YarnClientApplication에 GetNewApplicationResponse가 포함되어있습니다.

=> GetNewApplicationResponse에는 ApplicationSubmissionContext가 있는데, 여기에 애플리케이션 실행에 필요한 ID가 담겨있습니다.

  ```java
  MyClient.java
  ...

    int maxMem = appResponse.getMaximumResourceCapability().getMemory();
    LOG.info("Max mem capabililty of resources in this cluster " + maxMem);

    // A resource ask cannot exceed the max.
    if (amMemory > maxMem) {
      LOG.info("AM memory specified above max threshold of cluster. Using max value."
          + ", specified=" + amMemory
          + ", max=" + maxMem);
      amMemory = maxMem;
    }

    int maxVCores = appResponse.getMaximumResourceCapability().getVirtualCores();
    LOG.info("Max virtual cores capabililty of resources in this cluster " + maxVCores);

    if (amVCores > maxVCores) {
      LOG.info("AM virtual cores specified above max threshold of cluster. "
          + "Using max value." + ", specified=" + amVCores
          + ", max=" + maxVCores);
      amVCores = maxVCores;
    }

  ...
  ```

`GetNewApplicationResponse`에는 ID 뿐만 아니라 가용가능한 최대 자원 정보도 포함되어있습니다.

사용자가 요청한 리소스가 현재 가용가능한 리소스보다 크면 사용자가 설정한 값 대신에 가용가능한 리소스 값을 사용합니다.

  ```java
  MyClient.java
  ...

    // set the application name
    ApplicationSubmissionContext appContext = app.getApplicationSubmissionContext();
    ApplicationId appId = appContext.getApplicationId();

    appContext.setApplicationName(appName);

    // Set up resource type requirements
    // For now, both memory and vcores are supported, so we set memory and
    // vcores requirements
    Resource capability = Records.newRecord(Resource.class);
    capability.setMemory(amMemory);
    capability.setVirtualCores(amVCores);
    appContext.setResource(capability);

    // Set the priority for the application master
    Priority pri = Records.newRecord(Priority.class);
    pri.setPriority(amPriority);
    appContext.setPriority(pri);

    // Set the queue to which this application is to be submitted in the RM
    appContext.setQueue(amQueue);

  ...
  ```

`ApplicationId appId = appContext.getApplicationId();`를 통해 애플리케이션 ID를 조회합니다.

리소스매니저에게 애플리케이션 실행을 요청할 때는 ApplicationSubmissionContext를 전달해야합니다.

=> `appContext`에 전달되는 값은 ApplicationSubmissionContext의 설정 값입니다.

1. `appContext.setApplicationName(appName);`
=> 애플리케이션 이름 설정

2. `appContext.setResource(capability);`
=> 메모리, CPU 코어 개수 설정

3. `appContext.setPriority(pri);`
=> 클러스터 내에서 AM 우선순위 설정

4. `appContext.setQueue(amQueue);`
=> AM 큐 설정

  ```java
  MyClient.java
  ...

    // Set the ContainerLaunchContext to describe the Container ith which the ApplicationMaster is launched.
    appContext.setAMContainerSpec(getAMContainerSpec(appId.getId()));

    // Submit the application to the applications manager
    // SubmitApplicationResponse submitResp = applicationsManager.submitApplication(appRequest);
    // Ignore the response as either a valid response object is returned on success
    // or an exception thrown to denote some form of a failure
    LOG.info("Submitting application to ASM");

    yarnClient.submitApplication(appContext);

    // Monitor the application
    return monitorApplication(appId);
  }

  ...
  ```

애플리케이션마스터를 실행하는 컨테이너의 context 실행 정보를 설정합니다.

실행 정보는 ContainerLaunchContext에 정의되며, `setAMContainerSpec` 메서드를 사용합니다.

`submitApplication` 메서드를 통해 yarnClient에 context를 넘겨주고,

애플리케이션 실행 요청이 완료되면 실행 상태를 모니터링 합니다.

##### getAMContainerSpec 메서드

<details markdown="1">
<summary>접기/펼치기</summary>

  ```java
  MyClient.java - getAMContainerSpec method

    private ContainerLaunchContext getAMContainerSpec(int appId) throws IOException, YarnException {
    // Set up the container launch context for the application master
    ContainerLaunchContext amContainer = Records.newRecord(ContainerLaunchContext.class);

    FileSystem fs = FileSystem.get(conf);

    // set local resources for the application master
    // local files or archives as needed
    // In this scenario, the jar file for the application master is part of the local resources
    Map<String, LocalResource> localResources = new HashMap<String, LocalResource>();

    LOG.info("Copy App Master jar from local filesystem and add to local environment");
    // Copy the application master jar to the filesystem
    // Create a local resource to point to the destination jar path
    addToLocalResources(fs, appMasterJarPath, Constants.AM_JAR_NAME, appId,
        localResources, null);

    // Set local resource info into app master container launch context
    amContainer.setLocalResources(localResources);

    // Set the env variables to be setup in the env where the application master will be run
    LOG.info("Set the environment for the application master");
    amContainer.setEnvironment(getAMEnvironment(localResources, fs));

    // Set the necessary command to execute the application master
    Vector<CharSequence> vargs = new Vector<CharSequence>(30);

    // Set java executable command
    LOG.info("Setting up app master command");
    vargs.add(Environment.JAVA_HOME.$$() + "/bin/java");
    // Set Xmx based on am memory size
    vargs.add("-Xmx" + amMemory + "m");
    // Set class name
    vargs.add("com.wikibooks.hadoop.yarn.examples.MyApplicationMaster");
    // Set params for Application Master
    vargs.add("--container_memory " + String.valueOf(containerMemory));
    vargs.add("--container_vcores " + String.valueOf(containerVirtualCores));
    vargs.add("--num_containers " + String.valueOf(numContainers));
    vargs.add("--priority " + String.valueOf(requestPriority));
    vargs.add("1>" + ApplicationConstants.LOG_DIR_EXPANSION_VAR + "/AppMaster.stdout");
    vargs.add("2>" + ApplicationConstants.LOG_DIR_EXPANSION_VAR + "/AppMaster.stderr");

    // Get final command
    StringBuilder command = new StringBuilder();
    for (CharSequence str : vargs) {
      command.append(str).append(" ");
    }

    LOG.info("Completed setting up app master command " + command.toString());
    List<String> commands = new ArrayList<String>();
    commands.add(command.toString());
    amContainer.setCommands(commands);

    return amContainer;
  } 

  ```
  
</details>

애플리케이션마스터를 실행하는 컨테이너에게 필요한 ContainerLaunchContext를 생성합니다.

또한 애플리케이션마스터가 사용할 LocalResource 정보를 보관하는 HashMap 객체도 생성합니다.

##### addToLocalResources 메서드

`addToLocalResources` 메서드를 사용해서 HDFS에 파일을 업로드합니다.

<details markdown="1">
<summary>접기/펼치기</summary>

  ```java
  MyClient.java - addToLocalResources method
    
    // 
    private void addToLocalResources(FileSystem fs, String fileSrcPath,
                                    String fileDstPath, int appId, Map<String, LocalResource> localResources,
                                    String resources) throws IOException {
      String suffix = appName + "/" + appId + "/" + fileDstPath;
      Path dst =
          new Path(fs.getHomeDirectory(), suffix);
      if (fileSrcPath == null) {
        FSDataOutputStream ostream = null;
        try {
          ostream = FileSystem
              .create(fs, dst, new FsPermission((short) 0710));
          ostream.writeUTF(resources);
        } finally {
          IOUtils.closeQuietly(ostream);
        }
      } else {
        fs.copyFromLocalFile(new Path(fileSrcPath), dst);
      }
      FileStatus scFileStatus = fs.getFileStatus(dst);
      LocalResource scRsrc =
          LocalResource.newInstance(
              ConverterUtils.getYarnUrlFromURI(dst.toUri()),
              LocalResourceType.FILE, LocalResourceVisibility.APPLICATION,
              scFileStatus.getLen(), scFileStatus.getModificationTime());
      localResources.put(fileDstPath, scRsrc);
    }       

  ```

</details>

##### getAMEnvironment 메서드

`getAMEnvironment` 메서드를 사용해서 ContainerLaunchContext에 사용할 시스템 환경변수를 설정합니다.

<details markdown="1">
<summary>접기/펼치기</summary>

  ```java
  MyClient.java - getAMEnvironment method
    
    // 
    private Map<String, String> getAMEnvironment(Map<String, LocalResource> localResources
      , FileSystem fs) throws IOException{
    Map<String, String> env = new HashMap<String, String>();

    // Set ApplicationMaster jar file
    LocalResource appJarResource = localResources.get(Constants.AM_JAR_NAME);
    Path hdfsAppJarPath = new Path(fs.getHomeDirectory(), appJarResource.getResource().getFile());
    FileStatus hdfsAppJarStatus = fs.getFileStatus(hdfsAppJarPath);
    long hdfsAppJarLength = hdfsAppJarStatus.getLen();
    long hdfsAppJarTimestamp = hdfsAppJarStatus.getModificationTime();

    env.put(Constants.AM_JAR_PATH, hdfsAppJarPath.toString());
    env.put(Constants.AM_JAR_TIMESTAMP, Long.toString(hdfsAppJarTimestamp));
    env.put(Constants.AM_JAR_LENGTH, Long.toString(hdfsAppJarLength));

    // Add AppMaster.jar location to classpath
    // At some point we should not be required to add
    // the hadoop specific classpaths to the env.
    // It should be provided out of the box.
    // For now setting all required classpaths including
    // the classpath to "." for the application jar
    StringBuilder classPathEnv = new StringBuilder(Environment.CLASSPATH.$$())
        .append(ApplicationConstants.CLASS_PATH_SEPARATOR).append("./*");
    for (String c : conf.getStrings(
        YarnConfiguration.YARN_APPLICATION_CLASSPATH,
        YarnConfiguration.DEFAULT_YARN_CROSS_PLATFORM_APPLICATION_CLASSPATH)) {
      classPathEnv.append(ApplicationConstants.CLASS_PATH_SEPARATOR);
      classPathEnv.append(c.trim());
    }
    env.put("CLASSPATH", classPathEnv.toString());

    return env;
  }  

  ```

</details>

#### 14.2.3 애플리케이션 모니터링

애플리케이션 모니터링은 monitorApplication 메서드에서 실행됩니다.

YarnClient는 1초에 한 번씩 YarnChild의 getApplicationReport 인터페이스를 호출해 실행 상태를 출력합니다.

##### monitorApplication 메서드

  ```java
  MyClient.java
  ...

  private boolean monitorApplication(ApplicationId appId)
      throws YarnException, IOException {

    while (true) {
      // Check app status every 1 second.
      try {
        Thread.sleep(1000);
      } catch (InterruptedException e) {
        LOG.error("Thread sleep in monitoring loop interrupted");
      }

      // Get application report for the appId we are interested in
      ApplicationReport report = yarnClient.getApplicationReport(appId);
      YarnApplicationState state = report.getYarnApplicationState();
      FinalApplicationStatus dsStatus = report.getFinalApplicationStatus();
      if (YarnApplicationState.FINISHED == state) {
        if (FinalApplicationStatus.SUCCEEDED == dsStatus) {
          LOG.info("Application has completed successfully. "
              + " Breaking monitoring loop : ApplicationId:" + appId.getId());
          return true;
        }
        else {
          LOG.info("Application did finished unsuccessfully."
              + " YarnState=" + state.toString() + ", DSFinalStatus=" + dsStatus.toString()
              + ". Breaking monitoring loop : ApplicationId:" + appId.getId());
          return false;
        }
      }
      else if (YarnApplicationState.KILLED == state
          || YarnApplicationState.FAILED == state) {
        LOG.info("Application did not finish."
            + " YarnState=" + state.toString() + ", DSFinalStatus=" + dsStatus.toString()
            + ". Breaking monitoring loop : ApplicationId:" + appId.getId());
        return false;
      }

      if (System.currentTimeMillis() > (clientStartTime + clientTimeout)) {
        LOG.info("Reached client specified timeout for application. Killing application"
            + ". Breaking monitoring loop : ApplicationId:" + appId.getId());
        forceKillApplication(appId);
        return false;
      }
    }
  }

  ...

  ```

##### MyClient 실행(main code)

사용자가 커맨드 라인에서 MyClient 실행을 요청하면 MyClient의 main 메서드가 실행됩니다.

main 메서드에서 MyClient 객체를 선언하고 실행합니다. 

  ```java
  MyClient.java
  ...

  public static void main(String[] args) {
    boolean result = false;
    try {
      MyClient client = new MyClient();
      LOG.info("Initializing Client");
      try {
        boolean doRun = client.init(args);
        if (!doRun) {
          System.exit(0);
        }
      } catch (IllegalArgumentException e) {
        System.err.println(e.getLocalizedMessage());
        client.printUsage();
        System.exit(-1);
      }
      result = client.run();
    } catch (Throwable t) {
      LOG.fatal("Error running CLient", t);
      System.exit(1);
    }
    if (result) {
      LOG.info("Application completed successfully");
      System.exit(0);
    }
    LOG.error("Application failed to complete successfully");
    System.exit(2);
  }
  ...

  ```

### Reference

1. 시작하세요 하둡 프로그래밍
2. [Apache Hadoop]](https://hadoop.apache.org/docs/stable/hadoop-yarn/hadoop-yarn-site/YARN.html)