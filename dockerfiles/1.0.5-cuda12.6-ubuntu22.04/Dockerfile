FROM nvcr.io/nvidia/cuda-dl-base:24.10-cuda12.6-devel-ubuntu22.04

WORKDIR /root/
ARG PYTHON_VERSION=3.10
ENV LANG=C.UTF-8 LC_ALL=C.UTF-8

# Leave these args here to better use the Docker build cache
# renovate: datasource=custom.miniconda_installer
ARG INSTALLER_URL_LINUX64="https://repo.anaconda.com/miniconda/Miniconda3-py310_24.9.2-0-Linux-x86_64.sh"
ARG SHA256SUM_LINUX64="364869f004c6259268b09a667ed476080bf4f44e9c4ab06a293146df8990d43f"
# renovate: datasource=custom.miniconda_installer
ARG INSTALLER_URL_AARCH64="https://repo.anaconda.com/miniconda/Miniconda3-py310_24.9.2-0-Linux-aarch64.sh"
ARG SHA256SUM_AARCH64="5d022f16ecb01d33ac8c85a87a0a8dccfb025dc90b20dbd56c4b9ae8f9727578"

RUN set -x && \
    apt update && \
    apt install -y --no-install-recommends build-essential \
        bzip2 \
        ca-certificates \
        cmake \
        curl \
        gcc \
        git \
        libglib2.0-0 \
        libjpeg-dev \
        libpng-dev \
        libsm6 \
        libxext6 \
        libxrender1 \
        make \
        mercurial \
        openssh-client \
        procps \
        python3.10 \
        subversion \
        vim \
	    wget && \
    apt clean && \
    rm -rf /var/lib/apt/lists/* 

ENV PATH=/opt/conda/bin:$PATH

RUN set -x && \
    UNAME_M="$(uname -m)" && \
    if [ "${UNAME_M}" = "x86_64" ]; then \
    INSTALLER_URL="${INSTALLER_URL_LINUX64}"; \
    SHA256SUM="${SHA256SUM_LINUX64}"; \
    elif [ "${UNAME_M}" = "aarch64" ]; then \
        INSTALLER_URL="${INSTALLER_URL_AARCH64}"; \
        SHA256SUM="${SHA256SUM_AARCH64}"; \
    fi && \
    wget "${INSTALLER_URL}" -O miniconda.sh -q && \
    echo "${SHA256SUM} miniconda.sh" > shasum && \
    sha256sum --check --status shasum && \
    mkdir -p /opt && \
    bash miniconda.sh -b -p /opt/conda && \
    rm miniconda.sh shasum && \
    ln -s /opt/conda/etc/profile.d/conda.sh /etc/profile.d/conda.sh && \
    echo ". /opt/conda/etc/profile.d/conda.sh" >> ~/.bashrc && \
    find /opt/conda/ -follow -type f -name '*.a' -delete && \
    find /opt/conda/ -follow -type f -name '*.js.map' -delete && \
    /opt/conda/bin/conda clean -afy

RUN set -x && \
    conda create -n exp python=3.10 -y && \
    conda config --add channels conda-forge && \
    conda install -n exp pytorch==2.3.1 torchvision==0.18.1 torchaudio==2.3.1 pytorch-cuda=12.1 -c pytorch -c nvidia -y && \
    conda install -n exp conda-forge::tqdm -y && \
    conda install -n exp anaconda::numpy -y && \
    echo "conda activate exp" >> ~/.bashrc

CMD [ "/bin/bash" ]
