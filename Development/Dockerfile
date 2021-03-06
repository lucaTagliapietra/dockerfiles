FROM diegoferigo/tools
MAINTAINER Diego Ferigo <dgferigo@gmail.com>

# Install ROS Desktop Full
# ========================

# https://github.com/osrf/docker_images/blob/master/ros/
ENV ROS_DISTRO lunar
RUN apt-key adv --keyserver ha.pool.sks-keyservers.net \
        --recv-keys 421C365BD9FF1F717815A3895523BAEEB01FA116
RUN echo "deb http://packages.ros.org/ros/ubuntu xenial main" \
        > /etc/apt/sources.list.d/ros-latest.list
RUN apt-get update &&\
    apt-get install --no-install-recommends -y \
        python-rosdep \
        python-rosinstall \
        python-vcstools &&\
    rm -rf /var/lib/apt/lists/*
RUN rosdep init &&\
    rosdep update
RUN apt-get update &&\
    apt-get install -y \
        ros-${ROS_DISTRO}-desktop-full &&\
        #ros-${ROS_DISTRO}-fake-localization \
        #ros-${ROS_DISTRO}-map-server &&\
    rm -rf /var/lib/apt/lists/*

# Install libraries
# =================

RUN apt-get update &&\
    apt-get install -y \
        libeigen3-dev \
        libgsl-dev \
        libedit-dev \
        libace-dev \
        coinor-libipopt-dev \
        liboctave-dev \
        &&\
    rm -rf /var/lib/apt/lists/*

# Concurrent compilation jobs
ENV GCC_JOBS=6

# Install SWIG with Matlab support
# ... waiting its upstream merge
RUN apt-get update &&\
    apt-get install -y \
        autotools-dev \
        automake \
        bison \
        &&\
    rm -rf /var/lib/apt/lists/*
RUN git clone https://github.com/jaeandersson/swig.git &&\
    cd swig &&\
    git checkout matlab &&\
    sh autogen.sh &&\
    ./configure &&\
    make -j${GCC_JOBS} &&\
    make install &&\
    cd - &&\
    rm -r swig

# Install YARP, iCub and friends from sources
# ===========================================

ENV IIT_DIR=/iit
ENV IIT_INSTALL=${IIT_DIR}/local
ENV IIT_SOURCES=${IIT_DIR}/sources
ARG IIT_BIN=${IIT_INSTALL}/bin
ENV IIT_PATH=${IIT_PATH:+${IIT_PATH}:}${IIT_BIN}
ENV LD_LIBRARY_PATH=${LD_LIBRARY_PATH:+${LD_LIBRARY_PATH}:}${IIT_INSTALL}/lib/
ENV PATH=${IIT_PATH}:${PATH}

RUN mkdir -p ${IIT_SOURCES} ${IIT_BIN}

# Use docker cache for steps above
ARG IIT_DOCKER_SOURCES="20170703"

# Build Variables
ARG SOURCES_GIT_BRANCH=devel
ARG SOURCES_BUILD_TYPE=Debug

# Download all sources with git
# -----------------------------

RUN cd ${IIT_SOURCES} &&\
    git clone https://github.com/robotology/yarp.git &&\
    git clone https://github.com/robotology/icub-main.git &&\
    git clone https://github.com/robotology/icub-contrib-common.git &&\
    git clone https://github.com/robotology/robot-testing.git &&\
    git clone https://github.com/robotology/ycm.git &&\
    git clone https://github.com/robotology/gazebo-yarp-plugins.git &&\
    git clone https://github.com/robotology/codyco-superbuild.git &&\
    git clone https://github.com/robotology-playground/yarp-matlab-bindings.git &&\
    git clone https://github.com/robotology/idyntree.git

# Build all sources
# -----------------

RUN cd ${IIT_SOURCES}/yarp &&\
    git checkout ${SOURCES_GIT_BRANCH} &&\
    mkdir build && cd build &&\
    cmake -DCMAKE_BUILD_TYPE=${SOURCES_BUILD_TYPE} \
          -DCMAKE_INSTALL_PREFIX=${IIT_INSTALL} \
          -DCREATE_GUIS=ON \
          -DCREATE_LIB_MATH=ON \
          .. &&\
    make -j ${GCC_JOBS} install &&\
    ln -s ${IIT_SOURCES}/yarp/scripts/yarp_completion \
          /etc/bash_completion.d/yarp_completion
ENV YARP_DIR=${IIT_INSTALL}
ENV YARP_DATA_DIRS=${IIT_INSTALL}/share/yarp
ENV YARP_COLORED_OUTPUT=1
RUN yarp check
EXPOSE 10000/tcp

RUN cd ${IIT_SOURCES}/icub-main &&\
    git checkout ${SOURCES_GIT_BRANCH} &&\
    mkdir build && cd build &&\
    cmake -DCMAKE_BUILD_TYPE=${SOURCES_BUILD_TYPE} \
          -DCMAKE_INSTALL_PREFIX=${IIT_INSTALL} \
          -DENABLE_icubmod_cartesiancontrollerserver=ON \
          -DENABLE_icubmod_cartesiancontrollerclient=ON \
          -DENABLE_icubmod_gazecontrollerclient=ON \
          .. &&\
    make -j ${GCC_JOBS} install
ENV YARP_DATA_DIRS=${YARP_DATA_DIRS:+${YARP_DATA_DIRS}:}${IIT_INSTALL}/share/iCub

RUN cd ${IIT_SOURCES}/icub-contrib-common &&\
    mkdir build && cd build &&\
    cmake -DCMAKE_INSTALL_PREFIX=${IIT_INSTALL} \
          .. &&\
    make -j ${GCC_JOBS} install
ENV YARP_DATA_DIRS=${YARP_DATA_DIRS:+${YARP_DATA_DIRS}:}${IIT_INSTALL}/share/ICUBcontrib

RUN cd ${IIT_SOURCES}/robot-testing &&\
    git checkout ${SOURCES_GIT_BRANCH} &&\
    mkdir build && cd build &&\
    cmake -DCMAKE_BUILD_TYPE=${SOURCES_BUILD_TYPE} \
          -DCMAKE_INSTALL_PREFIX=${IIT_INSTALL} \
          -DENABLE_MIDDLEWARE_PLUGINS=ON \
          .. &&\
    make -j ${GCC_JOBS} install

RUN cd ${IIT_SOURCES}/ycm &&\
    git checkout ${SOURCES_GIT_BRANCH} &&\
    mkdir build && cd build &&\
    cmake -DCMAKE_INSTALL_PREFIX=${IIT_INSTALL} \
          .. &&\
    make -j ${GCC_JOBS} install

RUN cd ${IIT_SOURCES}/gazebo-yarp-plugins &&\
    git checkout ${SOURCES_GIT_BRANCH} &&\
    mkdir build && cd build &&\
    cmake -DCMAKE_BUILD_TYPE=${SOURCES_BUILD_TYPE} \
          -DCMAKE_INSTALL_PREFIX=${IIT_INSTALL} \
          .. &&\
    make -j ${GCC_JOBS} install
ENV YARP_DATA_DIRS=${YARP_DATA_DIRS:+${YARP_DATA_DIRS}:}${IIT_INSTALL}/share/ICUBcontrib
ENV GAZEBO_PLUGIN_PATH=${GAZEBO_PLUGIN_PATH:+${GAZEBO_PLUGIN_PATH}:}${IIT_INSTALL}/lib

RUN cd ${IIT_SOURCES}/yarp-matlab-bindings &&\
    git checkout ${SOURCES_GIT_BRANCH} &&\
    mkdir build && cd build &&\
    cmake -DCMAKE_BUILD_TYPE=${SOURCES_BUILD_TYPE} \
          -DCMAKE_INSTALL_PREFIX=${IIT_INSTALL} \
          -DYARP_USES_OCTAVE:BOOL=ON \
          -DYARP_GENERATE_MATLAB=ON \
          -DYARP_SOURCE_DIR=${IIT_SOURCES}/yarp \
          -DYARP_NO_DEPRECATED_WARNINGS:BOOL=ON \
          -DYCM_USE_DEPRECATED:BOOL=FALSE \
          -DYARP_INSTALL_MATLAB_LIBDIR=matlab \
          -DYARP_INSTALL_MATLAB_MFILESDIR=matlab \
          .. &&\
    make -j ${GCC_JOBS} install

RUN apt-get update &&\
    apt-get install -y \
        liborocos-kdl-dev \
        &&\
    rm -rf /var/lib/apt/lists/*

RUN cd ${IIT_SOURCES}/idyntree &&\
    git checkout ${SOURCES_GIT_BRANCH} &&\
    mkdir build && cd build &&\
    cmake -DCMAKE_BUILD_TYPE=${SOURCES_BUILD_TYPE} \
          -DCMAKE_INSTALL_PREFIX=${IIT_INSTALL} \
          -DIDYNTREE_USES_OCTAVE:BOOL=ON \
          -DIDYNTREE_INSTALL_MATLAB_LIBDIR=matlab \
          -DIDYNTREE_INSTALL_MATLAB_MFILESDIR=matlab \
          -DIDYNTREE_USES_KDL:BOOL=ON \
          .. &&\
    make -j ${GCC_JOBS} install

RUN cd ${IIT_SOURCES}/codyco-superbuild &&\
    mkdir build && cd build &&\
    cmake -DCMAKE_BUILD_TYPE=${SOURCES_BUILD_TYPE} \
          -DCODYCO_USES_GAZEBO:BOOL=ON \
          -DNON_INTERACTIVE_BUILD=ON \
          -DCMAKE_INSTALL_PREFIX=${IIT_INSTALL} \
          .. &&\
    make -j ${GCC_JOBS}
ENV CODYCO_SUPERBUILD_ROOT=${IIT_SOURCES}/codyco-superbuild
ARG CODYCO_SUPERBUILD_INSTALL=${CODYCO_SUPERBUILD_ROOT}/build/install
ENV IIT_PATH=${IIT_PATH:+${IIT_PATH}:}${CODYCO_SUPERBUILD_ROOT}/build/install/bin
ENV LD_LIBRARY_PATH=${LD_LIBRARY_PATH:+${LD_LIBRARY_PATH}:}${CODYCO_SUPERBUILD_INSTALL}/lib
ENV YARP_DATA_DIRS=${YARP_DATA_DIRS:+${YARP_DATA_DIRS}:}${CODYCO_SUPERBUILD_INSTALL}/share/codyco
ENV GAZEBO_PLUGIN_PATH=${GAZEBO_PLUGIN_PATH:+${GAZEBO_PLUGIN_PATH}:}${CODYCO_SUPERBUILD_INSTALL}/lib
ENV GAZEBO_MODEL_PATH=${GAZEBO_MODEL_PATH:+${GAZEBO_MODEL_PATH}:}${CODYCO_SUPERBUILD_INSTALL}/share/gazebo/models
ENV GAZEBO_RESOURCE_PATH=${GAZEBO_RESOURCE_PATH:+${GAZEBO_RESOURCE_PATH}:}${CODYCO_SUPERBUILD_INSTALL}/share/gazebo/worlds
ENV PATH=${IIT_PATH}:${ROOT_PATH}

# Misc setup of the image
# =======================

# Some QT-Apps/Gazebo don't show controls without this
ENV QT_X11_NO_MITSHM 1

# Include a custom bashrc
COPY bashrc /usr/etc/skel/bashrc-dev
COPY bashrc-colors /usr/etc/skel/bashrc-colors

# Include an additional entrypoint script
COPY entrypoint.sh /usr/sbin/entrypoint-dev.sh
RUN chmod 755 /usr/sbin/entrypoint-dev.sh
ENTRYPOINT ["/usr/sbin/entrypoint-dev.sh"]
