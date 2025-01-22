--
title: "[ROS2] ROS2 시작하기 (기본 용어와 설치)"
date: 2025-01-17 :23:40 +09:00
categories: SLAM
description: ROS2에 대해 학습하고 설치를 진행해본다.
pin: true
use_math: true
---

## 1. ROS(Robot Operating System)

ROS는 오픈 소스이면서, 로봇 응용 프로그램 개발을 위한 로봇 소프트웨어 프레임워크이다. 이는 로봇들간의 통신이나 센서들과의 통신, 컨트롤을 용이하도록 도와준다. 

> 왜 ROS는 OS라고 이름이 붙었을까?  
> 1. 하드웨어의 추상화
> PC에 설치된 하드웨어의 종류는 다양하고 각각 서로 다른 프로토콜을 사용한다. 하지만 OS가 이에 대한 것을 관리하고 표준을 제공하기 때문에 하드웨어에 대한 자세한 지식이 없어도 사용할 수 있다. 이처럼 ROS 또한, 로봇의 하드웨어를 관리하고 사용자에게 표준을 제공하여 이를 연동한 개발이 가능하다.  
> 2. 통신 관리  
> 운영체제는 프로세스 간의 통신을 관리한다. ROS에서 프로세스는 노드라고 부르는데, 노드 간의 토픽, 서비스, 액션을 통해 통신이 가능하다. 더 자세한 내용은 뒤에서 다룬다.  
> 3. 패키지 관리  
> 운영체제는 응용 프로그램과 그와 관련된 라이브러리의 설치, 의존을 관리한다. ROS는 이처럼 패키지 단위로 로봇의 코드와 기능을 구성하며, 패키지 간의 상호작용과 의존성을 관리한다.  
> 
> 이처럼 ROS는 소프트웨어 입장에서 운영체제와 유사한 역할을 수행한다. 

### 1-1. DDS
 
ROS2는 ROS1과 달리 DDS를 기본 통신 프로토콜로 사용한다. 이를 통해 더 높은 통신의 확장성과 실시간성, 신뢰성을 가진다. 

> DDS(Data Distribution Service)?  
> 분산 시스템에서 데이터 공유와 통신을 효율적으로 관리하는 표준이다. 실시간 데이터 전송을 제공하며, 데이터 중심의 모델을 사용한다. 

ROS2는 미들웨어로써, DDS와 로봇 어플리케이션 사이의 통신을 관리한다.

### 1-2. 워크스페이스(Workspace)

워크스페이스는 ROS 프로젝트를 관리하는 기본 단위로, 워크 스페이스 내에는 다양한 ROS 패키지와 패키지를 빌드하는데 사용되는 환경이 제공된다. 

그 중 ROS 설치 시 기본적으로 제공되는 ROS 기본 패키지와 기본 환경들을 담고 있는 워크 스페이스를 언더레이(underlay)라고 한다. 기존의 언더레이 위에 개발자는 자신의 개발된 코드나 패키지를 추가하여 워크스페이스 확장하는데, 이를 오버레이(overlay)라고 한다. 즉, 오버레이 워크스페이스는 개발 코드와 모든 패키지를 담은 폴더를 말한다.


## 2. ROS 구성요소

이제 ROS 개발에 필요한 구성 요소들에 대해 알아보고 어떻게 통신하고, 데이터를 공유하는지 알아보자.

### 2-1. 노드(Node)

노드는 ROS 시스템에서 기본 실행 단위이다. 즉, 로봇의 각 기능을 수행하는 독립적인 프로세스이다. 예를 들어, 센서 데이터를 읽는 노드, 로봇의 모터를 제어하는 노드 등으로 정의될 수 있다. 

### 2-2. 메세지(Message)

메세지는 노드 간의 주고 받는 정형화된 데이터의 구조이다. ROS는 여러 메세지 타입을 정의하여 다양한 데이터를 구조화하여 전달 할 수 있도록 한다. 예를 들어, 센서 데이터, 로봇의 위치 정보 등이 메세지 형태로 전송 될 수 있다. 

### 2-3. 토픽(Topic)

토픽은 비동기식 데이터 전송을 위한 통신 채널이다. 어떠한 노드가 특정 토픽(채널)으로 메세지(데이터)를 publish하면 다른 노드가 특정 토픽(채널)을 subcribe하여 메세지를 읽을 수 있다.

<img src="{{ site.baseurl }}/assets/img/post/ROS2/topic.png" alt="토픽" style="width: 70%">

