---
layout: single
title:  "[ToyProject] Jenkins를 활용한 EC2 자동 배포"
categories: ToyProject
tag: []
toc: true   
author_profile: false
sidebar:
    nav: "main"
search:
    true
---

# [ToyProject] Jenkins를 활용한 EC2 자동 배포

해당 장은 다음과 같이 구성됩니다. 

* [Docker 설치](Docker-설치)
* [Jenkins 설정](Jenkins-설정)
* [GitHub Webhook](Github-Webhook)



원래는 Jenkins 서버를 따로 두어 관리하는 것을 권장하지만, aws 프리티어를 사용하기 위해 Docker에 Jenkins를 띄웠습니다. 

[CI / CD 과정]

1. github에 개발된 내용을 push 
2. EC2 서버에서 도커환경에서 돌아가고 있는 Jenkins가 이를 감지 
3. 성공적으로 build 될 경우 build 파일을 S3에 업로드
4. Jenkins 에서 CodeDeploy에 배포요청 
5. S3에서 CodeDeploy에게 빌드 파일 전달. 
6. EC2에서 배포



## Docker 설치

* yum update

  * ```ruby
    sudo yum update -y
    ```

* Docker 설치 

  * ```ruby
    sudo amazon-linux-extras install -y docker
    ```

* Docker 실행

  * ```ruby
    sudo service docker start
    ```

* jenkins 설정 

  * ```ruby
    sudo docker run -d --name jenkins -p 9000:8080 -e TZ=Asia/Seoul jenkins/jenkins:jdk17
    ```

* jenkins 실행 확인 

  * ```ruby
    sudo docker ps -a
    ```



## Jenkins 설정

* 내가 할당 받은 ec2 ip주소:9000번으로 접속

![image-20230314113838409]({{site.url}}/images/2023-03-14-ToyProject3/image-20230314113838409.png)

Jenkins를 처음 구동 시키면 password가 필요하다.

* docker 에 구동중인 jenkins 접속 

  * ```ruby
    sudo docker exec -it jenkins bash
    ```

* initial Password 확인

  * ```ruby
    cat /var/jenkins_home/secrets/initialAdminPassword
    ```

* initial Password를 Jenkins 서버에 입력

* Install suggested plugins 선택

* 설치가 완료된다면 다음과 같은 화면이 나타난다.

![image-20230314120158580]({{site.url}}/images/2023-03-14-ToyProject3/image-20230314120158580.png)

이 부분은 자유롭게 적어주도록 하자.

* Instance Configuration의 경우 기본 설정으로 두고 `Save and Finish` 클릭
* 이로써 기본적인 Jenkins 구축은 완료 되었다.



추가적으로 Jenkins 설정을 해주자.

* Jenkins `관리` 탭에서 `플러그인 관리` 클릭

![image-20230314120601580]({{site.url}}/images/2023-03-14-ToyProject3/image-20230314120601580.png)



* `Available plugins` 에서 aws codedeploy 검색 후 설치 

![image-20230314120656522]({{site.url}}/images/2023-03-14-ToyProject3/image-20230314120656522.png)



* Dashboard에서 새로운 Item 클릭 
* item name : 자유롭게 생성 
* Freestyle project 클릭

![image-20230314150044528]({{site.url}}/images/2023-03-14-ToyProject3/image-20230314150044528.png)

