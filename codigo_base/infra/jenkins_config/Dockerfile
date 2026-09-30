FROM jenkins/jenkins:lts-jdk17

USER root

RUN apt-get update \
    && apt-get install -y --no-install-recommends maven curl \
    && rm -rf /var/lib/apt/lists/*

RUN curl -fsSL "https://download.docker.com/linux/static/stable/$(uname -m)/docker-27.3.1.tgz" \
    | tar xz --strip-components=1 -C /usr/local/bin docker/docker

RUN mkdir -p /usr/local/lib/docker/cli-plugins \
    && curl -fsSL "https://github.com/docker/compose/releases/download/v2.29.7/docker-compose-linux-$(uname -m)" \
       -o /usr/local/lib/docker/cli-plugins/docker-compose \
    && chmod +x /usr/local/lib/docker/cli-plugins/docker-compose

COPY plugins.txt /usr/share/jenkins/ref/plugins.txt
RUN jenkins-plugin-cli --plugin-file /usr/share/jenkins/ref/plugins.txt

USER jenkins
