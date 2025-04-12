## Useful Docker Commands

### Start Commands:

Start a container:
 - `start container`

Start a container with a compose .yml file:
 - `sudo docker compose up -d`

### Environment commands:
Get what shell a container is using:
 - `get shell prompt`

Login to a container from terminal:
 - `sudo docker exec -it CONTAINERNAME sh`

Login to a container as root from terminal:
 - `docker exec -u root -it CONTAINERNAME <shell>`
