FROM osrf/ros:noetic-desktop-full

# 컨테이너 재생성해도 살아남아야 하는 것들 (apt로 손수 깔면 up -d 때 날아감)
RUN apt-get update && apt-get install -y --no-install-recommends \
        terminator \
    && rm -rf /var/lib/apt/lists/*
