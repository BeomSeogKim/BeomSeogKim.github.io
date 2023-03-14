---
layout: single
title:  "[ToyProject] AWS 설정"
categories: ToyProject
tag: [AWS]
toc: true   
author_profile: false
sidebar:
    nav: "main"
search:
    true
---

# [ToyProject] AWS 설정 

> 이번 장에서는 AWS 설정을 하는 것에 대해 알아보려고 한다. 

큰 순서는 다음과 같다. 

- [IAM 역할 생성](IAM-역할-생성)
- [IAM 사용자 생성](IAM-사용자-생성)
- [S3 생성](S3-생성)
- [EC2 생성](EC2-생성)
- [CodeDeploy 생성](CodeDeploy-생성)
- [EC2 추가 설정](EC2-추가-설정)



EC2에 배포하기에 앞서 AWS-Amazon에 가입을 진행하자.

https://aws.amazon.com/ko/



## IAM 역할 생성

#### [IAM - EC2]

가입을 진행 한 후 AWS에 로그인을 한 후에 상단에 검색창에 `IAM` 검색 후 역할 만들기 클릭

![image-20230313183545763]({{site.url}}/images/2023-03-13-ToyProject2/image-20230313183545763.png)



사용 사례에 `EC2` 클릭

![image-20230313211958366]({{site.url}}/images/2023-03-13-ToyProject2/image-20230313211958366.png)



`AmazonCodeDeployFullAccess`, `AmazonS3FullAccess` 권한 추가

![image-20230313214805856]({{site.url}}/images/2023-03-13-ToyProject2/image-20230313214805856.png)



역할 이름 지정 (이름의 경우 원하는 대로 지정하면 된다.)

![image-20230313214928536]({{site.url}}/images/2023-03-13-ToyProject2/image-20230313214928536.png)

---

#### [IAM - CodeDeploy]

마찬가지로 `역할 생성`을 누른 후 Code Deploy 선택

![image-20230313215225273]({{site.url}}/images/2023-03-13-ToyProject2/image-20230313215225273.png)



추가적인 권한 추가 없이 역할 이름만 지정해 준다.

![image-20230313215316627]({{site.url}}/images/2023-03-13-ToyProject2/image-20230313215316627.png)



## IAM 사용자 생성 

IAM 에서 사용자 클릭 후 사용자 추가 클릭 

![image-20230314152530876]({{site.url}}/images/2023-03-13-ToyProject2/image-20230314152530876.png)



사용자 이름은 원하는 대로 설정 

`직접 정책 연결` 선택 후 다음 세가지 항목 추가

* AmazonEC2FullAccess 
* AmazonS3FullAccess
* AWSCodeDployFullAccess 



![image-20230314152742941]({{site.url}}/images/2023-03-13-ToyProject2/image-20230314152742941.png)



사용자 생성 후 사용자를 클릭한다. 

액세스 키 만들기를 선택한 후 `AWS 외부에서 실행되는 애플리케이션`을 선택하고 키 생성을 한다. 

액세스 키의 경우 자주 사용하니 csv 파일을 저장하여 찾기 쉬운 곳에 놔두자. 

**액세스 키의 경우 절대적으로 외부에 노출이 되면 안된다. 추후에 application.properties 설정을 할 때 gitignore 처리를 꼭 해야 한다.** 

![image-20230314153721290]({{site.url}}/images/2023-03-13-ToyProject2/image-20230314153721290.png)



## S3 생성 

> Jenkins를 활용한 압축 빌드 파일을 담기 위한 S3 생성 

S3를 검색 한 후 `버킷 만들기` 클릭

![image-20230313215557390]({{site.url}}/images/2023-03-13-ToyProject2/image-20230313215557390.png)



나머지 설정들은 Default 값으로 놔두고 버킷 이름만 지정 한 후 생성 해준다.

![image-20230313215734359]({{site.url}}/images/2023-03-13-ToyProject2/image-20230313215734359.png)



## EC2 생성

> 우리가 앞으로 실행할 가상 서버 EC2를 설정해 주자

`인스턴스 시작` 클릭

![image-20230313220002026]({{site.url}}/images/2023-03-13-ToyProject2/image-20230313220002026.png)



인스턴스 이름을 지정해준다.

![image-20230313220037377]({{site.url}}/images/2023-03-13-ToyProject2/image-20230313220037377.png)



EC2 서버의 경우  Amazon Linux 2 AMI 버전을 사용하자. 

![image-20230313220123102]({{site.url}}/images/2023-03-13-ToyProject2/image-20230313220123102.png)



