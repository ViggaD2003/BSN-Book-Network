# Build stage
FROM maven:3.8.7-openjdk-18 AS build
WORKDIR /build
COPY pom.xml .
RUN mvn dependency:resolve
COPY src ./src
RUN mvn clean package -DskipTests

# Runtime stage
FROM amazoncorretto:17
# Define arguments with default values
ARG PROFILE=dev
ARG APP_VERSION=latest
WORKDIR /app
COPY --from=build /build/target/book-network-*.jar /app/

EXPOSE 8088
ENV DB_URL=jdbc:postgresql://postgres-sql-bsn:5432/book_social_network
ENV ACTIVE_PROFILE=${PROFILE}
ENV JAR_VERSION=${APP_VERSION}

# Use a fallback JAR version if the specified version does not exist
RUN if [ "${JAR_VERSION}" = "latest" ]; then \
    JAR_NAME=$(ls /app/book-network-*.jar | sort -V | tail -n 1); \
    echo "Using latest JAR: $JAR_NAME"; \
    else \
    JAR_NAME="/app/book-network-${JAR_VERSION}.jar"; \
    fi && \
    mv $JAR_NAME /app/book-network.jar

CMD java -jar -Dspring.profiles.active=${ACTIVE_PROFILE} -Dspring.datasource.url=${DB_URL} /app/book-network.jar
