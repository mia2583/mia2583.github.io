--
title: "[ROS2] ROS2 실습 - Publisher"
date: 2025-01-17 :23:40 +09:00
categories: SLAM
description: ROS2로 간단한 publisher를 작성해본다.
pin: true
use_math: true
---

## 1. 워크스페이스 생성하기

아래 명령어를 통해 새로운 워크스페이스를 생성한다. 

```bash
mkdir -p ros2_ws/src    # 워크스페이스 생성
cd ros2_ws
colcon build            # 초기화
```

워크스페이스를 초기화하게 되면 워크스페이스 내에 자동으로 build, install, log 폴더가 생성된다. 

> 워크스페이스 구조?  
> build : 패키지와 소스코드가 빌드되는 중간 파일들이 저장되는 디렉토리
> install : 빌드된 패키지와 소스코드가 설치되는 디렉토리
> log : 빌드 및 실행 중에 생성된 로그 파일이 저장되는 디렉토리
> src : ROS 패키지의 소스코드들이 저장되는 디렉토리

이제 src 폴더 내에 패키지를 생성한다. 만약 파이썬 패키지를 생성하기를 원한다면 `build-type`을 `ament_python`으로 한다. 나의 경우에는 C++ 패키지를 생성하려고 한다.

```bash
cd src 
# ros2 pkg create --build-type [빌드_타입] [패키지명]
ros2 pkg create --build-type ament_cmake cpp_pubsub
```

그리고 새로 생성한 패키지를 포함해서 워크스페이스를 다시 빌드해준다. 이를 통해 빌드된 패키지의 결과물이 install 폴더에 저장된다.

```bash
cd ../
colcon build
```

<img src="{{ site.baseurl }}/assets/img/post/ROS2/build_success.png" alt="빌드 성공" style="width: 70%">

