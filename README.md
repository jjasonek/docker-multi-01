## Exposing port 27017 to localhost -p 27017:27017 we ensure that not dockerized BE
## is able to connect to the localhost:20017

docker run --name mongodb --rm -d -p 27017:27017 mongo
docker ps -a
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS                      NAMES
866a34304e3d   mongo     "docker-entrypoint.s…"   19 seconds ago   Up 18 seconds   0.0.0.0:27017->27017/tcp   mongodb
