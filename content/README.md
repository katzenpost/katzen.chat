katzen
=======

[![Build Status](https://github.com/katzenpost/katzen/actions/workflows/go.yml/badge.svg?branch=main)](https://github.com/katzenpost/katzen/actions/workflows/go.yml)

A multiplatform chat client using catshadow and gio

## Getting the source code

    git clone https://github.com/katzenpost/katzen
    cd katzen/

## Building katzen

The easiest way to build katzen is in a container managed by
[podman](https://podman.io/) or
[docker](https://en.wikipedia.org/wiki/Docker_(software)). Podman is a docker
replacement which has the advantages of not requiring root access or a daemon to
operate and what we use by default. If you already have Docker installed proceed
to next step, otherwise run:

```
# apt install podman
```

### Building for GNU/Linux

```
$ make docker-build-linux
```

The Makefile in this repository has targets to create a container image with the
build environment, and to create a container with that image, and build katzen
in the container. Although the make target names are prefixed with `docker-`,
podman will be used by default if it is installed. 

- `docker-build-linux` builds GNU/Linux binary (default target)
- `docker-build-windows` builds Windows
- `docker-build-android` builds Android APK
- `docker-shell` puts you in a shell inside an ephemeral build container with your local
checkout mounted at `/go/katzen`

**Docker or Podman**

If you have both Podman and Docker installed and want to force the use of
Docker, or if you want to specify the path to a different docker/podman
compatible binary, you can specify `docker=docker` as the first argument to the
`make` command. 

```
$ make docker=docker docker-build-linux
```

### Cleaning build

To remove the docker images and containers created by the Makefile, run: 

```
# make docker-clean
```

The Makefile contains targets which build intermediate images for the Debian 
and go module dependencies, so that local changes can be built without the 
need for internet access.

To build using local uncomitted changes from the Katzenpost monorepo, add
`replace github.com/katzenpost/katzenpost => ./katzenpost` to katzen's `go.mod`
file and clone the monorepo in your katzen checkout.

### Building without docker

Make sure you have a working Go environment (Go 1.19 or higher is required;
on Debian bullseye the backports repository can be used).

See the [golang install instructions](http://golang.org/doc/install.html).

#### Installing golang (Debian Bullseye example)

    apt install golang git ca-certificates
    export GOPATH=$HOME/go

#### Install debian dependencies (Debian Bullseye example)

    apt install --no-install-recommends build-essential libgles2 libgles2-mesa-dev libglib2.0-dev libxkbcommon-dev libxkbcommon-x11-dev libglu1-mesa-dev libxcursor-dev libwayland-dev libx11-xcb-dev libvulkan-dev

#### Download and verify dependencies

   go mod download && go mod verify

#### Build katzen

    go build -trimpath -ldflags=-buildid=

#### Cross-compilation dependencies for the arm64 architecture

    dpkg --add-architecture arm64 && apt update
    apt install --no-install-recommends crossbuild-essential-arm64 libgles2:arm64 libgles2-mesa-dev:arm64 libglib2.0-dev:arm64 libxkbcommon-dev libxkbcommon-x11-dev:arm64 libglu1-mesa-dev:arm64 libxcursor-dev:arm64 libwayland-dev:arm64 libx11-xcb-dev:arm64 libvulkan-dev:arm64

#### Building for Linux on arm64

    CC=aarch64-linux-gnu-gcc CGO_ENABLED=1 GOOS=linux GOARCH=arm64 go build -trimpath -ldflags=-buildid=

#### Building for Windows

    GOOS=windows CGO_ENABLED=1 CGO_LDFLAGS="-Wl,--no-as-needed -Wl,-allow-multiple-definition" CC=x86_64-w64-mingw32-gcc go build -trimpath -ldflags="-H windowsgui -buildid=" -o katzen.exe

#### Building for macOS (Intel), requires macOS and xcode

    GOARCH="amd64" go build -trimpath -ldflags=-buildid=

#### Building for macOS (Apple M1, arm), requires macOS 11 and xcode

    CGO_ENABLED=1 GOOS="darwin" GOARCH="arm64" go build -tags dynamic -trimpath -ldflags=-buildid=

#### Building for android

Note that you will need to have the android NDK and SDK installed and the
appropriate environment variables exported.

See the Dockerfile.android in this repository to set up a build environment if you wish.

First, get and install the gogio tool:

    go install gioui.org/cmd/gogio

Generate an Android signing key so you can update your app later:
(keytool is provided by the openjdk package: apt install openjdk-11-jdk, or use within the docker container)

    keytool -genkey -keystore sign.keystore -storepass YOURPASSWORD -alias android -keyalg RSA -keysize 2048 -validity 10000 -noprompt -dname CN=android

And then build the Android APK:

    gogio -arch arm64,amd64 -x -target android -appid chat.katzen -version 1 -signkey sign.keystore -signpass YOURPASSWORD .

#### Building for Nix

We slightly-tested Nix packaging. This command should work:

    nix build --extra-experimental-features flakes \
              --extra-experimental-features nix-command \
              github:katzenpost/katzen -L

(And produce reproducible output which you can compare with the hashes
published by the github CI run for the commit you built from...)

Note that when updating dependencies, the vendorHash line in katzen.nix will need to be updated manually as well.
After updating a dependency, run the above command (or use make docker-build-nix) which will output the updated vendorHash.

## Run it

    Usage of ./katzen:
      -f string
         Path to the client config file. (default to baked-in testnet configuration)
      -s string
         The catshadow state file path. (default "catshadow_statefile")

## supported by

[![NGI](https://nlnet.nl/image/logos/EC.svg)](https://www.ngi.eu/about/)
[![nlnet.nl](https://nlnet.nl/logo/banner.svg)](https://nlnet.nl)
[![nlnet.nl/assure](https://nlnet.nl/image/logos/NGIAssure_tag.svg)](https://nlnet.nl/assure)
[![nlnet.nl/NGI0](https://nlnet.nl/image/logos/NGI0PET_tag.svg)](https://nlnet.nl/NGI0)

This project has received funding from:

* European Union’s Horizon 2020 research and innovation programme under the Grant Agreement No 653497, Privacy and Accountability in Networks via Optimized Randomized Mix-nets (Panoramix).
* The Samsung Next Stack Zero grant.
* NGI0 PET Fund, a fund established by NLnet with financial support from the European Commission's Next Generation Internet programme, under the aegis of DG Communications Networks, Content and Technology under grant agreement No 825310.
* NGI Assure Fund, a fund established by NLnet with financial support from the European Commission's Next Generation Internet programme, under the aegis of DG Communications Networks, Content and Technology under grant agreement No 957073.
