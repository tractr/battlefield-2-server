# bf2-docker

Dockerized Battlefield 2 server based on [nihlen/bf2-docker](https://github.com/nihlen/bf2-docker). The base image is `debian:stretch-slim` and was tested on Linux containers in Windows 10 WSL2 and Debian 11. Uses multi-stage builds to keep the image sizes down.

## Prerequisites

- [Docker](https://docker.com/)
- [Docker Compose](https://docs.docker.com/compose/) (optional)

## Usage

Different server variations are placed in the [images](https://github.com/tractr/battlefield-2-server/tree/master/images) folder. To create your own, you can copy one of the existing images to use as a base, and then place your custom files in the assets/bf2 folder to overwrite any existing files.

Initial settings or passwords can be set using environment variables. Persisted files like settings, logs and demos are put in the `/volume` directory in the container using symbolic links and should be mapped to a host directory. If you want to have full visibility of the server files you can also map the `/home/bf2/srv` folder of the container.

To use these images on a remote host like a VPS you can either use the snippets below to build and run or you can build the images locally and then push them to a container registry like Docker Hub or Azure Container Registry (public or private).

Running multiple servers on the same host can be done by changing the ports in the environment variables and the mapped host port. For this use case I prefer using Docker Compose, an example is listed further down.

### [default](https://github.com/tractr/battlefield-2-server/tree/master/images/default)

- Battlefield 2 server (1.5.3153.0)

The basic image to run a Battlefield 2 server. Not practical since you can't play online but it can be used as a base.

```
docker build -t tractr/battlefield-2-server:default ./images/default
docker run --name bf2server -p 4711:4711/tcp -p 4712:4712/tcp -p 16567:16567/udp -p 27901:27901/udp -p 29900:29900/udp tractr/battlefield-2-server:default
```

### Docker Compose

To simplify setting up multiple servers on the same host you can use Docker Compose. The `image:` property can point to a locally built image or a URL to your container registry of choice. Note that the game server port and gamespy port need to match in the environment variables and in the Docker port configuration.

Here is an example of running two servers on the same host:

```yaml
version: "3.3"
services:
  bf2-docker-1-service:
    container_name: bf2-docker-1
    image: tractr/battlefield-2-server:default
    restart: on-failure
    environment:
      - ENV_SERVER_NAME=bf2-docker #1
      - ENV_MAX_PLAYERS=32
      - ENV_COOP_BOT_COUNT=32
      - ENV_COOP_BOT_DIFFICULTY=50
      - ENV_SERVER_PORT=16567
      - ENV_SERVER_INTERNET=0
      - ENV_SERVER_IP="0.0.0.0"
      - ENV_SERVER_MOTD="Welcome to bf2-docker"
      - ENV_GAMESPY_PORT=29900
      - ENV_RCON_PASSWORD=rconpw123
    volumes:
      - "/data/bf2/bf2-docker-1/server:/home/bf2/srv"
      - "/data/bf2/bf2-docker-1/volume:/volume"
    ports:
      - "8000:80/tcp"
      - "4711:4711/tcp"
      - "4712:4712/tcp"
      - "16567:16567/udp"
      - "27901:27901/udp"
      - "29900:29900/udp"
```

Place the docker-compose.yml on the host and run `docker-compose up -d --remove-orphans` to create the containers. If you are not using a container registry then the images need to be built on the host first.

## Maps list

The `maplist.con` is `images/default/assets/build/bf2/mods/bf2/settings/maplist.con`.

## Build the images

```shell
docker image build -t tractr/battlefield-2-server:latest ./images/default
docker image build -t tractr/battlefield-2-server:default ./images/default
```
