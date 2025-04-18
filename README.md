# cl-docker-containers

This is a series of dockerfiles, their respective names describe the contents of the final 
sbcl.core file. 

I only upload the sbcl.ql-swank-ultralisp-safe to ghcr.io/k1d77a/


### Container: sbcl.ql-and-slynk*
#### sbcl.ql-and-slynk-safe

Configures SBCL to use maximum safety settings.

#### sbcl.ql-and-slynk-default
Makes no adjustments to SBCLs compilation settings.


### Container: sbcl.ql-slynk-ultralisp*
#### sbcl.ql-slynk-ultralisp-safe
Configures SBCL to use maximum safety settings.

#### sbcl.ql-slynk-ultralisp-default
Makes no adjustments to SBCLs compilation settings.

### Container: sbcl.ql-and-swank*
#### sbcl.ql-and-swank-safe

Configures SBCL to use maximum safety settings.

#### sbcl.ql-and-swank-default
Makes no adjustments to SBCLs compilation settings.


### Container: ghcr.io/k1d77a/sbcl.ql-swank-ultralisp-safe 
Configures SBCL to use maximum safety settings. Comes with swank and ultralisp. 

### Container: sbcl.ql-swank-ultralisp-default
Makes no adjustments to SBCLs compilation settings.



To grab one from gchr.io
```
FROM ghcr.io/k1d77a/sbcl.ql-swank-ultralisp-safe:latest AS base
```

Here is an example of how to use the sbcl.ql-swank-ultralisp-safe in a multi stage build 

```
# syntax=docker/dockerfile:1

FROM ghcr.io/k1d77a/sbcl.ql-swank-ultralisp-safe AS dependencies

ARG CORE=ql-swank-ultralisp

WORKDIR /root/
COPY --from=ghcr.io/k1d77a/sbcl.ql-swank-ultralisp-safe /root/quicklisp /root/quicklisp
RUN apk --no-cache add git libev-dev make gcc musl-dev sqlite-dev

WORKDIR /root/quicklisp/local-projects/
RUN git clone https://github.com/K1D77A/carlyle.git
RUN git clone https://github.com/ruricolist/spinneret.git
RUN git clone https://github.com/K1D77A/cl-migratum.git
RUN git clone https://github.com/40ants/doc.git
    
WORKDIR /root/

COPY dependencies.lisp ./dependencies.lisp
RUN sbcl --eval '(ql:quickload (uiop:read-file-form "dependencies.lisp"))' \
         --eval '(slynk-loader:dump-image "with-dependencies")'

FROM dependencies AS development
WORKDIR /root/
COPY --from=dependencies /root/quicklisp /root/quicklisp
COPY --from=dependencies /root/with-dependencies /root/with-dependencies
COPY Makefile *.asd ./
COPY src ./src
COPY tests ./tests
#COPY migrations ./migrations
COPY docs ./docs
RUN make pipeline-build-testing


FROM alpine AS development-deploy
RUN apk --no-cache add expect openssl-dev ca-certificates zstd-libs sqlite-dev libev-dev
WORKDIR /app
COPY --from=development --chmod=755 /root/my-project.bin .
WORKDIR /
#RUN chown -R 1000 /root/src #test
RUN mkdir /config
COPY src/html/static /config/src/html/static
#COPY migrations /config/migrations/
EXPOSE 43333
EXPOSE 43331
EXPOSE 43335
WORKDIR /config
CMD ["/app/my-project.bin"]
ENTRYPOINT [""]


```





