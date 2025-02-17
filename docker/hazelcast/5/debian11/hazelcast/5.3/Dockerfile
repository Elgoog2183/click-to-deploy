# Copyright 2022 Google LLC
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

FROM marketplace.gcr.io/google/c2d-debian11

ENV DEBUG_DOCKER_ENTRYPOINT false

ENV HAZELCAST_VERSION 5.3.1
ENV C2D_RELEASE 5.3.1

# Versions of Hazelcast and Hazelcast plugins
ARG HZ_VERSION=5.3.1
ARG HZ_VERSION_SHA=

# Build constants
ARG HZ_HOME="/opt/hazelcast"

# Runtime constants / variables
ENV HZ_HOME="${HZ_HOME}" \
    CLASSPATH_DEFAULT="${HZ_HOME}/*" \
    JAVA_OPTS_DEFAULT="-Djava.net.preferIPv4Stack=true -Dhz.network.rest-api.enabled=true -Dhz.network.rest-api.endpoint-groups.DATA.enabled=true -Dhazelcast.rest.enabled=true -Dhazelcast.logging.type=log4j2 -XX:MaxRAMPercentage=80.0 -XX:MaxGCPauseMillis=5 --add-modules java.se --add-exports java.base/jdk.internal.ref=ALL-UNNAMED --add-opens java.base/java.lang=ALL-UNNAMED --add-opens java.base/java.nio=ALL-UNNAMED --add-opens java.base/sun.nio.ch=ALL-UNNAMED --add-opens java.management/sun.management=ALL-UNNAMED --add-opens jdk.management/com.sun.management.internal=ALL-UNNAMED" \
    PROMETHEUS_PORT="" \
    PROMETHEUS_CONFIG="${HZ_HOME}/config/jmx_agent_config.yaml" \
    CLASSPATH="" \
    JAVA_OPTS="" \
    HAZELCAST_CONFIG=config/hazelcast-docker.xml \
    LANG=C.UTF-8 \
    PATH=${HZ_HOME}/bin:$PATH

# Expose port
EXPOSE 5701

COPY *.jar get-hz-all-url.sh ${HZ_HOME}/

# Install
RUN echo "Installing new APK packages" \
    && apt-get --allow-releaseinfo-change update \
    && apt-get -y install openjdk-11-jdk-headless bash curl libxml2-utils zip unzip procps net-tools telnet inetutils-ping \
    && echo "Downloading Hazelcast distribution zip..." \
    && HAZELCAST_ZIP_URL=$(${HZ_HOME}/get-hz-all-url.sh) \
    && curl -sf -L ${HAZELCAST_ZIP_URL} --output ${HZ_HOME}/hazelcast-distribution.zip \
    && unzip -qq ${HZ_HOME}/hazelcast-distribution.zip 'hazelcast-*/**' -d ${HZ_HOME}/tmp/ \
    && mv ${HZ_HOME}/tmp/*/* ${HZ_HOME}/ \
    && echo "Setting Pardot ID to 'docker'" \
    && echo 'hazelcastDownloadId=docker' > "${HZ_HOME}/lib/hazelcast-download.properties" \
    && echo "Granting read permission to ${HZ_HOME}" \
    && chmod -R +r ${HZ_HOME} \
    && echo "Cleaning APK packages and redundant files/folders" \
    && rm -rf /var/cache/apk/* ${HZ_HOME}/get-hz-all-url.sh ${HZ_HOME}/hazelcast-distribution.zip ${HZ_HOME}/tmp


RUN echo "Adding license to /usr/share/hazelcast directory" \
    && mkdir -p /usr/share/hazelcast/ \
    && curl https://raw.githubusercontent.com/hazelcast/hazelcast/master/LICENSE -o /usr/share/hazelcast/LICENSE


COPY log4j2.properties jmx_agent_config.yaml ${HZ_HOME}/config/

WORKDIR ${HZ_HOME}

# Start Hazelcast server
CMD ["hz", "start"]
