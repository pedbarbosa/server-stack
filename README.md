# server-stack

To use the stack, make sure to create a .env file with the required variables

```
cp .env_example .env
vim .env
```

## nvenc for jellyfin

To use nvenc transcoding in jellyfin, on an Ubuntu host run the following:

Set up the nvidia-docker repository:

```
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | apt-key add -
curl -s -L https://nvidia.github.io/nvidia-docker/ubuntu22.04/nvidia-docker.list > /etc/apt/sources.list.d/nvidia-docker.list
```

Install the NVidia container toolkit and configure it:

```
apt update
apt install nvidia-container-toolkit

nvidia-ctk runtime configure
systemctl restart docker
```

If not available or not required, remove the 'runtime' and the 'NVIDIA_VISIBLE_DEVICES' environment variable from the 'jellyfin' container

