<a name="JPwWW"></a>
# Dockerfile 实践举例
<a name="v7aSJ"></a>
## Maven 项目
```dockerfile
FROM maven:3.8.4-openjdk-17-slim as build

WORKDIR /opt

COPY settings.xml /usr/share/maven/conf/settings.xml
COPY . .

RUN mvn clean package -Dmaven.test.skip=true

FROM openjdk:17-oracle as publish

ENV JAVA_OPTS=""

WORKDIR /opt

COPY --from=build /opt/target/*.jar ./app.jar

EXPOSE 8080/tcp

ENTRYPOINT exec java ${JAVA_OPTS} -jar app.jar
```
<a name="cmI8C"></a>
## Node 项目
```dockerfile
FROM node:16-slim as build

WORKDIR /opt

COPY .npmrc ~
COPY . .

RUN npm install pnpm -g \
      && pnpm install \
      && pnpm build

FROM nginx:1.21.1 as publish

COPY --from=build /opt/nginx.conf /etc/nginx/nginx.conf.d/website.conf

COPY --from=build /opt/dist/      /usr/share/nginx/html/

EXPOSE 80/tcp

ENTRYPOINT ["nginx", "-g", "daemon off;"]
```
