FROM node:16-alpine AS builder

ARG version='UNDEFINED'

WORKDIR '/app'
COPY ./package.json ./
RUN npm install
COPY . .
RUN sed -i 's/APP_VERSION/'"${version}"'/g' ./src/keys.js
RUN npm run build

FROM nginx:latest
EXPOSE 3000
COPY ./nginx/default.conf /etc/nginx/conf.d/default.conf
COPY --from=builder /app/build /usr/share/nginx/html
