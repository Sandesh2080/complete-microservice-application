# Dockerfile for the API Gateway service

# Use a lightweight JDK base image (both build and runtime)
FROM eclipse-temurin:17-jdk-jammy

# Use multi-stage if you want to build inside, but here we assume jar is built by Maven
ARG JAR_FILE=target/*.jar
COPY ${JAR_FILE} app.jar

# Expose default port
EXPOSE 8080
ENTRYPOINT ["java","-jar","/app.jar"]
