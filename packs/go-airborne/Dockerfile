FROM debian:buster-slim

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
      ca-certificates \
      procps curl net-tools dnsutils && \
    rm -rf /var/lib/apt/lists/*

EXPOSE 8080
COPY ./bin/ /
ENTRYPOINT ["/REPLACE_ME_APP_NAME"]
