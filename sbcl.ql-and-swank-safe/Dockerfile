# syntax=docker/dockerfile:1

FROM alpine AS build
ARG VERSION=2.4.5

RUN apk add --update build-base linux-headers zstd-dev git sbcl;

ADD https://github.com/sbcl/sbcl/archive/sbcl-$VERSION.tar.gz sbcl.tar.gz


RUN tar xvfz sbcl.tar.gz && rm sbcl.tar.gz
WORKDIR /sbcl-sbcl-$VERSION
RUN echo "\"$VERSION\"" > version.lisp-expr
RUN SBCL_HOME=/usr/lib64/sbcl/ sh make.sh --fancy --dynamic-space-size=4Gb
RUN INSTALL_ROOT=/usr/ sh install.sh
RUN ls /usr/lib/sbcl
WORKDIR /root/
ADD https://beta.quicklisp.org/quicklisp.lisp quicklisp.lisp
RUN sbcl --load quicklisp.lisp --eval '(quicklisp-quickstart:install)'

WORKDIR /root/quicklisp/local-projects/
RUN git clone https://github.com/slime/slime.git

WORKDIR /root/
RUN echo "(sb-ext:restrict-compiler-policy 'safety 3 3)" >> setup.lisp
RUN echo "(sb-ext:restrict-compiler-policy 'space 1 1)" >> setup.lisp
RUN echo "(sb-ext:restrict-compiler-policy 'speed 0 0)" >> setup.lisp
RUN echo "(load \"quicklisp/setup.lisp\")" >> setup.lisp
RUN echo "(load \"quicklisp/local-projects/slime/swank-loader.lisp\")" >> setup.lisp
RUN echo "(setf swank-loader::*contribs*" >> setup.lisp
RUN echo "      '(swank-arglists" >> setup.lisp
RUN echo "        swank-asdf" >> setup.lisp
RUN echo "        swank-buffer-streams" >> setup.lisp
RUN echo "        swank-clipboard" >> setup.lisp
RUN echo "        swank-c-p-c" >> setup.lisp
RUN echo "        swank-fancy-inspector" >> setup.lisp
RUN echo "        swank-fuzzy" >> setup.lisp
RUN echo "        swank-hyperdoc" >> setup.lisp
RUN echo "        swank-indentation" >> setup.lisp
RUN echo "        swank-listener-hooks" >> setup.lisp
RUN echo "        swank-macrostep" >> setup.lisp
RUN echo "        swank-media" >> setup.lisp
RUN echo "        swank-mrepl" >> setup.lisp
RUN echo "        swank-package-fu" >> setup.lisp
RUN echo "        swank-presentations" >> setup.lisp
RUN echo "        swank-presentation-streams" >> setup.lisp
RUN echo "        swank-quicklisp" >> setup.lisp
RUN echo "        swank-repl" >> setup.lisp
RUN echo "        swank-sbcl-exts" >> setup.lisp
RUN echo "        swank-snapshot" >> setup.lisp
RUN echo "        swank-sprof" >> setup.lisp
RUN echo "        swank-trace-dialog" >> setup.lisp
RUN echo "        swank-util))" >> setup.lisp
RUN echo "(ql:update-all-dists :prompt nil)" >> setup.lisp
RUN echo "(swank-loader:dump-image \"sbcl.core\" :load-contribs t)" >> setup.lisp

RUN sbcl --load setup.lisp --quit

FROM alpine AS sbcl.ql-and-swank-safe
RUN apk add --no-cache --update zstd-dev
WORKDIR /root/
COPY --from=build /usr/lib/sbcl /usr/lib/sbcl
RUN rm /usr/lib/sbcl/sbcl.core
COPY --from=build /usr/bin/sbcl /usr/bin/sbcl
COPY --from=build /root/quicklisp /root/quicklisp
COPY --from=build /root/sbcl.core /usr/lib/sbcl/