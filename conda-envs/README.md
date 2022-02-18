# Using conda environments

Assumes using Linux

## A good base environment:

First install mambaforge (https://github.com/conda-forge/miniforge#mambaforge):
```bash
curl -L -O "https://github.com/conda-forge/miniforge/releases/latest/download/Mambaforge-$(uname)-$(uname -m).sh"
bash Mambaforge-$(uname)-$(uname -m).sh
```
(or you can continue with conda or mamba if already installed)

Utilities to install in the base environment:
```bash
mamba install pipx
```

Utilities to install as *applications* using pipx:
```bash
pipx install nox pre-commit
```

## A development environment

`lab.yml` defines an environment containing `jupyterlab` & `code-server`, along with some extensions to improve JupyterLab and enable interactivity with some packages. Create this environment:
```bash
mamba env create --file lab.yml --name lab
```
(change the name how you like)

## Extending the development environment with packages you need

It is possible to create separate conda environments for different purposes, and the JupyterLab running inside `lab` will see these and you can connect notebooks to appropriate kernels. That is a good approach in order to have cleaner dependency management (an environment with an enormous list of packages can be difficult to work with if you need to change specific package versions, and is overly complicated to reproduce).

However, that doesn't work well if you want to use interactive elements in notebooks (which rely on JupyterLab extensions as well as tools within the running kernel). So, instead of creating separate environments, here I merge in my requirements into the `lab` environment:
```bash
mamba env update --name lab --file swarm.yml
```
This is mostly the same as the VRE packages, excluding eoxmagmod. The versions will be different though.

The resulting environment (with specific versions used) is stored in `swarmlab.yml` (from `mamba env export --file swarmlab.yml`). So you could just install that directly:
```bash
mamba env create --file swarmlab.yml --name swarmlab.yml
```