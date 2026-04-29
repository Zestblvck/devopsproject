# Build stage
FROM maven:3.9.6-eclipse-temurin-17 AS build
COPY . /usr/src/app
WORKDIR /usr/src/app
RUN mvn clean package -DskipTests

# Run stage
FROM eclipse-temurin:17-jre
RUN apt-get update && apt-get install -y iproute2
COPY --from=build /usr/src/app/target/quarkus-app/ /deployments/

EXPOSE 8080
ENTRYPOINT ["java", "-jar", "/deployments/quarkus-run.jar"]
