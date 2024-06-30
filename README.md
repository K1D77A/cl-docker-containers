# cl-docker-containers

Currently 1 Dockerfile for SBCL that will build a SBCL Core file that can be used with multi stage builds.

To build
```
docker buildx build -f Dockerfile . -t sbcl.ql-and-slynk
```

Here is an example of how to use the sbcl.ql-and-slynk in a multi stage build 

```
# syntax=docker/dockerfile:1

FROM sbcl.ql-and-slynk AS sbcl.with-ultralisp

ARG CORE=ql-and-slynk #brought from the OG Dockerfile

WORKDIR /root/
COPY --from=sbcl.ql-and-slynk /root/quicklisp /root/quicklisp
COPY --from=sbcl.ql-and-slynk /root/$CORE /root/$CORE
    
RUN sbcl --core $CORE \
         --eval '(ql-dist:install-dist "http://dist.ultralisp.org/" :prompt nil)' \
         --eval '(ql:update-dist "ultralisp" :prompt nil)' \
         --eval '(slynk-loader:dump-image "my-new-core")'
```
This will create a container to be used in the next stage where you can copy 'my-new-core' into the subsequent stage and build on top of that.


Here is an example of how you can load all of the dependencies for a CL project into an SBCL Core without actually loading the system itself. This is very useful within Docker because we all know how long it can take to compile libraries like Ironclad.


```
COPY my-project.asd ./
RUN sbcl --core ql-and-slynk \
         --load "my-project" \
         --eval '(ql:quickload (asdf:system-depends-on (asdf:find-system "my-project")))' \
         --eval '(slynk-loader:dump-image "my-projects-dependencies")'

```
Now you can use the SBCL Core file 'my-projects-dependencies' within the multi stage build in order to have a container that has all of the projects dependencies already loaded into SBCL. 



