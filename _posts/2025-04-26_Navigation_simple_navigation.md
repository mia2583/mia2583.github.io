---
title: "[Navigation] Nav2 설치와 네비게이션 시작하기"
date: 2025-04-26 23:40:00 +09:00
categories: 네비게이션
description: Nav2를 사용하여 간단하게 네비게이션을 실습해본다.
pin: true
use_math: true
---

로봇의 네비게이션 기능의 모든 것을 처음부터 제작하기에는 너무 많은 일이 필요하기에 Nav2 stack을 사용해본다. Nav2 stack은 로봇이 경로를 찾는 것을 돕고 시작점에서 목표점까지 안전히 주행하도록 돕는다. 

## Nav2 설치하기

먼저 Nav2 관련 패키지와 시뮬레이션에 사용할 로봇을 설치해보자

```bash
sudo apt update
# navigation 관련 설치
sudo apt install ros-humble-navigation2 ros-humble-nav2-bringup
# 터틀봇3(시뮬레이션용) 관련 패키지 설치
sudo apt install ros-humble-turtlebot3*
```

네비게이션을 하기 전에 사용할 터틀봇을 정의해준다. bashrc에 등록하지 않고 바로 export 명령어를 사용해도 된다.

```
vi ~/.bashrc
# .bashrc에 아래 내용을 추가하기
export TURTLEBOT3_MODEL=waffle
# .bashrc 적용하기
source ~/.bashrc
```

## 지도 생성하기

네비게이션은 기본적으로 주어진 지도를 바탕으로 경로를 탐색하기 때문에 지도가 먼저 필요하다. 간단하게 cartographer slam을 사용하여 지도를 생성해본다.

```bash
# 터틀봇3 시뮬레이션 실행하기
ros2 launch turtlebot3_gazebo turtlebot3_world.launch.py
# 새 터미널에서 slam 실행하기
ros2 launch turtlebot3_cartographer cartographer.launch.py use_sim_time:=True
# 새 터미널에서 키보드로 터틀봇3 조작하기
ros2 run turtlebot3_teleop teleop_keyboard
# 지도를 완성 후 아래 명령어를 통해서 지도 저장하기 => pgm, yaml 파일 생성
ros2 run nav2_map_server map_saver_cli -f [파일_이름]
```

<img src="{{ site.baseurl }}/assets/img/post/Navigation/turtlebot_map.png" alt="터틀봇으로 그린 지도" style="width: 100%">

지도는 총 3가지 색으로 구성된다. 흰색은 빈공간을, 검은색은 벽이나 장애물, 회색은 가지 못하는 공간 혹은 알지 못하는 공간을 의미한다. 
지도가 정확할 수록 네비게이션 또한 정확해지므로 벽이나 장애물이 뚜렷히 검은색으로 표시되는 것이 좋다. 하지만 현재 생성한 지도는 간단한 주행을 테스트하기 위해 임시로 만든 지도이므로 장애물이 완벽히 표시되지 않은 상태더라도 넘어가기로 한다.

<img src="{{ site.baseurl }}/assets/img/post/Navigation/map_yaml.png" alt="지도 yaml파일" style="width: 100%">

yaml 파일에는 어떠한 정보가 저장되는지 살짝 살펴보자.

- mode : 맵을 어떻게 해석할지를 의미. trinary는 위에서 설명한대로 지도를 3가지 색상으로 구분한다.
- resolution : 지도에 표시할 단위로 픽셀당 미터를 의미. 즉, 위의 이미지는 한 픽셀이 0.05m를 의미한다. pgm의 픽셀 수에 resolution을 곱하면 실제 맵의 크기를 알 수 있다.
- origin : 지도 이미지에서 제일 왼쪽 아래 꼭짓점의 좌표를 의미
- negate : 1로 하면 검은색과 흰색의 의미가 반대로 된다.
- occupied_thresh : threshold 이상의 픽셀값은 occupied(장애물)로 저장
- fee_thresh : threshold 이하의 픽셀값은 free space(빈공간)으로 저장

## 네비게이션

위치 추정이나 Path planning, 주행 등을 수행하기 위해서는 노드들끼리 원활한 통신이 필요하다. ros2의 커뮤니케이션은 DDS 프로토콜을 기반으로 이루어진다. 더 빠른 성능의 DDS를 사용하려면 아래와 같이 설정하면 된다.

```bash
# 설치
sudo apt install ros-humble-cyclonedds-cpp
#
vi ~/.bashrc
export RWM_IMPLEMENTATION=rmw_cyclonedds_cpp
```

로봇이 이동할 때 로봇의 속도나 회전 등의 동작을 어떻게 추적할지 정의할 수 있다. nav2의 amcl 알고리즘은 파티클 필터를 사용하여 로봇의 현재 위치를 추정한다. 이때 로봇이 이동하는 방식을 DifferentialMotionModel로 사용한다.

