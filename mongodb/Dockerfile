FROM alpine:edge
LABEL maintainer="kght6123"

RUN echo 'http://dl-3.alpinelinux.org/alpine/edge/main' >> /etc/apk/repositories
RUN echo 'http://dl-3.alpinelinux.org/alpine/edge/community' >> /etc/apk/repositories
RUN echo 'http://dl-3.alpinelinux.org/alpine/edge/testing' >> /etc/apk/repositories
RUN apk update
RUN apk add --update --no-cache mongodb

VOLUME /data/db
EXPOSE 27017 27017

ENTRYPOINT [ "mongod" ]
CMD [ "--bind_ip", "0.0.0.0" ]