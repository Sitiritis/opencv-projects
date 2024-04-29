# Container images for OpenCV

## Build OpenCV from sources with CUDA

Based on https://github.com/opencv/opencv-python.

### Limitations

- Supports only X64 for now
- Requires cuDNN 8 ([OpenCV 4.9.0 does not support cuDNN 9](https://github.com/opencv/opencv/issues/24983))

### Prerequisites

#### Download nvidia codec SDK

Download [Nvidia Video Codec SDK 12.1](https://developer.nvidia.com/video-codec-sdk-archive) and unpack it to `infra/image/build/deps/nvidia-codec-sdk`. Move everything from inside the directory to the `nvidia-codec-sdk` dir.

### Start the build container

Since the build might take some time and the is a possibility that it fail, it is more reliable to run it interactively in a container. When it is finished, the build artifacts can be copied to an attached volume to be accessed outside of the build container.

Create directory to share opencv build artifacts between the build container and host:

```shell
mkdir -p ./infra/opencv-build
```

Run the build container using `compose.build.yaml` file:

```shell
sudo nerdctl compose -f="./infra/compose.build.yaml" up -d
```

> [!WARNING]
>
> With this command, the container may not be able to see nvidia GPU for some reason. Be sure to check `sudo nerdctl logs opencv-python-build`.
>
> If it says:
> ```
> WARNING: The NVIDIA Driver was not detected.  GPU functionality will not be available.
>   Use the NVIDIA Container Toolkit to start this container with GPU support; see
>   https://docs.nvidia.com/datacenter/cloud-native/ .
> ```
>
> Then the gpu has not been detected. In this case,
> bring the container down:
> ```shell
> sudo nerdctl compose -f="./infra/compose.build.yaml" down
> ```
> And run the container manually (thankfully, the image has already been built so it won't be rebuilt):
> ```shell
> sudo nerdctl run -dt --gpus="all" --name="opencv-python-build" --volume="${PWD}/infra/image/deps/opencv-build:/home/opencv/build" --volume="${PWD}/infra/image/build/scripts:/home/opencv/scripts" opencv-python-build:latest
> ```

> Add `--build` after `up` if you changed the `Conainerfile.opencv-python-build` and want to rebuild the image.

This will start a build container. The container has 2 attached volumes:

- `./opencv-build:/home/opencv/build` - after the build is finished you can copy the outputs to `/home/opencv/build` inside the container and they will appear in `infra/opencv-build` directory. This directory is also excluded from git.
- `./build/scripts:/home/opencv/scripts` - contains scripts to facilitate build activities. E.g. `build-opencv-python.sh` will set build flags and start the build. You can change this script, if you want to compile opencv with different flags.

### Login to the build container

[After the build container has been started](#start-the-build-container), you can login to the container:

```shell
sudo nerdctl exec -it opencv-python-build /bin/bash
```

### Build

[Inside the build container](#login-to-the-build-container) run build with:

```shell
~/scripts/build-opencv-python.sh
```

### Copy OpenCV build to the host

Copy the built wheel:

```shell
cp opencv_*.whl ~/build/
```

Or, you can copy the whole build directory (it can reach 10Gb, so it's better to copy only wheel, because this is what is actually going to be used to install OpenCV):

```shell
cp -R ./ ~/build/
```

## Jupyter image with OpenCV compiled for CUDA

### Building jupyter image with CUDA OpenCV

<!-- When you [have built OpenCV with CUDA](#build-opencv-from-sources-with-cuda), copy the resulting wheel to the `./infra/image/jupyter/deps` directory.

```shell
cp ./infra/opencv-build/opencv_*.whl ./infra/image/deps/
```

When jupyter container is built, OpenCV will be installed from that wheel. -->

### Run Jupyter image with CUDA OpenCV

- [ ] TODO: describe
  - [ ] TODO: ensure that you have the OpenCV wheel build under `./infra/deps/opencv-build`

```shell
sudo nerdctl compose up -d
```

```shell
sudo nerdctl run -d --gpus="all" --name="opencv-projects" -p="8888:8888" --volume="${PWD}/src:/home/opencv/src" opencv-projects:latest
```