# Using Docker in a multi architecture environment.
With the release of Apple Silcon based workstations, I've needed to work with multiple processor architectures when using contaienrs, primarily amd64 and arm64, but also s390x and ppc64le.  This is technically done use The following are my notes on how to (mostly) successfully do this.

Note, I've only tested on when using Docker Desktop for macOS.  While this can be done on other operating systems and with Docker engine directly, it isnt' covered here at this time.

I'll be assuming that the workstation is amd64 based; if you are using an Apple Silcon machine, you can swap the order.

## Basic Dockerfile
I'll be starting with the following example Dockerfile:
```
FROM ubuntu:latest
RUN apt update && apt install -y curl golang-go git
```
This Dockerfile updates apt and installs both curl, git, and go.

## Building images without a registry
Building images is most easily done when using Buildx. The build command using buildx is very similar to standard build command.
```
docker buildx build --platform <platform> -t <repo>:<tag> --load .
```
where
- `--platform` - The platform to build for, e.g. linux/amd64, linux/arm64, linux/ppc64le, or linux/s390x. Note, when using load command, multiple platforms are not supported.
- `<repo>:<tag>` - the image name and tag
- `--load` - the load option makes the the resulting image available to your local docker enviroment.
This example assumes the Dockerfile is located in the same directory.

I'll start by building an amd64 image first.
```
docker buildx build --platform linux/amd64 -t ryan:amd64 --load .
```
You should see output similar to:
```
...                                                                                                                                                                                          
 => [internal] load .dockerignore                                                                                                                                                                                       
 => => transferring context: 2B                                                                                                                                                                                         
 => [internal] load build definition from Dockerfile                                                                                                                                                                    
 => => transferring dockerfile: 108B                                                                                                                                                                                    
 => [internal] load metadata for docker.io/library/ubuntu:latest                                                                                                                                                        
 => [1/2] FROM docker.io/library/ubuntu:latest@sha256:20fa2d7bb4de7723f542be5923b06c4d704370f0390e4ae9e1c833c8785644c1                                                                                                  
 => => resolve docker.io/library/ubuntu:latest@sha256:20fa2d7bb4de7723f542be5923b06c4d704370f0390e4ae9e1c833c8785644c1                                                                                                  
 => [2/2] RUN apt update && apt install -y curl golang-go git                                                                                                                                                    
 => exporting to oci image format                                                                                                                                                                                       
 => => exporting layers                                                                                                                                                                                                 
 => => exporting manifest sha256:7f44313b7846f93f8329770715e3e534a124e8830c4b125b3577b38b8d3361ab                                                                                                                       
 => => exporting config sha256:8bd1f828155feaa6ff638f95fddd4b600570081f1558214d4ef0972764ca6ca2                                                                                                                         
 => => sending tarball                                                                                                                                                                                                  
 => importing to docker   
``` 

I'll now build an image for arm64.

```
docker buildx build --platform linux/arm64 -t ryan:arm64 --load .
```
notice the different platform and tag names.

With an output similar to:
```
...
[+] Building 194.4s (7/7) FINISHED                                                                                                                                                                                           
 => [internal] load build definition from Dockerfile                                                                                                                                                                    
 => => transferring dockerfile: 108B                                                                                                                                                                                    
 => [internal] load .dockerignore                                                                                                                                                                                       
 => => transferring context: 2B                                                                                                                                                                                         
 => [internal] load metadata for docker.io/library/ubuntu:latest                                                                                                                                                        
 => CACHED [1/2] FROM docker.io/library/ubuntu:latest@sha256:20fa2d7bb4de7723f542be5923b06c4d704370f0390e4ae9e1c833c8785644c1                                                                                           
 => => resolve docker.io/library/ubuntu:latest@sha256:20fa2d7bb4de7723f542be5923b06c4d704370f0390e4ae9e1c833c8785644c1                                                                                                  
 => [2/2] RUN apt update && apt install -y curl golang-go git                                                                                                                                                         
 => exporting to oci image format                                                                                                                                                                                      
 => => exporting layers                                                                                                                                                                                                
 => => exporting manifest sha256:e7d176b3fe165fa9788fd07f2f6fbd65c97e0144cd74d5ff44595d5be8d55ee2                                                                                                                       
 => => exporting config sha256:8807632eb0c69548a49a81f66130265fcca1b7f78b24ab4d6bff77b623aa67b8                                                                                                                         
 => => sending tarball                                                                                                                                                                                                 
 => importing to docker 
```
To run these images, jump to the section `Running images`

