FROM ros:humble-ros-base

ARG USERNAME=rosdev
ARG UID=1000
ARG GID=$UID

# Install some dependencies packages
RUN apt update -q \
    && apt upgrade -q -y \
    && apt install -y --no-install-recommends \
    software-properties-common \
    python3-pip \
    python-is-python3 \
    xauth \
    && apt clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# Create and switch to user
RUN groupadd -g $GID $USERNAME \
    && useradd -lm -u $UID -g $USERNAME -s /bin/bash $USERNAME \
    && echo "$USERNAME ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers
USER $USERNAME

# Create workspace so that user own this directory
RUN mkdir -p /home/$USERNAME/ros2_ws/src
WORKDIR /home/$USERNAME/ros2_ws

# Copy configuration files
RUN echo 'source /opt/ros/'$ROS_DISTRO'/setup.bash' >> /home/$USERNAME/.bashrc \
    && echo 'source /home/'$USERNAME'/ros2_ws/install/setup.bash' >> /home/$USERNAME/.bashrc

# Use the system installation of Python (the one with ROS installed, etc).
# See https://juliapy.github.io/PythonCall.jl/dev/pythoncall/#pythoncall-config
ENV JULIA_CONDAPKG_BACKEND="Null"

# Install JuliaCall for Julia-Python integration. Import JuliaCall once to
# trigger installation Julia and required Julia packages. Then install common Julia
# packages to the Julia environment.
# See: https://juliapy.github.io/PythonCall.jl/stable/juliacall/
RUN pip install --no-cache-dir juliacall \
    && python -c 'from juliacall import Base' \
    && sudo ln -s $(python -c "from juliacall import Base; print(Base.julia_cmd()[1])") /usr/local/bin/julia \
    && export BDAI_JULIA_PYTHONCALL_PROJECT=$(python -c "from juliacall import Base; print(Base.joinpath(Base.splitpath(Base.active_project())[0:-1]))") \
    && julia --project="$BDAI_JULIA_PYTHONCALL_PROJECT" \
        -e 'using Pkg; Pkg.add("Revise"); Pkg.precompile()'

# Setup entrypoint
COPY ./ros_entrypoint.sh /
ENTRYPOINT ["/ros_entrypoint.sh"]
CMD ["bash"]
