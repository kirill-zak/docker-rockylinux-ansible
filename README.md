# docker-rockylinux-ansible
Rocky Linux docker container with built-in Ansible for playbook and role testing 

## Tags
- `8.x`: Based on branch `8.x`. Releases: `8.5`, `8.6`, `8.7`.
- `9.x`: Based on branch `9.x`. Releases: `9.0`, `9.1`.

## How to Build

To build the image on your own locally, do the following:

  1. [Install Docker](https://docs.docker.com/engine/installation/).
  2. `cd` into target directory. For example, target directory for `Rocky Linux 9.x` is `rocky9`
  3. Run `docker build -t docker-rocky-ansible . --build-arg VERSION=9.<release>`. Where `<release>` is release version of Rocky Linux. List of minor releases can found on official release page of Rocky Linux (http://dl.rockylinux.org/vault/rocky/)

  ## How to Use

  1. [Install Docker](https://docs.docker.com/engine/installation/).
  2. Pull pre-build image via `docker image pull kirillzak/docker-rocky-ansible:latest` or build image.
  3. Run a container from the image: `docker run --detach --privileged --volume /sys/fs/cgroup:/sys/fs/cgroup:ro docker-rocky-ansible:latest`

## Author

[Kirill Ziuzin](https://kirill-zak.ru/)