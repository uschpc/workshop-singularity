---
title: Software Containers with Singularity
author: Center for Advanced Research Computing <br> University of Southern California
date: 2021-06-07
---


## Outline

1 --- Overview and motivation  
2 --- Getting pre-built container images  
3 --- Building custom container images  
4 --- Running container images


## 1 --- Overview and motivation


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


## Singularity architecture

<img src="singularity-architecture.png" width="700" />

Source: [Singularity Community and SingularityPRO on high-performance servers](https://sylabs.io/assets/white-papers/Sylabs_Whitepaper_High_performance_server_v3.pdf)


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


## Getting and running images

- Getting images
  - Pull pre-existing images from container registries
  - Download images from software websites
  - Build your own custom image
- Running images
  - Run images in interactive or batch modes
  - Various options can be used to isolate/integrate with host system


## 2 --- Getting pre-built container images


## Pulling existing container images

- Pull from container registries
- [Singularity Cloud Library](https://cloud.sylabs.io/library)

```
singularity pull library://ubuntu:latest
```

- [Docker Hub](https://hub.docker.com/)

```
singularity pull docker://clearlinux:latest
```


## Exercise 1

Pull a container image from a registry


## 3 --- Building custom container images


## Building externally

- Need to build outside of CARC systems (requires superuser privileges)
- But need a Linux system
- Best approach is to use the cloud-based [Singularity Remote Builder](https://cloud.sylabs.io/home)
- Or use a virtual machine on your local computer (e.g., [Multipass](https://multipass.run/), [Virtual Box](https://www.virtualbox.org/))
- Alternatively, create a Docker container image locally, upload to Docker Hub, and then convert to Singularity format


## Building workflow

1. Create a definition file
2. Build image externally using the definition file
3. If error occurs, modify definition file and rebuild (back to step 1)
4. Transfer image file to CARC systems
5. Test image
6. If error occurs, modify definition file and rebuild (back to step 1)


## Definition files

- Recipe for building a container image
- Start with a base Linux OS (e.g., Debian, Ubuntu, CentOS, Clear Linux)
- Or start with existing container image
- Then install software, add files, etc.
- Similar to a Dockerfile, but different syntax


## Structure of a definition file

```
# Header (required)

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
    apt-get -y update
    apt-get -y upgrade
    apt-get -y install wget
    apt-get -y autoremove --purge
    apt-get -y clean
  
    cd /opt
    wget -q https://julialang-s3.julialang.org/bin/linux/x64/1.6/julia-1.6.1-linux-x86_64.tar.gz
    tar -xf julia-1.6.1-linux-x86_64.tar.gz
    rm julia-1.6.1-linux-x86_64.tar.gz

    mkdir /opt/juliadeot
    export JULIA_DEPOT_PATH=/opt/juliadepot
    /opt/julia-1.6.1/bin/julia -e 'using Pkg; Pkg.add("StatsKit")'

%test
    export PATH=/opt/julia-1.6.1/bin:$PATH
    julia --version

%environment
    export LC_ALL=C
    export PATH=/opt/julia-1.6.1/bin:$PATH
    export JULIA_DEPOT_PATH=/opt/juliadepot

%runscript
    julia

%help
    Ubuntu 20.04 with Julia 1.6.1 and the JuliaStats packages.
```


## Example definition file 2

```
Bootstrap: docker
From: clearlinux/r-base:4.0.5

%post
    Rscript -e 'install.packages("data.table", repos = c("https://cloud.r-project.org"))'

%test
    R --version

%environment
    export LC_ALL=C

%runscript
    R

%help
    Based on the Clear Linux r-base:4.0.5 Docker image. Base R 4.0.5 built with OpenBLAS. Packages installed: data.table
```


## Using Singularity Remote Builder

- [Singularity Container Services](https://cloud.sylabs.io/home)
- Free service with up to 11 GB of storage
- Log in with other account (GitHub, GitLab, Google, Microsoft)
- Set up access token on CARC systems
- Use web GUI interface to build


## Using Singularity Remote Builder (continued)

- Can also build remotely from the command line
- Builds image remotely and automatically downloads to current directory

```
singularity build --remote julia.sif julia.def
```


## Exercise 2

Create a definition file for a custom container image  
Use the Remote Builder to build the custom image  
Transfer the image to CARC systems  
[Some template definition files](https://github.com/uschpc/singularity)


## 4 --- Running container images


## Three commands to run images

- `singularity shell` &mdash; for an interactive shell within the container
- `singularity exec` &mdash; for executing commands within the container
- `singularity run` &mdash; for running a pre-defined runscript within the container
- A container process is like any other Linux process
- Just a different software environment


## Shell example

```
$ singularity shell julia.sif
Singularity> pwd
/home1/ttrojan/singularity
Singularity> exit
exit
$
```


## Execute example

```
$ singularity exec julia.sif julia -e 'println("Hello world")'
Hello world
```


## Runscript example

```
$ singularity run julia.sif
              _
   _       _ _(_)_     |  Documentation: https://docs.julialang.org
  (_)     | (_) (_)    |
   _ _   _| |_  __ _   |  Type "?" for help, "]?" for Pkg help.
  | | | | | | |/ _` |  |
  | | |_| | | | (_| |  |  Version 1.6.1 (2021-04-23)
 _/ |\__'_|_|_|\__'_|  |  Official https://julialang.org/ release
|__/                   |

julia>
```

Inspect runscripts:

```
$ singularity inspect --runscript julia.sif
#!/bin/sh

    julia
```


## Exercise 3

Run the command `echo 'Hello world'` within a container


## Bind mounting directories to containers

- Some directories are automatically mounted to the container
  - your `/home1` directory
  - `/tmp` or `TMPDIR`
- Use the `--bind` option for additional directories
- For example, to add your current working directory and `/scratch` directory:

```
singularity exec --bind $PWD,/scratch/<username> julia.sif julia script.jl
```


## Other useful options

- Often a good idea to use `--cleanenv` (or shorter `-e`)
- May need to use `--no-home` to exclude `/home1` directory (e.g., for Python, R, Julia containers)


## Exercise 4

Bind mount your `/scratch` directory to a container  
Show that it mounted correctly by listing files in that directory


## Running containers on GPUs

- Containers need to access host GPU driver
- Use `--nv` option to allow container access to driver
- Run `nvidia-smi` on GPU node to see current driver version and compatibility
- [https://sylabs.io/guides/3.7/user-guide/gpu.html](https://sylabs.io/guides/3.7/user-guide/gpu.html)
- Example for TensorFlow runscript:

```
singularity run --cleanenv --nv tf.sif
```


## Running containers with MPI

- Two approaches: hybrid vs. mount
- Pros and cons for each approach
- Less portable
- [https://sylabs.io/guides/3.7/user-guide/mpi.html](https://sylabs.io/guides/3.7/user-guide/mpi.html)
- Example for OpenMPI program:

```
srun --mpi=pmix_v2 -n $SLURM_NTASKS singularity exec openmpi.sif mpi_program.x
```


## Singularity environment variables

- Some useful environment variables can be set
- Add to `~/.bashrc` to automatically set every time you log in
- For example, change cache directory from `/home1` to `/scratch`:

```
export SINGULARITY_CACHEDIR=/scratch/<username>/.singularity
```

- For example, add certain bind paths to every container:

```
export SINGULARITY_BIND=/scratch/<username>,/project/<project_id>
```


## Singularity documentation

- [Official documentation](https://sylabs.io/guides/latest/user-guide/)  

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

- [Using Singularity on CARC Systems](https://carc.usc.edu/user-information/user-guides/software-and-programming/singularity)
- [CARC Singularity template definition files](https://github.com/uschpc/singularity)
- [Singularity website](https://sylabs.io/singularity/)
- [Singularity documentation](https://sylabs.io/guides/latest/user-guide/)
- [Singularity tutorial](https://singularity-tutorial.github.io/)
- [Singularity Remote Builder](https://cloud.sylabs.io/home)
- [Singularity Cloud Library](https://cloud.sylabs.io/library)
- [Docker Hub](https://hub.docker.com/)
- [BioContainers](https://biocontainers.pro)
- [NVIDIA GPU Cloud Catalog](https://ngc.nvidia.com/catalog)


## Getting help

- [Submit a support ticket](https://carc.usc.edu/user-information/ticket-submission)
- [User Forum](https://hpc-discourse.usc.edu/)
- Office Hours
  - Every Tuesday 2:30-5pm (currently via Zoom)
  - Register [here](https://carc.usc.edu/news-and-events/events)
