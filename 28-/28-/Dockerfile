FROM alpine:latest

ARG GLIBC_VERSION="2.28-r0"

ENV ANDROID_SDK_ROOT /home/alpine/android
ENV ANDROID_HOME /home/alpine/android

RUN apk -U update && apk -U add \
  openjdk8 \
  git \
  wget \
  zip \
  openssh-client \
  && wget https://alpine-pkgs.sgerrand.com/sgerrand.rsa.pub -O /etc/apk/keys/sgerrand.rsa.pub \
	&& wget https://github.com/sgerrand/alpine-pkg-glibc/releases/download/${GLIBC_VERSION}/glibc-${GLIBC_VERSION}.apk -O /tmp/glibc.apk \
	&& wget https://github.com/sgerrand/alpine-pkg-glibc/releases/download/${GLIBC_VERSION}/glibc-bin-${GLIBC_VERSION}.apk -O /tmp/glibc-bin.apk \
	&& apk add /tmp/glibc.apk /tmp/glibc-bin.apk \
  && rm -rf /tmp/* \
	&& rm -rf /var/cache/apk/* \
  #creation of android group and alpine user
  && addgroup -S android && adduser -DS alpine -G android

USER alpine

WORKDIR /home/alpine

#get cmdline-tools from web
RUN mkdir temp \
  && wget https://dl.google.com/android/repository/commandlinetools-linux-6858069_latest.zip \
  && unzip commandlinetools-linux-6858069_latest.zip -d ~/temp && rm commandlinetools-linux-6858069_latest.zip \
#creation of structure required
  && mkdir -p ~/android/cmdline-tools/tools \
  && mv ~/temp/cmdline-tools/* ~/android/cmdline-tools/tools/ \
  && rm -rf ~/temp \
  && mkdir .ssh \
#creation of build-tools
  && yes | ~/android/cmdline-tools/tools/bin/sdkmanager --licenses