> 재빌드 시 에러 발생?  
> `/usr/local/bin/cmake: error while loading shared libraries: libssl.so.1.1` 에러가 발생한다면 이는 OpenSSL 1.1이 설치되어 있지 않아서 발생하는 에러이다. 설치 후 재시도하자.  
> [설치 방법 참고](https://xaida.tistory.com/25)

이후 install/setup.bash 파일을 소싱하여 설치된 패키지를 활성화해준다. 

```bash
# 새로운 터미널을 연 후
cd ros2_ws/install
. setup.bash    # 활성화
ros2 pkg list   # 해당 터미널에 한해서만 패키지가 활성화되기 때문에 다른 터미널에서 시도하면 패키지가 보이지 않는다.
```

<img src="{{ site.baseurl }}/assets/img/post/ROS2/pkg_list.png" alt="패키지 리스트" style="width: 70%">

이로써, 설치된 ros2와 개발에 사용된 모든 패키지가 포함된 오버레이 워크스페이스가 된다. 즉, 작업 공간의 코드와 패키지들이 ros2에서 인식된다.

## 2. publisher

이제 생성된 패키지 내에 새로운 노드를 생성해보자. `ros2_ws/src/cpp_pubsub/src/` 안에 새로운 파일 `simple_publisher.cpp`를 생성해준다. 매 초마다 메세지를 publish하는 publisher를 만들고자 한다.

### 2-1. publisher 클래스 생성하기

가장 먼저 C++에서의 ROS2 라이브러리를 포함시켜 ROS2 시스템을 사용할 수 있도록 한다.

```cpp
#include <rclcpp/rclcpp.hpp>
```

그리고 새로운 노드 클래스를 정의한다. 생성자를 통해 노드의 이름을 `simple_publisher`로 하고, publish할 토픽의 이름을 `chatter`로, 큐의 크기를 10으로, 그리고 전송할 메세지 타입을 `std_msg::msg::String`으로 정의한다.

```cpp
#include <std_msgs/msg/string.hpp>

class SimplePublisher : public rclcpp::Node {
public:
    SimplePublisher() : Node("simple_publisher") {
        pub_ = create_publisher<std_msgs::msg::String>("chatter", 10);
    }

private:
    rclcpp::Publisher<std_msg::msg::String>::SharedPtr pub_;
};
```

> 스마트 포인터(`SharedPtr)란?  
> C++ 표준 라이브러리에서 제공하는 포인터로 객체의 메모리를 관리하는데 유용하다. 객체가 더 이상 사용되지 않으면 자동으로 메모리를 해제한다. 주로 동적으로 생성하는 객체에 사용된다.

해당 메세지를 1초당 전송하기 위해 시간과 관련된 C++ 표준 라이브러리 `chrono`를 사용한다.

```cpp

#include <chrono>

using namespace std::chrono_literals;

class SimplePublisher : public rclcpp::Node {
public:
    SimplePublisher() : Node("simple_publisher"), counter_(0) {
        pub_ = create_publisher<std_msgs::msg::String>("chatter", 10);
        timer_ = create_wall_timer(1s, std::bind(&SimplePublisher::timerCallback, this));     // 1초당 함수 timeCallBack()을 호출하는 타이머 생성 

        RCLCPP_INFO(get_logger(), "publishing at 1 Hz");
    }

private:
    unsigned int counter_;
    rclcpp::Publisher<std_msgs::msg::String>::SharedPtr pub_;
    rclcpp::TimerBase::SharedPtr timer_;

    void timerCallback() {     // 메세지를 publish하는 함수
        auto message = std_msgs::msg::String();
        message.data = "Hello ROS2 - counter: " + std::to_string(counter_++);

        pub_->publish(message);
    }
};
```

### 2-2. main 함수 생성

정의한 SimplePublisher 클래스의 인스턴스를 호출해 사용할 수 있도록 하자. 그전에 ROS2 시스템을 초기화해준다.

```cpp
int main(int argc, char *argv[]) {
    rclcpp::init(argc, argv);
}
```

이후, SimplePublisher 노드가 종료될 때까지 이벤트를 실행하고, 노드가 종료되면 ROS2 시스템도 종료한다.

```cpp
rclcpp::spin(std::make_shared<SimplePublisher>());    // 노드 실행
rclcpp::shutdown();   // ROS2 시스템 종료
return 0;
```

### 2-3. CMakeLists.txt 작성하기

소스코드가 모두 작성되었으니 이제 CMakeLists.txt에 이를 빌드하기 위한 설정을 작성해준다.
`project(cpp_pubsub)`는 프로젝트의 이름을 설정하며 이는 나중에 설치나 빌드 작업에서 사용된다. 
빌드 시에 사용하는 패키지들 모두를 `find_package()`를 통해서 작성하며, `REQUIRED`를 통해 해당 패키지가 필수임을 나타낸다.

```
find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(std_msgs REQUIRED)
```

소스코드 `src/simple_publisher.cpp`를 통해 실행파일 `simple_publisher`를 생성하며, 이 소스코드가 의존하는 패키지들을 정의해준다. 

```
add_executable(simple_publisher src/simple_publisher.cpp)
ament_target_dependencies(simple_publisher rclcpp std_msgs)
```

실행파일 `simple_publisher`가 빌드되면, 이를 설치 디렉토리 `lib/cpp_pubsub`에 설치하도록 작성한다. `${PROJECT_NAME}`은 위에서 정의한 `cpp_pubsub`을 의미한다. 

```
install(TARGETS
  simple_publisher
  DESTINATION lib/${PROJECT_NAME}
)
```

### 2-3. package.xml 작성하기

`package.xml` 파일은 패키지의 의존성 관리를 담당한다. 해당 파일 안에 사용한 패키지들을 정의해준다.

```xml
<!-- buildtool_depend 바로 아래에 작성한다. -->
<depend>rclcpp</depend>
<depend>std_msgs</depend>
```

## 3. 실행 결과

```bash
cd ros2_ws
colcon build
# 새 터미널 열기
source ros2_ws/install/setup.bash
# ros2 run [패키지명] [실행파일명]
ros2 run cpp_pubsub simple_publisher
```

<img src="{{ site.baseurl }}/assets/img/post/ROS2/simple_publisher.png" alt="퍼블리셔" style="width: 70%">

pushlish되고 있는 메세지를 확인하고 싶다면 아래 명령어를 통해 확인할 수 있다. 

```bash
# ros2 run ...을 실행중인 상태에서 새 터미널을 열고 아래 명령어들 입력
ros2 topic list     # 현재 publish 중인 토픽 확인하기
ros2 topic echo /chatter    # 토픽의 메세지 읽기
```

<img src="{{ site.baseurl }}/assets/img/post/ROS2/echo_topic.png" alt="토픽 메세지 읽기" style="width: 70%">

publish되고 있는 토픽에 대한 정보를 알고 싶다면 아래 명령어를 입력한다.

```bash
ros2 topic info /chatter --verbose   # 토픽에 대한 전체 개요 확인
ros2 topic hz /chatter  # 해당 토픽이 몇 hz로 메세지를 publish하는지 확인
```

## 4. 전체 코드

## 참고

ROS2 Documentation의 [Beginner:Client libraries](https://docs.ros.org/en/humble/Tutorials/Beginner-Client-Libraries.html)