```bash
cd /opt/ros/humble/share/turtlebot3_navigation2/param
sudo vi ./waffle.yaml
# 만약 robot_model_type이 "differential"로 정의되어 있다면 아래와 같이 수정
robot_model_type: "nav2_amcl::DifferentialMotionModel"
```

이제 모든 준비가 끝났으니 네비게이션을 실행해보자

```bash
# 시뮬레이션 실행
ros2 launch turtlebot3_gazebo turtlebot3_world.launch.py
# 새 터미널에서 네비게이션 실행
ros2 launch turtlebot3_navigation2 navigation2.launch.py use_sim_time:=True map:=[지도_파일].yaml
```

### 로봇의 초기 위치 지정하기

<img src="{{ site.baseurl }}/assets/img/post/Navigation/navigation_option.png" alt="네비게이션 옵션" style="width: 100%">

주행 전에 우선은 로봇의 초기 위치를 지정해줘야 한다. '2D Pose Estimate' 버튼을 클릭 후 지도에 로봇의 위치와 바라보는 방향으로 드래그 해주면 된다.

<img src="{{ site.baseurl }}/assets/img/post/Navigation/path_planner.png" alt="플래너" style="width: 100%">

로봇의 위치가 지정되면 위와 같이 화면이 바뀐다. 전체 지도를 바탕으로 가지고 있는 연한 파랑, 빨강, 보라 등을 global costmap라고 한다. 네비게이션 시에 전체 경로를 생성할 때 사용한다. 반대로 작은 진한 파란색 사각형 내부는 local costmap이라하며, 실시간 주행중에 장애물을 만나거나 지름길이 있는지 살펴보기 위해서 사용된다.

### 로봇 목적지로 보내기

이제 로봇의 목적지를 설정해주자. 초기 위치를 지정할때와 마찬가지로 'Nav2 Goal' 버튼을 클릭후, 목적지의 위치와 방향을 표시해준다.

<img src="{{ site.baseurl }}/assets/img/post/Navigation/direction.png" alt="방향" style="width: 100%">

ros에서는 주로 위와 같은 방향을 가지며, RVIZ에서 맵 프레임의 x축은 빨간색, y축은 초록색, z축은 파란색으로 표시된다. 

목적지가 설정되면, 로봇은 지도에 고정된 장애물을 피해서 주행을 한다. 만약 도달 불가능한 곳이 목적지로 지정되면 해당 goal은 aborted된다.

로봇은 네비게이션을 하면서 자신의 위치를 다시 조금씩 조정하게 된다.

## waypoint follower

이전의 Nav2 Goal은 목적지를 하나만 지정이 가능했다. 이번에는 여러 목적지를 설정하고 로봇이 연속적으로 주행하도록 해본다.

<img src="{{ site.baseurl }}/assets/img/post/Navigation/waypointion.png" alt="waypoint" style="width: 100%">

'Waypoint/Nav Through Poses Mode' 버튼을 클릭한 후 'Nav Goal' 버튼을 통해서 원하는 수만큼의 목적지를 지정해준다. 이후에 'Start Waypoint Following'을 누르면 순서대로 모든 goal을 방문한다.

혹은 'Start Nav Trough Poses'를 사용해도 되지만 해당 옵션은 stable하지 않기 때문에 실행중에 문제가 있을 수 있다.

## dynamic obstacle avoidance

지금까지의 주행은 벽이나 장애물이 모든 지도상에 고정된 상태였다. 그래서 로봇이 주행을 할때 이를 피해서 주행하는 것은 쉽다. 하지만 만약 사람과 같이 지도에 포함되지 않는 물체가 등장한다면 로봇은 어떻게 주행할까?

<img src="{{ site.baseurl }}/assets/img/post/Navigation/gazebo_obstacle.png" alt="가지보에서 장애물 추가하기" style="width: 100%">

일단 먼저 시뮬레이션 상에서 지도에 없는 새로운 물체를 넣어보자. gazebo 상에서 원래 지도에 없던 물체를 추가한다.

<img src="{{ site.baseurl }}/assets/img/post/Navigation/before_obstacle.png" alt="장애물 인식 전 경로" style="width: 100%">

그리고 RVIZ에서 Nav2 Goal을 지정하면 로봇이 알고있는 정보에는 아직 장애물이 없으므로 로봇은 장애물이 없는 상태에서의 최적의 경로를 생성한다.

<img src="{{ site.baseurl }}/assets/img/post/Navigation/after_obstacle.png" alt="장애물 인식 후 경로" style="width: 100%">

하지만 이동중 라이다에 장애물이 인식이 되고 해당 경로로 지나갈 수 없음을 알게된다. 그러자 장애물을 통과하지 않는, 새로운 경로를 생성한다.
