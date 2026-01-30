# Conda

I'll be using minconda, even though the differences should be investigated.
Basically it's an environment, a bit similar to venv bu much bigger, one can 
also change the version of python and other software. Basically if you are
in a conda environment when you call a command first the conda enviroment
gets searched and afterwards the parent one (I think, maybe not even always, 
probably depends on the command)

## Commands

- `conda create --name new-env-cloned --clone env-to-be-cloned` creates an env by cloning the target one
- `conda env create -f environment.yml` create an environment from .yml file
- `conda env list` lists the environments and `*` on the current one
- `conda env remove <env-to-be-removed>` deletes and environment
- `conda rename -n <current-env-name> <new-env-name>`

## Conda vs Mamba

Apparently you can also install packages in your conda environment with mamba, no idea what the difference
could be
