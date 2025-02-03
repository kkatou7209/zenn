FROM node:latest

SHELL [ "/bin/bash", "-c" ]

VOLUME [ "/articles" ]

COPY ../ /articles

WORKDIR /articles

RUN apt install git