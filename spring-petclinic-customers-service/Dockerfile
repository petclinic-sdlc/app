# =========================
# Stage 1: Build Stage
# =========================
FROM maven:3.9.6-eclipse-temurin-17-alpine AS builder

WORKDIR /build

# Copy source code
COPY . .

# Copy parent pom from root
COPY ../pom.xml /build/pom.xml

# Build application
RUN mvn clean package -DskipTests


# =========================
# Stage 2: Runtime Stage
# =========================
FROM eclipse-temurin:17-jdk-alpine

# Create non-root user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

WORKDIR /app

# Copy built JAR from builder stage
COPY --from=builder /build/target/*.jar app.jar

# Set ownership
RUN chown appuser:appgroup /app/app.jar

# Use non-root user
USER appuser

EXPOSE 8081

ENTRYPOINT ["java","-jar","/app/app.jar"]