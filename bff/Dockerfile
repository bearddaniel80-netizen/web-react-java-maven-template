# syntax=docker/dockerfile:1.7
# ---------- Common Git Stage ----------
FROM alpine:latest AS git-base

RUN --mount=type=cache,target=/var/cache/apk \
    apk update && \
    apk add git

# ------- assembly ---------------
FROM git-base AS assembly

ARG DOMAIN
ARG APP_LANG
ARG BUILD_FILE
ARG EXT

ENV BUILD_FILE=${BUILD_FILE}

ENV APP_LANG=${APP_LANG}

ENV EXT=${EXT}

ENV DOMAIN=${DOMAIN}

ENV BUILD_DIR=/build-downloads
ENV COMMON_DIR=/common-downloads
ENV DOMAIN_DIR=/${DOMAIN}-downloads

ENV GIT_REPO=web-${APP_LANG}-lib

ENV GIT_URL=https://github.com/bearddaniel80-netizen/${GIT_REPO}.git

RUN git clone -b build --single-branch ${GIT_URL} ${BUILD_DIR}

RUN git clone -b common --single-branch ${GIT_URL} ${COMMON_DIR}

RUN git clone -b ${DOMAIN} --single-branch ${GIT_URL} ${DOMAIN_DIR}

# ........ build file ..........
WORKDIR /app/build

RUN cp ${BUILD_DIR}/${BUILD_FILE} .

# ........ actual code ..........
WORKDIR /app/${APP_LANG}/src/main/${APP_LANG}/example

RUN cp ${COMMON_DIR}/Application.${EXT} .
RUN cp -R ${COMMON_DIR}/config .
RUN cp -R ${COMMON_DIR}/web .
RUN cp -R ${DOMAIN_DIR}/service .

# ........ resources ..............
WORKDIR /app/${APP_LANG}/src/main

RUN cp -R ${DOMAIN_DIR}/resources .

# ---- build stage ----
FROM maven:3.9-eclipse-temurin-21 AS build

ARG APP_LANG
ENV APP_LANG=${APP_LANG}

WORKDIR /app

# Copy only the pom first so dependency downloads are cached until it changes
COPY --from=assembly /app/build ./
RUN --mount=type=cache,target=/root/.m2 \
    mvn -B -q dependency:resolve dependency:resolve-plugins

COPY ${APP_LANG} .

COPY --from=assembly /app/${APP_LANG} .

# Remove flag -DskipTests to run spring test
RUN --mount=type=cache,target=/root/.m2 \
    mvn -B package -DskipTests

# ---------- Java runtime ----------
FROM eclipse-temurin:21-jre AS runtime
WORKDIR /app
COPY --from=build /app/target/aql-0.0.1.jar app.jar
ENTRYPOINT ["java", "-jar", "app.jar"]