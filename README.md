# Setting up Python Computing Interface with Micromamba

This page describes how to set up computing environments on the FHCC cluster, particularly focused on python.
We use micromamba, see manual here: "https://mamba.readthedocs.io/en/latest/user_guide/micromamba.html", which is a lightweight package manager that is employed in a similar fashion as conda.

To install micromamba, use the following command: ```"${SHELL}" <(curl -L micro.mamba.pm/install.sh)```
This automatically installs micromamba in the directory ```~/.local/bin```.  Make sure directory is in ```PATH``` environment variable in ```~/.bash_profile```. Can use the following code:
```
# >>> mamba initialize >>>
# !! Contents within this block are managed by 'micromamba shell init' !!
export MAMBA_EXE='/home/ayeh/.local/bin/micromamba'; # Defines full path to mamba executable
export MAMBA_ROOT_PREFIX='/home/ayeh/micromamba'; # Dictates where environments and packages are stored
__mamba_setup="$("$MAMBA_EXE" shell hook --shell bash --root-prefix "$MAMBA_ROOT_PREFIX" 2> /dev/null)"
if [ $? -eq 0 ]; then
    eval "$__mamba_setup"
else
    alias micromamba="$MAMBA_EXE"  # Fallback on help from micromamba activate
fi
unset __mamba_setup
# <<< mamba initialize <<<
```

## Micromamba environments
To create a new environment, type ```micromamba create -n ENV_NAME python```.

To activate environment, type ```micromamba activate ENV_NAME```.

To list available environments, type in ```micromamba env list```.

## Micromamba channels
To list channels used, type in ```micromamba config list```

Note that FHCC cannot install from conda-forge directly, and we have to use in-house version, which include:
```https://conda-forge.fredhutch.org/bioconda/```
```https://conda-forge.fredhutch.org/conda-forge/```

To install the proper channels, type in: 
```
conda config --remove channels defaults # Removes existing channels
micromamba config set channel_alias https://conda-forge.fredhutch.org --file ~/.mambarc
micromamba config set channel_alias https://conda-forge.fredhutch.org --file ~/.condarc
conda config --add channels conda-forge
conda config --add channels bioconda
```

Can change ```~/.mambarc``` (runtime configuration file to dictate what channel to install packages from). In YAML format:
```
channel_alias: https://my.local.repo/
channels:
  - local
  - conda-forge
channel_priority: strict
```

To install a package into a channel.  First, load the channel, than install package:
``` 
micromamba activate ENV_NAME
micromamba install -c conda-forge opencv
```
Note that this will install from the directory "[channel_alias] / [channel_name] / [platform] / repodata.json", e.g. 
```https://conda-forge.fredhutch.org/conda-forge/linux-64/repodata.json```, where the "repodata.json" is a compressed index of all available packages.


## Jupyter notebook environment setup

Create the following .sh script below and run it on the computing cluster with sbatch.  This should give you a node name that you can then paste into the following URL to access the jupyter environment:
```
http://[node name].fhcrc.org:25032 # Put random port number here that is in the .sh script below
```

```
#!/bin/bash -
#SBATCH --job-name=jupyter_server
#SBATCH --output=/home/%u/jupyter_logs/%x_job-%j_%N.log
#SBATCH --mem=32G
#SBATCH --cpus-per-task=1
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --time=1-00:00:00

source $HOME/.bash_profile

# Use the first command line argument as conda environment name
envName="${1:-jupyterenv}"
echo "Using the conda environment $envName"

# make sure micromamba is enabled
eval "$(micromamba shell hook --shell=bash)"

# enter the environment
micromamba activate "$envName"

# make sure the module system is enabled
source /etc/profile.d/modules.sh

# If SLURM is managing CPUs, limit threads for numpy to optimize performance
if [[ -n $SLURM_CPUS_PER_TASK ]]; then
    export MKL_NUM_THREADS=$SLURM_CPUS_PER_TASK
    export NUMEXPR_MAX_THREADS=$SLURM_CPUS_PER_TASK
    export NUMEXPR_NUM_THREADS=$SLURM_CPUS_PER_TASK
    export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK
fi

export JUPYTER_ALLOW_INSECURE_WRITES=1 # Seems to fix a weird bug

# Navigate to the home directory
cd "$HOME"

# Start JupyterLab on the designated port number
jupyter lab --ip=$(hostname) --port=25032 --no-browser

```





