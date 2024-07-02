FROM ghcr.io/k1d77a/sbcl.ql-and-slynk:latest AS sbcl.ql-slynk-ultralisp

ARG CORE=ql-and-slynk

WORKDIR /root/

COPY --from=ghcr.io/k1d77a/sbcl.ql-and-slynk:latest /root/quicklisp /root/quicklisp
COPY --from=ghcr.io/k1d77a/sbcl.ql-and-slynk:latest /root/$CORE /root/$CORE

RUN sbcl --core $CORE \
         --eval '(ql-dist:install-dist "http://dist.ultralisp.org/" :prompt nil)' \
         --eval '(ql:update-dist "ultralisp" :prompt nil)' \
         --eval '(slynk-loader:dump-image "ql-slynk-ultralisp")' \
         --quit

