---
title: "[OpenCV] OpenCV 설치와 테스트 프로젝트"
date: 2025-01-30 15:00:00 +09:00
categories: OpenCV
description: 우분투에 OpenCV를 설치하고 C++ 환경에서 빌드를 시도한다. 
pin: true
use_math: true
---

OpenCV는 open source Computer Vision Library을 약어로, 이미지/영상 처리 및 딥러닝과 같은 컴퓨터 비전 관련 기능을 제공하는 라이브러리이다.
이번 포스팅에서는 OpenCV를 설치하고 테스트 프로젝트를 실행시켜본다.

## 1. OpenCV(C++) 설치

최신 버전의 OpenCV를 설치하기 위해서는 많은 패키지들을 미리 설치해야 한다. 
터미널을 열어 아래 명령어들을 입력하자.

```
sudo apt-get install build-essential cmake
sudo apt-get install pkg-config
sudo apt-get install libjpeg-dev libtiff5-dev libpng-dev
sudo apt-get install ffmpeg libavcodec-dev libavformat-dev libswscale-dev libxvidcore-dev libx264-dev libxine2-dev
sudo apt-get install libv4l-dev v4l-utils
sudo apt-get install libglx-dev
sudo apt-get install libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev
sudo apt-get install libgtk-3-dev
sudo apt-get install libgtk2.0-dev
sudo apt-get install libgtk3.0-cil-dev
sudo apt-get install mesa-utils libgl1-mesa-dri libgtkgl2.0-dev libgtkglext1-dev
sudo apt-get install libatlas-base-dev gfortran libeigen3-dev
sudo apt-get install python3-dev python3-numpy
```

