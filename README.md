# Quick reference
- Maintained by: FPC Docker team
- Where to get help: [the FPC Community Forums](https://forum.lazarus.freepascal.org/index.php), [the Docker Community Forums](https://forums.docker.com/), or [Stack Overflow](https://stackoverflow.com/search?tab=newest&q=docker)



# Supported tags and respective Dockerfile links
- [`3.2.2-focal-full`, `latest-focal-full`](https://gitlab.com/freepascal.org/fpc/docker/-/blob/3.2.2/Ubuntu/Dockerfile)
- [`3.2.2-full`, `latest-full`](https://gitlab.com/freepascal.org/fpc/docker/-/blob/3.2.2/AlpineLinux/Dockerfile)
- [`3.2.2-slim`, `latest-slim`](https://gitlab.com/freepascal.org/fpc/docker/-/blob/3.2.2/AlpineLinux/Dockerfile)
- [`3.2.2-minimal`, `latest-minimal`](https://gitlab.com/freepascal.org/fpc/docker/-/blob/3.2.2/AlpineLinux/Dockerfile)
- [`3.2.0-focal-full`](https://gitlab.com/freepascal.org/fpc/docker/-/blob/3.2.0/Ubuntu/Dockerfile)
- [`3.2.0-full`](https://gitlab.com/freepascal.org/fpc/docker/-/blob/3.2.0/AlpineLinux/Dockerfile)
- [`3.2.0-slim`](https://gitlab.com/freepascal.org/fpc/docker/-/blob/3.2.0/AlpineLinux/Dockerfile)
- [`3.2.0-minimal`](https://gitlab.com/freepascal.org/fpc/docker/-/blob/3.2.0/AlpineLinux/Dockerfile)

**Later (help wanted!)**
  - 3.2.x-x86_64-to-armv7 (cross compiled binaries, RTL + all packages, examples and documentation on Ubuntu)
  - 3.2.x-x86_64-to-arm64v8 (cross compiled binaries, RTL + all packages, examples and documentation on Ubuntu)
  - 3.2.x-x86_64-to-win64 (cross compiled binaries, RTL + all packages, examples and documentation on Ubuntu)
  - ...
  - nightly-full (full installation incl. all packages on Alpine Linux without examples and documentation of the current main branch)
  - nightly-slim (binaries and full set of RTL on Alpine Linux of the current main branch)
  - nightly-minimal (binaries and files needed to compile a new compiler and its test suite on Alpine Linux of the current main branch)
  - pas2js (binaries and packages for the Pascal to JavaScript transpiler)



# Quick reference (cont.)
- Where to file issues: https://gitlab.com/freepascal.org/fpc/docker
- Supported architectures: ([more info](https://github.com/docker-library/official-images#architectures-other-than-amd64)) amd64, arm32v7, arm64v8, i386
<!-- - Published image artifact details: repo-info repo's repos/alpine/ directory (history) (image metadata, transfer size, etc) -->
<!-- - Image updates: official-images repo's library/alpine label  
    official-images repo's library/alpine file (history) -->
- Source of this description: [docs repo's directory](https://gitlab.com/freepascal.org/fpc/docker/-/blob/main/README.md) ([history](https://gitlab.com/freepascal.org/fpc/docker/-/commits/main/README.md))



# What is Free Pascal Compiler?
Free Pascal is a mature, versatile, open source Pascal compiler. It can target many processor architectures: Intel x86 (16 and 32 bit), AMD64/x86-64, PowerPC, PowerPC64, SPARC, SPARC64, ARM, AArch64, MIPS, Motorola 68k, AVR, and the JVM. Supported operating systems include Windows (16/32/64 bit, CE, and native NT), Linux, Mac OS X/iOS/iPhoneSimulator/Darwin, FreeBSD and other BSD flavors, DOS (16 bit, or 32 bit DPMI), OS/2, AIX, Android, Haiku, Nintendo GBA/DS/Wii, AmigaOS, MorphOS, AROS, Atari TOS, and various embedded platforms. Additionally, support for RISC-V (32/64), Xtensa, and Z80 architectures, and for the LLVM compiler infrastructure is available in the development version. <!-- Additionally, the Free Pascal team maintains a transpiler for pascal to Javascript called pas2js. -->

![alt text](https://www.freepascal.org/pic/logo.gif "Open source compiler for Pascal and Object Pascal")



# How to use this image

## Start a FPC instance running your app
The most straightforward way to use this image is to use a container as both the build and runtime environment. In your Dockerfile, writing something along the lines of the following will compile and run your project:
```docker
FROM freepascal/fpc:3.2.2-focal-full

WORKDIR /usr/src/myapp
COPY . .

RUN fpc myapp.pas

CMD ["myapp"]
```
Then, build and run the Docker image:
```console
$ docker build -t my-fpc-app .
$ docker run -it --rm --name my-running-app my-fpc-app
```
This creates an image that has all of the FPC tooling in the image. If you just want the compiled application:
```docker
FROM freepascal/fpc:3.2.2-focal-full as builder
WORKDIR /usr/src/myapp
COPY . .
RUN fpc myapp.pas

FROM ubuntu:focal
RUN apt-get update && apt-get upgrade && rm -rf /var/lib/apt/lists/*
COPY --from=builder /usr/src/myapp/myapp /usr/local/bin/myapp
CMD ["myapp"]
```
This method will create an image that only contains the program's executable. If you switch to using the Alpine-based FPC image, you might be able to save some space.  
See [multi-stage builds](https://docs.docker.com/develop/develop-images/multistage-build/) for more information.

## Compile your app inside the Docker container
There may be occasions where it is not appropriate to run your app inside a container. To compile, but not run your app inside the Docker instance, you can write something like:
```console
$ docker run --rm --user "$(id -u)":"$(id -g)" -v "$PWD":/usr/src/myapp -w /usr/src/myapp freepascal/fpc:3.2.2-focal-full fpc -MDelphi myapp.lpr
```
This will add your current directory, as a volume, to the container, set the working directory to the volume, and run the command `fpc -MDelphi myapp.lpr`. This tells FPC to compile the project `myapp` to output an executable.



# Image Variants
The `FPC` images come in different flavors, each designed for a specific use case.

## Ubuntu
[Ubuntu](https://hub.docker.com/_/ubuntu) is a Debian-based Linux operating system that runs from the desktop to the cloud. It is the world's most popular operating system across public clouds and OpenStack clouds. It is the number one platform for containers; from Docker to Kubernetes to LXD, Ubuntu can run your containers at scale. Fast, secure and simple, Ubuntu powers millions of PCs worldwide.
### `<version>`-`<ubuntu>`-full
This image contains everything you would find in a full local installation and thus also the documentation and examples.

## Alpine Linux
These images are based on the popular [Alpine Linux project](https://alpinelinux.org/), available in [the `alpine` official image](https://hub.docker.com/_/alpine). Alpine Linux is much smaller than most distribution base images (~5MB), and thus leads to much slimmer images in general.

This variant is useful when final image size being as small as possible is your primary concern. The main caveat to note is that it does use musl libc instead of glibc and friends, so software will often run into issues depending on the depth of their libc requirements/assumptions. See [this Hacker News comment thread](https://news.ycombinator.com/item?id=10782897) for more discussion of the issues that might arise and some pro/con comparisons of using Alpine-based images.

To minimize image size, it's uncommon for additional related tools (such as `git` or `bash`) to be included in Alpine-based images. Using this image as a base, add the things you need in your own Dockerfile (see the [`alpine` image description](https://hub.docker.com/_/alpine/) for examples of how to install packages if you are unfamiliar).
### `<version>`-full
This image contains everything that you will find if you do a normal installation of FPC with the official installer packages. The only exception is that it comes without the examples and documentation.
### `<version>`-slim
This image reduces the size even further by removing most packages and keeping only the core packages - the full RTL - and all binaries.
### `<version>`-minimal
This image contains all binaries, a minimal RTL and might only be used to compile a new compiler and the corresponding tests.



# License
View [license information](https://www.freepascal.org/faq.html#general-license) for the software contained in this image.

As with all Docker images, these likely also contain other software which may be under other licenses (such as Bash, etc from the base distribution, along with any direct or indirect dependencies of the primary software being contained).

<!-- Some additional license information which was able to be auto-detected might be found in the repo-info repository's alpine/ directory. -->

As for any pre-built image usage, it is the image user's responsibility to ensure that any use of this image complies with any relevant licenses for all software contained within.
