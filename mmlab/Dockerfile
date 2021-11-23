ARG PYTORCH="1.7.0"
ARG CUDA="11.0"
ARG CUDNN="8"

FROM pytorch/pytorch:${PYTORCH}-cuda${CUDA}-cudnn${CUDNN}-devel

##### 换源
RUN apt clean -y && rm -rf /var/lib/apt/lists/*
RUN mv /etc/apt/sources.list /etc/apt/sources.list.bak
COPY sourcelist /etc/apt/sources.list

##### 配置用户
# Create a no-passowrd sudo user
RUN apt update \
    && apt install -y sudo \
    && useradd -m ubuntu -s /bin/bash && adduser ubuntu sudo \
    && echo "ubuntu ALL=(ALL) NOPASSWD : ALL" | tee /etc/sudoers.d/nopasswd4sudo
# Adjust Timezone
ENV DEBIAN_FRONTEND=noninteractive
RUN apt install -y tzdata \
    && ln -fs /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

##### 安装zsh
USER ubuntu
WORKDIR /home/ubuntu
COPY --chown=ubuntu oh-my-zsh /home/ubuntu/.oh-my-zsh
RUN sudo apt install -y git zsh \
    && cp ~/.oh-my-zsh/templates/zshrc.zsh-template ~/.zshrc \
    && sed -i "s/robbyrussell/bira/" ~/.zshrc \
    && sudo usermod -s /bin/zsh ubuntu
COPY --chown=ubuntu zsh-autosuggestions /home/ubuntu/.oh-my-zsh/custom/plugins/zsh-autosuggestions
COPY --chown=ubuntu zsh-syntax-highlighting /home/ubuntu/.oh-my-zsh/custom/plugins/zsh-syntax-highlighting
COPY --chown=ubuntu zshrc /home/ubuntu/.zshrc
USER root

##### Install MMCV
ENV TORCH_CUDA_ARCH_LIST="6.0 6.1 7.0+PTX"
ENV TORCH_NVCC_FLAGS="-Xfatbin -compress-all"
ENV CMAKE_PREFIX_PATH="$(dirname $(which conda))/../"

RUN apt-get update && apt-get install -y ffmpeg libsm6 libxext6 git ninja-build libglib2.0-0 libsm6 libxrender-dev libxext6

COPY opencv_python-4.5.4.60-cp38-cp38-manylinux_2_17_x86_64.manylinux2014_x86_64.whl ./opencv_python-4.5.4.60-cp38-cp38-manylinux_2_17_x86_64.manylinux2014_x86_64.whl
RUN pip install opencv_python-4.5.4.60-cp38-cp38-manylinux_2_17_x86_64.manylinux2014_x86_64.whl
RUN rm opencv_python-4.5.4.60-cp38-cp38-manylinux_2_17_x86_64.manylinux2014_x86_64.whl
COPY matplotlib-3.5.0-cp38-cp38-manylinux_2_5_x86_64.manylinux1_x86_64.whl ./matplotlib-3.5.0-cp38-cp38-manylinux_2_5_x86_64.manylinux1_x86_64.whl
RUN pip install matplotlib-3.5.0-cp38-cp38-manylinux_2_5_x86_64.manylinux1_x86_64.whl
RUN rm matplotlib-3.5.0-cp38-cp38-manylinux_2_5_x86_64.manylinux1_x86_64.whl

RUN pip install mmcv-full==1.3.17 -f https://download.openmmlab.com/mmcv/dist/cu110/torch1.7.0/index.html

##### Install MMDetection
WORKDIR /home/ubuntu
RUN conda clean --all
# RUN git clone https://github.com/open-mmlab/mmdetection.git /mmdetection
COPY --chown=ubuntu mmdetection ./mmdetection
WORKDIR ./mmdetection
ENV FORCE_CUDA="1"
RUN pip install -r requirements/build.txt
RUN pip install --no-cache-dir -e .

##### Install MMClassification
WORKDIR /home/ubuntu
RUN conda clean --all
# RUN git clone https://github.com/open-mmlab/mmclassification.git
COPY --chown=ubuntu mmclassification ./mmclassification
WORKDIR ./mmclassification
RUN pip install --no-cache-dir -e .

# 切换路径
USER ubuntu
# Clean apt-cache
RUN sudo apt autoremove -y \
    && sudo apt clean -y \
    && sudo rm -rf /var/lib/apt/lists/*

WORKDIR /home/ubuntu
ENTRYPOINT ["/bin/zsh"]

