# ============================================================
# Stage 1: runtime ——  最终镜像
# ============================================================
FROM ubuntu:22.04 AS base

ENV DEBIAN_FRONTEND=noninteractive \
    PATH=/opt/miniconda3/bin:$PATH

# ---- 系统依赖（不常变 → 强缓存层）----
RUN set -eux; \
    apt-get update && \
    apt-get install -y --no-install-recommends \
        wget bzip2 ca-certificates git parallel \
        libopenblas-dev cmake && \
    rm -rf /var/lib/apt/lists/*

# ---- Miniforge（不常变 → 强缓存层）----
RUN set -eux; \
    wget -q https://github.com/conda-forge/miniforge/releases/download/24.9.2-0/Miniforge3-24.9.2-0-Linux-x86_64.sh \
         -O /tmp/miniforge.sh && \
    bash /tmp/miniforge.sh -b -p /opt/miniconda3 && \
    rm /tmp/miniforge.sh && \
    conda clean -afy

# ---- cnmf_env（偶尔变 → 弱缓存层）----
# 只用 conda-forge + bioconda，**不混 defaults**（避免 channel 冲突）
RUN conda create -n cnmf_env --yes \
        -c conda-forge -c bioconda \
        python \
        fastcluster matplotlib numpy \
        palettable pandas scipy \
        scikit-learn pyyaml scanpy \
        parallel jupyterlab cnmf harmonypy scikit-misc\
    && conda clean -afy

WORKDIR /home
CMD ["/bin/bash"]
