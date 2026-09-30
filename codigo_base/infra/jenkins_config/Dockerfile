# Imagen de Jenkins con el CLI de Docker instalado, para que el propio
# controlador (montando /var/run/docker.sock) pueda construir y publicar
# las imágenes del backend y del frontend durante el pipeline.
FROM jenkins/jenkins:lts-jdk17

USER root

RUN apt-get update \
    && apt-get install -y --no-install-recommends docker.io maven curl \
    && rm -rf /var/lib/apt/lists/*

COPY plugins.txt /usr/share/jenkins/ref/plugins.txt
RUN jenkins-plugin-cli --plugin-file /usr/share/jenkins/ref/plugins.txt

USER jenkins
