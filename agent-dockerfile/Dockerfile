FROM maven:3.11-eclipse-temurin-11

# Install Docker CLI
USER root
RUN apt-get update && apt-get install -y \
    docker.io \
    && rm -rf /var/lib/apt/lists/*


