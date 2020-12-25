nvidia gpu을 사용한 AMBER18, AMBER20, GROMACS을 이용한 MD simuation을 수행할 때, 우분투(20.04 LTS)환경에서 CUDA, CuDNN, NVIDIA-Driver 설치하여 MD simulation 환경을 셋팅을 하겠다. 
이번에 컴퓨터 포멧 및 그래픽 카드 교체로 인해서 다시 환경을 셋팅을 해야할 필요가 있어고 rtx 3090의 경우 

고준수 박사님께서 작성하신 'Compute Node Setup Manual(Ubuntu 18.04).pdf' 문서와 https://ropiens.tistory.com/34 을 참고하였다.

1. 필수 라이브러리 설치
    $ sudo apt install -y libglvnd-dev

2. 이전 라이브러리 제거(optional)
기존에 설치되어 있는 cuda 관련 라이브러리들을 제거 
    $ sudo apt purge nvidia*
    $ sudo apt purge cuda*
    $ sudo apt purge libcudnn*
    $ sudo apt purge nccl-*
    $ sudo apt autoremove
    $ sudo apt autoclean
    $ sudo rm -rf /usr/local/cuda *

3. CUDA 라이브러리 설치
사실 AMBER18 때문에 CUDA-10.1을 설치하고 싶었으나, 장착된 그래픽 카드가 rtx 3090이다보니 참고한대로 설치를 진행해도 CUDA-11.2가 설치되어버렸다...
NVIDIA 홈페이지에 접속해서 (접속 경로 적기) 'cuda-repo-ubuntu1804-10-1-local-10.1.105-418.39_1.0-1_amd64.deb'을 다운로드 받는다.
    $ cd
    $ mkdir ENVs/ && cd ENVs
    $ mkdir NVIDIA
    $ sudo dpkg -i cuda-repo-ubuntu1804-10-1-local-10.1.105-418.39_1.0-1_amd64.deb4
    $ sudo apt-key add /var/cuda-repo-10-1-local-10.1.243-418.87.00/7fa2af89.pub
    $ sudo apt update
    $ sudo apt install -y cuda (약 5~10 정도 소요)

 환경 설정 및 적용
아래 내용의 bash script을 작성하여 수행
#!/bin/bash

sudo cat <<_EOF > /etc/profile.d/cuda.sh
export PATH=/usr/local/cuda/bin:${PATH}
export LD_LIBRARY_PATH=/usr/local/cuda/lib64:${LD_LIBRARY_PATH}
export CUDA_HOME=/usr/local/cuda
_EOF

source /etc/profile.d/cuda.sh

$ chmod 755 config_cuda.sh
$ sudo ./config_cuda.sh

 설치 확인
$ nvcc --version
$ nvidia-smi





