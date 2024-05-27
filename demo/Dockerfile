# Stage 1: Build stage
FROM gradle:jdk17 AS builder

# Copy app files and Gradle dependencies
COPY --chown=gradle:gradle . /home/gradle/project
WORKDIR /home/gradle/project
RUN ./gradlew build --exclude-task test

# Stage 2: Prepare the container
FROM openjdk:17-jdk-alpine

# Set up workspace in /app
WORKDIR /app

# Copy app JAR built in stage 1
COPY --from=builder /home/gradle/project/build/libs/demo-0.0.1-SNAPSHOT.jar /app/demo.jar

# Copy files to the container in /app/crypto
COPY crypto/serverErat.ks /app/crypto/serverErat.ks
COPY crypto/ec-cacert.pem /app/crypto/ec-cacert-pem
COPY crypto/ec-cakey.jks /app/crypto/ec-cakey.jks
COPY resources/temperatura /app/resources/temperatura

# Expose port 5000
EXPOSE 5000

# Define building args
ARG PDP_PORT
ARG PDP_CONFIG
ARG PDP_KS
ARG PDP_PW
ARG PDP_ALIAS
ARG DLT_IP
ARG DLT_PORT
ARG IDAGENT_KS
ARG IDAGENT_PW
ARG IDAGENT_ALIAS
ARG IDAGENT_CERT
ARG IDAGENT_PORT
ARG IDAGENT_IP
ARG RESOURCES
# Set up environment variables
ENV PDP_PORT=${PDP_PORT}
ENV PDP_CONFIG=${PDP_CONFIG}
ENV PDP_KS=${PDP_KS}
ENV PDP_PW=${PDP_PW}
ENV PDP_ALIAS=${PDP_ALIAS}
ENV DLT_IP=${DLT_IP}
ENV DLT_PORT=${DLT_PORT}
ENV IDAGENT_KS=${IDAGENT_KS}
ENV IDAGENT_PW=${IDAGENT_PW}
ENV IDAGENT_ALIAS=${IDAGENT_ALIAS}
ENV IDAGENT_CERT=${IDAGENT_CERT}
ENV IDAGENT_PORT=${IDAGENT_PORT}
ENV IDAGENT_IP=${IDAGENT_IP}
ENV RESOURCES=${RESOURCES}
# Set up app entrypoint
CMD ["java", "-jar", "demo.jar"]
