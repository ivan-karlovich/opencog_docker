#
# Container for the Hanson Robotics Eva head (stand-alone Owyl version).
# This provides the complete Owyl-behavior-tree version of the demo head.
# It includes UVC webcam vision input, pi_vision to discover faces, and
# behavior controlled by Owyl trees.  This is a stand-alone version of
# the Eva head, it does not depend on or use any OpenCog software.
# In particular, it is not capable of chat.
#
# To build:
# sudo docker build -t opencog/eva-owyl .
#
FROM opencog/ros-indigo-blender
MAINTAINER Linas Vepštas "linasvepstas@gmail.com"

# Install required packages
ENV LAST_OS_UPDATE 2022-05-29
RUN apt-get -y update
RUN apt-get -y upgrade
RUN apt-get -y install ros-indigo-tf ros-indigo-driver-common
RUN apt-get -y install ros-indigo-cv-bridge ros-indigo-image-transport
RUN apt-get -y install ros-indigo-openni-camera ros-indigo-mjpeg-server
RUN apt-get -y install ros-indigo-usb-cam
# The below are needed by perception node
RUN apt-get -y install ros-indigo-dynamixel-msgs
RUN apt-get -y install ros-indigo-robot-state-publisher
RUN apt-get -y install ros-indigo-joint-state-publisher

# need it to fix the following error thrown by face-tracker node:
# `Failed to load module “canberra-gtk-module”`
# see https://askubuntu.com/a/872397
RUN apt install -y libcanberra-gtk-module libcanberra-gtk3-module

# Owyl setup
WORKDIR /opt/
RUN git clone https://github.com/eykd/owyl.git
WORKDIR /opt/owyl/
RUN python setup.py install

# Install catkinized packages
WORKDIR /catkin_ws/src

# RUN git clone https://github.com/hansonrobotics/pi_vision.git
# need ros-indigo tag as master branch is for ROS noetic
RUN git clone https://github.com/opencog/pi_vision.git --branch ros-indigo --single-branch
# RUN git clone https://github.com/hansonrobotics/perception.git
RUN git clone https://github.com/opencog/perception
# RUN git clone https://github.com/hansonrobotics/eva_behavior.git
# basic_head_api.msg package is required by eva_behavior
RUN git clone https://github.com/ivan-karlovich/hansonrobotics_basic_head_api.git basic_head_api
#RUN git clone https://github.com/Gaboose/eva_behavior.git
RUN git clone https://github.com/ivan-karlovich/eva_behavior.git
#RUN git clone https://github.com/hansonrobotics/blender_api_msgs.git
#RUN git clone https://github.com/hansonrobotics/blender_api.git
RUN git clone https://github.com/opencog/blender_api_msgs.git
RUN git clone https://github.com/opencog/blender_api.git

# The pau2motors package defines the PAU messages, which are
# needed by the perception (vision-geometry) module.
# RUN git clone https://github.com/hansonrobotics/pau2motors.git
# RUN git clone https://github.com/hansonrobotics/robots_config.git
RUN git clone https://github.com/opencog/pau2motors.git
RUN git clone https://github.com/opencog/robots_config.git

# The motor side of things is not needed for the software head.
# RUN git clone https://github.com/geni-lab/ros_pololu_servo.git
# RUN git clone https://github.com/hansonrobotics/ros_pololu_servo.git
# RUN git clone https://github.com/hansonrobotics/ros_motors_webui.git

# Change line below on rebuild. Will use cache up to this line
ENV LAST_SOFTWARE_UPDATE 2015-02-19

# Git pull for all packages
RUN cd /catkin_ws/src/ && find . -maxdepth 1 -mindepth 1 -type d \
	-execdir git --git-dir=$PWD/{}/.git --work-tree=$PWD/{} pull \;

# The blender API has not been fully catkinized yet.
RUN pip3 install ./blender_api_msgs/

# Some versions of ros_pololu_server will fail during build; the
# work-around for this is to build twice (This because the missing
# files are autogened, and the pololu Cmakefiles fail to list them as
# dependencies.)
WORKDIR /catkin_ws
RUN bash -l -c "/opt/ros/indigo/bin/catkin_make" || true
RUN bash -l -c "/opt/ros/indigo/bin/catkin_make"

RUN echo source /catkin_ws/devel/setup.bash >> ~/.bashrc
# fix the following issue with ALSA:
# "ALSA lib pcm_dmix.c:1022:(snd_pcm_dmix_open) unable to open slave"
# see https://github.com/kevineye/docker-shairport-sync/issues/5
RUN echo "options snd_hda_intel index=1" > /etc/modprobe.d/default.conf

COPY /scripts/eva.sh /catkin_ws/eva.sh

ENTRYPOINT bash -l -c "./eva.sh; bash"
