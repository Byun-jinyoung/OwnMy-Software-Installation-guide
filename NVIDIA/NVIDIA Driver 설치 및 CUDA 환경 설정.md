nvidia gpu을 사용한 AMBER18, AMBER20, GROMACS을 이용한 MD simuation을 수행할 때, 우분투(20.04 LTS)환경에서 CUDA, CuDNN, NVIDIA-Driver 설치하여 MD simulation 환경을 셋팅을 하겠다. 
이번에 컴퓨터 포멧 및 그래픽 카드 교체로 인해서 다시 환경을 셋팅을 해야할 필요가 있어고 rtx 3090의 경우 

고준수 박사님께서 작성하신 'Compute Node Setup Manual(Ubuntu 18.04).pdf' 문서와 https://ropiens.tistory.com/34 을 참고하였다.

# 1. 필수 라이브러리 설치

    $ sudo apt install -y libglvnd-dev

# 2. 이전 라이브러리 제거(optional)
기존에 설치되어 있는 cuda 관련 라이브러리들을 제거 
    
    $ sudo apt purge nvidia*
    $ sudo apt purge cuda*
    $ sudo apt purge libcudnn*
    $ sudo apt purge nccl-*
    $ sudo apt autoremove
    $ sudo apt autoclean
    $ sudo rm -rf /usr/local/cuda *

# 3. CUDA 라이브러리 설치
현재 사용하는 그래픽 카드와 NVIDIA Driver, CUDA 사이에 호환성이 존재하며, 호환성을 벗어난다고 해서 Driver나 CUDA가 제대로 작동하지는 않는 것 같지만 그래도 NVIDIA에서 제시하는 최소한의 호환성을 지키면서 설치하고자 했다.(제대로 지키긴 했나?) 사실 CUDA-10.1까지 지원하는 AMBER18 때문에 CUDA-10.1을 설치하고 싶었으나, 장착된 그래픽 카드가 rtx 3090이다보니 참고한대로 설치를 진행해도 CUDA-11.2가 설치되어버렸다. 왜 그런지 이유는 모르겠다... 그리고 심지어 CUDA-11.2도 최신 버전이긴 하지만 왜 인지 설치 중간에 제대로 진행되지 않아서 다시 설치를 진행했다. 
그리고 더 알아보니 rtx 3090부터는 CUDA-11.0 이상의 버전을 설치해야한다고 한다. 따라서 AMBER18은 포기하였다. 
 
## 그래픽 카드 정보 확인
 
    $ ubuntu-drivers devices
    == /sys/devices/pci0000:00/0000:00:03.1/0000:26:00.0 ==
    modalias : pci:v000010DEd00002204sv000010DEsd00001454bc03sc00i00
    vendor   : NVIDIA Corporation
    driver   : nvidia-driver-455 - third-party free
    driver   : nvidia-driver-460 - third-party free recommended
    driver   : xserver-xorg-video-nouveau - distro free builtin

추천 드라이버는 nvidia-driver-460이지만 너무 높은 버전으로 설치하면 또 CUDA-10.x 버전이 설치되지 않을까봐 nvidia-driver-455로 설치를 진행하기로 결정하였다. 
몇몇의 블로그에서는 수동 설치를 권하고 있지 않지만 고준수 박사님 manual에서도 수동 설치를 하기 때문에 나는 수동 설치을 하기로 진행했다.(자동 설치의 경우 https://pstudio411.tistory.com/entry/Ubuntu-2004-Nvidia%EB%93%9C%EB%9D%BC%EC%9D%B4%EB%B2%84-%EC%84%A4%EC%B9%98%ED%95%98%EA%B8%B0, https://linuxize.com/post/how-to-nvidia-drivers-on-ubuntu-20-04/ 에서 설치 방법들을 설명해주고 있다.)
NVIDIA 홈페이지에 www.nvidia.com/drivers 에서 원하는 버전의 run file을 다운로드 받는다.

    $ mv ~/Download/NVIDIA-Linux-x86_64-455.23.04.run ~/ENVs/NVIDIA
    $ chmod 755 NVIDIA-Linux-x86_64-455.23.04.run
    $ sudo ./NVIDIA-Linux-x86_64-455.23.04.run
위와 같이 진행했을 때, 문제가 없으면 좋지만 나는 기존에 있던 CUDA가 제대로 제거가 되지 않았는지 무슨 kernel얘기하면서 오류가 발생하여 자동 설치 방법으로 진행하였다.


##  PPA 저장소를 사용하여 그래픽 드라이버 설치
기존에 있던 Nvidia DRIVER 때문에 설치 과정 중 문제가 생길 수 있으므로 아예 새로 지우고 클린 설치를 진행했다. 
    
    $ dpkg -l | grep -i nvidia
    $ sudo apt-get remove --purge nvidia-*
    $ sudo apt-get autoremove
    $ sudo apt-get autoclean
    $ sudo apt-get update
    $ sudo add-apt-repository ppa:graphics-drivers/ppa
    $ sudo apt update
    $ sudo apt install nvidia-driver-455
    $ sudo apt update
    $ sudo apt upgrade
    $ sudo reboot
    

## CUDA 라이브러리 설치
CUDA Toolkit Archive에서 들어가서 원하는 버전의 CUDA Toolkit 버전을 선택하여 다운로드 받는다.
    
    $ mkdir ENVs/ && cd ENVs
    $ mkdir NVIDIA
    $ wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/cuda-ubuntu2004.pin
    $ sudo mv cuda-ubuntu2004.pin /etc/apt/preferences.d/cuda-repository-pin-600
    $ wget https://developer.download.nvidia.com/compute/cuda/11.0.3/local_installers/cuda-repo-ubuntu2004-11-0-local_11.0.3-450.51.06-1_amd64.deb
    $ sudo dpkg -i cuda-repo-ubuntu2004-11-1-local_11.1.1-455.32.00-1_amd64.deb
    $ sudo apt-key add /var/cuda-repo-ubuntu2004-11-1-local/7fa2af80.pub
    $ sudo apt update
    $ sudo apt install -y cuda 
위와 같이 deb(local) 방식으로 진행하면 멋대로 CUDA-11.2을 설치해버리고 문제가 생긴다. runfile 속편히 설치하자
    

## 환경 설정 및 적용
아래 내용의 bash script을 작성하여 수행
```
#!/bin/bash

sudo cat <<_EOF > /etc/profile.d/cuda.sh
export PATH=/usr/local/cuda/bin:${PATH}
export LD_LIBRARY_PATH=/usr/local/cuda/lib64:${LD_LIBRARY_PATH}
export CUDA_HOME=/usr/local/cuda
_EOF

source /etc/profile.d/cuda.sh
```

    $ chmod 755 config_cuda.sh
    $ sudo ./config_cuda.sh

 설치 확인
 
    $ nvcc --version
    $ nvidia-smi
    $ sudo reboot




