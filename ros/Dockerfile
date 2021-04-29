FROM balenalib/jetson-nano-ubuntu:bionic

# Prevent apt-get prompting for input
ENV DEBIAN_FRONTEND noninteractive

# Update to 32.4.4 repository, only if base image uses an older one:
RUN sed -i 's/r32 main/r32.4 main/g' /etc/apt/sources.list.d/nvidia.list

RUN apt-get update && apt-get install -y wget tar lbzip2 \
    && \
    wget https://developer.nvidia.com/embedded/L4T/r32_Release_v4.4/r32_Release_v4.4-GMC3/T210/Tegra210_Linux_R32.4.4_aarch64.tbz2 && \
    tar xf Tegra210_Linux_R32.4.4_aarch64.tbz2 && \
    cd Linux_for_Tegra && \
    sed -i 's/config.tbz2\"/config.tbz2\" --exclude=etc\/hosts --exclude=etc\/hostname/g' apply_binaries.sh && \
    sed -i 's/install --owner=root --group=root \"${QEMU_BIN}\" \"${L4T_ROOTFS_DIR}\/usr\/bin\/\"/#install --owner=root --group=root \"${QEMU_BIN}\" \"${L4T_ROOTFS_DIR}\/usr\/bin\/\"/g' nv_tegra/nv-apply-debs.sh && \
    sed -i 's/chroot . \//  /g' nv_tegra/nv-apply-debs.sh && \
    ./apply_binaries.sh -r / --target-overlay && cd .. \
    rm -rf Tegra210_Linux_R32.4.2_aarch64.tbz2 && \
    rm -rf Linux_for_Tegra && \
    echo "/usr/lib/aarch64-linux-gnu/tegra" > /etc/ld.so.conf.d/nvidia-tegra.conf && ldconfig

# Install X and xfce
RUN \
  apt-get install -y \
  xserver-xorg-input-evdev \
  xserver-xorg-video-fbdev \
  xserver-xorg \
  xinit \
  xfce4 \
  xfce4-terminal \
  x11-xserver-utils \
  dbus-x11 \
  xterm

ENV LD_LIBRARY_PATH=/usr/lib/aarch64-linux-gnu/tegra
ENV XFCE_PANEL_MIGRATE_DEFAULT=1
ENV UDEV=1

# Prevent screen from turning off
RUN echo "#!/bin/bash" > /etc/X11/xinit/xserverrc \
  && echo "" >> /etc/X11/xinit/xserverrc \
  && echo 'exec /usr/bin/X -s 0 dpms' >> /etc/X11/xinit/xserverrc

RUN apt-get install -y locales python3 python3-pip git usbutils nano sudo

ARG ROS_PKG=ros_base
ENV ROS_DISTRO=eloquent
ENV ROS_ROOT=/opt/ros/${ROS_DISTRO}

WORKDIR /workspace

# change the locale from POSIX to UTF-8
RUN locale-gen en_US en_US.UTF-8 && update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
ENV LANG=en_US.UTF-8

# add the ROS deb repo to the apt sources list
RUN apt-get install -y --no-install-recommends git cmake build-essential curl gnupg2 lsb-release

RUN wget --no-check-certificate https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc && apt-key add ros.asc
RUN sh -c 'echo "deb [arch=$(dpkg --print-architecture)] http://packages.ros.org/ros2/ubuntu $(lsb_release -cs) main" > /etc/apt/sources.list.d/ros2-latest.list'

# install ROS packages
ENV RMW_IMPLEMENTATION=rmw_cyclonedds_cpp

RUN apt-get update && apt-get install -y --no-install-recommends \
	ros-eloquent-ros-base \
	ros-eloquent-launch-xml \
	ros-eloquent-launch-yaml \
	ros-eloquent-rviz2 \
	ros-eloquent-vision-msgs \
        ros-eloquent-image-tools \
	libpython3-dev \
	python3-colcon-common-extensions \
	python3-rosdep

RUN apt-get install -y --no-install-recommends \
      ros-eloquent-cyclonedds \
      ros-eloquent-rmw-cyclonedds-cpp

# init/update rosdep
RUN cd ${ROS_ROOT} && \
    rosdep init && \
    sudo rosdep fix-permissions && \
    rosdep update

# compile yaml-cpp-0.6, which some ROS packages may use (but is not in the 18.04 apt repo)
RUN git clone --branch yaml-cpp-0.6.0 https://github.com/jbeder/yaml-cpp yaml-cpp-0.6 && \
    cd yaml-cpp-0.6 && \
    mkdir build && \
    cd build && \
    cmake -DBUILD_SHARED_LIBS=ON .. && \
    make -j$(nproc) && \
    cp libyaml-cpp.so.0.6.0 /usr/lib/aarch64-linux-gnu/ && \
    ln -s /usr/lib/aarch64-linux-gnu/libyaml-cpp.so.0.6.0 /usr/lib/aarch64-linux-gnu/libyaml-cpp.so.0.6

