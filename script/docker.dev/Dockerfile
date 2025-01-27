FROM satantime/puppeteer-node:23.6.1

ENV NODE_ENV "development"
WORKDIR /app

ENTRYPOINT ["/usr/bin/docker-entrypoint.sh"]
CMD ["/usr/bin/supervisord", "-c", "/etc/supervisord.conf"]
ENV PATH "$PATH:/app/node_modules/.bin"

RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get -q -y install supervisor && \
    apt-get -q -y autoremove

COPY /docker.dev/. /
RUN chmod 0755 /usr/bin/docker-entrypoint.sh && \
    chmod 0755 /docker-entrypoint.d/*.sh
