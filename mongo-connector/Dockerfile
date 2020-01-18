FROM alpine:edge
LABEL maintainer="kght6123"

RUN apk update
RUN apk add --update --no-cache wget rpm
RUN wget https://dl.fedoraproject.org/pub/epel/7/x86_64/Packages/e/epel-rpm-macros-7-23.noarch.rpm
#RUN rpm -ivh epel-rpm-macros-7-23.noarch.rpm
#RUN apk add --update --no-cache epel-release
RUN apk add --update --no-cache \
    musl \
    linux-headers \
    gcc \
    g++ \
    make \
    gfortran \
    openblas-dev \
    python3 \
    python3-dev
RUN pip3 install --upgrade pip

RUN pip3 install mongo-connector[elastic5]

ENTRYPOINT [ "mongo-connector" ]
CMD [ "-m", "mongodb1", "-t", "elasticsearch:9200", "-d", "elastic2_doc_manager", "--continue-on-error", "--auto-commit-interval=0" ]
