# # BUILD STEP # #
FROM node:18-bullseye AS builder
ENV CRONICLE_VERSION=0.9.83
WORKDIR /opt/cronicle
RUN curl -L -o /tmp/Cronicle-${CRONICLE_VERSION}.tar.gz https://github.com/jhuckaby/Cronicle/archive/refs/tags/v${CRONICLE_VERSION}.tar.gz
RUN tar zxvf /tmp/Cronicle-${CRONICLE_VERSION}.tar.gz -C /tmp/ && \
    mv /tmp/Cronicle-${CRONICLE_VERSION}/* . && \
    rm -rf /tmp/* && \
    yarn
COPY ./patches /tmp/patches
RUN patch -p3 < /tmp/patches/engine.patch lib/engine.js
COPY docker-entrypoint.js ./bin/


# # FINAL IMAGE # #
FROM python:3.13-alpine
ARG USERNAME=cronicle

# Install requirements & additional reqs for Chrome (https://pptr.dev/troubleshooting#chrome-doesnt-launch-on-linux)
RUN apk update --no-cache && apk add --no-cache procps curl bash nodejs \
		ca-certificates \
    fonts-liberation \
    libasound2 \
    libatk-bridge2.0-0 \
    libatk1.0-0 \
    libc6 \
    libcairo2 \
    libcups2 \
    libdbus-1-3 \
    libexpat1 \
    libfontconfig1 \
    libgbm1 \
    libgcc1 \
    libglib2.0-0 \
    libgtk-3-0 \
    libnspr4 \
    libnss3 \
    libpango-1.0-0 \
    libpangocairo-1.0-0 \
    libstdc++6 \
    libx11-6 \
    libx11-xcb1 \
    libxcb1 \
    libxcomposite1 \
    libxcursor1 \
    libxdamage1 \
    libxext6 \
    libxfixes3 \
    libxi6 \
    libxrandr2 \
    libxrender1 \
    libxss1 \
    libxtst6 \
    lsb-release \
    wget \
    xdg-utils
RUN addgroup ${USERNAME} && adduser ${USERNAME} -D -G ${USERNAME} --shell=/bin/bash

COPY --from=builder --chown=${USERNAME}:${USERNAME} /opt/cronicle/ /opt/cronicle/

ENV CRONICLE_foreground=1
ENV CRONICLE_echo=1
ENV CRONICLE_color=1
ENV debug_level=1
ENV HOSTNAME=main

USER cronicle
WORKDIR /opt/cronicle
RUN node bin/build.js dist && bin/control.sh setup

CMD ["node", "bin/docker-entrypoint.js"]