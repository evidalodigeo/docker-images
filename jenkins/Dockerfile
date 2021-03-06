FROM esthervidal78/centos7-dev-ds-infra-042019:latest
MAINTAINER "E.Vidal" <esther.vidal@edreamsodigeo.com>

ARG user=jenkins
ARG group=jenkins
ARG uid=1001
ARG gid=1001
ARG http_port=8080
ARG agent_port=50000
ARG JENKINS_HOME=/var/lib/jenkins_home

ENV JENKINS_HOME $JENKINS_HOME
ENV JENKINS_SLAVE_AGENT_PORT ${agent_port}

RUN yum -y install zip unzip

# Jenkins is run with user `jenkins`, uid = 1000
# If you bind mount a volume from the host or a data container,
# ensure you use the same uid
RUN mkdir -p $JENKINS_HOME \
  && chown ${uid}:${gid} $JENKINS_HOME \
  && groupadd -g ${gid} ${group} \
  && useradd -d "$JENKINS_HOME" -u ${uid} -g ${gid} -m -s /bin/bash ${user}

RUN mkdir -p $JENKINS_HOME/data \
  && chown -R ${uid}:${gid} $JENKINS_HOME/data

RUN mkdir -p /jenkins_data/backup \
  && chown -R ${uid}:${gid} $JENKINS_HOME/data

RUN mkdir -p ${JENKINS_HOME}_restore \
  && chown -R ${uid}:${gid} ${JENKINS_HOME}_restore \
  && chmod 0777 /var/lib/
# Jenkins home directory is a volume, so configuration and build history
# can be persisted and survive image upgrades
VOLUME $JENKINS_HOME

# `/usr/share/jenkins/ref/` contains all reference configuration we want
# to set on a fresh new installation. Use it to bundle additional plugins
# or config file with your custom jenkins Docker image.
RUN mkdir -p /usr/share/jenkins/ref/init.groovy.d

# jenkins version being bundled in this docker image
ARG JENKINS_VERSION
ENV JENKINS_VERSION ${JENKINS_VERSION:-2.195}

# jenkins.war checksum, download will be validated using it
ARG JENKINS_SHA=6030b16e190c377b65a42999f1d5a97b290774debcea95c17d51c2a34eaab881
#6030b16e190c377b65a42999f1d5a97b290774debcea95c17d51c2a34eaab881

# Can be used to customize where jenkins.war get downloaded from
ARG JENKINS_URL=https://repo.jenkins-ci.org/public/org/jenkins-ci/main/jenkins-war/${JENKINS_VERSION}/jenkins-war-${JENKINS_VERSION}.war

# could use ADD but this one does not check Last-Modified header neither does it allow to control checksum
# see https://github.com/docker/docker/issues/8331
RUN curl -fsSL ${JENKINS_URL} -o /usr/share/jenkins/jenkins.war

ENV JENKINS_UC https://updates.jenkins.io
ENV JENKINS_UC_EXPERIMENTAL=https://updates.jenkins.io/experimental
ENV JENKINS_INCREMENTALS_REPO_MIRROR=https://repo.jenkins-ci.org/incrementals
RUN chown -R ${user} "$JENKINS_HOME" /usr/share/jenkins/ref

# for main web interface:
EXPOSE ${http_port}

# will be used by attached slave agents:
EXPOSE ${agent_port}

ENV COPY_REFERENCE_FILE_LOG $JENKINS_HOME/copy_reference_file.log
USER ${user}

COPY jenkins-support /usr/local/bin/jenkins-support
COPY jenkins.sh /usr/local/bin/jenkins.sh
COPY plugins/install-plugins.sh /usr/local/bin/install-plugins.sh
COPY plugins/plugins.txt /usr/share/jenkins/ref/plugins.txt
RUN /usr/local/bin/install-plugins.sh < /usr/share/jenkins/ref/plugins.txt

COPY backup/backup_20190918_1601.zip /jenkins_data/backup

ENTRYPOINT ["/usr/local/bin/jenkins.sh"]
