## example dockerizing FE, BE and DB into 3 docker containers.

## Exposing port 27017 to localhost -p 27017:27017 we ensure that not dockerized BE
## is able to connect to the localhost:20017

docker run --name mongodb --rm -d -p 27017:27017 mongo
docker ps -a
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS                      NAMES
866a34304e3d   mongo     "docker-entrypoint.s…"   19 seconds ago   Up 18 seconds   0.0.0.0:27017->27017/tcp   mongodb


## Dockerize BE

docker build -t goals-node .

## we don't use detached mode (-d) for now to see the log.
docker run --name goals-backend --rm goals-node
CONNECTED TO MONGODB

## we know that BE is connecting to the DB but we still need to expose a port

docker stop goals-backend
docker run --name goals-backend --rm -d -p 80:80 goals-node

docker ps -a
CONTAINER ID   IMAGE        COMMAND                  CREATED              STATUS              PORTS                      NAMES
a0b1f479ab41   goals-node   "docker-entrypoint.s…"   About a minute ago   Up About a minute   0.0.0.0:80->80/tcp         goals-backend
866a34304e3d   mongo        "docker-entrypoint.s…"   55 minutes ago       Up 55 minutes       0.0.0.0:27017->27017/tcp   mongodb