이후 빌드할 OpenCV 소스코드를 다운받아 압축해제 한다. 
나의 경우에는 완전 최신보다는 2023년 12월에 배포된 4.9.0 버전을 설치하였다.
배포 버전은 [opencv/release](https://opencv.org/releases/)에서 확인할 수 있다.

```
wget -O opencv.zip https://github.com/opencv/opencv/archive/4.9.0.zip
wget -O opencv_contrib.zip https://github.com/opencv/opencv_contrib/archive/4.9.0.zip
unzip opencv.zip
unzip opencv_contrib.zip
```

압축해제한 opencv 폴더 내에 빌드 폴더를 생성해준다.

```
cd opencv-4.9.0
sudo mkdir bulid
cd bulid
```

cmake 명령어를 통해 Makefile을 생성한다. 
나의 경우에는 권한 에러로 인해 sudo명령어를 사용하였고, `libssl.so.1.1` 에러로 라이브러리 경로를 직접 설정하였다.

```
sudo LD_LIBRARY_PATH=$LD_LIBRARY_PATH cmake -D CMAKE_BUILD_TYPE=RELEASE \
-D CMAKE_INSTALL_PREFIX=/usr/local \
-D WITH_TBB=OFF \
-D WITH_IPP=OFF \
-D WITH_1394=OFF \
-D BUILD_WITH_DEBUG_INFO=OFF \
-D BUILD_DOCS=OFF \
-D INSTALL_C_EXAMPLES=ON \
-D INSTALL_PYTHON_EXAMPLES=ON \
-D BUILD_EXAMPLES=OFF \
-D BUILD_PACKAGE=OFF \
-D BUILD_TESTS=OFF \
-D BUILD_PERF_TESTS=OFF \
-D WITH_QT=OFF \
-D WITH_GTK=ON \
-D WITH_OPENGL=OFF \
-D BUILD_opencv_python3=ON \
-D OPENCV_EXTRA_MODULES_PATH=../../opencv_contrib-4.9.0/modules \
-D WITH_V4L=ON \
-D WITH_FFMPEG=ON \
-D WITH_XINE=ON \
-D OPENCV_ENABLE_NONFREE=ON \
-D BUILD_NEW_PYTHON_SUPPORT=ON \
-D OPENCV_SKIP_PYTHON_LOADER=ON \
-D OPENCV_GENERATE_PKGCONFIG=ON \
-D WITH_JAVA=OFF ../
```

또한 make 시에 발생한 JAVA 에러와 OpenGL에러를 해결하기 위해 `-D WITH_OPENGL=OFF \`와 `-D WITH_JAVA=OFF`를 옵션으로 설정하였다.

그리고 make 실행에 권한 에러를 해결하기 위해 일부 폴더의 권한 설정을 수정하였다.

```
sudo chown -R USER:USER [경로]/opencv/opencv-4.9.0
sudo chmod -R 777 [경로]/opencv
```
https://sanghyunpark01.github.io/ubuntu/tips/Ubuntu_opencv/ 참고

이후 make로 빌드한다. 나의 경우에는 `-j${nproc}` 옵션 사용시에 노트북이 멈추는 현상이 발생하여 옵션 없이 빌드하였다.

```
make
```

빌드가 완료되면 설치를 진행한다. 

```
sudo make install
```

설치가 완료되면 아래 명령어를 통해 `/usr/local/lib`가 포함되어 있는지 확인한다.

```
cat /etc/ld.so.conf.d/*
```

이후 설치를 마무리하기 위해 아래 명령어를 입력한다.

```
sudo ldconfig
```

마지막으로 잘 설치되었는지 opencv 버전을 출력해 확인한다. 

```
pkg-config --modversion opencv4
```

## 2. OpenCV 테스트

OpenCV 작동을 확인하기 위해서 간단한 edge detection 프로젝트를 실행해본다. 
먼저 프로젝트 폴더를 만들어주고 아래에 `CMakeLists.txt` 파일과 `main.cpp` 파일을 생성한다.

CMakeLists.txt 파일은 아래와 같이 간단하게 openCV 경로와 링크, 빌드할 소스파일을 정의해준다.

```
cmake_minimum_required(VERSION 3.10)
project(OpenCVTest)

# OpenCV 경로 설정
find_package(OpenCV REQUIRED)

# 빌드할 실행파일 이름과 소스 파일 목록 설정
add_executable(edge_detection main.cpp)

# OpenCV 라이브러리 링크
target_link_libraries(edge_detection PRIVATE ${OpenCV_LIBS})

# C++ 표준 설정 (C++11 이상 권장)
set(CMAKE_CXX_STANDARD 11)
```

그리고 main.cpp 파일은 웹캠을 열어서 웹캠의 영상에 엣지를 추출하도록 하며, q가 입력되면 종료되도록 코드를 작성한다.

```
#include <opencv2/opencv.hpp>
#include <iostream>

int main() {
    // 웹캠 열기
    cv::VideoCapture cap(0); // 기본 웹캠
    if (!cap.isOpened()) {
        std::cout << "웹캠 열기 실패" << std::endl;
        return -1;
    }

    cv::Mat frame, gray, edges;
    while (true) {
        cap >> frame; // 웹캠에서 프레임 받아오기
        if (frame.empty()) {
            std::cout << "웹캠에서 프레임 받아오기 실패" << std::endl;
            break;
        }

        // 프레임을 그레이스케일로 변환
        cv::cvtColor(frame, gray, cv::COLOR_BGR2GRAY);

        // 엣지 추출 (Canny Edge Detection)
        cv::Canny(gray, edges, 100, 200);

        // 엣지 이미지를 화면에 표시
        cv::imshow("Edge Detection", edges);

        // 'q'를 눌러서 종료
        if (cv::waitKey(1) == 'q') {
            break;
        }
    }

    cap.release();
    cv::destroyAllWindows();
    return 0;
}
```

그리고 .vscode 폴더 아래, `tasks.json` 파일을 생성한다.

```
{
    "tasks": [
        {
            "type": "shell",
            "label": "CMake: build",
            "command": "cmake --build ${workspaceFolder}/build",
            "args": [
                "-fdiagnostics-color=always",
                "-g",
                "${file}",
                "-o",
                "${fileDirname}/${fileBasenameNoExtension}"
            ],
            "options": {
                "cwd": "${fileDirname}"
            },
            "problemMatcher": [
                "$gcc"
            ],
            "group": {
                "kind": "build",
                "isDefault": true
            },
            "detail": "Task generated by Debugger."
        }
    ],
    "version": "2.0.0"
}
```

마지막으로 opencv의 경로를 연결하기 위해, `settings.json` 파일을 열고 아래의 코드를 추가해준다.

```
{
    "C_Cpp.default.configurationProvider": "ms-vscode.cmake-tools",
    "C_Cpp.default.includePath": [
        "/usr/include/opencv4",  // OpenCV 헤더 파일 경로
        "/usr/local/include/opencv4",  // 다른 OpenCV 경로 (만약 있으면)
    ]
}
```

그리고 터미널을 열어, 아래 명령어를 순서대로 수행하여 build 폴더 아래 생성된 edge_detection을 실행한다. 

```
mkdir bulid
cd build
cmake ..
make
./edge_detection
```

## 참고

Sanghyun님의 [Ubuntu에 OpenCV 설치하기](https://sanghyunpark01.github.io/ubuntu/tips/Ubuntu_opencv/)
