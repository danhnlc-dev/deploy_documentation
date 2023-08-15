Switch to other branch to get your goals

# Dockerfile Example

- Example Dockerfile

## NODEJS

```bash
# Stage 1
FROM node:18-alpine as build-step
RUN mkdir -p /app
WORKDIR /app
# ENV
ENV NODE_OPTIONS="--max-old-space-size=8192"
ENV APP_ENVIRONMENT=""
ENV PORT=""
ENV TIMEZONE=""
#
EXPOSE 3000
COPY package.json .
RUN npm install --verbose
COPY . .
CMD [ "npm", "start" ]
```

## REACT APP WITH NGINX

```bash
# Stage 1
FROM node:18 as build-step
RUN mkdir -p /app
WORKDIR /app
ENV NODE_OPTIONS="--max-old-space-size=8192"
ENV REACT_APP_API_BASE_URL=""
COPY package.json .
RUN npm install --verbose
COPY . .
RUN npm run build

# Stage 2
FROM nginx:stable-alpine
EXPOSE 80
COPY --from=build-step /app/build /usr/share/nginx/html
COPY --from=build-step /app/nginx.conf /etc/nginx/conf.d/default.conf
CMD ["nginx", "-g", "daemon off;"]
```

```bash
# Stage 1
FROM 109234046957.dkr.ecr.ap-southeast-1.amazonaws.com/sr-devops:node-18.16.0 as node
WORKDIR /app
ENV NODE_OPTIONS="--max-old-space-size=8192"
ARG REACT_APP_API_BASE_URL
ARG REACT_APP_AWS_ACCESS_KEY_ID
ARG REACT_APP_AWS_REGION
ARG REACT_APP_AWS_S3_BUCKET
ARG REACT_APP_AWS_SECRET_ACCESS_KEY
ARG REACT_APP_GOOGLE_MAP_API_KEY
ARG REACT_APP_PREFIX_API
COPY package.json /app
RUN npm install -f
COPY . /app
RUN npm run build

# Stage 2
FROM 109234046957.dkr.ecr.ap-southeast-1.amazonaws.com/sr-devops:nginx-stable-alpine
COPY --from=node /app/build/ /usr/share/nginx/html/
COPY --from=node /app/nginx.conf /etc/nginx/conf.d/default.conf
CMD ["nginx", "-g", "daemon off;"]
```
