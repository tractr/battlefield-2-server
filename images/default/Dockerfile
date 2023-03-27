# Build stage
FROM debian:stretch-slim AS build

# Set working directory
WORKDIR /home/bf2/tmp

# Get required packages
RUN apt -y update && \
    apt -y install wget expect

# Download missing assets
RUN wget -nc -q --show-progress --progress=bar:force:noscroll ftp://ftp.bf-games.net/server-files/bf2/bf2-linuxded-1.5.3153.0-installer.tgz

# Add assets to image
COPY ./assets/build ./

# Extract server files from the installer
RUN tar -xvf /home/bf2/tmp/bf2-linuxded-1.5.3153.0-installer.tgz -C /home/bf2/tmp && \
    chmod +x /home/bf2/tmp/bf2-linuxded-1.5.3153.0-installer.sh ./extract && \
    ./extract

# Clean up unused folders
RUN rm -r /home/bf2/tmp/srv/bin/ia-32

# Replace with our own BF2 server files (custom settings and scripts)
RUN cp -r "/home/bf2/tmp/bf2/." "/home/bf2/tmp/srv"

# Create empty server folder to copy our files into if it's empty on the host system
RUN mkdir -p /home/bf2/srv && \
    chmod -R 700 /home/bf2/srv/

# Runtime stage
FROM debian:stretch-slim AS runtime
WORKDIR /home/bf2/tmp
LABEL maintainer=nihlen

# Environment variables
ENV SERVER_NAME="bf2-docker"

# Copy runtime assets
COPY ./assets/runtime ./

# Install required packages and set permissions
RUN bash -x ./setup.sh

# Copy server files from the build stage
COPY --from=build /home/bf2/tmp/srv ./srv

# Move server files to persisted folder and start server
CMD ./run.sh
