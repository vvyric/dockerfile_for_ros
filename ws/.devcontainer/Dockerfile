##### Base Stage #####

FROM nvidia/cuda:11.8.0-cudnn8-devel-ubuntu20.04 as base

LABEL MAINTAINER="runcong.wang@tum.de"

SHELL [ "/bin/bash", "-c" ]

# setup environment
ENV LANG=en_US.UTF-8
ENV LC_NUMERIC=en_US.UTF-8
ENV LC_ALL=C
ENV DEBIAN_FRONTEND=noninteractive

# setup timezone
RUN echo 'Etc/UTC' > /etc/timezone && \
  ln -s -f /usr/share/zoneinfo/Etc/UTC /etc/localtime && \
  apt-get update && \
  apt-get install -q -y --no-install-recommends tzdata && \
  rm -rf /var/lib/apt/lists/*

# USE German mirror to speed up things
RUN cp /etc/apt/sources.list /etc/apt/sources.list.old \
  && sed -i -e 's/http:\/\/archive\.ubuntu\.com\/ubuntu\// \
  http:\/\/de.archive\.ubuntu\.com\/ubuntu/' /etc/apt/sources.list
RUN apt-key update && apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 4B63CF8FDE49746E98FA01DDAD19BAB3CBF125EA

# install packages
RUN apt-get update && apt-get install -q -y --no-install-recommends \
  terminator \
  vim \
  git \
  bash-completion \ 
  software-properties-common \
  ca-certificates \
  curl \
  wget \
  ssh \
  dirmngr \
  gnupg2 \
  lsb-release \
  python3-pip \
  && rm -rf /var/lib/apt/lists/*

# Fix ssl certificate issue
RUN apt-get update
RUN apt-get install ca-cacert \
  && update-ca-certificates \
  && rm -rf /var/lib/apt/lists/*


##### ROS Stage #####

FROM base as ros

# install ros
RUN sh -c 'echo "deb [arch=amd64] http://packages.ros.org/ros/ubuntu focal main" > /etc/apt/sources.list.d/ros-latest.list'
RUN curl -s https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc | apt-key add -
RUN apt-get update
RUN apt-get install -y ros-noetic-desktop-full
RUN echo "source /opt/ros/noetic/setup.bash" >> ~/.bashrc
RUN source ~/.bashrc

RUN apt-get update && apt-get install --no-install-recommends -y \
  python3-rosdep \
  python3-rosinstall \
  python3-rosinstall-generator \
  python3-wstool \
  python3-vcstools \
  python3-catkin-tools\
  && rm -rf /var/lib/apt/lists/*

# bootstrap rosdep
RUN rosdep init \
  && rosdep update

ENV ROS_PACKAGE_PATH=/opt/ros/noetic/share

##### User Stage #####

FROM ros as user

ARG USERNAME=developer
ARG USER_UID=1000
ARG USER_GID=$USER_UID

RUN groupadd --gid $USER_GID $USERNAME \
  && useradd --uid $USER_UID --gid $USER_GID -m $USERNAME \
  && apt-get update \
  && apt-get install -q -y sudo \
  && rm -rf /var/lib/apt/lists/* \
  && echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME \
  && chmod 0440 /etc/sudoers.d/$USERNAME

USER $USERNAME

CMD ["bash"]

RUN echo "source /opt/ros/noetic/setup.bash" >> ~/.bashrc

RUN sudo mkdir -p /home/$USERNAME/Downloads/openssl/ssl \
  && sudo cp -r /etc/ssl/* /home/$USERNAME/Downloads/openssl/ssl/ \
  && sudo rm /etc/ros/rosdep/sources.list.d/20-default.list \
  && sudo rosdep init \
  && rosdep update

##### Runtime Stage #####

FROM user as runtime

ENV NVIDIA_VISIBLE_DEVICES \
  ${NVIDIA_VISIBLE_DEVICES:-all}

ENV NVIDIA_DRIVER_CAPABILITIES compute,graphics,utility,video,display

# detailed fix for the ssl certificate issue
RUN sudo mkdir -p /home/$USERNAME/Downloads/openssl/ssl \
  && sudo cp -r /etc/ssl/* /home/$USERNAME/Downloads/openssl/ssl/ \
  && sudo rm /etc/ros/rosdep/sources.list.d/20-default.list \
  && sudo rosdep init \
  && rosdep update


##### Dependencies Stage #####

# FROM runtime as dev_containers_target_stage
FROM runtime as dep
RUN sudo apt update
# WORKDIR /workspaces/tiago_ws/

# # Install Tiago packages
# RUN sudo wget https://raw.githubusercontent.com/pal-robotics/tiago_tutorials/noetic-devel/tiago_public-noetic.rosinstall
# RUN rosinstall /workspaces/tiago_ws/src /opt/ros/noetic /workspaces/tiago_ws/tiago_public-noetic.rosinstall
# RUN echo 'source /workspaces/tiago/tiago_ws/devel/setup.bash' >> ~/.bashrc



RUN echo 'export ROS_HOME=~/.ros' >> ~/.bashrc
RUN source /home/$USERNAME/.bashrc
RUN echo "alias sss='source devel/setup.bash'" >> ~/.bashrc

RUN sudo apt install -qqy lsof \
  net-tools\
  lshw\
  iputils-ping\
  python-is-python3


RUN sudo apt install -qqy ros-noetic-moveit
RUN pip install --upgrade pip numpy protobuf
RUN pip install networkx==3.1
RUN pip install torch==2.1.1 torchvision==0.16.1 torchaudio==2.1.1 --index-url https://download.pytorch.org/whl/cu118
RUN pip install opencv-python pycocotools matplotlib onnxruntime onnx
RUN pip install ftfy regex tqdm tensorflow tensorboard

RUN pip install opencv-python smash

# jax jaxlib NOTE: YOU HAVE TO INSTALL MANUALLY AFTER THE CONTAINER IS BUILT
RUN pip install --upgrade "jax[cuda11_cudnn86]" -f https://storage.googleapis.com/jax-releases/jax_cuda_releases.html

# WORKDIR /workspaces/LSHM

RUN pip install numpy \
                scipy\
                matplotlib\
                scikit-image\
                scikit-learn\
                ipython\
                pandas\
                h5py\
                seaborn\
                tensorflow-datasets\
                ipywidgets

RUN sudo apt install -y ros-noetic-base-local-planner
# RUN rosdep update
## RUN rosdep install --from-paths /workspaces/tiago_ws/src --ignore-src --rosdistro noetic -y
# RUN rosdep install --from-paths /workspaces/tiago_ws/src --ignore-src -r -y

