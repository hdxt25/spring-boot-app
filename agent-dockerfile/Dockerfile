FROM maven:3.9.11-eclipse-temurin-17

# Install Docker CLI
USER root
RUN apt-get update && apt-get install -y \
    docker.io \
    && rm -rf /var/lib/apt/lists/*


