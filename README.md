# cl-docker-containers

Currently 2 Dockerfiles for SBCL that will build a SBCL Core file that can be used with multi stage builds.

They are available in the ghcr.io/k1d77a/

They are currently SBCL version 2.4.5



## Container: ghcr.io/k1d77a/sbcl.ql-and-slynk
### Core: ql-and-slynk


## Container: ghcr.io/k1d77a/sbcl.ql-slynk-ultralisp
### Core: ql-slynk-ultralisp 

To build
```
docker buildx build -f Dockerfile . -t sbcl.ql-and-slynk
```

To grab from gchr.io
```
FROM ghcr.io/k1d77a/sbcl.ql-and-slynk:latest AS base
```

Here is an example of how to use the sbcl.ql-and-slynk in a multi stage build 

```
# syntax=docker/dockerfile:1

FROM sbcl.ql-and-slynk AS sbcl.with-ultralisp

ARG CORE=ghcr.io/k1d77a/sbcl.ql-and-slynk #brought from the OG Dockerfile

WORKDIR /root/
COPY --from=sbcl.ql-and-slynk /root/quicklisp /root/quicklisp
COPY --from=sbcl.ql-and-slynk /root/$CORE /root/$CORE
COPY my-project.asd ./
RUN sbcl --core ql-and-slynk \
         --load "my-project" \
         --eval '(ql:quickload (asdf:system-depends-on (asdf:find-system "my-project")))' \
         --eval '(slynk-loader:dump-image "my-projects-dependencies")'

```
Now you can use the SBCL Core file 'my-projects-dependencies' within the multi stage build in order to have a container that has all of the projects dependencies already loaded into SBCL.




