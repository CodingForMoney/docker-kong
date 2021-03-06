FROM alpine:3.11

LABEL maintainer="Kong <support@konghq.com>"

ARG ASSET=ce
ENV ASSET $ASSET

ARG EE_PORTS

COPY kong.tar.gz /tmp/kong.tar.gz
COPY kong.conf /tmp/kong.conf
COPY lets-encrypt.nginx.conf /tmp/lets-encrypt.nginx.conf

ARG KONG_VERSION=2.1.3
ENV KONG_VERSION $KONG_VERSION

ARG KONG_SHA256="2f2a73d468d95f3ded495aa4199b55d2148d1ad5cb5050372621a1768b34ba8a"

RUN set -ex; \
    if [ "$ASSET" = "ce" ] ; then \
        apk add --no-cache --virtual .build-deps curl wget tar ca-certificates && \
        curl -fL "https://bintray.com/kong/kong-alpine-tar/download_file?file_path=kong-$KONG_VERSION.amd64.apk.tar.gz" -o /tmp/kong.tar.gz && \
        echo "$KONG_SHA256  /tmp/kong.tar.gz" | sha256sum -c -; \
        apk del .build-deps; \
    fi; \
    mkdir /kong; \
	tar -C /kong -xzf /tmp/kong.tar.gz && \
	mv /kong/usr/local/* /usr/local && \
	mv /kong/etc/* /etc && \
	mv /tmp/kong.conf /etc/kong/kong.conf && \
	mv /tmp/lets-encrypt.nginx.conf /etc/kong/lets-encrypt.nginx.conf && \
	rm -rf /kong && \
	apk add --no-cache libstdc++ libgcc openssl pcre perl tzdata libcap zip bash zlib zlib-dev git ca-certificates && \
	adduser -S kong && \
	mkdir -p "/usr/local/kong" && \
	mkdir -p "/home/kong/acme-challenge" && \
	mkdir -p "/home/kong/ssl" && \
	chown -R kong:0 /usr/local/kong && \
	chown kong:0 /usr/local/bin/kong && \
	chmod -R g=u /usr/local/kong && \
	rm -rf /tmp/kong.tar.gz && \
	luarocks install kong-plugin-ice-grpc-gateway && \
	luarocks install kong-plugin-ice-jaeger && \
  if [ "$ASSET" = "ce" ] ; then \
    kong version ; \
  fi;

COPY docker-entrypoint.sh /docker-entrypoint.sh

USER kong

COPY test.pem /home/kong/fullchain.pem
COPY testkey.pem /home/kong/privkey.pem
COPY dhparam.pem /home/kong/kong.pem

ENTRYPOINT ["/docker-entrypoint.sh"]

EXPOSE 8000 8443 8001 8444 $EE_PORTS

STOPSIGNAL SIGQUIT

CMD ["kong", "docker-start"]
