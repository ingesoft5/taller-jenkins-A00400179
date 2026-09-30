# Contenedor liviano que ejecuta smee-client: reenvía los eventos del
# webhook de GitHub (recibidos en un canal público de smee.io) hacia el
# endpoint /github-webhook/ de Jenkins, sin necesidad de exponer Jenkins
# a Internet.
FROM node:20-alpine

RUN npm install -g smee-client

# TODO
ENV SMEE_TARGET=http://jenkins:8080/github-webhook/

ENTRYPOINT ["sh", "-c", "smee -u \"$SMEE_URL\" -t \"$SMEE_TARGET\""]

FROM jenkins/jenkins:lts-jdk17

USER root

RUN apt-get update \
    && apt-get install -y --no-install-recommends docker.io maven curl \
    && rm -rf /var/lib/apt/lists/*

RUN mkdir -p /usr/local/lib/docker/cli-plugins \
    && curl -fsSL "https://github.com/docker/compose/releases/download/v2.29.7/docker-compose-linux-$(uname -m)" \
       -o /usr/local/lib/docker/cli-plugins/docker-compose \
    && chmod +x /usr/local/lib/docker/cli-plugins/docker-compose

COPY plugins.txt /usr/share/jenkins/ref/plugins.txt
RUN jenkins-plugin-cli --plugin-file /usr/share/jenkins/ref/plugins.txt

USER jenkins