* Configure 설정 하기 

  * 설명 

    * 알아보기 쉽게 편하게 작성

  * 소스코드 관리

    * Git 선택
    * Repository Url : 해당 프로젝트가 올라가 있는 GitHub url을 입력
    * Credential : 만약 private 이면 적어주면 된다. public인 경우 그냥 넘어가도 된다.

  * Branch to build

    * Branch Specifier : 소스코드 빌드 유발하는 branch 명을 입력하자. 

      

  ![image-20230314151218298]({{site.url}}/images/2023-03-14-ToyProject3/image-20230314151218298.png)

  

  * 빌드 유발 
    * GitHub hook trigger for GITScm polling 선택
  * Build Steps
    * `Add build step`을 누른 뒤 `Execute shell` 선택
    * command에는 ./gradlew clean build 입력 

  ![image-20230314151326249]({{site.url}}/images/2023-03-14-ToyProject3/image-20230314151326249.png)

  

  * 빌드 후 조치 
    * Deploy an application to AWS CodeDeploy 선택 
      * Application Name : CodeDeploy Application 이름 기입
      * Deployment Group : Application에 할당된 배포그룹 이름 기입
      * AWS Region : 한국으로 생성 했으므로 AP_NORTHEAST_2 선택
      * S3 bucket : 앞서 생성 했던 S3 버킷 이름을 넣어준다.

  ![image-20230314151348601]({{site.url}}/images/2023-03-14-ToyProject3/image-20230314151348601.png)

  

  * AWS Acess key, AWS Secret key에서 앞전에 다운 받았던 key를 넣어주면된다.

    ![image-20230314154024851]({{site.url}}/images/2023-03-14-ToyProject3/image-20230314154024851.png)

  * Apply 후 저장하면 된다.



## GitHub Webhook

* Github Repository에서 Settings 선택
  * Code and automation 탭에서 Webhooks 선택
    * add web hook
      * PayLoad URL : http://자기 서버 ip:9000/github-webhook/
      * Content type : application/json



다음과 같이 체크표시가 나타난다면 설정이 된 것이다.

![image-20230314154609848]({{site.url}}/images/2023-03-14-ToyProject3/image-20230314154609848.png)



## Project 설정 



[build.gradle]

build.gradle 파일에 다음 코드를 넣어주자. (plain jar 생성을 금지하는 코드 )

```java
// plain.jar 생성 금지
jar{
    enabled=false
}
```



[appspec.yml & scripts]

다음 경로에 `appspec.yml`, `scripts/deploy.sh`를 생성해준다.

![image-20230314155152172]({{site.url}}/images/2023-03-14-ToyProject3/image-20230314155152172.png)



[appspec.yml]

```java
version: 0.0
os: linux
files:
  - source:  /
    destination: /home/ec2-user/toy-project		{복사할 directory}
    overwrite: yes

permissions:
  - object: /
    pattern: "**"
    owner: ec2-user
    group: ec2-user

hooks:
  ApplicationStart:
    - location: scripts/deploy.sh
      timeout: 60
      runas: ec2-user
```



[deploy.sh]

```shell
#!/bin/bash
BUILD_JAR=$(ls /home/ec2-user/toy-project/build/libs/*.jar)
JAR_NAME=$(basename $BUILD_JAR)
echo "> build : $JAR_NAME" >> /home/ec2-user/deploy.log

echo "> build 파일 복사" >> /home/ec2-user/deploy.log
DEPLOY_PATH=/home/ec2-user/
cp $BUILD_JAR $DEPLOY_PATH

echo "> 실행중인 애플리케이션 pid 확인" >> /home/ec2-user/deploy.log
CURRENT_PID=$(pgrep -f $JAR_NAME)

if [ -z $CURRENT_PID ]
then
  echo "> 실행중인 애플리케이션이 없으므로 종료하지 않음" >> /home/ec2-user/deploy.log
else
  echo "> kill -15 $CURRENT_PID" >> /home/ec2-user/deploy.log
  kill -15 $CURRENT_PID
  sleep 10
fi

DEPLOY_JAR=$DEPLOY_PATH$JAR_NAME
echo "> DEPLOY_JAR 배포"    >> /home/ec2-user/deploy.log
nohup java -jar $DEPLOY_JAR >> /home/ec2-user/deploy.log 2>/home/ec2-user/deploy_err.log &
```



마지막으로 github에 push 를 하면 jenkins에서 build를 시작 할 것이다. 

![image-20230314231735808]({{site.url}}/images/2023-03-14-ToyProject3/image-20230314232156331.png)

다음과 같이 정상적으로 build가 되었다면 

`http://ec2주소:8080/health-check`를 요청하였을 때 "ok"가 return 된다면 정상적으로 환경 구성이 완료 된 것이다.



혹시나 궁금한 점 있으면 언제든지 연락 주시길 바랍니다.
