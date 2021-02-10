# Combined ideas from:
# https://github.com/pangeo-data/pangeo-docker-images/blob/master/base-image/Dockerfile
# https://gitlab.eox.at/esa/vires_vre_ops/-/blob/staging/docker/vre-base-notebook/Dockerfile

FROM jupyter/scipy-notebook

USER root

COPY apt.txt .
RUN apt-get update \
    && xargs -a apt.txt apt-get install -yq --no-install-recommends \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* \
    && rm apt.txt

USER jovyan

# Use mamba to speed up environment solving
RUN conda install -n base --yes 'conda-forge::mamba'
# Extend base scipy environment with ours
COPY environment.yml .
RUN mamba env update -n base --file environment.yml \
    && mamba clean -yaf \
    && rm environment.yml

COPY requirements.txt .
RUN pip install --upgrade pip \
    && pip install -r requirements.txt \
    && rm requirements.txt

# NB: JupyterLab extensions will be installable through pip from v3 on
RUN jupyter labextension install \
    jupyter-matplotlib \
    jupyterlab-topbar-extension \
    jupyterlab-system-monitor \
    jupyterlab-theme-toggle \
    jupyter-leaflet \
    @jupyter-widgets/jupyterlab-manager \
    @pyviz/jupyterlab_pyviz \
    dask-labextension
    
RUN jupyter lab build

RUN pip install --upgrade git+https://github.com/ESA-VirES/VirES-Python-Client@staging#egg=viresclient

# Magnetic Models and dependencies, including spacepy
RUN mkdir $HOME/build \
    && cd $HOME/build \
    && wget --no-verbose https://github.com/ESA-VirES/MagneticModel/archive/staging.zip -O MagneticModel-staging.zip \
    && unzip MagneticModel-staging.zip \
    && mv MagneticModel-staging MagneticModel
RUN cd $HOME/build \
    && conda build MagneticModel/libcdf \
    && conda build MagneticModel/qdipole
RUN mamba install --quiet --yes --use-local qdipole cdf
RUN pip install spacepy
RUN /bin/bash -c 'source activate base && pip install build/MagneticModel/eoxmagmod'
ENV CDF_LIB /opt/conda/lib
RUN conda build purge \
    && cd $HOME \
    && rm -fR $HOME/build \
    && mamba clean -yaf
