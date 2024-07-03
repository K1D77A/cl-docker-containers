# syntax=docker/dockerfile:1

FROM alpine AS sbcl.ql-and-slynk-default
ARG VERSION=2.4.5

RUN apk add --update build-base linux-headers zstd-dev git sbcl;

ADD https://github.com/sbcl/sbcl/archive/sbcl-$VERSION.tar.gz sbcl.tar.gz

RUN tar xvfz sbcl.tar.gz && rm sbcl.tar.gz
WORKDIR /sbcl-sbcl-$VERSION
RUN echo "\"$VERSION\"" > version.lisp-expr
RUN SBCL_HOME=/usr/lib64/sbcl/ sh make.sh --fancy --dynamic-space-size=4Gb
RUN sh install.sh
WORKDIR /root/
ADD https://beta.quicklisp.org/quicklisp.lisp quicklisp.lisp
RUN sbcl --load quicklisp.lisp --eval '(quicklisp-quickstart:install)'

WORKDIR /root/quicklisp/local-projects/
RUN git clone https://github.com/joaotavora/sly.git

WORKDIR /root/
RUN echo "(load \"quicklisp/setup.lisp\")" >> setup.lisp
RUN echo "(load \"quicklisp/local-projects/sly/slynk/slynk-loader.lisp\")" >> setup.lisp
RUN echo "(ql:update-all-dists :prompt nil)" >> setup.lisp
RUN echo "(slynk-loader:dump-image \"sbcl.core\")" >> setup.lisp


RUN sbcl --load setup.lisp --quit

FROM alpine AS sbcl.ql-and-slynk-safe
RUN apk add --no-cache --update zstd-dev
WORKDIR /root/
COPY --from=build /usr/local/lib/sbcl /usr/lib/sbcl
RUN rm /usr/lib/sbcl/sbcl.core
COPY --from=build /usr/local/bin/sbcl /usr/bin/sbcl
COPY --from=build /root/quicklisp /root/quicklisp
COPY --from=build /root/sbcl.core /usr/lib/sbcl/
