FROM maven:3.9.9-eclipse-temurin-17-alpine AS build
WORKDIR /app


COPY pom.xml .
RUN mvn dependency:go-offline -B

COPY src ./src
RUN mvn clean package -DskipTests -B


FROM eclipse-temurin:17-jre-alpine AS production
WORKDIR /app

RUN addgroup -S appgroup && \
    adduser -S -G appgroup -H -s /sbin/nologin appuser

COPY --from=build /app/target/*.jar app.jar
RUN chown appuser:appgroup app.jar

USER appuser

ENV SERVER_PORT=8081
EXPOSE 8081

ENTRYPOINT ["java", "-jar", "app.jar"]
