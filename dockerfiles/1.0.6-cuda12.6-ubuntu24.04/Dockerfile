FROM nvcr.io/nvidia/cuda-dl-base:24.12-cuda12.6-devel-ubuntu24.04

WORKDIR /root/
ARG PYTHON_VERSION=3.12
ENV LANG=C.UTF-8 LC_ALL=C.UTF-8

# Leave these args here to better use the Docker build cache
# renovate: datasource=custom.miniconda_installer
ARG INSTALLER_URL_LINUX64="https://repo.anaconda.com/miniconda/Miniconda3-py312_25.9.1-3-Linux-x86_64.sh"
ARG SHA256SUM_LINUX64="07fc00c146a481f7ae2657a31eb7ec0d5d77a107b981d58ab5d7cd8bc99be778"
# renovate: datasource=custom.miniconda_installer
ARG INSTALLER_URL_AARCH64="https://repo.anaconda.com/miniconda/Miniconda3-py312_25.9.1-3-Linux-aarch64.sh"
ARG SHA256SUM_AARCH64="8c5c1d38011fed8053d7bdaaf7fab372c3f71dbb77791e07692fb8059b7828bd"

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
    conda tos accept --override-channels --channel https://repo.anaconda.com/pkgs/main && \
    conda tos accept --override-channels --channel https://repo.anaconda.com/pkgs/r && \
    conda create -n exp python=3.12 -y && \
    conda config --add channels conda-forge && \
    conda install -n exp pytorch==2.4.1 torchvision==0.19.1 torchaudio==2.4.1 pytorch-cuda=12.4 -c pytorch -c nvidia -y && \
    conda install -n exp conda-forge::tqdm -y && \
    conda install -n exp anaconda::numpy -y && \
    echo "conda activate exp" >> ~/.bashrc

ENV CONDA_DEFAULT_ENV=exp
ENV PATH=/opt/conda/envs/exp/bin:$PATH

RUN echo '#!/bin/bash\n\
source /opt/conda/etc/profile.d/conda.sh\n\
conda activate exp\n\
exec "$@"' > /entrypoint.sh && \
    chmod +x /entrypoint.sh

ENTRYPOINT ["/entrypoint.sh"]
CMD [ "/bin/bash" ]
