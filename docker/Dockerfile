FROM ros:humble-ros-base-jammy
ARG USERNAME=dev
ARG UID=1000
ARG GID=1000
SHELL ["/bin/bash", "-c"]

# 1. Dependensi sistem (jarang berubah, jadi di atas agar ter-cache)
RUN apt-get update && apt-get install -y --no-install-recommends \
python3-colcon-common-extensions \
ros-humble-demo-nodes-cpp \
ros-humble-demo-nodes-py \
&& rm -rf /var/lib/apt/lists/*

#2. User non-root agar file di bind mount tidak milik root
RUN groupadd -g ${GID} ${USERNAME} 2>/dev/null || true && \
useradd -m -u ${UID} -g ${GID} -s /bin/bash ${USERNAME}

COPY docker/entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh
USER ${USERNAME}
WORKDIR /ws

#3. Kode sumber (paling sering berubah, jadi paling bawah)
COPY --chown=${UID}:${GID} src/ /ws/src/
RUN source /opt/ros/humble/setup.bash && colcon build
ENTRYPOINT ["/entrypoint.sh"]
CMD ["bash"]
