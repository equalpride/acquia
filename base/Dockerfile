FROM ubuntu:12.04.5
MAINTAINER Nicholas Alipaz <nicholas.alipaz@heremedia.com>

# Change to root user
USER root

COPY container /

RUN readonly UBUNTU_RELEASE_NAME="$(cat /etc/lsb-release | grep CODENAME | cut -d '=' -f2)" &&\
 sed -i -r\
 -e "1s/^/deb mirror:\/\/mirrors\.ubuntu\.com\/mirrors\.txt "${UBUNTU_RELEASE_NAME}"-security main restricted universe multiverse\n\n/"\
 -e "1s/^/deb mirror:\/\/mirrors\.ubuntu\.com\/mirrors\.txt "${UBUNTU_RELEASE_NAME}"-backports main restricted universe multiverse\n/"\
 -e "1s/^/deb mirror:\/\/mirrors\.ubuntu\.com\/mirrors\.txt "${UBUNTU_RELEASE_NAME}"-updates main restricted universe multiverse\n/"\
 -e "1s/^/deb mirror:\/\/mirrors\.ubuntu\.com\/mirrors\.txt "${UBUNTU_RELEASE_NAME}" main restricted universe multiverse\n/"\
 -e "1s/^/\# Enable Ubuntu mirrors and multiverse\n/"\
 /etc/apt/sources.list

RUN apt-get update &&\
 apt-get upgrade -y &&\
 cat /tmp/UBUNTU_PACKAGES | xargs apt-get install --no-install-recommends -y &&\
 apt-get clean

############ Setup Dumb Init ############
ENV DUMB_INIT_VERSION 1.2.0
WORKDIR /tmp/dumb-init
RUN curl -sSLO\
 https://github.com/Yelp/dumb-init/releases/download/v${DUMB_INIT_VERSION}/dumb-init_${DUMB_INIT_VERSION}_amd64.deb &&\
 dpkg -i dumb-init_${DUMB_INIT_VERSION}_amd64.deb &&\
 rm -rf /tmp/dumb-init