## Building images with a repository
Building images is most easily done when using Buildx. The build command using buildx is very similar to standard build command.
```
docker buildx build --platform <platform> -t <registry>/<repo>:<tag> --push .
```
where
- `--platform` - The platform to build for, e.g. linux/amd64, linux/arm64, linux/ppc64le, or linux/s390x. Note, when using push command, multiple platforms are supported and buildx builds the multi architecture manifest.
- `<registry>` the registry to push the image(s) into.
- `<repo>:<tag>` - the image name and tag
- `--push` - the push option pushes the image or images to the repository.  
This example assumes the Dockerfile is located in the same directory.

I'll start by building both the amd64 and arm64 images
```
docker buildx build --platform linux/amd64,linux/arm64 -t ryandejana/ryan:multi --push .
```
You should see output similar to:
```
[+] Building 510.5s (12/12) FINISHED                                                                                                                                                                                         
 => [internal] load .dockerignore                                                                                                                                                                                       
 => => transferring context: 2B                                                                                                                                                                                         
 => [internal] load build definition from Dockerfile                                                                                                                                                                    
 => => transferring dockerfile: 108B                                                                                                                                                                                    
 => [linux/arm64 internal] load metadata for docker.io/library/ubuntu:latest                                                                                                                                            
 => [linux/amd64 internal] load metadata for docker.io/library/ubuntu:latest                                                                                                                                            
 => [auth] library/ubuntu:pull token for registry-1.docker.io                                                                                                                                                           
 => [linux/arm64 1/2] FROM docker.io/library/ubuntu:latest@sha256:20fa2d7bb4de7723f542be5923b06c4d704370f0390e4ae9e1c833c8785644c1                                                                                      
 => => resolve docker.io/library/ubuntu:latest@sha256:20fa2d7bb4de7723f542be5923b06c4d704370f0390e4ae9e1c833c8785644c1                                                                                                  
 => [linux/amd64 1/2] FROM docker.io/library/ubuntu:latest@sha256:20fa2d7bb4de7723f542be5923b06c4d704370f0390e4ae9e1c833c8785644c1                                                                                      
 => => resolve docker.io/library/ubuntu:latest@sha256:20fa2d7bb4de7723f542be5923b06c4d704370f0390e4ae9e1c833c8785644c1                                                                                                  
 => CACHED [linux/arm64 2/2] RUN apt update && apt install -y curl golang-go git                                                                                                                                        
 => CACHED [linux/amd64 2/2] RUN apt update && apt install -y curl golang-go git                                                                                                                                        
 => exporting to image                                                                                                                                                                                                
 => => exporting layers                                                                                                                                                                                                 
 => => exporting manifest sha256:7f44313b7846f93f8329770715e3e534a124e8830c4b125b3577b38b8d3361ab                                                                                                                       
 => => exporting config sha256:8bd1f828155feaa6ff638f95fddd4b600570081f1558214d4ef0972764ca6ca2                                                                                                                         
 => => exporting manifest sha256:e7d176b3fe165fa9788fd07f2f6fbd65c97e0144cd74d5ff44595d5be8d55ee2                                                                                                                       
 => => exporting config sha256:8807632eb0c69548a49a81f66130265fcca1b7f78b24ab4d6bff77b623aa67b8                                                                                                                         
 => => exporting manifest list sha256:74dd8b78ab9c6e04db5e502d5d3d22016cdfbaf5fb6ba9d0cbb0c4b70c4535cd                                                                                                                  
 => => pushing layers                                                                                                                                                                                                 
 => => pushing manifest for docker.io/ryandejana/ryan:multi@sha256:74dd8b78ab9c6e04db5e502d5d3d22016cdfbaf5fb6ba9d0cbb0c4b70c4535cd                                                                                     
 => [auth] ryandejana/ryan:pull,push token for registry-1.docker.io                                                                                                                                                     
 => [auth] ryandejana/ryan:pull,push token for registry-1.docker.io 
``` 


