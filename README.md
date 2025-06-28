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
micromamba config set channel_alias https://conda-forge.fredhutch.org --file ~/.mambarc
micromamba config set channel_alias https://conda-forge.fredhutch.org --file ~/.condarc
conda config --add channels conda-forge
conda config --add channels bioconda
```



