# Stage 1: Build Stage 
FROM maven:3.9.6-eclipse-temurin-17-alpine AS builder

WORKDIR /build

# Copy full repository (IMPORTANT)
COPY . .

# Build only customers service with parent modules
RUN mvn -pl spring-petclinic-customers-service -am clean package -DskipTests


# Stage 2: Runtime Stage
FROM eclipse-temurin:17-jdk-alpine

RUN addgroup -S appgroup && adduser -S appuser -G appgroup

WORKDIR /app

COPY --from=builder /build/spring-petclinic-customers-service/target/*.jar app.jar

RUN chown appuser:appgroup app.jar

USER appuser

EXPOSE 8081

ENTRYPOINT ["java","-jar","app.jar"]