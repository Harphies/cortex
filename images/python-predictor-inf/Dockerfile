FROM ubuntu:18.04

RUN apt-get update -qq && apt-get install -y -q \
    wget \
    gnupg && \
    echo "deb https://apt.repos.neuron.amazonaws.com bionic main" >> /etc/apt/sources.list.d/neuron.list && \
    wget -qO - https://apt.repos.neuron.amazonaws.com/GPG-PUB-KEY-AMAZON-AWS-NEURON.PUB | apt-key add - && \
    apt-get update -qq && apt-get install -y -q \
    aws-neuron-tools=1.4.2.0 \
    aws-neuron-runtime=1.4.3.0 && \
    apt-get clean -qq && rm -rf /var/lib/apt/lists/*

RUN wget -P /tmp/ https://github.com/just-containers/s6-overlay/releases/download/v2.1.0.2/s6-overlay-amd64-installer && \
    chmod +x /tmp/s6-overlay-amd64-installer && /tmp/s6-overlay-amd64-installer / && rm /tmp/s6-overlay-amd64-installer

ENV S6_BEHAVIOUR_IF_STAGE2_FAILS 2

ENV PATH=/opt/aws/neuron/bin/:$PATH

RUN apt-get update -qq && apt-get install -y -q \
    build-essential \
    pkg-config \
    software-properties-common \
    curl \
    git \
    unzip \
    zlib1g-dev \
    locales \
    nginx=1.14.* \
    && apt-get clean -qq && rm -rf /var/lib/apt/lists/*

RUN locale-gen en_US.UTF-8
ENV LANG=en_US.UTF-8 LANGUAGE=en_US.UTF-8 LC_ALL=en_US.UTF-8

ENV PATH=/opt/conda/bin:$PATH
ENV PYTHONVERSION=3.6.9
ENV CORTEX_IMAGE_TYPE=python-predictor-inf

# conda needs an untainted base environment to function properly
# that's why a new separate conda environment is created
RUN curl "https://repo.anaconda.com/miniconda/Miniconda3-4.7.12.1-Linux-x86_64.sh" --output ~/miniconda.sh && \
    /bin/bash ~/miniconda.sh -b -p /opt/conda && \
    rm -rf ~/.cache ~/miniconda.sh

# split the conda installations because the dev boxes have limited memory
RUN /opt/conda/bin/conda create -n env -c conda-forge python=$PYTHONVERSION pip=19.* && \
    /opt/conda/bin/conda clean -a && \
    ln -s /opt/conda/etc/profile.d/conda.sh /etc/profile.d/conda.sh && \
    echo ". /opt/conda/etc/profile.d/conda.sh" > ~/.env && \
    echo "conda activate env" >> ~/.env && \
    echo "source ~/.env" >> ~/.bashrc

ENV BASH_ENV=~/.env
SHELL ["/bin/bash", "-c"]

COPY pkg/cortex/serve/serve.requirements.txt /src/cortex/serve/serve.requirements.txt
COPY pkg/cortex/serve/cortex_internal.requirements.txt /src/cortex/serve/cortex_internal.requirements.txt
RUN pip install --no-cache-dir \
    -r /src/cortex/serve/serve.requirements.txt \
    -r /src/cortex/serve/cortex_internal.requirements.txt

COPY pkg/cortex/serve/init/install-core-dependencies.sh /usr/local/cortex/install-core-dependencies.sh
RUN chmod +x /usr/local/cortex/install-core-dependencies.sh

COPY pkg/cortex/serve/ /src/cortex/serve
COPY pkg/cortex/client/ /src/cortex/client
ENV CORTEX_LOG_CONFIG_FILE /src/cortex/serve/log_config.yaml

RUN pip install --no-deps /src/cortex/serve/ && \
    pip install /src/cortex/client && \
    rm -r /src/cortex/client
RUN mv /src/cortex/serve/init/bootloader.sh /etc/cont-init.d/bootloader.sh

ENTRYPOINT ["/init"]