메세지를 publish 하는 노드를 publisher, subscribe하는 노드를 subscriber라고 하며, 하나의 토픽에 대해 여러 개의 publisher와 subscriber가 존재할 수 있다. 또, 하나의 노드는 여러 토픽에 대해 publisher이자, 다른 여러 토픽에 대해 subscriber 역할을 할 수 있다. 

예를 들어, 로봇 이동 노드는 여러 토픽을 subscribe하여 현재 위치를 담은 메세지와 목적지 위치를 담은 메세지를 얻어 목적지로 이동 후 완료 메세지를 담아 publish 한다.

### 2-4. 서비스(Service)

<img src="{{ site.baseurl }}/assets/img/post/ROS2/service.png" alt="서비스" style="width: 70%"

서비스는 동기식 요청-응답 방식의 통신으로, 클라이언트는 서비스를 요청하고 서버는 요청된 서비스에 대한 응답을 반환한다. 

### 2-5. 액션(Action)

<img src="{{ site.baseurl }}/assets/img/post/ROS2/action.png" alt="액션" style="width: 70%">

액션은 서비스와 유사하지만 작업 중간 진행사항에 대한 피드백을 받을 수 있어서 시간이 오래 걸리는 작업을 처리할 때 사용된다. 또한 액션은 서비스와 달리 작업을 중간에 취소할 수 있다. 

### 2-6. 런치 파일(Launch File)

런치 파일은 여러 노드를 한번에 실행할 수 있도록 설정해주는 XML 형식의 파일이다. 이를 통해 로봇 시스템의 구성과 실행을 효율적으로 관리할 수 있다. 

### 2-7. 파라미터(Parameter)

파라미터는 노드에서 사용될 수 있는 설정값이다. 예를 들어, 로봇의 이동을 담당하는 노드에서는 이동 속도, 회전 속도, 이동 방향 등을 파라미터로 가질 수 있다. 

### 2-8. 파라미터 서버(Prameter Server)

파라미터 서버는 노드 간에 공유되는 설정 값(파라미터)를 저장하고 관리한다. 런치파일을 통해 파라미터 서버를 초기화하거나 값을 설정할 수 있다. 

## 3. ROS2 Humble 설치

현재 사용하는 Ubuntu의 버전이 22.04이므로 그에 맞는 ROS2 Humble을 설치해서 사용하려고 한다. ROS2 Humble은 [ROS2 공식문서](https://docs.ros.org/en/humble/Installation/Ubuntu-Install-Debs.html)를 따라서 실행하면 큰 문제 없이 설치가 가능하다. 

> Environment setup?  
> 설치 과정의 대부분은 문서에 내용을 터미널로 그대로 복사 붙여넣기하면 된다. 하지만 마지막 단계 Environment setup에서 막힌 사람들이 있을 거 같아서 추가설명을 적고자한다.  
> 새로운 터미널을 열 때마다 추가작업 없이 ros2의 명령어와 기능을 사용할 수 있도록 ros2를 소싱해주어야 한다. 우분투의 경우에는 `vi .bashrc`하여 해당 파일 맨 끝에 `source /opt/ros/humble/setup.bash`를 추가 작성해준다.  
> 이후 새 터미널을 열면 ros2 사용이 가능하다.

<img src="{{ site.baseurl }}/assets/img/post/ROS2/ros2_help.png" alt="ros2 help" style="width: 70%">

만약 설치가 모두 완료되었다면 터미널에 `ros2 --help`를 입력했을 때, 위의 이미지처럼 ros2에서 사용가능한 명령어들이 출력되어야 한다. 

이후 VSCode에서 ros2를 사용하기 위해 도와줄 확장자들을 설치해준다.
- python, C++, ROS
- CMake, CMake Tools, XML, XML Tools

> 리눅스 터미널의  `terminator`를 설치해주면 여러 터미널 세션을 하나의 화면에서 볼 수 있는 등 터미널 사용이 더욱 편리하다.  
> `sudo apt-get install terminator` 설치 이후 `terminator`로 실행  
> <img src="{{ site.baseurl }}/assets/img/post/ROS2/terminator.png" alt="터미네이터" style="width: 70%">  
> - `Ctrl+Shift+e` : 오른쪽에 새로운 터미널 열기  
> - `Ctrl+Shift+o` : 아래에 새로운 터미널 열기

## 참고

ROS2 Documentation의 [Basic Concepts](https://docs.ros.org/en/humble/Concepts/Basic.html)