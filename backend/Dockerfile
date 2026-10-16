# Stage 1: Build stage
FROM eclipse-temurin:21-jdk-alpine AS builder

WORKDIR /app

# Copy gradle wrapper files and build configuration first for dependency caching
COPY gradlew .
COPY gradle gradle
COPY build.gradle settings.gradle ./

# Normalize line endings for gradlew (prevent CRLF issues on Windows) and grant execute permissions
RUN sed -i 's/\r$//' gradlew && chmod +x gradlew

# Download dependencies to cache them in docker layer
RUN ./gradlew dependencies --no-daemon || true

# Copy source code and build application jar (skip tests during container build)
COPY src src
RUN ./gradlew bootJar -x test --no-daemon

# Stage 2: Runtime stage
FROM eclipse-temurin:21-jre-alpine

WORKDIR /app

# Create a non-privileged system user for security
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

# Copy built artifact from builder stage
COPY --from=builder --chown=appuser:appgroup /app/build/libs/*.jar app.jar

USER appuser

# Expose backend application port
EXPOSE 8091

# Run the Spring Boot application
ENTRYPOINT ["java", "-jar", "app.jar"]
