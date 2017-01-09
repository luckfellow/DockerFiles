# Pull base image.
FROM ubuntu:16.04

MAINTAINER luckfellow <luckfellow@gmail.com>

ENV DEBIAN_FRONTEND noninteractive

USER root

# Install.
RUN \
  sed -i 's/# \(.*multiverse$\)/\1/g' /etc/apt/sources.list && \
  apt-get update && \
  apt-get install -y build-essential && \
  apt-get install -y ccache xfsprogs && \
  apt-get install -y software-properties-common && \
  apt-get install -y byobu curl git htop man zip unzip gzip vim wget && \
  apt-get install -y subversion sudo expect && \
  apt-get install -y python-software-properties && \
  rm -rf /var/lib/apt/lists/*

# Install Java.
RUN \
  apt-key adv --keyserver keyserver.ubuntu.com --verbose --refresh-keys && \
  echo oracle-java8-installer shared/accepted-oracle-license-v1-1 select true | debconf-set-selections && \
  add-apt-repository -y ppa:webupd8team/java && \
  apt-get update && \
  apt-get install -y oracle-java8-installer && \
  rm -rf /var/lib/apt/lists/* && \
  rm -rf /var/cache/oracle-jdk8-installer

# Define working directory.
WORKDIR /data

# Define commonly used JAVA_HOME variable
ENV JAVA_HOME /usr/lib/jvm/java-8-oracle

################################################################################
# Install Jenkins.
# made from https://github.com/jenkinsci/docker/blob/83ce6f6070f1670563a00d0f61d04edd62b78f4f/Dockerfile
ENV JENKINS_HOME /var/jenkins_home
ENV JENKINS_SLAVE_AGENT_PORT 50000

ARG user=jenkins
ARG group=jenkins
ARG uid=1001
ARG gid=1001

# Jenkins is run with user `jenkins`, uid = 1001
# If you bind mount a volume from the host or a data container,
# ensure you use the same uid
# and change password for jenkins account.
RUN \
  groupadd -g ${gid} ${group} && \
  useradd -d "$JENKINS_HOME" -u ${uid} -g ${gid} -m -s /bin/bash ${user} && \
  echo "jenkins:jenkins" | chpasswd


# Jenkins home directory is a volume, so configuration and build history
# can be persisted and survive image upgrades
VOLUME /var/jenkins_home

# `/usr/share/jenkins/ref/` contains all reference configuration we want
# to set on a fresh new installation. Use it to bundle additional plugins
# or config file with your custom jenkins Docker image.
RUN mkdir -p /usr/share/jenkins/ref/init.groovy.d

ENV TINI_VERSION 0.13.1
ENV TINI_SHA 0f78709a0e3c80e7c9119fdc32c2bc0f4cfc4cab

# Use tini as subreaper in Docker container to adopt zombie processes
RUN curl -fsSL https://github.com/krallin/tini/releases/download/v${TINI_VERSION}/tini-static-amd64 -o /bin/tini && chmod +x /bin/tini \
  && echo "$TINI_SHA  /bin/tini" | sha1sum -c -
RUN chmod +x /bin/tini

COPY init.groovy /usr/share/jenkins/ref/init.groovy.d/tcp-slave-agent-port.groovy

# jenkins version being bundled in this docker image
ARG JENKINS_VERSION
ENV JENKINS_VERSION ${JENKINS_VERSION:-2.32.1}

# jenkins.war checksum, download will be validated using it
ARG JENKINS_SHA=1b65dc498ba7ab1f5cce64200b920a8716d90834

# Can be used to customize where jenkins.war get downloaded from
ARG JENKINS_URL=https://repo.jenkins-ci.org/public/org/jenkins-ci/main/jenkins-war/${JENKINS_VERSION}/jenkins-war-${JENKINS_VERSION}.war

# could use ADD but this one does not check Last-Modified header
# see https://github.com/docker/docker/issues/8331
RUN curl -fsSL ${JENKINS_URL} -o /usr/share/jenkins/jenkins.war \
  && echo "${JENKINS_SHA}  /usr/share/jenkins/jenkins.war" | sha1sum -c -

ENV JENKINS_UC https://updates.jenkins.io
# add jenkins account in sudoers
# change timezone from UTC to Asia/Seoul
RUN \
  chown -R ${user} "$JENKINS_HOME" /usr/share/jenkins/ref && \
  adduser jenkins sudo && \
  chmod u+w /etc/sudoers && \
  echo "jenkins ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers && \
  chmod u-w /etc/sudoers && \
  ln -sf /usr/share/zoneinfo/Asia/Seoul /etc/localtime

# for main web interface:
EXPOSE 8080

# will be used by attached slave agents:
EXPOSE 50000

ENV COPY_REFERENCE_FILE_LOG $JENKINS_HOME/copy_reference_file.log

USER ${user}

COPY jenkins-support /usr/local/bin/jenkins-support
COPY jenkins.sh /usr/local/bin/jenkins.sh
ENTRYPOINT ["/bin/tini", "--", "/usr/local/bin/jenkins.sh"]

USER root
RUN chmod +x /usr/local/bin/jenkins.sh

USER ${user}

# from a derived Dockerfile, can use `RUN plugins.sh active.txt` to setup /usr/share/jenkins/ref/plugins from a support bundle
COPY plugins.sh /usr/local/bin/plugins.sh
COPY install-plugins.sh /usr/local/bin/install-plugins.sh
################################################################################</luckfellow@gmail.com>