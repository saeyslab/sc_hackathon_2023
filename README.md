# sc_hackathon_2023
Ghent hackathon on Python/R interoperability for single cell and spatial bioinformatics

## Installation

Simultaneous development of Python and R packages be tricky, as they have different packages managers and practices. Below is an example workflow to setup an Python and R conda environment to get the code examples up and running. VS Code is presumed as IDE, but Jupyter Lab and RStudio also support conda environments.

- If you have Windows, you can use the [Windows Subsystem for Linux](https://learn.microsoft.com/en-us/windows/wsl/install) to install a Linux distribution. You can then use the Linux instructions below.
- If you have a Macbook with M1 chip, you can use the instructions at the bottom of this page.
- If you have Linux, install [conda](https://conda.io/projects/conda/en/latest/user-guide/install/index.html) (or mamba for faster dependency resolving)
```
conda env update -f env_mixed.yaml
conda activate ghent_hackathon_mixed
```

## Usage

In the `examples` folder, there are some examples of how to use the packages. They're either Jupyter notebooks `.ipynb` or [Quarto](https://quarto.org/) `.qmd`. You can open the notebooks using Jupyter Lab or VS Code. You can open the Quarto files using VS Code or RStudio. You can convert between the two using the `quarto convert` [command line tool](https://quarto.org/docs/tools/vscode-notebook.html).

## Working with conda kernels

You can use the `conda` environment as a kernel in Jupyter Lab or VS Code. This will allow you to use the environment in a notebook. You can also use the `conda` environment as a kernel in RStudio. Make sure the environment you want to use has a kernel installed such as `ipykernel` or `r-irkernel`. VS Code should pick up on the kernel, especially if you do a `Reload Window`. You can also use Jupyter Lab and [nb_conda_kernels](https://github.com/Anaconda-Platform/nb_conda_kernels) in the main environment.

You can use quarto within the conda environment, but using quarto to access an external conda environment requires [an extra configuration step](https://quarto.org/docs/computations/python.html#kernel-selection).

## Working with git development branches

When hacking or patching external packages, try to use a local development branch. This will make it easier to track changes and merge them back to the original package. Fork the package repository and clone it to a different location. Apply your changes and install the package in editable mode. For example:
```python
pip install -e ../anndata
```

For R, you can use the `devtools` package to install a package from a local directory:
```R
devtools::install_local("../anndataR")
``` 

To share your changes, you can push them to your forked repository and create a pull request. You can reference the pull request in your environment file using the `pip` section. For R packages, you can add install devtools install instructions at the top of the script.
```R
if (!requireNamespace("devtools", quietly = TRUE))
    install.packages("devtools")
if (!requireNamespace("anndataR", quietly = TRUE))
    devtools::install_github("scverse/anndataR@issue-82-nullable-vectors-and-boolean-enums")
```

## Macbook ARM support
If you want to run on a Macbook with M1 chip, you need to do one of the following if you want to use Conda and R packages (as [Bioconda does not yet support ARM on Mac](https://github.com/bioconda/bioconda-recipes/issues/23454)):
1. SLOW: use a Linux virtual machine (e.g. [UTM and VS Code](https://medium.com/@lizrice/linux-vms-on-an-m1-based-mac-with-vscode-and-utm-d73e7cb06133))
2. WORKS: use a remote Linux workstation (this can have problems if you want to use graphical interfaces such as RStudio, napari, plotting...)
3. FAILS: install using the `env_mixed_arm.yaml` (without bioconda channel) instead of `env_mixed.yaml`. The BioCManager will then manually install all remaining packages (>5min).
4. FAILS: install using the `env_mixed.yaml`, but emulate the x86 architecture using [Rosetta](https://support.apple.com/en-us/HT211861). This will but will be faster than option 2, but hacking low level packages could give wrong architecture errors due to the emulation.
    ```
    CONDA_SUBDIR=osx-64 conda env update -f env_mixed.yaml
    ```
5. WORKS: Use docker with an RStudio server inside it, like e.g. [here](https://neurogenomics.github.io/MAGMA_Celltyping/articles/docker.html).
6. FAILS: use pixi (see below) to resolve the R/Python environment. This also uses the conda packages, so will have similar issues.

When using option 4, you can test you're actively using Rosetta using a `python` shell after environment activation:
```python
import platform, subprocess
platform.processor() # should return 'i386' if using Rosetta
subprocess.run(["sysctl", "-n", "sysctl.proc_translated"]) # should return 1 if using Rosetta
```


## Using conda kernels in Pixi

Install [pixi](https://prefix.dev)
```bash
# OR create pixi.toml and install deps
pixi init
pixi add ...

# OR start from pixi.toml
pixi install

# setup kernels
## find out jupyter PATH
which jupyter
## install kernel
PATH=$PATH:/srv/scratch/benjaminr/anaconda3/bin pixi run R
IRkernel::installspec(name = 'ir43pixi', displayname = 'R 4.3 pixi')

## check if install
jupyter kernelspec list

## VS Code Reload, should be able to select the kernel now
```