WORKDIR /


# ----------------------------------------------------------
# install OpenCV (with GStreamer support)
# ----------------------------------------------------------
RUN apt-key adv --fetch-key https://repo.download.nvidia.com/jetson/jetson-ota-public.asc

RUN echo "deb https://repo.download.nvidia.com/jetson/common r32.4 main" > /etc/apt/sources.list.d/nvidia-l4t-apt-source.list && \
    apt-get update && \
    apt-get install -y --no-install-recommends \
    libopencv-python && apt-get clean


# ----------------------------------------------------------
# PyCUDA
# ----------------------------------------------------------
ENV PATH="/usr/local/cuda/bin:${PATH}"
ENV LD_LIBRARY_PATH="/usr/local/cuda/lib64:${LD_LIBRARY_PATH}"
RUN echo "$PATH" && echo "$LD_LIBRARY_PATH"

RUN apt-get update && apt-get install -y python3 python3-pip
RUN apt-get install -y libopenblas-base libopenmpi-dev cuda-toolkit-10-2 && apt-get clean
RUN apt-get update && apt-get install -y libcudnn8 python3-libnvinfer-dev cuda-compiler-10-2 && apt-get clean
RUN apt-get update && apt-get install -y -o Dpkg::Options::=--force-confdef nvidia-l4t-gstreamer libgstreamer1.0-0 gstreamer1.0-plugins-base gobject-introspection gstreamer1.0-plugins-good gstreamer1.0-plugins-bad gstreamer1.0-plugins-ugly gstreamer1.0-tools && apt-get clean 
RUN pip3 install pycuda --verbose


# ----------------------------------------------------------
# Gst Python
# ----------------------------------------------------------
RUN apt update && apt install python3-gi python3-dev python3-gst-1.0 -y


# ----------------------------------------------------------
# PyTorch Installations
# ----------------------------------------------------------
#
# install prerequisites (many of these are for numpy)
#
ENV PATH="/usr/local/cuda-10.2/bin:${PATH}"
ENV LD_LIBRARY_PATH="/usr/local/cuda-10.2/lib64:/usr/local/cuda-10.2/extras/CUPTI/lib64:${LD_LIBRARY_PATH}"

RUN apt-get update && \
    ldconfig && \
    apt-get install -y --no-install-recommends \
        python3-pip \
	python3-dev \
	libopenblas-dev \
	libopenmpi2 \
        openmpi-bin \
        openmpi-common \
	gfortran

RUN pip3 install setuptools Cython wheel
RUN pip3 install numpy --verbose


# ----------------------------------------------------------
# PyTorch (for JetPack 4.4 DP)
# ----------------------------------------------------------
#  PyTorch v1.2.0 https://nvidia.box.com/shared/static/lufbgr3xu2uha40cs9ryq1zn4kxsnogl.whl (torch-1.2.0-cp36-cp36m-linux_aarch64.whl)
#  PyTorch v1.3.0 https://nvidia.box.com/shared/static/017sci9z4a0xhtwrb4ps52frdfti9iw0.whl (torch-1.3.0-cp36-cp36m-linux_aarch64.whl)
#  PyTorch v1.4.0 https://nvidia.box.com/shared/static/c3d7vm4gcs9m728j6o5vjay2jdedqb55.whl (torch-1.4.0-cp36-cp36m-linux_aarch64.whl)
#  PyTorch v1.5.0 https://nvidia.box.com/shared/static/3ibazbiwtkl181n95n9em3wtrca7tdzp.whl (torch-1.5.0-cp36-cp36m-linux_aarch64.whl)
#

ARG PYTORCH_URL=https://nvidia.box.com/shared/static/cs3xn3td6sfgtene6jdvsxlr366m2dhq.whl
ARG PYTORCH_WHL=torch-1.7.0-cp36-cp36m-linux_aarch64.whl

RUN wget --quiet --show-progress --progress=bar:force:noscroll --no-check-certificate ${PYTORCH_URL} -O ${PYTORCH_WHL} && \
    pip3 install ${PYTORCH_WHL} --verbose && \
    rm ${PYTORCH_WHL}


# ----------------------------------------------------------
# torchvision 0.4
# ----------------------------------------------------------
ARG TORCHVISION_VERSION=v0.7.0
#ARG PILLOW_VERSION="pillow<7"
ARG TORCH_CUDA_ARCH_LIST="7.2"

