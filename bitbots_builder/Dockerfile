# Use upstream melodic images as base
FROM docker.io/ros:melodic-ros-base-bionic AS bitbots-builder

# Install system dependencies
RUN apt-get update
RUN apt-get remove -y ros-melodic-rosdoc-lite python-sphinx
RUN apt-get install -y sudo python3-pip python-pip python-coverage python-xmlrunner python-rospkg python-catkin-pkg \
    python-catkin-lint python-rosdep dia python3-sphinx python3-sphinx-rtd-theme python3-breathe
RUN python3 -m pip install rospkg catkin-pkg exhale
RUN python3 -m pip install git+https://github.com/catkin/catkin_tools

# Install sudoers file
ADD sudoers /etc/sudoers

# Add user
ARG UID=150
RUN useradd -M -d /catkin_ws -s /bin/bash -u $UID builder

# Build empty install space in shared install space
ARG SHARED_INSTALL_SPACE="/srv/catkin_install"
RUN . /opt/ros/melodic/setup.sh && \
    mkdir -p /catkin_ws/src && cd /catkin_ws && \
    catkin init >/dev/null && \
    catkin config --install --merge-install --install-space $SHARED_INSTALL_SPACE >/dev/null && \
    catkin build && \
    chown -R builder:builder $SHARED_INSTALL_SPACE && \
    rm -r /catkin_ws

# Setup catkin workspace
RUN . /opt/ros/melodic/setup.sh && \
    mkdir -p /catkin_ws/src; cd /catkin_ws && \
    catkin init && \
    catkin config --profile default --extend $SHARED_INSTALL_SPACE --install --isolate-install && \
    catkin config --profile install --extend $SHARED_INSTALL_SPACE --install --merge-install --install-space $SHARED_INSTALL_SPACE && \
    catkin build && \
    chown -R builder:builder /catkin_ws && \
    chmod -R 740 /catkin_ws

# Setup rosdep
RUN su -c "rosdep update" builder

# Setup runtime
ENV DEBIAN_FRONTEND=readline
ENV ROS_PYTHON_VERSION=3
VOLUME ["/srv/catkin_install"]
USER builder:builder
COPY entrypoint.sh /usr/local/bin/entrypoint
ENTRYPOINT ["/usr/local/bin/entrypoint"]
CMD ["bash"]
