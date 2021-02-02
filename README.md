# Docker Snap

This repository contains the source for the `docker` snap package.  The package provides a distribution of Docker Community Edition (CE) for Ubuntu Core 16 (and other snap-compatible) systems.  It is built from an upstream Docker CE release tag with some patches to fit the snap format and is available on `armhf`, `arm64`, `amd64`, `i386`, and `ppc64el` architectures.  The rest of this page describes installation, usage, and development.

> NOTE: Docker's official documentation ([https://docs.docker.com](https://docs.docker.com)) does not yet discuss the `docker` snap package.

## Installation

To install the latest stable release of Docker CE using `snap`:

```shell
sudo snap install docker
```

If you are using Ubuntu Core 16,

* Connect the `docker:home` plug as it's not auto-connected by default:

```shell
sudo snap connect docker:home
```

If you are using an alternative snap-compatible Linux distribution ("classic" in snap lingo), and would like to run `docker` as a normal user:

* Create and join the `docker` group.

```shell
sudo addgroup --system docker
sudo adduser $USER docker
newgrp docker
```

* You will also need to disable and re-enable the `docker` snap if you added the group while it was running.

```shell
sudo snap disable docker
sudo snap enable docker
```

## Usage

Docker should function normally, with the following caveats:

* All files that `docker` needs access to should live within your `$HOME` folder.

  * If you are using Ubuntu Core 16, you'll need to work within a subfolder of `$HOME` that is readable by root. https://github.com/docker/docker-snap/issues/8

* Additional certificates used by the Docker daemon to authenticate with registries need to be located in `/var/snap/docker/common/etc/certs.d` instead of `/etc/docker/certs.d`.

### Examples

* [Setup a secure private registry](registry-example.md)

## Development

Developing the `docker` snap package is typically performed on a "classic" Ubuntu distribution.  The instructions here are written for Ubuntu 16.04 "Xenial".

* Install the snap tooling (requires `snapd>2.21` and `snapcraft>=2.26`):

```shell
sudo apt-get install snapd snapcraft
sudo snap install core
```

* Checkout this repository and build the `docker` snap package:

```shell
git clone https://github.com/docker/docker-snap
cd docker-snap
sudo snapcraft
```

* Install the newly-created snap package:

```shell
sudo snap install --dangerous docker_[VER]_[ARCH].snap
```

* Manually connect the relevant plugs and slots which are not auto-connected:

```shell
sudo snap connect docker:privileged :docker-support
sudo snap connect docker:support :docker-support
sudo snap connect docker:firewall-control :firewall-control
sudo snap connect docker:docker-cli docker:docker-daemon
sudo snap disable docker
sudo snap enable docker
```

  You should end up with output similar to:

```shell
sudo snap interfaces docker
    Slot                  Plug
    :docker-support       docker:privileged,docker:support
    :firewall-control     docker
    :home                 docker
    :network              docker
    :network-bind         docker
    docker:docker-daemon  docker:docker-cli
```

## Testing

We rely on spread (https://github.com/snapcore/spread) to run full-system test on Ubuntu Core 16. We also provide a utility script (run-spread-test.sh) to launch the spread test. It will

1. Fetch primary snaps( kernel, core, gadget) and build custom Ubuntu Core image with them
2. Boot the image in qemu emulator
3. Deploy test suits in emulation environment
4. Execute full-system testing

Firstly, install ubuntu-image tool since we need to create a custom Ubuntu Core image during test preparation.

```shell
sudo snap install --beta --classic ubuntu-image
```

Secondly, install qemu-kvm package since we use it as the backend to run the spread test.

```shell
sudo apt install qemu-kvm
```

Meanwhile, you need a classic-mode supported spread binary to launch kvm from its context. You can either build spread from this [branch](https://github.com/rmescandon/spread/tree/snap-as-classic) or download the spread snap package [here](http://people.canonical.com/~gary-wzl77/spread_2017.05.24_amd64.snap).

```shell
sudo snap install --classic --dangerous spread_2017.05.24_amd64.snap
```

You may build the docker snap locally in advance and then execute the spread tests with the following commands:

```shell
snapcraft
./run-spread-tests.sh
```

When doing a local build, you can also specify --test-from-channel to fetch the snap from the specific channel of the store. The snap from `candidate` channel is used by default as test target if `--channel` option is not specified.

```shell
./run-spread-tests.sh --test-from-channel --channel=stable
```

In order to run an individual spread test, please run the following command:

```shell
spread spread/main/installation
```

This will run the test case under spread/main/installation folder.
You can specify the `SNAP_CHANNEL` environment variable to install a snap from a specific channel for the testing as well.

```shell
SNAP_CHANNEL=candidate spread spread/main/update_policy
```
