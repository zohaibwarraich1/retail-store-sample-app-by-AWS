# Build Stage
FROM maven:3.9-eclipse-temurin-21-alpine AS build
WORKDIR /app

# Copy shared resources
COPY src/misc ./src/misc

# Copy source code
COPY src/orders ./src/orders

# Build the application
RUN mvn -f src/orders/pom.xml clean package -DskipTests

# Runtime Stage
FROM gcr.io/distroless/java21-debian12
WORKDIR /app

# Copy artifact
COPY --from=build /app/src/orders/target/*.jar app.jar

USER nonroot:nonroot

EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]