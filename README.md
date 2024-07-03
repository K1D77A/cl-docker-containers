# cl-docker-containers

Currently 4 Dockerfiles for SBCL that will build a SBCL Core file that can be used with multi stage builds.

They are available @ ghcr.io/k1d77a/

They are currently SBCL version 2.4.5

The Ultralisp versions are updated every 24h. 


## Container: ghcr.io/k1d77a/sbcl.ql-and-slynk*
### Core: ql-and-slynk

This comes with a core file containing Quicklisp and Slynk.
#### sbcl.ql-and-slynk-safe
Configures SBCL to use maximum safety settings.

#### sbcl.ql-and-slynk-default
Makes no adjustments to SBCLs compilation settings.


## Container: ghcr.io/k1d77a/sbcl.ql-slynk-ultralisp*
### Core: ql-slynk-ultralisp
This comes with a core file containing Quicklisp, Slynk and Ultralisp.

#### sbcl.ql-slynk-ultralisp-safe
Configures SBCL to use maximum safety settings.

#### sbcl.ql-slynk-ultralisp-default
Makes no adjustments to SBCLs compilation settings.


To grab one from gchr.io
```
FROM ghcr.io/k1d77a/sbcl.ql-and-slynk-safe:latest AS base
```

Here is an example of how to use the sbcl.ql-and-slynk-safe in a multi stage build 

```
# syntax=docker/dockerfile:1

FROM ghcr.io/k1d77a/sbcl.ql-and-slynk-safe AS sbcl.with-ultralisp

ARG CORE=ql-and-slynk

WORKDIR /root/
COPY --from=sbcl.ql-and-slynk-safe /root/quicklisp /root/quicklisp
COPY --from=sbcl.ql-and-slynk-safe /root/$CORE /root/$CORE
COPY my-project.asd ./
RUN sbcl --core ql-and-slynk \
         --load "my-project" \
         --eval '(ql:quickload (asdf:system-depends-on (asdf:find-system "my-project")))' \
         --eval '(slynk-loader:dump-image "my-projects-dependencies")'

```
Now you can use the SBCL Core file 'my-projects-dependencies' within the multi stage build in order to have a container that has all of the projects dependencies already loaded into SBCL.




