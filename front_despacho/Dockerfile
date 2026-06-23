FROM node:20-alpine AS build
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci --frozen-lockfile
COPY . .
RUN npm run build

FROM nginx:1.27-alpine AS production
RUN addgroup -S nginxgroup && \
    adduser -S -G nginxgroup -H -s /sbin/nologin nginxuser
RUN rm /etc/nginx/conf.d/default.conf
COPY nginx-ecs.conf /etc/nginx/conf.d/default.conf
COPY --from=build /app/dist /usr/share/nginx/html
RUN chown -R nginxuser:nginxgroup /usr/share/nginx/html && \
    chown -R nginxuser:nginxgroup /var/cache/nginx && \
    chown -R nginxuser:nginxgroup /var/log/nginx && \
    touch /var/run/nginx.pid && \
    chown -R nginxuser:nginxgroup /var/run/nginx.pid
USER nginxuser
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
