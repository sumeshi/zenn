FROM node:21-bookworm

RUN apt -y update && apt upgrade -qqy && apt -y install \
    git \
    fish \
    && apt clean

RUN chsh -s /usr/bin/fish
RUN npm install -g zenn-cli textlint

USER node
WORKDIR /home/node/workspace
