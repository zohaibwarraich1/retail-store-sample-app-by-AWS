# Build Stage
FROM maven:3.9-eclipse-temurin-21-alpine AS build
WORKDIR /app

# Copy shared resources
COPY src/misc ./src/misc

# Copy OpenAPI specs (Required for client generation)
COPY src/cart/openapi.yml ./src/cart/openapi.yml
COPY src/catalog/openapi.yml ./src/catalog/openapi.yml
COPY src/checkout/openapi.yml ./src/checkout/openapi.yml
COPY src/orders/openapi.yml ./src/orders/openapi.yml

# Copy source code
COPY src/ui ./src/ui

# Build the application
RUN mvn -f src/ui/pom.xml clean package -DskipTests

# Runtime Stage
FROM gcr.io/distroless/java21-debian12
WORKDIR /app

# Copy artifact
COPY --from=build /app/src/ui/target/*.jar app.jar

# Use non-root user (standard in distroless)
USER nonroot:nonroot

EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]