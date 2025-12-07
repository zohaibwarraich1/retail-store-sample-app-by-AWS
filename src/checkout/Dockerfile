# Build Stage
FROM node:20-alpine AS build
WORKDIR /app

# Copy package files
COPY src/checkout/package.json src/checkout/yarn.lock ./src/checkout/

# Install dependencies
WORKDIR /app/src/checkout
# frozen-lockfile ensures that the dependencies are installed in the same way every time with same exact versions
RUN yarn install --frozen-lockfile 

# Copy source code
COPY src/checkout/ .

# Build the application
RUN yarn build

# Runtime Stage
FROM gcr.io/distroless/nodejs20-debian12
WORKDIR /app

# Copy built artifacts and dependencies
COPY --from=build /app/src/checkout/dist ./dist
COPY --from=build /app/src/checkout/node_modules ./node_modules
COPY --from=build /app/src/checkout/package.json ./package.json

# Use non-root user that comes already in distroless
USER nonroot:nonroot

EXPOSE 8080
CMD ["dist/main.js"]