키 페어가 있는 사용자의 경우 있는 키 페어를 사용하고 없는 사람들은 `새 키 페어 생성`을 클릭해준다.

![image-20230313220237084]({{site.url}}/images/2023-03-13-ToyProject2/image-20230313220237084.png)



키 페어 이름의 경우 알 수 있게끔 이름을 생성해주면 된다. 

*필자의 경우 aws 계정이 daum계정임을 확인하기위해 daum-key 라고 설정했다.*

**다운받은 pem 키파일의 경우 알기 쉬운 곳에 잘 보관 해 놓을 것!**

후에 ssh 접속을 할 때 사용해야 하므로 꼭 알기 쉬운 곳에 잘 보관해 놓자.

![image-20230313220326820]({{site.url}}/images/2023-03-13-ToyProject2/image-20230313220326820.png)



보안 그룹의 경우 `SSH`, `HTTP`를 허용해준다.

![image-20230313220657521]({{site.url}}/images/2023-03-13-ToyProject2/image-20230313220657521.png)



스토리지의 경우 프리티어는 30 GB까지 가능하므로 30GB로 설정해준다.

![image-20230313220827759]({{site.url}}/images/2023-03-13-ToyProject2/image-20230313220827759.png)

설정을 다 해주었다면 인스턴스 시작을 눌러주자.



인스턴스 목록에서 IAM 역할 수정 클릭

![image-20230313221240158]({{site.url}}/images/2023-03-13-ToyProject2/image-20230313221240158.png)



앞서 생성했던 `IAM 역할` 클릭 후 업데이트 클릭

![image-20230313221332996]({{site.url}}/images/2023-03-13-ToyProject2/image-20230313221332996.png)



역할을 수정해 주었다면 인스턴스 상태에서 재부팅을 클릭해 준다.

(재부팅을 해야 IAM 역할이 정상적으로 작동한다.)

![image-20230313221537921]({{site.url}}/images/2023-03-13-ToyProject2/image-20230313221537921.png)



EC2 대시보드에서 `보안`  클릭 후 `보안그룹` 에서 `인바운드 규칙 편집` 을 누른다.

다음과 같이 설정 후 규칙 저장 (8080 : WAS port, 9000 : jenkins port)

![image-20230313221831889]({{site.url}}/images/2023-03-13-ToyProject2/image-20230313221831889.png)



서버를 껐다 켜도 동일한 IP를 가질 수 있도록 `탄력적 IP 주소 할당` 클릭

![image-20230313222938443]({{site.url}}/images/2023-03-13-ToyProject2/image-20230313222938443.png)



별 다른 설정 없이 생성하면 다음과 같이 생성된다.

![image-20230313223117863]({{site.url}}/images/2023-03-13-ToyProject2/image-20230313223117863.png)



`작업`을 누른 후 탄력적 IP 주소 연결 클릭

앞서 만든 EC2 인스턴스를 선택해 주고 IP주소를 클릭해 준 후 연결 클릭

(EC2 대시보드에서 잘 할당되었는지 꼭 확인해주자)

![image-20230313223444551]({{site.url}}/images/2023-03-13-ToyProject2/image-20230313223444551.png)



## CodeDeploy 생성 

CodeDeploy 검색 후 `애플리케이션 생성` 클릭

![image-20230313222019053]({{site.url}}/images/2023-03-13-ToyProject2/image-20230313222019053.png)



`컴퓨팅 플랫폼`은 EC2/온프레미스, 이름은 자유롭게 지어준다.

![image-20230313222136184]({{site.url}}/images/2023-03-13-ToyProject2/image-20230313222136184.png)



배포 그룹 생성을 클릭 해준다.

배포 그룹 이름은 알기 쉽도록 작성해준다. 서비스 역할은 앞전에 생성했던 IAM을 선택해준다.

![image-20230313222311220]({{site.url}}/images/2023-03-13-ToyProject2/image-20230313222311220.png)



`배포 유형`은 현재위치

`환경 구성`은 Amazon EC2 인스턴스 

태그 그룹 키는 Name, 값은 생성한 EC2를 선택해준다.

![image-20230313222424160]({{site.url}}/images/2023-03-13-ToyProject2/image-20230313222424160.png)



AWS CodeDeploy 에이전트의 경우 직접 설치할 예정이므로 `안함` 선택

![image-20230313222618198]({{site.url}}/images/2023-03-13-ToyProject2/image-20230313222618198.png)



로드 밸런서의 경우 아직은 적용 예정이 없으므로 비활성화 시켜준다.

설정을 마쳤으면 `배포그룹 생성` 클릭

