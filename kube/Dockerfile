FROM ubuntu:20.04


ENV DEBIAN_FRONTEND=noninteractive


RUN apt-get update && apt-get install -y \
    software-properties-common \
    build-essential \
    curl \
    wget \
    gnupg2 \
    apt-transport-https \
    ca-certificates


RUN apt-get install -y python2 python2-dev python3 python3-dev python3-pip
RUN apt-key adv --keyserver keyserver.ubuntu.com --recv-keys E298A3A825C0D65DFD57CBB651716619E084DAB9 && \
    add-apt-repository 'deb https://cloud.r-project.org/bin/linux/ubuntu focal-cran40/' && \
    apt-get update && apt-get install -y --no-install-recommends r-base


RUN apt-get clean && rm -rf /var/lib/apt/lists/*


CMD ["tail", "-f", "/dev/null"]
