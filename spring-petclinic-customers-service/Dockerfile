# =========================
# Stage 1: Build Stage
# =========================
FROM maven:3.9.6-eclipse-temurin-17-alpine AS builder

WORKDIR /build

# Copy FULL repository (important for multi-module Maven)
COPY . .

# Build ONLY customers service module
RUN mvn -pl spring-petclinic-customers-service -am clean package -DskipTests


# =========================
# Stage 2: Runtime Stage
# =========================
FROM eclipse-temurin:17-jdk-alpine

# Create non-root user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

WORKDIR /app

# Copy built JAR from builder stage
COPY --from=builder /build/spring-petclinic-customers-service/target/*.jar app.jar

# Set ownership
RUN chown appuser:appgroup /app/app.jar

USER appuser

EXPOSE 8081

ENTRYPOINT ["java","-jar","/app/app.jar"]