![image-20230313222702743]({{site.url}}/images/2023-03-13-ToyProject2/image-20230313222702743.png)

이로써 기본적인 AWS 설정은 마쳤으므로 EC2 추가 설정을 해주도록 하자.



## EC2 추가 설정



* pem 키 파일 권한 변경 

  * sudo chmod 400 (다운받은 pem 키파일 드래그하기)

* ssh 접속 

  * ```ruby
    ssh -i 받은 pem 키페어를끌어다놓기 ec2-user@AWS에적힌내아이피
    ```

* ~ yes / no 문구가 뜬다면 yes 타이핑 후 엔터



**[시간 동기화]**

* ec2 시간 확인

  * ```ruby
    date
    ```

* chrony 방식으로 시간 동기화를 하도록 요청

  * ```ruby
    sudo chkconfig chronyd on
    ```

* 169.254.169.123 IP 주소를 사용하여 시간 동기화를 하고 있는지 확인

  * ```ruby
    chronyc sources -v
    ```

* 타임존 설정

  * ```ruby
    sudo vim /etc/sysconfig/clock
    ```

* 값 변경

  * ```ruby
    ZONE="Asia/Seoul"
    KST=True
    ```

* 타임존 확인 

  * ```ruby
    cat /etc/localtime
    ```

* 만약 UTC로 되어 있다면 기존 설정 삭제

  * ```ruby
    sudo rm /etc/localtime
    ```

* 대한민국 표준 시간대 정보 설정

  * ```shell
    sudo ln -s /usr/share/zoneinfo/Asia/Seoul /etc/localtime
    ```

* 시간 확인 

  * ```ruby
    date
    ```

**[java 17 설치]**

* 설치 명령어 

  * ```ruby
    sudo yum install java-17-amazon-corretto
    ```

* 자바 버전 확인

  * ```ruby
    java --version
    ```

**[메모리 설정]**

프리티어인 t2.micro의 경우 메모리는 1GB로 WAS와 Jenkins를 둘다 띄우기에 무리일 수 있으므로 swap 설정을 추가적으로 해준다.



권장하는 SWAP 메모리의 크기는 다음과 같다.

![swap]({{site.url}}/images/2023-03-13-ToyProject2/swap.png)



* dd 명령어를 통한 swap 메모리 할당

  * ```ruby
    sudo dd if=/dev/zero of=/swapfile bs=128M count=16
    ```

* swap 파일의 읽기 및 쓰기 권한 업데이트 

  * ```ruby
    sudo chmod 600 /swapfile
    ```

* Linux swap 영역 설정 

  * ```ruby
    sudo mkswap /swapfile
    ```

* swap 공간에 swap file 추가

  * ```ruby
    sudo swapon /swapfile
    ```

* 설정 확인 

  * ```ruby
    sudo swapon -s
    ```

* fstab에 /swapfile 설정 추가

  * ```ruby
    sudo vi /etc/fstab
    ```

  * 마지막 줄에 다음 코드 삽입

    * ```ruby
      /swapfile swap swap defaults 0 0
      ```

* 적용 내용 확인 

![image-20230314112320460]({{site.url}}/images/2023-03-13-ToyProject2/image-20230314112320460.png)



**[codedeploy agent 설정]**

* aws-cli 설치 확인 

  * ```ruby
    aws --version 
    ```

* aws 설정 

  * ```ruby
    sudo aws configure
    {아래 코드는 직접 기입해주면 됩니다.}
    AWS Access Key ID [None]: Access Key
    AWS Secret Access Key [None]: Secret Access Key
    Default region name [None]: ap-northeast-2 {혹시 서버를 한국이 아닌 다른곳으로 했을 경우 변경}
    Default output format [None]: json
    ```

* codedeploy agent 설치 파일 다운로드

  * ```ruby
    wget https://aws-codedeploy-ap-northeast-2.s3.amazonaws.com/latest/install
    ```

* ruby 설치 

  * ```ruby
    sudo yum install ruby -y
    chmod +x ./install
    sudo ./install auto
    ```

* codedeploy-agent 동작 확인 

  * ```ruby
    sudo service codedeploy-agent status
    ```



***다음 장에서는 Jenkins를 활용한 자동 배포를 알아보자.***

---



참고 링크 

[Linux 설정](https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/set-time.html)

[swap 관련](https://aws.amazon.com/ko/premiumsupport/knowledge-center/ec2-memory-swap-file/)

[java17 설치](https://docs.aws.amazon.com/corretto/latest/corretto-17-ug/amazon-linux-install.html)

