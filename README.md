# containers

## `swarm-dev`

- Based on [`jupyter/scipy-notebook`](https://jupyter-docker-stacks.readthedocs.io/en/latest/using/selecting.html#jupyter-scipy-notebook)
- Additional libraries merged in from apt (`apt.txt`), then conda (`environment.yml`), then pip (`requirements.txt`)
- Provides reasonably recent "latest" versions
- Our Python environment is all in the `base` conda environment

```
cd swarm-dem
docker build . -t swarm-dev
docker run --rm -it -t swarm-dev bash
docker run --rm -it -p 8889:8889 -v XXXXX:/home/jovyan -t swarm-dev jupyter lab --port 8889
```

## `pangeo-swarm`

- Based on [`pangeo/base-image`](https://github.com/pangeo-data/pangeo-docker-images)
    - Smarter(?) setup suitable for Pangeo Jupyter & Binder Hubs (+ Dask gateway), & Repo2Docker.  
    - Better organisation around the conda environment, for reproducability (via `conda-lock`) & customisability  
    - NB: pip packages specified in `requirements.txt` which are installed after the conda(-lock) packages, so break reproducability
- Our Python environment is in the `notebook` conda environment, and managed in Jupyter with `nb_conda_kernels`
- Things needing fortran compiler (e.g. apexpy) wouldn't work even though it is present in the conda  (TBC...)

```
cd pangeo-swarm
# Update the lock with new packages
conda-lock -f environment.yml -p linux-64
# Build image
# - Uses `conda-linux-64.lock`, or, if not available, uses `environment.yml` directly
# - Uses requirements.txt for pip packages
docker build . -t pangeo-swarm
docker run --rm -it -t pangeo-swarm bash
docker run --rm -it -p 8890:8890 -v XXXXX:/home/jovyan -t pangeo-swarm jupyter lab --port 8890 --ip 0.0.0.0
```

## `swarm-dashboards`

A docker image to directly serve a panel dashboard, given the VirES access token as `$VIRES_TOKEN`.
```
cd swarm-dashboards
docker build . -t swarm-dashboards
docker run --rm -it -e "VIRES_TOKEN=$VIRES_TOKEN" -p 5006:5006 swarm-dashboards
```

## TODO

- Containers:
    - Extend with more swarm/geomag/spacephys packages; track problems where they arise
    - Find and fix installation of compile-dependent packages in `pangeo-swarm`
    - See if it's reasonable to move to pangeo as a dependable base in general
    - Test & instructions for Singularity images with these as base - or will it motivate a rewrite for something more "singularity-native"?
- Dashboards:
    - Find out how to serve a set of notebooks from `swarm-dashboards`
    - Options for how to serve to the world
      - shared access to a common panel server (shared VirES auth)
      - through existing JHub (see [jupyter-panel-proxy](https://github.com/holoviz/jupyter-panel-proxy)) - lose benefit of panel state caching