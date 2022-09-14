# Using Docker in a multi architecture environment.
With the release of Apple Silcon based workstations, I've needed to work with multiple processor architectures when using contaienrs, primarily amd64 and arm64, but also s390x and ppc64le.  The following are my notes on how to (mostly) successfully do this.

Note, I've only tested on when using Docker Desktop for macOS.  While this can be done on other operating systems and with Docker engine directly, it isnt' covered here at this time.

## Basic Dockerfile
I'll be starting with the following example Dockerfile:
```
From ubuntu:latest
RUN apt update && apt install -y curl golang-go git
```
This Dockerfile updates apt and installs both curl, git, and go.

## Building images without a registry
Building images is most easily done when using Buildx.  


## Building images with
