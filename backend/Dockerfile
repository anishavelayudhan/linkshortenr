# Multi-stage build
FROM maven:3.9.9-eclipse-temurin-23 AS builder

# Set working directory
WORKDIR /app

# Copy pom.xml and source code
COPY pom.xml .
COPY src ./src

# Build the application (Maven 3.9.9 can compile Java 23 code)
RUN mvn clean package -DskipTests

# Runtime stage - Use Java 23 for running
FROM eclipse-temurin:23-jdk

# Set working directory
WORKDIR /app

# Copy the built JAR from builder stage
COPY --from=builder /app/target/linkshortenr-1.0.0.jar app.jar

# Expose port
EXPOSE 8080

# Run the application
ENTRYPOINT ["java", "-jar", "app.jar"]