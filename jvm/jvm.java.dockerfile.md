
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
