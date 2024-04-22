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

## Dockerize FE

## default port for the React application is 3000

docker build -t goals-react .
docker run --name goals-frontend --rm -d -p 3000:3000 goals-react
docker ps -a                                                     
CONTAINER ID   IMAGE         COMMAND                  CREATED             STATUS             PORTS                      NAMES
badb04cbdfab   goals-react   "docker-entrypoint.s…"   6 seconds ago       Up 4 seconds       0.0.0.0:3000->3000/tcp     goals-frontend
a0b1f479ab41   goals-node    "docker-entrypoint.s…"   27 minutes ago      Up 27 minutes      0.0.0.0:80->80/tcp         goals-backend
866a34304e3d   mongo         "docker-entrypoint.s…"   About an hour ago   Up About an hour   0.0.0.0:27017->27017/tcp   mongodb

## Well now the FE application stopped for the instructor but for me it still worked
## Maybe it happened because his node version is older. The instructor says that this can be react specific problem.
## on http://localhost:3000/.
## workaround for the instructor was using interactive mode> (-it) so I do the same.

docker run --name goals-frontend --rm -p 3000:3000 -it goals-react

...
Compiled successfully!

You can now view docker-frontend in the browser.

Local:            http://localhost:3000
On Your Network:  http://172.17.0.4:3000

Note that the development build is not optimized.
To create a production build, use npm run build.

webpack compiled successfully





