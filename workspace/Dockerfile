ARG CUDA_VER=11.3.1
ARG CUDA_PREFIX_VERSION=11.3

# Preamble from extension [nvidia]
FROM nvidia/opengl:1.0-glvnd-devel-ubuntu16.04 as glvnd
FROM nvidia/cuda:${CUDA_VER}-cudnn8-devel-ubuntu16.04

USER root

# Minimal setup
RUN apt-get update \
 && apt-get install -y locales lsb-release curl --no-install-recommends \
    libgl1 \
    libgles2 \
    && rm -rf /var/lib/apt/lists/*

# Snippet from extension [nvidia]
# Open nvidia-docker2 GL support
COPY --from=glvnd /usr/local/lib/x86_64-linux-gnu /usr/local/lib/x86_64-linux-gnu
COPY --from=glvnd /usr/local/lib/i386-linux-gnu /usr/local/lib/i386-linux-gnu
COPY --from=glvnd /usr/lib/x86_64-linux-gnu /usr/lib/x86_64-linux-gnu
COPY --from=glvnd /usr/lib/i386-linux-gnu /usr/lib/i386-linux-gnu
# if the path is alreaady present don't fail because of being unable to append
RUN ( echo '/usr/local/lib/x86_64-linux-gnu' >> /etc/ld.so.conf.d/glvnd.conf && ldconfig || grep -q /usr/local/lib/x86_64-linux-gnu /etc/ld.so.conf.d/glvnd.conf ) && \
    ( echo '/usr/local/lib/i386-linux-gnu' >> /etc/ld.so.conf.d/glvnd.conf &&  ldconfig || grep -q /usr/local/lib/i386-linux-gnu /etc/ld.so.conf.d/glvnd.conf )
ENV LD_LIBRARY_PATH /usr/local/lib/x86_64-linux-gnu:/usr/local/lib/i386-linux-gnu${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}

COPY --from=glvnd /usr/local/share/glvnd/egl_vendor.d/10_nvidia.json /usr/local/share/glvnd/egl_vendor.d/10_nvidia.json

# not sure if these two lines are needed anymore
ARG DEBIAN_FRONTEND=noninteractive
RUN dpkg-reconfigure locales

ENV NVIDIA_VISIBLE_DEVICES ${NVIDIA_VISIBLE_DEVICES:-all}
# ENV NVIDIA_DRIVER_CAPABILITIES graphics,utility,compute
ENV NVIDIA_DRIVER_CAPABILITIES all

# install kinetic for tiago
RUN sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'
RUN sh -c "curl -s https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc | apt-key add -"
RUN apt-get update \
 && apt-get install -y git python-rosinstall ros-kinetic-desktop-full python-catkin-tools ros-kinetic-range-sensor-layer ros-kinetic-teb-local-planner \
 ros-kinetic-joint-state-controller ros-kinetic-tf2-sensor-msgs ros-kinetic-twist-mux ros-kinetic-ompl ros-kinetic-controller-manager ros-kinetic-moveit-core ros-kinetic-moveit-ros-perception ros-kinetic-moveit-ros-move-group ros-kinetic-moveit-kinematics ros-kinetic-moveit-ros-planning-interface ros-kinetic-moveit-simple-controller-manager ros-kinetic-moveit-planners-ompl ros-kinetic-joy ros-kinetic-joy-teleop ros-kinetic-teleop-tools ros-kinetic-control-toolbox ros-kinetic-sound-play ros-kinetic-navigation ros-kinetic-eband-local-planner ros-kinetic-depthimage-to-laserscan  ros-kinetic-openslam-gmapping ros-kinetic-gmapping ros-kinetic-moveit-commander ros-kinetic-humanoid-nav-msgs ros-kinetic-urdf-geometry-parser ros-kinetic-four-wheel-steering-msgs

RUN apt-get install -y --no-install-recommends python-rosdep

RUN rosdep init \
 && rosdep fix-permissions \
 && rosdep update
RUN echo "source /opt/ros/kinetic/setup.bash" >> ~/.bashrc

# audio stuff
RUN apt-get update && apt-get install -y alsa-base pulseaudio

ARG TIAGO_USER
SHELL ["/bin/bash", "-c"]
RUN echo "${TIAGO_USER} ALL=(ALL) NOPASSWD:ALL" | EDITOR="tee -a" visudo

RUN apt-get install wget vim openssh-server apt-utils -y

RUN sed -i "s/Port 22/Port 2222/" /etc/ssh/sshd_config
COPY entrypoint.sh /entrypoint.sh

COPY pythonr /usr/bin/pythonr
RUN chown root.root /usr/bin/pythonr
RUN chmod 777 /usr/bin/pythonr


# set up ROS environment
RUN mkdir -p /opt/tiago_public_ws
WORKDIR /opt/tiago_public_ws
RUN wget https://raw.githubusercontent.com/pal-robotics/tiago_tutorials/kinetic-devel/tiago_public.rosinstall
RUN rosinstall src /opt/ros/kinetic tiago_public.rosinstall
RUN rm -f /etc/ros/rosdep/sources.list.d/20-default.list


RUN rosdep init
RUN rosdep update --include-eol-distros
RUN rosdep install --from-paths src --ignore-src --rosdistro kinetic --skip-keys="opencv2 opencv2-nonfree pal_laser_filters speed_limit_node sensor_to_cloud hokuyo_node libdw-dev python-graphitesend-pip python-statsd pal_filters pal_vo_server pal_usb_utils pal_pcl pal_pcl_points_throttle_and_filter pal_karto pal_local_joint_control camera_calibration_files pal_startup_msgs pal-orbbec-openni2 dummy_actuators_manager pal_local_planner gravity_compensation_controller current_limit_controller dynamic_footprint dynamixel_cpp tf_lookup opencv3 joint_impedance_trajectory_controller urdf_test uvc_camera range_sensor_layer teb_local_planner pointcloud_to_laserscan octomap_server spatio_temporal_voxel_layer openni2_launch moveit_ros_visualization slam_toolbox sick_tim libuvc_camera mynteye_description stockbot_description" -y
RUN apt-get install -y zip unzip

WORKDIR /
RUN apt-get install -y python-pip && python -m pip install pip==18.1
RUN python2.7 -m pip install rospkg future torch torchvision

WORKDIR /opt/tiago_public_ws
RUN apt-get update
RUN apt-get install ros-kinetic-catkin python-catkin-tools python-pip sudo ros-kinetic-ros-numpy ros-kinetic-moveit-ros-visualization -y
RUN source /opt/ros/kinetic/setup.bash; catkin build -DCATKIN_ENABLE_TESTING=0

#update cuda paths
ARG CUDA_VER=11.3
ENV LD_LIBRARY_PATH=/usr/local/${CUDA_VER}/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
ENV PATH=/usr/local/${CUDA_VER}/bin${PATH:+:${PATH}}