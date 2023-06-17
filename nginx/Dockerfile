FROM nginx:1.25.1-alpine3.17-slim

COPY ./nginx_configs/nginx.conf /etc/nginx/nginx.conf
COPY ./nginx_configs/default.conf /etc/nginx/conf.d/default.conf

WORKDIR /app

RUN adduser -u 1001 -D -s /sbin/nologin test

RUN chown -R test:test /app && chmod -R 755 /app && \
        chown -R test:test /var/cache/nginx && \
        chown -R test:test /var/log/nginx && \
        chown -R test:test /etc/nginx/conf.d
RUN touch /var/run/nginx.pid && \
        chown -R test:test /var/run/nginx.pid

USER test

EXPOSE 80
