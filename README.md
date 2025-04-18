<h1>CTPO: CUDA + TensorFlow + PyTorch + OpenCV Docker containers</h1>

Latest release: 20241219

`Dockerfile`s to build containers with support for CPU and GPU (NVIDIA CUDA) containers with support for TensorFlow, PyTorch and OpenCV (or combinations of), based on `nvidia/cuda` and Ubuntu 22.04 container images.

The tool's purpose is to enable developers, ML and CV enthusiasts to build and test solutions `FROM` a docker container, allowing fast prototyping and release of code to the community.

We have published examples of use of the tool to support efforts in Computer Vision, Data Science, Machine Learning and Natural Language Processing.
Go to https://github.com/Infotrend-Inc/CTPO-Demo_Projects for details.

- [1. Builds and Notes](#1-builds-and-notes)
  - [1.1. Tag naming conventions](#11-tag-naming-conventions)
  - [1.2. Building](#12-building)
    - [1.2.1. A note on build: time \& space](#121-a-note-on-build-time--space)
    - [1.2.2. Building (using Makefile)](#122-building-using-makefile)
    - [1.2.3. Building (using Dockerfile)](#123-building-using-dockerfile)
  - [1.3. Available builds on DockerHub](#13-available-builds-on-dockerhub)
    - [1.3.1. TensorRT build](#131-tensorrt-build)
  - [1.4. Build Details](#14-build-details)
  - [1.5. Jupyter build](#15-jupyter-build)
  - [1.6. Unraid build](#16-unraid-build)
- [2. Usage and more](#2-usage-and-more)
  - [2.1. A note on supported GPU in the Docker Hub builds](#21-a-note-on-supported-gpu-in-the-docker-hub-builds)
  - [2.2. Using the container images](#22-using-the-container-images)
  - [2.3. Podman usage](#23-podman-usage)
  - [2.4. docker compose](#24-docker-compose)
  - [2.5. virtualenv for shared Jupyter Lab usage](#25-virtualenv-for-shared-jupyter-lab-usage)
  - [2.6. Disabling GPUs when nvidia is the default Docker runtime](#26-disabling-gpus-when-nvidia-is-the-default-docker-runtime)
- [3. Version History](#3-version-history)


The CTPO (CUDA + TensorFlow + PyTorch + OpenCV) project aims to address the following challenges and provide solutions:
- **Containerized Development Environment**: CTPO offers Docker containers with pre-configured environments containing CUDA, TensorFlow, PyTorch, and OpenCV. This allows developers to work within a consistent and isolated environment.
- **Fast Prototyping and Testing**: The project facilitates fast prototyping and testing by providing pre-built containers. Developers can quickly iterate on their code within the containerized environment.
- **Versioned Frameworks and Dependencies**: The project uses versioned Docker containers, making it easier for developers to work with specific versions of TensorFlow, PyTorch, OpenCV, and other components.
- **Jupyter Lab Integration**: CTPO includes Jupyter Lab builds, allowing developers to use a web-based interface for interactive development, visualization, and documentation.

Building each container independently is made possible by the `Dockerfile` available in the `BuildDetails/<release>/<container>-<tag>` directories.
Building each container takes resources and time (counted in many cores, GB of memory and build hours).

Pre-built containers are available from Infotrend Inc.'s Docker account at https://hub.docker.com/r/infotrend/
Details on the available container and build are discussed in this document.

A Jupyter Lab and Unraid version of this WebUI-enabled version are also available on our Docker Hub, as well as able to be built from the `Makefile`.

Note: this tool was built earlier in 2023, iterations of its Jupyter Lab were made available to Infotrend's data scientists, and we are releasing it to help the developer community.

#  1. Builds and Notes

The base for those container images is pulled from Dockerhub's official `ubuntu:22.04` or `nvidia/cuda:[...]-devel-ubuntu22.04` images. 

More details on the Nvidia base images are available at https://hub.docker.com/r/nvidia/cuda/
In particular, please note that "By downloading these images, you agree to the terms of the license agreements for NVIDIA software included in the images"; with further details on DockerHub version from https://docs.nvidia.com/cuda/eula/index.html#attachment-a

For GPU-optimized versions, we will need to build the `cuda_` versions on a host with the supported hardware.
When using GPU and building the container, we need to install the NVIDIA Container Toolkit found at https://github.com/NVIDIA/nvidia-container-toolkit
Note that the NVIDIA video driver on the Linux host needs to support the version of CUDA that we are trying to build (we can see the supported CUDA version and driver version information when running the `nvidia-smi` command)

For CPU builds, simply build the non-`cuda_` versions.

Pre-built images are available for download on Infotrend's DockerHub (at https://hub.docker.com/r/infotrend/). 
Those are built using the same method provided by the `Makefile` and the corresponding `Dockerfile` used for those builds is stored in the matching `BuildDetails/<release>/<container>-<tag>` directory.

##  1.1. Tag naming conventions

The tag naming convention follows a `_`-components split after the base name of `infotrend/ctpo-` followed by the "release" tag (Docker container images are always lowercase).
`-` is used as a feature separator, in particular for `jupyter` or `unraid` specific builds.
Any `cuda_` build is a `GPU` build while all non-`cuda_` ones are `CPU` only.

For example, for `infotrend/ctpo-tensorflow_pytorch_opencv:2.12.0_2.0.1_4.7.0-20231120`, this means: `"base name"-"component1"_"compoment2"_"component3":"component1_version"_"component2_version"_"component3_version"-"release"` with:
- `base name`=`infotrend/ctpo-`
- `component1` + `component1_version` = `tensorflow` `2.12.0`
- `component2` + `component2_version` = `pytorch` `2.0.1`
- `component3` + `component3_version` = `opencv` `4.7.0`
- `release`=`20231120`
As such, this was "Infotrend's CTPO release 20231120 with TensorFlow 2.12.0, PyTorch 2.0.1, and OpenCV 4.7.0 without any CUDA support." 
(Since no `cuda_` was part of the name, this is a `CPU` build)

Similarly, `infotrend/ctpo-jupyter-cuda_tensorflow_pytorch_opencv-unraid:11.8.0_2.12.0_2.0.1_4.7.0-20231120` can be read as:
- `base name`=`infotrend/ctpo-`
- `feature1` = `jupyter`
- `component1` + `component1_version` = `cuda` `11.8.0`
- `component2` + `component2_version` = `tensorflow` `2.12.0`
- `component3` + `component3_version` = `pytorch` `2.0.1`
- `component4` + `component4_version` = `opencv` `4.7.0`
- `feature2` = `unraid`
- `release`=`20231120`
 "Infotrend's CTPO release 20231120 with a Jupyter Lab and Unraid specific components with PyTorch 2.0.1, OpenCV 4.7.0 and GPU (CUDA) support."

There will be a variable number of components or features in the full container name as shown above. 
It is left to the end user to follow the naming convention.

## 1.2. Building

There are two methods to build local containers:
1. when building multiple versions of the container or to have a dedicated Docker `buildx` builder used; we recommend using the `Makefile`
2. when building a specific version of the container; use the matching  `BuildDetails/<RELEASE>/<COMPONENTS>-<COMPONENTS_VERSIONS>-<RELEASE>/Dockerfile`

When using the `Makefile` method: we will use the `make` command to generate the content of the `BuildDetails` directory which will place all files required for building the container within that directory. For a `20241219` release, for `pytorch_opencv` a `BuildDetails/20241219/pytorch_opencv-2.5.1_4.10.0-20241219` directory (following the naming convention) is created and contains the build artifacts including a `Dockerfile` 

When using the `BuildDetails/<RELEASE>/<COMPONENTS>-<COMPONENTS_VERSIONS>-<RELEASE>/Dockerfile` method: we can build the container using the files within that directory.

### 1.2.1. A note on build: time & space

Building the containers requires the docker build step to have internet access. The process is CPU and Memory intensive and will require storage space for the build and the final image.
For reference, on a system with an AMD 5950x (16-cores, ie `NUMPROC=32`), 128GB of memory and NVMe-based storage, the build time of `make build_tpo build_ctpo build_ctpo_tensort` (for the `20241219` release) --ie making use of layers caching when possible-- was about 60 minutes for the `tensorflow_opencv` or `pytorch_opencv` containers (about 7GB each), about 80 minutes for `tensorflow_pytorch_opencv` (about 8GB), close to 130 minutes (and a 17GB container) for `cuda_tensorflow_opencv`, over 200 minutes (also 17GB) for `cuda_pytorch_opencv`, over 220 minutes (for a 19GB container) for  `cuda_tensorflow_pytorch_opencv`. The `TensorRT` variant took 240 minutes to build and its image size is close to 30GB.

When generating all images, there is about 100GB of reclaimable space generated by Docker `buildx`.


###  1.2.2. Building (using Makefile)

This method is a per-release method, as it will generate a build specific `Dockerfile` 

Type `make` to get the list of targets and some details of the possible builds.

It is possible to adapt the content of the `Makefile` to build custom solutions. For example, the default is not to build OpenCV non-free or build FFmpeg with libnpp, as those would make the images unredistributable and any release on our Dockerhub is made with "redistributable" packages.

We will see the result of this command for the `20241219` release:

```
**** Docker Image tag ending: 20241219
**** Docker Runtime: GPU
  To switch between GPU/CPU: add/remove "default-runtime": "nvidia" in /etc/docker/daemon.json then run: sudo systemctl restart docker

*** Available Docker images to be built (make targets):
  build_tpo (requires CPU Docker runtime):
    tensorflow_opencv OR pytorch_opencv OR tensorflow_pytorch_opencv (aka TPO, for CPU):
      tensorflow_opencv-2.18.0_4.10.0
      pytorch_opencv-2.5.1_4.10.0
      tensorflow_pytorch_opencv-2.18.0_2.5.1_4.10.0
  build_ctpo (requires GPU Docker runtime):
    cuda_tensorflow_opencv OR cuda_pytorch_opencv OR cuda_tensorflow_pytorch_opencv (aka CTPO, for NVIDIA GPU):
      cuda_tensorflow_opencv-12.5.1_2.18.0_4.10.0
      cuda_pytorch_opencv-12.5.1_2.5.1_4.10.0
      cuda_tensorflow_pytorch_opencv-12.5.1_2.18.0_2.5.1_4.10.0
  build_ctpo_tensorrt (requires GPU Docker runtime):
    cuda_tensorflow_pytorch_opencv (aka CTPO, for NVIDIA GPU with TensorRT): same as cuda_tensorflow_pytorch_opencv but installing TensorRT libraries
      Note:TensorRT is not supported by TensorFlow since 2.18.0

*** Jupyter Labs ready containers (requires the base TPO & CTPO container to either be built locally or docker will attempt to pull otherwise)
  jupyter_tpo:
      jupyter-tensorflow_pytorch_opencv-2.18.0_2.5.1_4.10.0
  jupyter_ctpo:
      jupyter-cuda_tensorflow_pytorch_opencv-12.5.1_2.18.0_2.5.1_4.10.0
```

In this usage are multiple sections:
- The `Docker Image tag ending` matches the software release tag.
- The `Docker Runtime` explains the current default runtime. For `GPU` (CTPO) builds it is recommended to add `"default-runtime": "nvidia"` in the `/etc/docker/daemon.json` file and restart the docker daemon. Similarly, for `CPU` (TPO) builds, it is recommended that the `"default-runtime"` should be removed (or commented,) but because switching runtime on a system is not always achievable, we will use `NVIDIA_VISIBLE_DEVICES=void` ([details](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/docker-specialized.html).) We can check the current status of the docker runtime by running: `docker info | grep "Default Runtime"`
- The `Available Docker images to be built` section allows us to select the possible build targets. For `GPU`, the `cuda_` variants. For `CPU` the non `cuda_` variants. Naming conventions and tags follow the guidelines specified in the "Tag naming conventions" section.
- The `Jupyter Labs ready containers` are based on the containers built in the "Available Docker images[...]" and adding a running "Jupyter Labs" following the specific `Dockerfile` in the `Jupyter_build` directory. The list of built containers is limited to the most components per `CPU` and `GPU` to simplify distribution.



Note: Local builds will not have the `infotrend/ctpo-` added to their base name as those are only for release to Docker hub by maintainers.

###  1.2.3. Building (using Dockerfile)

Each time we request a specific `make` target, a dedicated `Dockerfile` is built in the `BuildDetails/<release>/<target>` directory.

That `Dockerfile` contains `ARG` and `ENV` values that match the specific build parameters.
For example in release `20241219`, when building the `tensorflow_opencv` target, the `BuildDetails/20241219/tensorflow_opencv-2.18.0_4.10.0-20241219/Dockerfile` is created and used to build the `tensorflow_opencv:2.18.0_4.10.0-20241219` container image.
In that file, we will see content such as:
```
ARG CTPO_FROM=ubuntu:24.04
FROM ${CTPO_FROM}
[...]
## Download & Building TensorFlow from source in same RUN
ENV LATEST_BAZELISK=1.22.1
ENV CTPO_TENSORFLOW_VERSION=2.18.0
ENV CTPO_TF_CONFIG=""
ENV TF_CUDA_COMPUTE_CAPABILITIES=""  
[...]
# No Magma (PyTorch GPU only)

# No PyTorch, Torch Audio or Torch Video
[...]
```
, which is specific to the CPU build of TensorFlow and OpenCV (without PyTorch).

This `Dockerfile` can be used directly by developers to build and integrate their modifications to build a specific feature.

When the maintainers upload this image to Dockerhub, that image will be preceded by `infotrend/ctpo-`.

When choosing to build a container image on specific hardware, please be patient, building any of those images might take a long time (counted in hours).
To build it this way, find the corresponding `Dockerfile` and in the directory where the file is located: `docker build -f Dockerfile --tag custombuild:local .`

For example, to build the `BuildDetails/20241219/tensorflow_opencv-2.18.0_4.10.0-20241219/Dockerfile` and tag it as `to:test`, run:
```
% cd BuildDetails/20241219/tensorflow_opencv-2.18.0_4.10.0-20241219
% docker build -f Dockerfile --tag to:test .
```

> ℹ️ When using an existing `Dockerfile`, please update the `ARG CTPO_NUMPROC=` line with the value of running the `nproc --all` command.
> The value in the `Dockerfile` reflects the build as it was performed for release to Docker Hub and might not represent the build system.

The `Makefile` contains most of the variables that define the versions of the different frameworks.
The file has many comments that allow developers to tailor the build.

For example, any release on our Dockerhub is made with "redistributable" packages, the `CTPO_ENABLE_NONFREE` variable in the `Makefile` controls that feature: 
> `The default is not to build OpenCV non-free or build FFmpeg with libnpp, as those would make the images unredistributable.`
> `Replace "free" by "unredistributable" if you need to use those for a personal build`

##  1.3. Available builds on DockerHub

The `Dockerfile` used for a Dockerhub pushed built is shared in the `BuildDetails` directory (see the [Dockerfile](#Dockerfile) section above)

We will publish releases into [Infotrend Inc](https://hub.docker.com/r/infotrend/)'s Docker Hub account.
There you can find other releases from Infotrend.

The tag naming reflects the [Tag naming conventions](#Tagnamingconventions) section above.
`latest` is used to point to the most recent release for a given container image.

The different base container images that can be found there are:
- CPU builds:
  - https://hub.docker.com/r/infotrend/ctpo-tensorflow_opencv
  - https://hub.docker.com/r/infotrend/ctpo-pytorch_opencv
  - https://hub.docker.com/r/infotrend/ctpo-tensorflow_pytorch_opencv
- GPU builds:
  - https://hub.docker.com/r/infotrend/ctpo-cuda_tensorflow_opencv
  - https://hub.docker.com/r/infotrend/ctpo-cuda_pytorch_opencv
  - https://hub.docker.com/r/infotrend/ctpo-cuda_tensorflow_pytorch_opencv
- Jupyter Lab CPU & GPU builds:
  - https://hub.docker.com/r/infotrend/ctpo-jupyter-tensorflow_pytorch_opencv
  - https://hub.docker.com/r/infotrend/ctpo-jupyter-cuda_tensorflow_pytorch_opencv
- Unraid enabled Jupyter Lab, CPU & GPU builds:
  - https://hub.docker.com/r/infotrend/ctpo-jupyter-tensorflow_pytorch_opencv-unraid
  - https://hub.docker.com/r/infotrend/ctpo-jupyter-cuda_tensorflow_pytorch_opencv-unraid

### 1.3.1. TensorRT build

Note that we are not releasing TensorRT based containers on DockerHub. We provide a `Dockerfile` in the corresponding `BuildDetails` (ending in `-TensoRT`) for local builds, follow the "Building (using Dockerfile)" instructions to build the container.

We can check the TensorRT Python version present in the locally built container by:

```bash
% docker run --gpus all --rm -it cuda_tensorflow_pytorch_opencv:12.5.1_2.18.0_2.5.1_4.10.0-20241219-TensorRT
# python3
>>> import tensorrt
>>> print(tensorrt.__version__)
10.7.0
```

##  1.4. Build Details

The [`README-BuildDetails.md`](README-BuildDetails.md) file is built automatically from the content of the `BuildDetails` directory and contains links to different files stored in each sub-directory.

It reflects each build's detailed information, such as (where relevant) the Docker tag, version of CUDA, cuDNN, TensorFlow, PyTorch, OpenCV, FFmpeg and Ubuntu. Most content also links to sub-files that contain further insight into the system package, enabled build parameters, etc.

##  1.5. Jupyter build

Jupyter Lab containers are built `FROM` the `tensorflow_pytorch_opencv` or `cuda_tensorflow_pytorch_opencv` containers.

A "user" version (current user's UID and GID are passed to the internal user) can be built using `make JN_MODE="-user" jupyter_tpo jupyter_ctpo`.

The specific details of such builds are available in the `Jupyter_build` directory, in the `Dockerfile` and `Dockerfile-user` files.

The default Jupyter Lab's password (`iti`) is stored in the `Dockerfile` and can be modified by the builder by replacing the `--IdentityProvider.token='iti'` command line option.

When using the Jupyter-specific container, it is important to remember to expose the port used by the tool (here: 8888), as such in the `docker run` command, make sure to add `-p 8888:8888` to the command line.

Pre-built containers are available, see the [Available builds on DockerHub](#AvailablebuildsonDockerHub) section above.

##  1.6. Unraid build

Those are specializations of the Jupyter Lab's builds, and container images with a `sudo`-capable `jupyter` user using Unraid's specific `uid` and `gid` and the same default `iti` Jupyter Lab's default password.

The Unraid version can be built using `make JN_MODE="-unraid" jupyter_tpo jupyter_ctpo`.

The build `Dockerfile` is `Jupyter_build/Dockerfile-unraid`.

Pre-built containers are available, see the [Available builds on DockerHub](#AvailablebuildsonDockerHub) section above.

#  2. Usage and more

##  2.1. A note on supported GPU in the Docker Hub builds

A minimum Nvidia driver version is needed to run the CUDA builds. 
[Table 1: CUDA Toolkit and Compatible Driver Versions](https://docs.nvidia.com/deploy/cuda-compatibility/index.html#binary-compatibility__table-toolkit-driver) and [Table 2: CUDA Toolkit and Minimum Compatible Driver Versions](https://docs.nvidia.com/cuda/cuda-toolkit-release-notes/index.html) as well as the `nvidia-smi` command on the host will help us determine if a specific version of CUDA will be supported.

Not all GPUs are supported in the Docker Hub builds. 
The containers are built for "compute capability (version)" (as defined in the [GPU supported](https://en.wikipedia.org/wiki/CUDA#GPUs_supported) Wikipedia page) of 6.0 and above (ie Pascal and above). 

If we need a different GPU compute capability, we can edit the `Makefile` and alter the various `DNN_ARCH_` matching the one that we need to build and add the needed architecture. Then type `make` to see the entire list of containers that the release we have obtained can build and use the exact tag that we want to build to build it locally (on Ubuntu, we will need `docker` and `build-essential` installed --at least-- to do this). 
Building a container image takes a lot of CPU and can take multiple hours, so we recommend to build only the target needed.

##  2.2. Using the container images

Build or obtain the container image required from DockerHub.

We understand the image names are verbose. This is to avoid confusion between the different builds.
It is possible to `tag` containers with shorter names for easy `docker run`.

The `WORKDIR` for the containers is set as `/iti`, as such, to map the current working directory within the container and test functions, we can `-v` as `/iti`.

When using a GPU image, make sure to add `--gpus all` to the `docker run` command line.

For example to run the GPU-Jupyter container and expose the WebUI to port 8765, one would:
```
% docker run --rm -v `pwd`:/iti --gpus all -p 8765:8888 infotrend/ctpo-jupyter-cuda_tensorflow_pytorch_opencv:11.8.0_2.12.0_2.0.1_4.7.0-20231120
```
By going to http://localhost:8765 we will be shown the Jupyter `Log in` page. As a reminder, the default token is `iti`.
After log in, we will see the Jupyter Lab interface and the list of files mounted in `/iti` in the interface.
From that WebUI, using `File -> Shutdown` will exit the container.

The non-Jupyter containers are set to provide the end users with a `bash`.
`pwd`-mounting the `/iti` directory to a directory where the developer has some code for testing enables the setup of a quick prototyping/testing container-based environment. 
For example to run some of the content of the `test` directory on a CPU, in the directory where this `README.md` is located:
```bash
% docker run --rm -it -v `pwd`:/iti infotrend/ctpo-tensorflow_opencv:2.12.0_4.7.0-20231120
      [this starts the container in interactive mode and we can type command in the provided shell]
root@b859b8aced9c:/iti# python3 ./test/tf_test.py
Tensorflow test: CPU only

On CPU:
tf.Tensor(
[[22. 28.]
 [49. 64.]], shape=(2, 2), dtype=float32)

Time (s) to convolve 32x7x7x3 filter over random 100x100x100x3 images (batch x height x width x channel). Sum of ten runs.
CPU (s): 0.483618629979901
Tensorflow test: Done
```

Note that the base container runs as `root`.
To run it as a non-root user, add `-u $(id -u):$(id -g)` to the `docker` command line and ensure access to the directories we will work in.

##  2.3. Podman usage

The built image is compatible with other GPU-compatible container runtimes, such as [`podman`](https://podman.io/).

Follow the instructions to install the [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html) and [Support for Container Device Interface](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/cdi-support.html). 

We will need a version of `podman` above 4.1.0 to be able to:

```bash
% podman run -it --rm --device nvidia.com/gpu=all infotrend/ctpo-cuda_tensorflow_pytorch_opencv:latest

root@2b8d77a97c5b:/iti# python3 /iti/test/pt_test.py 
Tensorflow test: GPU found
On cpu:
[...]
On cuda:
[...]

root@2b8d77a97c5b:/iti# touch file
```
, that last command will create a `file` owned by the person who started the container.

> ℹ️ On Ubuntu 22.04, install [HomeBrew](https://brew.sh/) and `brew install podman`, which at the time of this writeup provided version 4.8.2

##  2.4. docker compose

It is also possible to run the container in `docker compose`.

Follow the [GPU support](https://docs.docker.com/compose/gpu-support/) instructions to match the usage, and adapt the following `compose.yml` example as needed:

```yaml
version: "3.8"
services:
  jupyter_ctpo:
    container_name: jupyter_ctpo
    image: infotrend/ctpo-jupyter-cuda_tensorflow_pytorch_opencv:latest
    restart: unless-stopped
    ports:
      - 8888:8888
    volumes:
      - ./iti:/iti
      - ./home:/home/jupyter
    environment:
      - TZ="America/New_York"
      - NVIDIA_VISIBLE_DEVICES=all
      - NVIDIA_DRIVER_CAPABILITIES=all
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: 1
              capabilities: [gpu]
```

## 2.5. virtualenv for shared Jupyter Lab usage

When using the Jupyter version of CTPO with other users, it might be better to use a `virtualenv` for the packages to be installed in. 
In the following, we will create a `myvenv` virtual environment in the `/iti` directory, that will show up in the list of available kernels.

In a Terminal (preferably running a `bash` shell) in the Jupyter Lab, run:

```
python3 -m venv --system-site-packages myvenv
source myvenv/bin/activate
pip3 install ipykernel
python -m ipykernel install --user --name=myvenv --name=myvenv --display-name="Python (myvenv)"
```

Make sure to select the proper kernel in the notebook.
When using this kernel, it is still recommendeded to run any `pip` command from the terminal with the virtual environment activated to ensure the packages are installed in the expected location (i.e. not a global installation). As an alternative for `pip` commands to run with the proper installation directory, we will need to use `! . ./myvenv/bin/activate` before the command. For example: `!. ./myvenv/bin/activate; pip install -r requirements.txt` 

## 2.6. Disabling GPUs when nvidia is the default Docker runtime

If `"default-runtime": "nvidia"` in set in `/etc/docker/daemon.json` and want to hide the GPUs from a running container, add `NVIDIA_VISIBLE_DEVICES=void` before the `docker run` command.

# 3. Version History

- 20241219: First Ubuntu 24.04 based release (with Python 3.12), with support for CUDA 12.5.1, TensorFlow 2.18.0, PyTorch 2.5.1 and OpenCV 4.10.0 (and a Dockerfile to build with TensorRT)
- 20241125: Release with support for CUDA 12.5.1, TensorFlow 2.18.0, PyTorch 2.5.1 and OpenCV 4.10.0 (last Ubuntu 22.04 based version, with Python 3.10)
- 20240421: Release with support for CUDA 12.3.2, TensorFlow 2.16.1, PyTorch 2.2.2 and OpenCV 4.9.0
- 20231201: Release with support for CUDA 11.8.0, TensorFlow 2.14.1, PyToch 2.1.1 and OpenCV 4.8.0
- 20231120: Initial Release, with support for CUDA 11.8.0, TensorFlow 2.12.0, PyTorch 2.0.1 and OpenCV 4.7.0.
- November 2023: Preparation for public release.
- June 2023: engineered to support clean `Dockerfile` generation, supporting the same versions as 20231120 releases.
