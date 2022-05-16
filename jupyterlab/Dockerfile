FROM mambaorg/micromamba:0.14.0 as build

# Install basic tools
RUN micromamba install -qy -c conda-forge python nodejs yarn \
    && useradd --shell /bin/bash jovyan \
    && chown jovyan $HOME

# Install npm packages - faster build thanks to caching
## package_json.tar.gz contains all package.json files using
## `tar cvf package_json.tar.gz package.json packages/*/package.package_json`
ADD ./jupyterlab/package.json /tmp/jupyterlab-dev/package.json 
COPY ./jupyterlab/yarn.lock /tmp/jupyterlab-dev

RUN cd /tmp/jupyterlab-dev \
    && yarn install --ignore-scripts

# Install python dependencies - faster build thanks to caching
COPY ./jupyterlab/setup.cfg /tmp

RUN list_package=$(python -c "from configparser import ConfigParser; c = ConfigParser(); c.read('/tmp/setup.cfg'); print(' '.join(c['options']['install_requires'].strip().splitlines()))") \
    && micromamba install -qy -c conda-forge $list_package \
    && micromamba clean -ay \
    && rm /tmp/setup.cfg

# Install JupyterLab
COPY --chown=jovyan ./jupyterlab/builder/ /tmp/jupyterlab-dev/builder/
COPY --chown=jovyan ./jupyterlab/buildutils/ /tmp/jupyterlab-dev/buildutils/
COPY --chown=jovyan ./jupyterlab/dev_mode/ /tmp/jupyterlab-dev/dev_mode/
COPY --chown=jovyan ./jupyterlab/jupyterlab/ /tmp/jupyterlab-dev/jupyterlab/
COPY --chown=jovyan ./jupyterlab/packages/ /tmp/jupyterlab-dev/packages/
COPY --chown=jovyan ./jupyterlab/scripts/ /tmp/jupyterlab-dev/scripts/
COPY --chown=jovyan ./jupyterlab/*.* ./LICENSE /tmp/jupyterlab-dev/

RUN pushd /tmp/jupyterlab-dev \
    && pip install -e .[ui-tests]

USER jovyan
WORKDIR ${HOME}

ENV PATH="/home/micromamba/.local/bin:$PATH"

RUN mkdir -p /home/micromamba/jlab_root

COPY ./jupyterlab/docker/jupyter_server_config.json /etc/jupyter/

EXPOSE 8888

ENTRYPOINT ["jupyter", "lab"]
