FROM node:20-alpine AS builder 

WORKDIR /app
COPY package*.json . 
RUN npm ci

COPY src ./src/
COPY angular.json tsconfig.json tsconfig.app.json .

RUN npm run build --configuration=production

FROM nginx:1.30.4-alpine AS runtime

RUN apk update && apk upgrade --no-cache
RUN apk add curl
LABEL "org.opencontainers.image.title"="ic-campus-web" 
LABEL "org.opencontainers.image.vendor"="IC-TechDojo"
LABEL "org.opencontainers.image.version"="1.0"

RUN adduser -D icweb --uid 1101

RUN chown -R icweb /usr/share/nginx/html && chmod -R 755 /usr/share/nginx/html && chown -R icweb /var/cache/nginx && chown -R icweb /var/log/nginx && chown -R icweb /etc/nginx/conf.d 

RUN touch /var/run/nginx.pid && chown -R icweb /var/run/nginx.pid 

COPY /nginx/default.conf.template /etc/nginx/templates/default.conf.template
COPY --from=builder /app/dist/ic-campus-web/ /usr/share/nginx/html 
USER icweb
HEALTHCHECK --interval=15s --timeout=3s --start-period=10s --retries=3 CMD curl -f http://localhost:8080/health || exit
 
EXPOSE 8080
