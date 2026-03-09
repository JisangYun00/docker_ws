# docker_ws

Ubuntu 22.04 호스트에서 다양한 Docker 환경을 관리하는 공용 작업 공간.

## 폴더 구조

```
~/docker_ws/
├── compose/                # docker-compose 파일 모음
│   ├── ros1.yml            # ROS1 Noetic 환경
│   └── ubuntu24.yml        # Ubuntu 24.04 환경 (추후)
├── ubuntu20_ros1/          # ROS1 Noetic 작업 공간
│   └── .bashrc_ros1        # ROS 전용 bashrc (컨테이너 내부: /root/.bashrc)
├── ubuntu24/               # Ubuntu 24.04 작업 공간 (추후)
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
mkdir -p ~/docker_ws/ubuntu24
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
```

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

## 공통 Docker 명령어

```bash
docker ps          # 실행 중인 컨테이너 목록
docker ps -a       # 전체 컨테이너 목록 (정지 포함)
docker images      # 다운받은 이미지 목록
docker logs ros1_container   # 컨테이너 로그 확인
```

---

## 주의사항

- 컨테이너를 `down` 해도 마운트된 폴더 안의 파일은 호스트에 유지됨
- 컨테이너 내부에서 마운트 폴더 외부에 저장한 파일은 `down` 시 삭제됨
- `stop` / `start` 는 컨테이너 상태를 보존 (내부 변경사항 유지)
- `down` / `up` 은 컨테이너를 재생성 (내부 변경사항 삭제)
