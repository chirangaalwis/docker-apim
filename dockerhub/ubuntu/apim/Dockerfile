# ------------------------------------------------------------------------
#
# Copyright 2019 WSO2, Inc. (http://wso2.com)
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
# http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License
#
# ------------------------------------------------------------------------

# set base Docker image to Ubuntu Docker image
FROM ubuntu:18.04
LABEL maintainer="WSO2 Docker Maintainers <dev@wso2.org>"

# set Docker image build arguments
# build arguments for user/group configurations
ARG USER=wso2carbon
ARG USER_ID=802
ARG USER_GROUP=wso2
ARG USER_GROUP_ID=802
ARG USER_HOME=/home/${USER}
# build arguments for java installation
ARG JDK_VERSION
ARG JDK_CHECKSUM
ARG JAVA_HOME=/usr/java
# build arguments for wso2 product installation
ARG WSO2_SERVER_NAME=wso2am
ARG WSO2_SERVER_VERSION=2.6.0
ARG WSO2_SERVER=${WSO2_SERVER_NAME}-${WSO2_SERVER_VERSION}
ARG WSO2_SERVER_HOME=${USER_HOME}/${WSO2_SERVER}
# build arguments for external artifacts
ARG SQL_CONNECTOR_VERSION=5.1.47
# build argument for MOTD
ARG MOTD="\n\
Welcome to WSO2 Docker resources.\n\
------------------------------------ \n\
This Docker container comprises of a WSO2 product, running with its latest GA release \n\
which is under the Apache License, Version 2.0. \n\
Read more about Apache License, Version 2.0 here @ http://www.apache.org/licenses/LICENSE-2.0.\n"

# create the user and group
RUN \
    groupadd --system -g ${USER_GROUP_ID} ${USER_GROUP} \
    && useradd --system --create-home --home-dir ${USER_HOME} --no-log-init -g ${USER_GROUP_ID} -u ${USER_ID} ${USER}

# install required packages
RUN \
    apt-get update \
    && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
        java-common \
        netcat \
        unzip \
        wget \
    && rm -rf /var/lib/apt/lists/* \
    && echo '[ ! -z "${TERM}" -a -r /etc/motd ] && cat /etc/motd' >> /etc/bash.bashrc; echo "${MOTD}" > /etc/motd

# install the wso2 product distribution
RUN \
    wget --no-check-certificate -O ${WSO2_SERVER}.zip "https://bintray.com/wso2/binary/download_file?file_path=${WSO2_SERVER}.zip" \
    && unzip -d ${USER_HOME} ${WSO2_SERVER}.zip \
    && chown wso2carbon:wso2 -R ${WSO2_SERVER_HOME} \
    && rm -rf ${WSO2_SERVER_HOME}/jdk \
    && rm -f ${WSO2_SERVER}.zip

# install Amazon Corretto
RUN \
    mkdir -p ${JAVA_HOME} \
    && wget --no-check-certificate -O amzon-corretto-linux-x64.tar.gz https://d2znqt9b1bc64u.cloudfront.net/amazon-corretto-${JDK_VERSION}-linux-x64.tar.gz \
    && echo "${JDK_CHECKSUM} amzon-corretto-linux-x64.tar.gz" | md5sum -c - \
    && tar -xf amzon-corretto-linux-x64.tar.gz -C ${JAVA_HOME} --strip-components=1 \
    && chown wso2carbon:wso2 -R ${JAVA_HOME} \
    && rm -f amzon-corretto-linux-x64.tar.gz

# add mysql jdbc connector to server home as a third party library
ADD --chown=wso2carbon:wso2 http://central.maven.org/maven2/mysql/mysql-connector-java/${SQL_CONNECTOR_VERSION}/mysql-connector-java-${SQL_CONNECTOR_VERSION}.jar ${WSO2_SERVER_HOME}/repository/components/lib/

# copy docker image's entrypoint script to user home
COPY --chown=wso2carbon:wso2 docker-entrypoint.sh ${USER_HOME}/

# set the user and work directory
USER ${USER_ID}
WORKDIR ${USER_HOME}

# set environment variables
ENV JAVA_HOME=${JAVA_HOME} \
    PATH=${JAVA_HOME}/bin:$PATH \
    WORKING_DIRECTORY=${USER_HOME} \
    WSO2_SERVER_HOME=${WSO2_SERVER_HOME}

# expose ports
EXPOSE 9763 9443 9999 11111 8280 8243 5672 9711 9611 7711 7611 10397 9099

# initiate container and start WSO2 Carbon server
ENTRYPOINT ["/home/wso2carbon/docker-entrypoint.sh"]