To run these images, jump to the section `Running images from the registry`


## Running images
Running images follows standard approach.  I'll start with our amd64 images and we'll simply run the uname command to display the container's architecture.
```
docker run -it --rm  ryan:amd64 uname -a
```
You should see output similar to:
```
Linux e607b4af87ff 5.10.104-linuxkit #1 SMP Thu Mar 17 17:08:06 UTC 2022 x86_64 x86_64 x86_64 GNU/Linux
```
Notice, this is an x86_64 (amd64) based container.

Now I'll run the arm64 container.
```
docker run -it --rm  ryan:arm64 uname -a
```
And you'll get output similar to:
```
WARNING: The requested image's platform (linux/arm64) does not match the detected host platform (linux/amd64) and no specific platform was requested
Linux e2518a7e58a5 5.10.104-linuxkit #1 SMP Thu Mar 17 17:08:06 UTC 2022 aarch64 aarch64 aarch64 GNU/Linux
```
There are a couple of things here.  First we can see we are running an aarch64 container and second we have a warning that the image doesn't match the the host.

To deal with the warning, the run command can use the `--platform` arguement.
```
docker run -it --rm  --platform linux/arm64 ryan:arm64 uname -a
```
The output now no longer has the warning.
```
Linux 4065807843e4 5.10.104-linuxkit #1 SMP Thu Mar 17 17:08:06 UTC 2022 aarch64 aarch64 aarch64 GNU/Linux
```

## Running images from a registry
As before, this is very similar to our normal run command. Starting with our amd64 image, assuming you are on an amd64 mac (if you are on Apple Silcon, what happens?):
```
 docker run -it --rm ryandejana/ryan:multi uname -a
```
Which results in:
```
Linux 6a7198553252 5.10.104-linuxkit #1 SMP Thu Mar 17 17:08:06 UTC 2022 x86_64 x86_64 x86_64 GNU/Linux
```
You can also be a bit more explict and use ask for the amd64 rather levarage the default behavior.  
```
docker run -it --rm --platform linux/amd64 ryandejana/ryan:multi uname -a
```
And our expected results are seen:
```
Linux 6a7198553252 5.10.104-linuxkit #1 SMP Thu Mar 17 17:08:06 UTC 2022 x86_64 x86_64 x86_64 GNU/Linux
```
For the arm64 image, use `--platform linux/arm64` to indicate that the arm64 image is to be used.
```
 docker run -it --rm --platform linux/arm64 ryandejana/ryan:multi uname -a
```
And the now expected results:
```
Linux 3c04bc2b0a64 5.10.104-linuxkit #1 SMP Thu Mar 17 17:08:06 UTC 2022 aarch64 aarch64 aarch64 GNU/Linux
```


## Closing

Using the `--platform`` option, you can request the specific image archiecture for an arbitrary image, assuming the image suports that architecture.
For example:
```
docker run -it --rm  --platform linux/arm64 ubuntu uname -a
```
Which results in:
```
Linux 8177a64ae053 5.10.104-linuxkit #1 SMP Thu Mar 17 17:08:06 UTC 2022 aarch64 aarch64 aarch64 GNU/Linux
```
We are not limited to just amd64 or arm65, we can try s390x.
```
docker run -it --rm  --platform linux/s390x ubuntu uname -a
```
And our expected results!
```
Linux a49e4cb9b25e 5.10.104-linuxkit #1 SMP Thu Mar 17 17:08:06 UTC 2022 s390x s390x s390x GNU/Linux
```

Hopefully this will get you started with using multiple architectures with Docker Desktop and macOS.

Enjoy!
