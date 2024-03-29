# An incomplete base Docker image for running JupyterHub
#
# Add your configuration to create a complete derivative Docker image.
#
# Include your configuration settings by starting with one of two options:
#
# Option 1:
#
# FROM jupyterhub/jupyterhub:latest
#
# And put your configuration file jupyterhub_config.py in /srv/jupyterhub/jupyterhub_config.py.
#
# Option 2:
#
# Or you can create your jupyterhub config and database on the host machine, and mount it with:
#
# docker run -v $PWD:/srv/jupyterhub -t jupyterhub/jupyterhub
#
# NOTE
# If you base on jupyterhub/jupyterhub-onbuild
# your jupyterhub_config.py will be added automatically
# from your docker directory.

ARG BASE_IMAGE=ubuntu:focal-20200729
FROM $BASE_IMAGE AS builder

USER root

ENV DEBIAN_FRONTEND noninteractive
RUN apt-get update \
 && apt-get install -yq --no-install-recommends \
    build-essential \
    ca-certificates \
    locales \
    python3-dev \
    python3-pip \
    python3-pycurl \
    nodejs \
    npm \
 && apt-get clean \
 && rm -rf /var/lib/apt/lists/*

RUN python3 -m pip install --upgrade setuptools pip wheel

# copy everything except whats in .dockerignore, its a
# compromise between needing to rebuild and maintaining
# what needs to be part of the build
COPY ./jupyterhub/ /src/jupyterhub/
WORKDIR /src/jupyterhub

# Build client component packages (they will be copied into ./share and
# packaged with the built wheel.)
RUN python3 setup.py bdist_wheel
RUN python3 -m pip wheel --wheel-dir wheelhouse dist/*.whl


FROM $BASE_IMAGE

USER root

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update \
 && apt-get install -yq --no-install-recommends \
    ca-certificates \
    curl \
    gnupg \
    locales \
    python3-pip \
    python3-pycurl \
    nodejs \
    npm \
 && apt-get clean \
 && rm -rf /var/lib/apt/lists/*

ENV SHELL=/bin/bash \
    LC_ALL=en_US.UTF-8 \
    LANG=en_US.UTF-8 \
    LANGUAGE=en_US.UTF-8

RUN  locale-gen $LC_ALL

# always make sure pip is up to date!
RUN python3 -m pip install --no-cache --upgrade setuptools pip

RUN npm install -g configurable-http-proxy@^4.2.0 \
 && rm -rf ~/.npm

# install the wheels we built in the first stage
COPY --from=builder /src/jupyterhub/wheelhouse /tmp/wheelhouse
RUN python3 -m pip install --no-cache /tmp/wheelhouse/*

RUN mkdir -p /srv/jupyterhub/
WORKDIR /srv/jupyterhub/

# **** jupyter lab 연결 **** 
WORKDIR /srv/jupyterhub/
COPY ./jupyterlab /srv/jupyterhub/

RUN pip install --upgrade pip 
#WORKDIR /srv/jupyterhub/jupyterlab
RUN cd jupyterlab
RUN pip3 install nbconvert
RUN pip3 install jupyterlab
RUN npm install -g n
RUN n stable

RUN python3 setup.py install
RUN pip3 install -e .
RUN jlpm install

WORKDIR /srv/jupyterhub/
# my config file
RUN pip install jupyterhub-nativeauthenticator 
COPY ./jupyterhub/jupyterhub_config.py /srv/jupyterhub/

RUN apt update 
RUN apt install sudo 
RUN apt install wget 
RUN wget https://bootstrap.pypa.io/pip/2.7/get-pip.py 
RUN sudo python2 get-pip.py
RUN python2 -m pip install ipykernel
RUN python2 -m ipykernel install 

EXPOSE 8000

LABEL maintainer="Jupyter Project <jupyter@googlegroups.com>"
LABEL org.jupyter.service="jupyterhub"

CMD ["jupyterhub"]