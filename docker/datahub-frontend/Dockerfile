# Defining environment
ARG APP_ENV=prod

FROM alpine:3 AS base

RUN addgroup -S datahub && adduser -S datahub -G datahub

# Upgrade Alpine and base packages
# PFP-260: Upgrade Sqlite to >=3.28.0-r0 to fix https://security.snyk.io/vuln/SNYK-ALPINE39-SQLITE-449762
RUN apk --no-cache --update-cache --available upgrade \
    && apk --no-cache add curl sqlite libc6-compat java-snappy \
    && apk --no-cache add openjdk11-jre --repository=http://dl-cdn.alpinelinux.org/alpine/edge/community \
    && apk --no-cache add jattach --repository http://dl-cdn.alpinelinux.org/alpine/edge/community/

ENV LD_LIBRARY_PATH="/lib:/lib64"

FROM base as prod-install

COPY ./datahub-frontend.zip /
RUN unzip datahub-frontend.zip && rm datahub-frontend.zip
COPY ./docker/monitoring/client-prometheus-config.yaml /datahub-frontend/
RUN chown -R datahub:datahub /datahub-frontend && chmod 755 /datahub-frontend

ENV JMX_VERSION=0.18.0
RUN wget https://github.com/open-telemetry/opentelemetry-java-instrumentation/releases/download/v1.24.0/opentelemetry-javaagent.jar -O opentelemetry-javaagent.jar \
    && wget https://repo1.maven.org/maven2/io/prometheus/jmx/jmx_prometheus_javaagent/${JMX_VERSION}/jmx_prometheus_javaagent-${JMX_VERSION}.jar -O jmx_prometheus_javaagent.jar

FROM base as dev-install
# Dummy stage for development. Assumes code is built on your machine and mounted to this image.
# See this excellent thread https://github.com/docker/cli/issues/1134
VOLUME [ "/datahub-frontend" ]

FROM ${APP_ENV}-install as final
COPY ./docker/datahub-frontend/start.sh /
RUN chown datahub:datahub /start.sh && chmod 755 /start.sh
USER datahub

ARG SERVER_PORT=9002
ENV SERVER_PORT=$SERVER_PORT
RUN echo $SERVER_PORT
EXPOSE $SERVER_PORT

HEALTHCHECK --start-period=2m --retries=4 CMD curl --fail http://localhost:$SERVER_PORT/admin || exit 1
CMD ./start.sh
