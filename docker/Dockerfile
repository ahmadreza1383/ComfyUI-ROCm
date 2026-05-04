# ROCm ComfyUI Dockerfile
# Based on AMD's official ROCm PyTorch container

FROM rocm/pytorch:rocm6.1.2_ubuntu20.04_py3.9_pytorch_release-2.1.2

# Set working directory
WORKDIR /workspace

# Install system dependencies + FFmpeg dev libs
RUN apt-get update && apt-get install -y --no-install-recommends \
    git \
    wget \
    curl \
    ffmpeg \
    libsm6 \
    libxext6 \
    libxrender-dev \
    libavcodec-dev \
    libavdevice-dev \
    libavfilter-dev \
    libavformat-dev \
    libavutil-dev \
    libswscale-dev \
    libswresample-dev \
    pkg-config \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Upgrade pip, setuptools, and wheel (pip داخل این ایمیج خیلی قدیمی است)
RUN python -m pip install --upgrade pip setuptools wheel

# Clone ComfyUI
RUN git clone https://github.com/comfyanonymous/ComfyUI.git

# Set ComfyUI as working directory
WORKDIR /workspace/ComfyUI

# Create necessary directories
RUN mkdir -p /workspace/ComfyUI/models/checkpoints && \
    mkdir -p /workspace/ComfyUI/models/vae && \
    mkdir -p /workspace/ComfyUI/models/loras && \
    mkdir -p /workspace/ComfyUI/models/embeddings && \
    mkdir -p /workspace/ComfyUI/models/upscale_models && \
    mkdir -p /workspace/ComfyUI/models/controlnet && \
    mkdir -p /workspace/ComfyUI/output && \
    mkdir -p /workspace/ComfyUI/input && \
    mkdir -p /workspace/ComfyUI/custom_nodes

# Copy ROCm-tested requirements file
COPY docker/requirements_rocm.txt /workspace/

# Install Python dependencies using ROCm-compatible requirements
RUN pip install --no-cache-dir -r /workspace/requirements_rocm.txt

# Copy startup script, models config, and sample workflow
COPY docker/startup.sh /workspace/startup.sh
COPY docker/download_models.py /workspace/download_models.py
COPY docker/models.yaml /workspace/models.yaml
COPY docker/sample_workflow.json /workspace/ComfyUI/

# Make startup script executable
RUN chmod +x /workspace/startup.sh

# Environment variables for model download behavior
ENV MODEL_DOWNLOAD=default

# Expose ComfyUI port
EXPOSE 8188

# Health check to ensure ComfyUI is running
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:8188/ || exit 1

# Use smart startup script that handles model downloads
CMD ["/workspace/startup.sh"]
