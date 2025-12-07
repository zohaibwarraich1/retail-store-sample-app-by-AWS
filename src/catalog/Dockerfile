# Build Stage
FROM golang:1.23-alpine AS build
WORKDIR /app

# Copy dependencies first for caching
COPY src/catalog/go.mod src/catalog/go.sum ./
RUN go mod download

# Copy source code
COPY src/catalog/ .

# Build the binary (statically linked)
RUN CGO_ENABLED=0 GOOS=linux go build -o /catalog .

# Runtime Stage
FROM gcr.io/distroless/static-debian12
WORKDIR /app

# Copy binary and JSON assets
COPY --from=build /catalog ./catalog
COPY --from=build /app/repository ./repository

# Use non-root user (standard in distroless)
USER nonroot:nonroot

EXPOSE 8080
ENTRYPOINT ["./catalog"]