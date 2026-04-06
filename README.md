# docker_ws

Ubuntu 22.04 호스트에서 다양한 Docker 환경을 관리하는 공용 작업 공간.

## 폴더 구조

```
~/docker_ws/
├── compose/                # docker-compose 파일 모음
│   ├── ros1.yml            # ROS1 Noetic 환경
│   └── ros2.yml            # ROS2 Humble 환경
├── ubuntu20_ros1/          # ROS1 Noetic 설정
│   └── .bashrc_ros1        # ROS1 전용 bashrc (컨테이너 내부: /root/.bashrc)
├── ubuntu22_ros2/          # ROS2 Humble 설정
│   └── .bashrc_ros2        # ROS2 전용 bashrc (컨테이너 내부: /root/.bashrc)
├── shared_workspace/       # 환경 공통 작업 폴더 (컨테이너 내부: /workspace)
├── git/                    # git 프로젝트 폴더 (컨테이너 내부: /git)
└── README.md
```

---

## 최초 설치 (한 번만)

### 1. Docker 그룹에 유저 추가

```bash
sudo usermod -aG docker $USER
newgrp docker  # 재로그인 없이 즉시 적용 (현재 터미널만)
```

> 완전 영구 적용은 로그아웃 후 재로그인

### 2. 폴더 생성

```bash
mkdir -p ~/docker_ws/compose
mkdir -p ~/docker_ws/ubuntu20_ros1
mkdir -p ~/docker_ws/ubuntu22_ros2
mkdir -p ~/docker_ws/shared_workspace
mkdir -p ~/docker_ws/git
```

---

## ubuntu20_ros1 - ROS1 Noetic

### compose/ros1.yml 설명

```yaml
services:
  ros1:
    image: osrf/ros:noetic-desktop-full   # 공식 ROS Noetic 이미지 (Ubuntu 20.04 기반)
    container_name: ros1_container
    stdin_open: true   # 터미널 입력 허용 (-i)
    tty: true          # 터미널 출력 허용 (-t)
    network_mode: host # ROS 노드 간 통신에 필요
    hostname: ros1-noetic  # 프롬프트 표시 이름 (root@ros1-noetic)
    privileged: true   # USB, 시리얼포트 등 하드웨어 접근
    environment:
      - DISPLAY=$DISPLAY              # 호스트 화면에 GUI 출력
      - ROS_MASTER_URI=http://localhost:11311
    volumes:
      - /tmp/.X11-unix:/tmp/.X11-unix                        # GUI 소켓 마운트
      - /home/jisang/docker_ws/shared_workspace:/workspace   # 공용 작업 폴더
      - /home/jisang/docker_ws/git:/git                      # git 프로젝트 폴더
      - /home/jisang/docker_ws/ubuntu20_ros1/.bashrc_ros1:/root/.bashrc  # 전용 bashrc
    working_dir: /
    command: bash
```

### 마운트 구조

```
호스트                                         컨테이너
~/docker_ws/shared_workspace/          →   /workspace/
~/docker_ws/git/                       →   /git/
~/docker_ws/ubuntu20_ros1/.bashrc_ros1 →   /root/.bashrc
```

### .bashrc_ros1 주요 내용

```bash
source /opt/ros/noetic/setup.bash

export ROS_MASTER_URI=http://localhost:11311
export ROS_HOSTNAME=localhost

alias cb='catkin_make'
alias cbr='catkin_make -DCMAKE_BUILD_TYPE=Release'
alias sd='source devel/setup.bash'
alias sr='source ~/.bashrc'
```

### alias 설명

| alias | 명령 | 설명 |
|-------|------|------|
| `cb`  | `catkin_make` | 현재 워크스페이스에서 빌드 |
| `cbr` | `catkin_make -DCMAKE_BUILD_TYPE=Release` | Release 모드 빌드 |
| `sd`  | `source devel/setup.bash` | 현재 워크스페이스 devel source |
| `sr`  | `source ~/.bashrc` | bashrc 재로드 |

### 사용법

#### 처음 실행 (이미지 다운로드 + 컨테이너 생성, 약 3GB)

```bash
xhost +local:docker   # GUI 허용 (로그인 후 한 번)

cd ~/docker_ws/compose
docker compose -f ros1.yml up -d
```

#### 매일 사용

```bash
# 컨테이너 시작
docker compose -f ~/docker_ws/compose/ros1.yml start

# 컨테이너 안으로 터미널 진입
docker exec -it ros1_container bash

# 작업 후 나오기 (컨테이너는 계속 실행 중)
exit

# 컨테이너 정지
docker compose -f ~/docker_ws/compose/ros1.yml stop
```

#### 설정 변경 후 재적용

```bash
docker compose -f ~/docker_ws/compose/ros1.yml down
docker compose -f ~/docker_ws/compose/ros1.yml up -d
```

---