RUN printenv && echo "torchvision version = $TORCHVISION_VERSION" && echo "pillow version = $PILLOW_VERSION" && echo "TORCH_CUDA_ARCH_LIST = $TORCH_CUDA_ARCH_LIST"

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    	git \
	build-essential \
	libjpeg-dev \
	zlib1g-dev

RUN git clone -b ${TORCHVISION_VERSION} https://github.com/pytorch/vision torchvision && \
    cd torchvision && \
    python3 setup.py install && \
    cd ../ && \
    rm -rf torchvision


# ----------------------------------------------------------
# torch2trt installations
# ----------------------------------------------------------
RUN git clone https://github.com/NVIDIA-AI-IOT/torch2trt && \
    cd torch2trt && \
    python3 setup.py install --plugins


# ----------------------------------------------------------
# trt_pose installation
# ----------------------------------------------------------
RUN apt-get update && \
    apt-get install -y python3-matplotlib
RUN pip3 install tqdm cython pycocotools

RUN git clone https://github.com/NVIDIA-AI-IOT/trt_pose && \
    cd trt_pose && \
    python3 setup.py install 


# ----------------------------------------------------------
# FastRTPS installation
# ----------------------------------------------------------
RUN git clone --recursive https://github.com/eProsima/Fast-RTPS.git -b 1.8.x /workspace/FastRTPS-1.8.2 && \
    cd /workspace/FastRTPS-1.8.2 && \
    mkdir build && cd build && \
    cmake -DTHIRDPARTY=ON -DSECURITY=ON .. && \
    make && \
    make install 


# ----------------------------------------------------------
# Creating ROS2 workspace and adding ROS2 packages
# ----------------------------------------------------------
RUN cd /workspace && \
    mkdir -p ros2_ws/src && \
    cd ros2_ws/src && \ 
    git clone https://github.com/NVIDIA-AI-IOT/ros2_trt_pose.git && \ 
    git clone https://github.com/NVIDIA-AI-IOT/ros2_trt_pose_hand.git  


# ----------------------------------------------------------
# Gesture Classification dependency
# ----------------------------------------------------------
RUN pip3 install -U scipy
RUN pip3 install -U scikit-learn

RUN apt-get update && apt-get install -y libqt5svg5 libffi-dev htop


# ----------------------------------------------------------
# Finish up
# ----------------------------------------------------------
RUN curl -sL https://deb.nodesource.com/setup_12.x -o nodesource_setup.sh && /bin/sh nodesource_setup.sh && apt-get update && apt-get upgrade -y nodejs && apt-get clean && rm -rf /var/lib/apt/lists/*
RUN pip3 install jupyter jupyterlab && jupyter labextension install @jupyter-widgets/jupyterlab-manager
RUN cd trt_pose/tasks/human_pose && wget --no-check-certificate 'https://docs.google.com/uc?export=download&id=1XYDdCUdiF2xxx4rznmLb62SdOUZuoNbd' -O resnet18_baseline_att_224x224_A_epoch_249.pth

# setup entrypoint 
RUN echo "source /opt/ros/eloquent/setup.bash" >> ~/.bashrc

# build ros2_trt_pose

COPY ros2_trt_pose/package.xml /workspace/ros2_ws/src/ros2_trt_pose/ros2_trt_pose/package.xml
COPY pose_msgs/package.xml /workspace/ros2_ws/src/ros2_trt_pose/pose_msgs/package.xml
COPY launch/pose-estimation.launch.py /workspace/ros2_ws/src/ros2_trt_pose/ros2_trt_pose/launch/pose-estimation.launch.py 
RUN . /opt/ros/eloquent/setup.sh && cd /workspace/ros2_ws && rosdep update --rosdistro $ROS_DISTRO && rosdep install --from-paths src --ignore-src --rosdistro eloquent -y && colcon build && . install/local_setup.sh

# Start XFCE desktop and ros2_trt_pose

COPY start.sh /start.sh
COPY pose-estimation.desktop /root/.config/autostart/pose-estimation.desktop
CMD ["/bin/sh", "start.sh"]

## Sample Gstreamer playback in webterminal:
## $ export DISPLAY=0.0
## $ gst-launch-1.0 filesrc location="file_example.mp4" ! qtdemux name=demux ! h264parse ! omxh264dec ! nvoverlaysink -e

##  Optional: Sample CUDA Clock sample run in webterminal:
##  apt-get install nvidia-l4t-cuda nvidia-cuda cuda-samples-10-2 && cd /usr/local/cuda-10.2/samples/0_Simple/clock/ && make && ./clock
##  Sample output:
##  CUDA Clock sample
##  GPU Device 0: "Maxwell" with compute capability 5.3
##  Average clocks/block = 3294.203125


