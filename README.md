[![License](https://img.shields.io/badge/license-MIT-blue)](./LICENSE)
[![Ubuntu](https://img.shields.io/badge/ubuntu-22.04-orange)](https://ubuntu.com)
![ARM64](https://img.shields.io/badge/linux%2farm64-Yes-red)
![AMD64](https://img.shields.io/badge/linux%2famd64-Yes-red)

# gp-base-devenv: Build a dev containers for VSCode

Copyright (c) 2023, Greg PFISTER. MIT License

<div id="about" />

## About

This is a simple Ubuntu container to use as base image for building development
to be used as Visual Studio Code Remote Container.

This is image is provided with Ubuntu 22.04.

Along with the basic requirements, the image provides:

- zsh + starship
- vim with a few plugins (airline, syntastic, ripgrep, nerdcommenter,
  vim-colorschemes, ctrlp.vim)
- additionnal tools like `less`, `curl`, `nmap`, `htop`, ...
- a `vscode` user account
- `sudo` passwordless commands
- `tmux`
- `nginx` light

The image can be found
[here](https://github.com/gp-devenv/gp-base-devenv/pkgs/container/gp-base-devenv).

The following image are using this base image:

- [ghcr.io/gp-devenv/gp-node-devenv](https://github.com/gp-devenv/gp-node-devenv/pkgs/container/gp-node-devenv):
  a base image to build a Node.js app in a VS Code dev container
  - [ghcr.io/gp-devenv/gp-firebase-devenv](https://github.com/gp-devenv/gp-firebase-devenv/pkgs/container/gp-firebase-devenv):
    a base image to build a Firebase app in a VS Code dev container
  - [ghcr.io/gp-devenv/gp-angular-devenv](https://github.com/gp-devenv/gp-angular-devenv/pkgs/container/gp-angular-devenv):
    a base image to build an Angular app in a VS Code dev container
    - [ghcr.io/gp-devenv/gp-firebase-angular-hosting-devenv](https://github.com/gp-devenv/gp-firebase-angular-hosting-devenv/pkgs/container/gp-firebase-angular-hosting-devenv):
      a base image to build a Angular app hosted on Firebase in a VS Code dev container
- [ghcr.io/gp-devenv/gp-docker-devenv](https://github.com/gp-devenv/gp-docker-devenv/pkgs/container/gp-docker-devenv):
  a base image to build a Docker app in a VS Code dev container

<div id="volumes" />

## Volumes

In order to persist user data, a volume for the /home folder is set. The root
user will not be persisted.

| Volume | Description                                        |
| ------ | -------------------------------------------------- |
| /home  |  Persist the user data stored in their home folder |

<div id="build-run-scan-push" />

## Build, scan and push

### Versioning

Image version contains the Ubuntu version and the build version, using the
format `<Ubuntu version>-<Build version>`. The build version refers to the
latest Dockerfile script, when modification consists of fixing (patch change),
or adding or removing something significant (minor change) or breaking (major).

For example:

| Image                                        | Description                               |
| -------------------------------------------- | ----------------------------------------- |
| ghcr.io/gp-devenv/gp-base-devenv:22.04       | The latest build using Ubuntu 22.04       |
| ghcr.io/gp-devenv/gp-base-devenv:22.04-1     | The latest build 1.x using Ubuntu 22.04   |
| ghcr.io/gp-devenv/gp-base-devenv:22.04-1.1   | The latest build 1.1.x using Ubuntu 22.04 |
| ghcr.io/gp-devenv/gp-base-devenv:22.04-1.1.0 | The latest build 1.1.0 using Ubuntu 22.04 |
| ghcr.io/gp-devenv/gp-base-devenv:22.04-1.0   | The latest build 1.0.x using Ubuntu 22.04 |
| ghcr.io/gp-devenv/gp-base-devenv:22.04-1.0.0 | The latest build 1.0.0 using Ubuntu 22.04 |

In addition, `-arm64` and `-amd64` tags are available, should you need to force
a specific arch.

For CI/CD, the build version is store in `.version` file. The build version is
in the format
[SemVer](https://en.wikipedia.org/wiki/Software_versioning#Semantic_versioning).

Only the latest version (MAJOR, MAJOR.MINOR and MAJOR.MINOR.PATCH) is been built
daily.

### Testing locally using `-dev` images

When you are making change to the image, use :develop at the end of the
[build](#build_dev), [run](#run_dev) and [scan](#scan_dev) commands. The `*-dev`
tag should never be pushed...

### Cross-platform building

#### Setup

In order to build x-platform, `docker buildx` must be enabled (more info
[here](https://docs.docker.com/buildx/working-with-buildx/)). Then, instead of
`build` command, `buildx` command should be used (for example:
`npm run buildx:develop` will create a cross-platform image tagged `develop`).

You will need to create a multiarch builder:

```sh
./src/scripts/buildx/setup.sh
```

Up successful completion, it should at least have platforms `linux/arm64` and
`linux/amd64`:

```sh
[+] Building 5.8s (1/1) FINISHED
 => [internal] booting buildkit                                             5.8s
 => => pulling image moby/buildkit:buildx-stable-1                            7s
 => => creating container buildx_buildkit_multiarch0                          1s
Name:   multiarch
Driver: docker-container

Nodes:
Name:      multiarch0
Endpoint:  unix:///var/run/docker.sock
Status:    running
Platforms: linux/arm64, linux/amd64, linux/amd64/v2, linux/riscv64,
           linux/ppc64le, linux/s390x, linux/386, linux/mips64le, linux/mips64,
           linux/arm/v7, linux/arm/v6
```

#### Build commands

Once the previous step is completed, simpy run to build the current version:

```sh
(cd src && ./scripts/buildx/build.sh)
```

### Build using local architecture (for local testing)

<div id="build_dev" />

#### Build the development image

To build using a specific Ubuntu version, use:

```sh
(cd scr && ./scripts/dev/image/build.sh <UBUNTU_VERSION>)
```

where `UBUNTU_VERSION` must 22.04.

It will create an image `ghcr.io/gp-devenv/gp-base-devenv` tagged with the current
version (see `src/.version` file) and `-dev` suffix.

You may alter the `.src/.version` file should you want to have different tags or
names, however if you PR your change, it will be rejected. The ideal solution
is to run the `docker build` command instead.

To remove the created image (named:
`ghcr.io/gp-devenv/gp-base-devenv:<UBUNTU_VERIONS>-<VERSION>-dev`), simply use:

```sh
(cd scr && ./scripts/dev/image/rm.sh <UBUNTU_VERSION>)
```

<div id="run_dev" />

#### Run or create/start/stop/exec a container

To run an interactive container of a give Ubuntu version, simple use:

```sh
(cd src && ./scripts/dev/container/run.sh <UBUNTU_VERSION>)
```

where `UBUNTU_VERSION` must 22.04.

Alternatively, you can create and start a container to run in background, and
execute scripts on this container, using the following scripts:

| Action | Script                                                                   |
| ------ | ------------------------------------------------------------------------ |
| create | `(cd src && ./scripts/dev/container/create.sh <UBUNTU_VERSION>)`         |
| start  | `(cd src && ./scripts/dev/container/start.sh <UBUNTU_VERSION>)`          |
| stop   | `(cd src && ./scripts/dev/container/stop.sh <UBUNTU_VERSION>)`           |
| exec   | `(cd src && ./scripts/dev/container/exec.sh <UBUNTU_VERSION>) <COMMAND>` |

To remove the created container (named:
`gp-base-devenv:<UBUNTU_VERIONS>-<VERSION>-dev`), simply use:

```sh
(cd src && ./scripts/dev/container/rm.sh <UBUNTU_VERSION>)
```

<div id="scan_dev" />

#### Scan

To scan the image of a give Ubuntu version, simple use:

```sh
(cd src && ./scripts/dev/scan.sh <UBUNTU_VERSION>)
```

where `UBUNTU_VERSION` must 22.04.

<div id="build-from-this-image" />

## Build from this image

Should you want to make other changes, the ideal solution is to build from this
image. For example, here's the way to set the image to a different timezone than
"Europe/Paris" (the default one):

```Dockerfile
FROM ghcr.io/gp-devenv/gp-base-devenv:22.04

ENV TZ="America/New_York"

# Switch to root
USER root

# Reconfigure tzdata
RUN dpkg-reconfigure -f noninteractive tzdata

# Update all packages
RUN apt-get update && \
    apt-get upgrade -y && \
    apt-get autoremove -y && \
    apt-get autoclean

# Switch back to vscode
USER vscode
```

**Important:** Unless you really want to use the root user, you should always
make sure the `vscode` is the last one activated.

**Updating the image:** There is yet no plan to create nighly build, ensuring
the image is always up to date. Therefore, when building yours from this one,
run the update process.

<div id="faq" />

## FAQ

1. [How to require password for sudo command ?](#faq1)
2. [How to change `nginx` settings ?](#faq2)

<div id="faq1" />

### 1. How to require password for sudo command ?

You will have to [build from this image](#build-from-this-image) to disable the
the password less sudo command. Typically create a `Dockerfile` like:

```Dockerfile
FROM ghcr.io/gp-devenv/gp-base-devenv:22.04

ARG VSCODE_PASSWORD="dummy"

# Switch to root to make changes
USER root

# Remove the specific config for sudo and add to sudo group
RUN rm /etc/sudoers.d/vscode && \
    usermod -aG sudo vscode

# Change the password.
RUN usermod -p $VSCODE_PASSWORD vscode

# Switch back to vscode
USER vscode
```

If you simply want to get rid of `sudo`:

```Dockerfile

FROM ghcr.io/gp-devenv/gp-base-devenv:22.04

# Switch to root to make changes
USER root

# Remove the specific config for sudo and add to sudo group
RUN rm /etc/sudoers.d/vscode && \
    apt-get purge -y sudo

# Switch back to vscode
USER vscode
```

<div id="faq2" />

### 2. How to change `nginx` settings ?

`nginx` can be setup by updating the configuration found on `/etc/nginx`.

```Dockerfile

FROM ghcr.io/gp-devenv/gp-base-devenv:22.04

# Switch to root to make changes
USER root

# Install whatever should be served using SSL

# Remove provious config
RUN rm /etc/nginx/sites-enabled/*
COPY <SITE_CONFIGURATION> /etc/nginx/sites-available/
RUN ln -s /etc/nginx/site-enabled/<SITE_CONFIGURATION> /etc/nginx/site-available/<SITE_CONFIGURATION>

# Remove the specific config for sudo and add to sudo group
RUN rm /etc/sudoers.d/vscode && \
    apt-get purge -y sudo

# Switch back to vscode
USER vscode
WORKDIR /workspace

EXPOSE <PORT>

ENTRYPOINT [ "sudo", "/entrypoint.sh"]

```

<div id="known_issues" />

## Known issues

See known issues [here](./KNOWN_ISSUES.md).

<div id="contrib" />

## Contributions

See instructions [here](./CONTRIBUTING.md).

<div id="license" />

## License

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS
FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR
COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER
IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN
CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

See license [here](./LICENSE).
