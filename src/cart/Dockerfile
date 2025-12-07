# Build stage
FROM maven:3.9-eclipse-temurin-21-alpine AS build
WORKDIR /app

# Copy the shared misc directory (required for checkstyle)
COPY src/misc ./src/misc

# Copy the cart service source code
COPY src/cart ./src/cart

# Build the application
# We run from the root but point to the cart module
RUN mvn -f src/cart/pom.xml clean package -DskipTests

# Runtime stage
FROM gcr.io/distroless/java21-debian12
WORKDIR /app

# Copy the built artifact from the build stage
COPY --from=build /app/src/cart/target/*.jar app.jar

USER nonroot:nonroot

# Expose the application port
EXPOSE 8080

# Run the application
ENTRYPOINT ["java", "-jar", "app.jar"]
