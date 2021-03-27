# Ref:
# https://gitlab.eox.at/esa/vires_vre_ops/-/blob/staging/docker/

# https://hub.docker.com/r/jupyter/scipy-notebook/tags
# # Uses JupyterLab 3 (is downgraded to 2 in this Dockerfile):
FROM jupyter/scipy-notebook:4d9c9bd9ced0
# # Uses JupyterLab 2:
# FROM jupyter/scipy-notebook:aec555e49be6

USER root

RUN sudo apt-get update \
    && sudo apt-get install -yq --no-install-recommends \
    octave gnuplot ghostscript epstool fig2dev pstoedit dvipng \
    curl mc htop less vim-tiny openssh-client \
    iputils-ping net-tools dnsutils inetutils-traceroute \
    && rm -rf /var/lib/apt/lists/*

USER jovyan

# Generic conda packages
# Use mamba to speed up environment solving
# mamba is included in recent scipy-notebook images
RUN mamba install --yes \
   'conda-forge::pytest' \
   'conda-forge::nbval' \
   'conda-forge::watermark' \
   'conda-forge::octave_kernel' \
   'conda-forge::ipyparallel' \
   'conda-forge::cartopy' \
   'conda-forge::fortran-compiler' \
   'conda-forge::cxx-compiler' \
   'conda-forge::make' \
   'conda-forge::dask-ml' \
   'conda-forge::zarr' \
   'conda-forge::intake' \
   'conda-forge::intake-xarray' \
   'conda-forge::geojson' \
   'conda-forge::geoplot' \
   && mamba clean --all -f -y
# [geojson geoplot] for https://github.com/MagGeo/MagGeo-Annotation-Program/blob/master/MagGeoRequirements.yml

# Pyviz advanced visualisation
RUN mamba install --yes \
   'conda-forge::holoviews' \
   'conda-forge::geoviews' \
   'conda-forge::hvplot' \
   'conda-forge::datashader' \
   'conda-forge::panel' \
   'conda-forge::param' \
   'conda-forge::colorcet' \
   'conda-forge::pyct[cmd]' \
   'conda-forge::netcdf4' \
   'conda-forge::fastparquet' \
   'conda-forge::selenium' \
   'conda-forge::pyviz_comms' \
   && mamba clean --all -f -y

# Jupyter extensions
RUN mamba install --yes \
   'conda-forge::jupyterlab<3' \
   'conda-forge::jupyterlab-git' \
   'conda-forge::nbgitpuller' \
   'conda-forge::nbdime' \
   'conda-forge::ipyleaflet' \
   && mamba clean --all -f -y \
   && jupyter labextension install @pyviz/jupyterlab_pyviz \
                                   jupyterlab-topbar-extension \
                                   jupyterlab-logout \
                                   jupyterlab-theme-toggle \
                                   @jupyterlab/toc \
                                   @jupyter-widgets/jupyterlab-manager \
                                   jupyter-leaflet \
   && jupyter lab build

# Include? (requires configuration)
# Notebook resource usage:
# 'conda-forge::nbresuse'
# jupyter labextension install jupyterlab-system-monitor
# Dask cluster management:
# 'conda-forge::dask-labextension'
# jupyter labextension install dask-labextension

# Magnetic Models and dependencies
RUN mkdir $HOME/build \
    && cd $HOME/build \
    && wget --no-verbose https://github.com/ESA-VirES/MagneticModel/archive/staging.zip -O MagneticModel-staging.zip \
    && unzip MagneticModel-staging.zip \
    && mv MagneticModel-staging MagneticModel \
    && conda install --quiet --yes conda-build conda-verify \
    && conda build MagneticModel/libcdf \
    && conda build MagneticModel/qdipole \
    && conda build purge \
    && conda install --quiet --yes --use-local qdipole cdf \
    && conda remove --quiet --yes --force conda-build conda-verify \
    && conda clean --all -f -y \
    && pip install --upgrade pip \
    && pip install spacepy \
    && /bin/bash -c 'source activate base && pip install MagneticModel/eoxmagmod' \
    && cd $HOME \
    && rm -fR $HOME/build
ENV CDF_LIB /opt/conda/lib

# Swarm/domain-specific packages
RUN pip install --upgrade pip \
    && pip install viresclient==0.7.2 \
    && pip install pyamps==1.4.0 \
    && pip install apexpy==1.1.0 \
    && pip install magpysv==1.0 \
    && pip install swarmpyfac==0.2.2 \
    && pip install chaosmagpy==0.6.0 \
    && /bin/bash -c 'source activate base && pip install aacgmv2==2.6.2' \
    && pip install hapiclient==0.1.7

# pyAMPS upgrade
RUN pip install --upgrade pip \
    && pip install --upgrade 'git+https://github.com/klaundal/pyAMPS@0661c5d6faba0f0f0be567118ab4b0307e462d5e#egg=pyamps' \
    && rm -fR $HOME/.cache/pip/

# experimental installation of the IPB model
RUN mkdir $HOME/build \
    && cd $HOME/build \
    && wget https://gitext.gfz-potsdam.de/rother/ibp-model/-/archive/master/ibp-model-master.zip -O ibp-model-master.zip \
    && unzip ibp-model-master.zip \
    && mv ibp-model-master ibp-model \
    && cat ibp-model/requirements.txt \
    && sed -i -e '/\(getopt\|sphinx\|numpydoc\)/d' ibp-model/requirements.txt \
    && sed -i -e "s/,'getopt'//g" ibp-model/setup.py \
    && pip install ./ibp-model/ \
    && cd $HOME \
    && rm -fR $HOME/build

# # VRE JupyterLab customization (requires Lab < 2)
# RUN mkdir $HOME/build \
#     && cd $HOME/build \
#     && wget --no-verbose https://github.com/ESA-VirES/exampleLoader/archive/jsonload.zip -O exampleLoader-jsonload.zip \
#     && unzip exampleLoader-jsonload.zip \
#     && mv exampleLoader-jsonload exampleLoader \
#     && cd exampleLoader \
#     && jlpm \
#     && jlpm build \
#     && jupyter labextension install . \
#     && cd $HOME \
#     && rm -fR $HOME/build $HOME/.cache/yarn/ $HOME/.cache/pip/

# Upgrade viresclient
RUN pip install --upgrade git+https://github.com/ESA-VirES/VirES-Python-Client@staging#egg=viresclient

# Experimental jupyter-panel-proxy install
RUN pip install git+https://github.com/smithara/jupyter-panel-proxy@swarm-vre#egg=jupyter-panel-proxy

# Experimental demo of jupyter-panel-proxy
WORKDIR $HOME
RUN mkdir -p shared \
    && cd shared \
    && git clone https://github.com/Swarm-DISC/Swarm_notebooks \
    && cd Swarm_notebooks \
    && git checkout staging \
    && cd dashboards \
    && wget https://raw.githubusercontent.com/smithara/panel-experiment/main/demo.ipynb \
    && cd $HOME