## ubuntu22_ros2 - ROS2 Humble

### compose/ros2.yml 설명

```yaml
services:
  ros2:
    image: osrf/ros:humble-desktop-full   # 공식 ROS2 Humble 이미지 (Ubuntu 22.04 기반)
    container_name: ros2_container
    stdin_open: true   # 터미널 입력 허용 (-i)
    tty: true          # 터미널 출력 허용 (-t)
    network_mode: host # DDS 통신에 필요
    hostname: ros2-humble  # 프롬프트 표시 이름 (root@ros2-humble)
    privileged: true   # USB, 시리얼포트 등 하드웨어 접근
    environment:
      - DISPLAY=$DISPLAY   # 호스트 화면에 GUI 출력
    volumes:
      - /tmp/.X11-unix:/tmp/.X11-unix                        # GUI 소켓 마운트
      - /home/jisang/docker_ws/shared_workspace:/workspace   # 공용 작업 폴더
      - /home/jisang/docker_ws/git:/git                      # git 프로젝트 폴더
      - /home/jisang/docker_ws/ubuntu22_ros2/.bashrc_ros2:/root/.bashrc  # 전용 bashrc
    working_dir: /
    command: bash
```

> ROS2는 ROS_MASTER_URI가 없음. DDS 기반 통신이므로 `ROS_DOMAIN_ID`로 네트워크 격리.

### 마운트 구조

```
호스트                                         컨테이너
~/docker_ws/shared_workspace/          →   /workspace/
~/docker_ws/git/                       →   /git/
~/docker_ws/ubuntu22_ros2/.bashrc_ros2 →   /root/.bashrc
```

### .bashrc_ros2 주요 내용

```bash
source /opt/ros/humble/setup.bash

export ROS_DOMAIN_ID=0   # 같은 네트워크에서 다른 ROS2 환경과 격리할 때 변경

# colcon alias
alias cb='colcon build'
alias cbr='colcon build --cmake-args -DCMAKE_BUILD_TYPE=Release'
alias sd='source install/setup.bash'
```

### alias 설명

| alias | 명령 | 설명 |
|-------|------|------|
| `cb`  | `colcon build` | 현재 워크스페이스에서 빌드 |
| `cbr` | `colcon build --cmake-args -DCMAKE_BUILD_TYPE=Release` | Release 모드 빌드 |
| `sd`  | `source install/setup.bash` | 현재 워크스페이스 install source |
| `sr`  | `source ~/.bashrc` | bashrc 재로드 |

### 사용법

#### 처음 실행 (이미지 다운로드 + 컨테이너 생성, 약 3GB)

```bash
xhost +local:docker   # GUI 허용 (로그인 후 한 번)

cd ~/docker_ws/compose
docker compose -f ros2.yml up -d
```

#### 매일 사용

```bash
# 컨테이너 시작
docker compose -f ~/docker_ws/compose/ros2.yml start

# 컨테이너 안으로 터미널 진입
docker exec -it ros2_container bash

# 작업 후 나오기 (컨테이너는 계속 실행 중)
exit

# 컨테이너 정지
docker compose -f ~/docker_ws/compose/ros2.yml stop
```

#### 설정 변경 후 재적용

```bash
docker compose -f ~/docker_ws/compose/ros2.yml down
docker compose -f ~/docker_ws/compose/ros2.yml up -d
```

#### colcon workspace 빌드 예시

```bash
# 컨테이너 진입 후 — 워크스페이스 이름은 자유롭게
mkdir -p /my_ws/src
cd /my_ws
cb                         # colcon build
sd                         # source install/setup.bash
```

---

## 공통 Docker 명령어

```bash
docker ps          # 실행 중인 컨테이너 목록
docker ps -a       # 전체 컨테이너 목록 (정지 포함)
docker images      # 다운받은 이미지 목록
docker logs ros1_container   # 컨테이너 로그 확인
docker logs ros2_container   # 컨테이너 로그 확인
```

---

## 프로젝트별 환경변수 추가 예시

프로젝트 전용 라이브러리나 경로가 필요한 경우 각 `.bashrc`에 직접 추가.

```bash
# 예시: acados 라이브러리 사용 시
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:"/git/my_project/lib/acados/lib"
export ACADOS_SOURCE_DIR="/git/my_project/lib/acados"
```

---

## 주의사항

- 컨테이너를 `down` 해도 마운트된 폴더 안의 파일은 호스트에 유지됨
- 컨테이너 내부에서 마운트 폴더 외부에 저장한 파일은 `down` 시 삭제됨
- `stop` / `start` 는 컨테이너 상태를 보존 (내부 변경사항 유지)
- `down` / `up` 은 컨테이너를 재생성 (내부 변경사항 삭제)
- ROS1과 ROS2 컨테이너는 동시에 실행 가능 (network_mode: host이므로 ROS_DOMAIN_ID로 격리)
