FROM ghcr.io/k1d77a/sbcl.ql-and-slynk-default:latest AS build

WORKDIR /root/

COPY --from=ghcr.io/k1d77a/sbcl.ql-and-slynk-default:latest /root/quicklisp /root/quicklisp
RUN sbcl --core $CORE \
         --eval '(ql-dist:install-dist "http://dist.ultralisp.org/" :prompt nil)' \
         --eval '(ql:update-dist "ultralisp" :prompt nil)' \
         --eval '(slynk-loader:dump-image "sbcl.core")' \
         --quit

FROM alpine AS sbcl.ql-slynk-ultralisp-default
RUN apk add --no-cache --update zstd-dev
WORKDIR /root/
COPY --from=build /usr/local/lib/sbcl /usr/lib/sbcl
RUN rm /usr/lib/sbcl/sbcl.core
COPY --from=build /usr/local/bin/sbcl /usr/bin/sbcl
COPY --from=build /root/quicklisp /root/quicklisp
COPY --from=build /root/sbcl.core /usr/lib/sbcl/
