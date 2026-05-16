# =========================
# Stage 1: Dependency Cache Layer
# =========================
FROM maven:3.9.6-eclipse-temurin-17-alpine AS dependencies

WORKDIR /build

# Copy only POM files to leverage Docker layer caching
# This layer is reused if dependencies haven't changed
COPY pom.xml .
COPY spring-petclinic-customers-service/pom.xml ./spring-petclinic-customers-service/

# Download dependencies (this layer will be cached)
RUN mvn -B dependency:go-offline \
    -DdownloadSources=false \
    -DdownloadJavadoc=false


# =========================
# Stage 2: Build Stage
# =========================
FROM maven:3.9.6-eclipse-temurin-17-alpine AS builder

WORKDIR /build

# Copy cached dependencies from previous stage
COPY --from=dependencies /root/.m2 /root/.m2

# Copy entire source code
COPY . .

# Build the customers service module
# Using -o for offline mode to use cached dependencies
RUN mvn -B -o \
    -pl spring-petclinic-customers-service \
    -am clean package \
    -DskipTests \
    -Dorg.slf4j.simpleLogger.defaultLogLevel=warn

# Verify JAR was created
RUN ls -lh spring-petclinic-customers-service/target/*.jar


# =========================
# Stage 3: Runtime Stage (Distroless for security)
# =========================
FROM eclipse-temurin:17-jdk-alpine

# Metadata labels for container management
LABEL maintainer="PetClinic Team"
LABEL version="4.0.1"
LABEL description="Spring PetClinic Customers Service"

# Create non-root user for security
RUN addgroup -S appgroup && \
    adduser -S appuser -G appgroup

# Set working directory
WORKDIR /app

# Copy built JAR from builder stage
COPY --from=builder /build/spring-petclinic-customers-service/target/*.jar app.jar

# Set ownership to non-root user
RUN chown -R appuser:appgroup /app && \
    chmod 755 /app

# Switch to non-root user
USER appuser

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=40s --retries=3 \
    CMD java -cp app.jar \
    -Dspring.profiles.active=health \
    -c "sh -c 'wget -q --spider http://localhost:8081/actuator/health || exit 1'" || exit 1

# Expose port
EXPOSE 8081

# Runtime optimizations
ENV JAVA_OPTS="-XX:+UseG1GC \
               -XX:MaxRAMPercentage=75.0 \
               -XX:+ParallelRefProcEnabled \
               -Djava.security.egd=file:/dev/./urandom"

ENTRYPOINT ["java", "-jar", "/app/app.jar"]