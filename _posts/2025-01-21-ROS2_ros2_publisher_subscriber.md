---
title: "[ROS2] ROS2 실습 - Publisher, Subscriber"
date: 2025-01-21 20:23:00 +09:00
categories: SLAM
description: ROS2로 간단한 publisher와 subscriber를 작성해본다.
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

<img src="{{ site.baseurl }}/assets/img/post/ROS2/build_success.png" alt="빌드 성공" style="width: 100%">

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

<img src="{{ site.baseurl }}/assets/img/post/ROS2/pkg_list.png" alt="패키지 리스트" style="width: 90%">

이로써, 설치된 ros2와 개발에 사용된 모든 패키지가 포함된 오버레이 워크스페이스가 된다. 즉, 작업 공간의 코드와 패키지들이 ros2에서 인식된다.

## 2. Publisher

이제 생성된 패키지 내에 새로운 노드를 생성해보자. `ros2_ws/src/cpp_pubsub/src/` 안에 새로운 파일 `simple_publisher.cpp`를 생성해준다. 매 초마다 메세지를 publish하는 publisher를 만들고자 한다.

### 2-1. Publisher 클래스 생성하기

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
        // 1초당 함수 timeCallBack()을 호출하는 타이머 생성 
        timer_ = create_wall_timer(1s, std::bind(&SimplePublisher::timerCallback, this));     

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

### 2-4. package.xml 작성하기

`package.xml` 파일은 패키지의 의존성 관리를 담당한다. 해당 파일 안에 사용한 패키지들을 정의해준다.

```xml
<!-- buildtool_depend 바로 아래에 작성한다. -->
<depend>rclcpp</depend>
<depend>std_msgs</depend>
```

### 2-5. 실행 결과

```bash
cd ros2_ws
colcon build
# 새 터미널 열기
source ros2_ws/install/setup.bash
# ros2 run [패키지명] [실행파일명]
ros2 run cpp_pubsub simple_publisher
```

<img src="{{ site.baseurl }}/assets/img/post/ROS2/simple_publisher.png" alt="퍼블리셔" style="width: 100%">

publish 되고 있는 메세지를 확인하고 싶다면 아래 명령어를 통해 확인할 수 있다. 

```bash
# ros2 run ...을 실행중인 상태에서 새 터미널을 열고 아래 명령어들 입력
ros2 topic list     # 현재 publish 중인 토픽 확인하기
ros2 topic echo /chatter    # 토픽의 메세지 읽기
```

<img src="{{ site.baseurl }}/assets/img/post/ROS2/echo_topic.png" alt="토픽 메세지 읽기" style="width: 90%">

publish 되고 있는 토픽에 대한 정보를 알고 싶다면 아래 명령어를 입력한다.

```bash
ros2 topic info /chatter --verbose   # 토픽에 대한 전체 개요 확인
ros2 topic hz /chatter  # 해당 토픽이 몇 hz로 메세지를 publish하는지 확인
```

### 3. Subscriber

터미널로 메세지를 읽는게 아닌 특정 토픽을 subscribe하는 subscriber 노드를 생성해보자. 먼저 노드를 생성하기위해 ros2 기본 라이브러인 `rclcpp`와 메세지 타입인 `std_msgs` 패키지를 포함한다.

```cpp
#include <rclcpp/rclcpp.hpp>
#include <std_msgs/msg/string.hpp>
```

### 3-1. Subscriber 클래스 생성하기

이제 토픽 `chatter`로부터 메세지 타입이 `std_msg::msg::String`인 메세지를 읽어서 이를 출력해주는 subscriber 클래스를 정의해준다.

```cpp
using std::placeholders::_1;

class SimpleSubscriber : public rclcpp::Node {    // subscriber 클래스
public:
    SimpleSubscriber() : Node("simple_subscriber") {
        sub_ = create_subscription<std_msgs::msg::String>("chatter", 10, std::bind(&SimpleSubscriber::msgCallback, this, _1));
    }

private:
    rclcpp::Subscription<std_msgs::msg::String>::SharedPtr sub_;

    void msgCallback(const std_msgs::msg::String &msg) const {    // 읽은 메세지 출력하기
        RCLCPP_INFO_STREAM(get_logger(), "I heard: " << msg.data.c_str());
    }
};
```

`std::placeholder`은 `std::bind()`와 함께 사용되며, 매개변수의 자리 표시자를 의미한다. 
이는 `create_subscrition`이 메세지를 받으면 이를 `msgCallback` 함수의 첫 번째 인자(`_1`)인 `const std_msgs::msg::String`으로 전달된다.

### 3-2. main 함수 생성

publisher와 마찬가지로 subscriber 노드를 생성 후, 노드가 종료될 때까지 계속 실행한다.

```cpp
int main(int argc, char* argv[]) {
    rclcpp::init(argc, argv);
    auto node = std::make_shared<SimpleSubscriber>();
    rclcpp::spin(node);
    rclcpp::shutdown();
    return 0;
}
```

### 3-3. CMakeLists.txt 작성하기

publisher를 빌드하기 위해서 사용한 CMakeLists.txt에 subscriber 노드에 대해서도 빌드를 할 수 있게 아래 코드를 추가해준다.

```
add_executable(simple_subscriber src/simple_subscriber.cpp)
ament_target_dependencies(simple_subscriber rclcpp std_msgs)

install(TARGETS
  simple_publisher
  simple_subscriber
  DESTINATION lib/${PROJECT_NAME}
)
```

### 3-5. 실행 결과

```bash
cd ros2_ws
colcon build
# 새 터미널 열기
source ros2_ws/install/setup.bash
# ros2 run [패키지명] [실행파일명]
ros2 run cpp_pubsub simple_subscriber
# 새 터미널 열기
source ros2_ws/install/setup.bash
ros2 run cpp_pubsub simple_publisher
```

<img src="{{ site.baseurl }}/assets/img/post/ROS2/pubsub.png" alt="퍼블리셔와 서브스크라이버" style="width: 90%">

이때, publisher 노드와 subscriber 노드가 서로 다른 언어로 작성되었어도 publish한 메세지를 subscribe 가능하다. 
그리고 만약 publisher가 아니라 터미널로 하나의 메세지를 publish하고 싶다면 아래의 명령어를 입력한다.

```bash
# ros2 topic pub [토픽] [메세지_타입] [메세지_형식+메세지] -> tab tab으로 힌트를 얻을 수 있음
ros2 topic pub /chatter std_msgs/msg/String "data: 'Hello'"
```

<img src="{{ site.baseurl }}/assets/img/post/ROS2/subscriber.png" alt="토픽 서브스크라이브" style="width: 90%">

publish되고 있는 토픽에 대한 정보를 알고 싶다면 아래 명령어를 입력한다.

```bash
ros2 topic info /chatter --verbose   # 토픽에 대한 전체 개요 확인
ros2 topic hz /chatter  # 해당 토픽이 몇 hz로 메세지를 publish하는지 확인
```

## 4. 전체 코드

[cpp_pubsub 패키지](https://github.com/mia2583/Robotics/tree/main/ros2_ws/src/cpp_pubsub)

## 참고

ROS2 Documentation의 [Beginner:Client libraries](https://docs.ros.org/en/humble/Tutorials/Beginner-Client-Libraries.html)

