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
        python=3.7 \
        fastcluster=1.1.26 matplotlib=3.3.2 numpy=1.19.2 \
        palettable=3.3.0 pandas=1.1.3 scipy=1.5.2 \
        scikit-learn=0.23.2 pyyaml=5.3.1 scanpy=1.6.0 \
        parallel jupyterlab \
    && conda clean -afy

# 切换后续 RUN 的 shell 自动进入 cnmf_env
SHELL ["conda", "run", "-n", "cnmf_env", "/bin/bash", "-c"]

# ---- 业务依赖（偶尔变 → 弱缓存层）----
RUN pip install --no-build-isolation --no-cache-dir scikit-misc harmonypy

# ---- cNMF 本体（经常变 → 弱缓存层，放最后）----
WORKDIR /opt
RUN git clone --depth 1 https://github.com/dylkot/cNMF.git /opt/cNMF \
    && pip install --no-build-isolation --no-cache-dir /opt/cNMF \
    && rm -rf /opt/cNMF/.git

WORKDIR /home
CMD ["/bin/bash"]
