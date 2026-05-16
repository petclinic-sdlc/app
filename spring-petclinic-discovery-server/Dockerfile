# Discovery server 
# Stage 1
FROM maven:3.9.6-eclipse-temurin-17 AS builder

WORKDIR /app
COPY . .

RUN mvn -pl spring-petclinic-discovery-server -am clean package -DskipTests

# Stage 2
FROM eclipse-temurin:17-jdk-alpine

WORKDIR /app

RUN addgroup -S appgroup && adduser -S appuser -G appgroup

COPY --from=builder /app/spring-petclinic-discovery-server/target/*.jar app.jar

RUN chown appuser:appgroup app.jar

USER appuser

EXPOSE 8761

ENTRYPOINT ["java","-jar","app.jar"]