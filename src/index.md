---
title: Software Containers with Singularity
author: Derek Strong <br> dstrong[at]usc.edu <br> Research Computing Associate <br> CARC at USC <br>
date: 2021-05-07
---


## Outline

- Software containers
- Getting pre-built container images
- Building your own container images
- Running container images


## What are software containers?

- Isolated, secure, stable, portable, and reproducible software environments
- Packages main application and all dependencies
- OS-level virtualization
- Provides a custom user space
- Different from a virtual machine
  - Containers have direct access to the kernel
  - Virtual machines have indirect access (performance penalty)


## Singularity containers

- A container format designed for shared HPC systems
- Most popular container format for HPC
- Can convert from Docker to Singularity format


## Why use Singularity for research?

- Install anything you want (based on any Linux OS)
- Ease installation issues by using pre-built container images
- Ensure the same software stack is used among a research group
- Use the same software stack across Linux systems (e.g., any HPC center)
- Run the same workflows across Linux systems by embedding runscripts in container images


## Some specific Singularity use cases

- Converting Docker images to Singularity images
- Using applications that need older or cutting-edge or very specific versions of software
- Bundling complex software stacks for easier distribution
- Using proprietary software (typically distributed as binaries) that depends on other software not available on host system


## Some limits of Singularity

- Built for Linux systems
- Portability depends on a few factors
  - CPU architecture format (x86 vs. ARM)
  - Binary format (ELF)
  - Kernel, glibc, other API compatibility
- Not always backwards compatible
- Need superuser privileges to build images


## The container image

- A single image file (executable `.sif`)
- Bundles application and all software dependencies needed to run it
- Intended to be immutable
- If need to modify, rebuild
- A container is a running instance of a container image


## Getting Singularity container images

- Pull pre-existing images from container registries
- Download images from software websites
- Build your own image


## Pulling existing container images

- Container registries:
  - [Singularity Library](https://cloud.sylabs.io/library)
    - Example: `singularity pull library://ubuntu:latest`
  - [Docker Hub](https://hub.docker.com/)
    - Example: `singularity pull docker://alpine:latest`


## Building your own container images

- Need to build outside of CARC systems (requires superuser privileges)
- But need a Linux system
- Best approach is to use the cloud-based [Singularity remote builder](https://cloud.sylabs.io/home)
- Or use a virtual machine on your local computer (e.g., [Multipass](https://multipass.run/), [Virtual Box](https://www.virtualbox.org/))
- General workflow for CARC systems:
  1. Build externally using a definition file
  2. Transfer image file to CARC systems
  3. Run container image


## Definition files

- Recipe for building a container image
- Start with a base Linux OS (e.g., Debian, Ubuntu, CentOS, Clear Linux)
- Or start with existing container image
- Then install software, add files, etc.


## Structure of a definition file

```
# Header

Bootstrap: ...
From: ...

# Sections (optional)

%files
    Copy files to the container from host (/source /destination)

%post
    Install software and libraries, write configuration files, download files from the internet, etc.

%test
    Run commands to validate build process

%environment
    Define environment variables that will be set at runtime

%startscript
    Run commands when singularity instance start command is used

%runscript
    Run commands when singularity run command is used    

%labels
    Add metadata labels (name-value pair)

%help
    Describe the container and its intended use
```


## Example definition file

```
Bootstrap: library
From: ubuntu:20.04

%post
    # Install required software packages
    apt-get -y update
    apt-get -y upgrade
    apt-get -y install wget
    apt-get -y autoremove --purge
    apt-get -y clean

    # Download and extract Julia binary
    cd /opt
    wget -q https://julialang-s3.julialang.org/bin/linux/x64/1.6/julia-1.6.1-linux-x86_64.tar.gz
    tar -xf julia-1.6.1-linux-x86_64.tar.gz
    rm julia-1.6.1-linux-x86_64.tar.gz

    # Add Julia packages
    julia -e 'using Pkg; Pkg.add.(["StatsBase", "StatsModels", "DataFrames", "Distributions"])'

%test
    export PATH=/opt/julia-1.6.1/bin:$PATH
    julia --version

%environment
    export LC_ALL=C
    export PATH=/opt/julia-1.6.1/bin:$PATH

%runscript
    julia

%help
    This container runs Julia 1.6.1 with several statistical packages on Ubuntu 20.04
```


## Using Singularity remote builder

- Free service with up to 11 GB of storage
- Log in with other account (GitHub, GitLab, Google, Microsoft)
- [https://cloud.sylabs.io/home](https://cloud.sylabs.io/home)


## Running Singularity containers

- Three commands:
  - `singularity shell` &mdash; for an interactive shell within the container
  - `singularity exec` &mdash; for executing commands within the container
  - `singularity run` &mdash; for running a pre-defined runscript within the container
- A container process is like any other Linux process
- Just a different software environment


## Useful options

- Often a good idea to use `--cleanenv` (or shorter `-e`)
- May need to use `--no-home` (e.g., for Python, R, Julia containers)
- Use `--bind` to bind mount directories on host to the container


## Running containers on GPUs

- Containers need to access host GPU driver
- Use `--nv` option to allow container access to driver
- Run `nvidia-smi` on GPU node to see current driver version and compatibility
- [https://sylabs.io/guides/3.6/user-guide/gpu.html](https://sylabs.io/guides/3.7/user-guide/gpu.html)
- Example for TensorFlow runscript:

```
singularity run --cleanenv --nv tf.sif
```


## Running containers with MPI

- Two approaches: hybrid vs. mount
- Pros and cons for each approach
- [https://sylabs.io/guides/3.6/user-guide/mpi.html](https://sylabs.io/guides/3.7/user-guide/mpi.html)
- Example for OpenMPI program:

```
srun --mpi=pmix_v2 -n $SLURM_NTASKS singularity exec <container_image> mpi_program.x
```


## Singularity environment variables

- Can be useful to set environment variables (add to `~/.bashrc`)
- Change cache directory from home to scratch

```
export SINGULARITY_CACHEDIR=/scratch/<username>/.singularity
```

- Can also set bind paths:

```
export SINGULARITY_BIND=/scratch/<username>,/project/<project_id>
```


## Getting help

- [Singularity documentation](https://sylabs.io/guides/latest/user-guide/)  

```
$ singularity help

Linux container platform optimized for High Performance Computing (HPC) and
Enterprise Performance Computing (EPC)

Usage:
  singularity [global options...]

Description:
  Singularity containers provide an application virtualization layer enabling
  mobility of compute via both application and environment portability. With
  Singularity one is capable of building a root file system that runs on any
  other Linux system where Singularity is installed.

Options:
  -c, --config string   specify a configuration file (for root or
                        unprivileged installation only) (default
                        "/etc/singularity/singularity.conf")
  -d, --debug           print debugging information (highest verbosity)
.
.
.
```


## Additional resources

- [CARC User Guide for Singularity](https://carc.usc.edu/user-information/user-guides/software-and-programming/singularity)
- [CARC Singularity template definition files](https://github.com/uschpc/singularity)  
- [Singularity website](https://sylabs.io/singularity/)  

- [Singularity remote builder](https://cloud.sylabs.io/home)  
- [Singularity tutorial](https://singularity-tutorial.github.io/)  
- [Singularity Library](https://cloud.sylabs.io/library)  
- [Docker Hub](https://hub.docker.com/)  
- [BioContainers](https://biocontainers.pro)  
- [NVIDIA GPU Cloud Catalog](https://ngc.nvidia.com/catalog)
