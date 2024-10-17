# Base image from NVIDIA CUDA with Ubuntu 22.04
FROM nvidia/cuda:11.8.0-devel-ubuntu22.04

# Install necessary packages
RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
    sudo git zip curl \
    libopencv-dev libffi-dev liblapack-dev libsqlite3-dev \
    build-essential libssl-dev libbz2-dev libreadline-dev \
    ssh-client wget vim python3-opencv && \
    rm -rf /var/lib/apt/lists/* /var/cache/apt/archives/*

# User setting with specified UID (default: 1000)
ARG UID=1000  # Default UID to 1000 if not passed
RUN useradd docker -l -u ${UID} -G sudo -s /bin/bash -m
RUN echo 'Defaults visiblepw' >> /etc/sudoers
RUN echo 'docker ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers
USER docker

# Python environment setup using Pyenv
ENV PYENV_ROOT=/home/docker/.pyenv
ENV PATH="$PYENV_ROOT/shims:$PYENV_ROOT/bin:$PATH"
RUN curl -L https://pyenv.run | bash

# Install specific Python version using Pyenv
ARG PYTHON_VERSION=3.8.4  # Default to 3.8.4 if not passed
RUN pyenv install ${PYTHON_VERSION} && pyenv global ${PYTHON_VERSION}

# Upgrade pip and install Torch, Torchvision, and dependencies
ARG TORCH_VERSION=1.12.1
ARG TORCHVISION_VERSION=0.13.1
ARG TORCH_CUDA_ARCH_LIST=6.0+PTX  # Ensure correct CUDA architecture is passed
RUN pip install --upgrade pip setuptools wheel
RUN pip install torch==${TORCH_VERSION} --index-url https://download.pytorch.org/whl/cu116 \
    torchvision==${TORCHVISION_VERSION} torchaudio

# Install MMCV and MIM
ARG MIM_VERSION=0.3.3
ARG MMCV_VERSION=1.7.0
RUN pip install openmim==${MIM_VERSION}
RUN mim install mmcv-full==${MMCV_VERSION}

# Clone and install MMDetection
ARG MMDET_VERSION=2.28.0
WORKDIR /home/docker/
RUN git config --global http.postBuffer 524288000
RUN git clone https://github.com/open-mmlab/mmdetection.git mmdetection -b v${MMDET_VERSION} --depth 1
WORKDIR /home/docker/mmdetection

# Install additional Python dependencies from requirements.txt
COPY requirements.txt /tmp/requirements.txt
RUN pip install --cache-dir /tmp/pip-cache -r /tmp/requirements.txt

# Install lane NMS (Non-Maximum Suppression)
COPY --chown=docker libs/models/layers/nms /tmp/nms
WORKDIR /tmp/nms
RUN python setup.py install

# Ensure correct CUDA paths
ENV LD_LIBRARY_PATH="/usr/local/cuda/lib64:$LD_LIBRARY_PATH"
ENV PATH="/usr/local/cuda/bin:$PATH"

# Set Python path to include work and home directories
ENV PYTHONPATH="/work:/home/docker:/home/docker/mmdetection"

# Final working directory
WORKDIR /work

# Default entry point
ENTRYPOINT ["/bin/bash"]

