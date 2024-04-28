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


## So now our containers communicate through our **localhost** machine, because ve have
## their **ports published**.
## Let's make it better using the docker network

docker stop goals-frontend goals-backend mongodb

docker network create goals-net

docker network ls              
NETWORK ID     NAME            DRIVER    SCOPE
edffa3a59832   bridge          bridge    local
8352974c56c2   favorites-net   bridge    local
**e9c53018ead8   goals-net       bridge    local**
57a477632d1e   host            host      local
e462d5990d80   none            null      local

docker run --name mongodb --rm -d --network goals-net mongo

docker build -t goals-node .
docker run --name goals-backend --rm -d --network goals-net goals-node

docker build -t goals-react .
docker run --name goals-frontend -d --rm --network goals-net -p 3000:3000 goals-react

## now we have error from FE dev tools console:
Failed to load resource: net::ERR_NAME_NOT_RESOLVED
goals-backend/goals:1
## The reason is, that the React runs in a **browser**, not container. And the browser has no idea 
## about docker network, hence "goals-backend"

docker stop goals-frontend
docker stop goals-backend

docker run --name goals-frontend -d --rm -p 3000:3000 goals-react
docker run --name goals-backend --rm -d --network goals-net -p 80:80 goals-node

docker ps -a                                                                   
CONTAINER ID   IMAGE         COMMAND                  CREATED         STATUS         PORTS                    NAMES
38ce33778a3e   goals-node    "docker-entrypoint.s…"   4 seconds ago   Up 3 seconds   0.0.0.0:80->80/tcp       goals-backend
403adf57eca1   goals-react   "docker-entrypoint.s…"   2 minutes ago   Up 2 minutes   0.0.0.0:3000->3000/tcp   goals-frontend
56642c6cbc6e   mongo         "docker-entrypoint.s…"   2 hours ago     Up 2 hours     27017/tcp                mongodb

## for database data persistence we will use a named volume

docker stop mongodb
docker run --name mongodb --rm -d --network goals-net -v data:/data/db mongo


## for credentials, we use environment variables

## this part did not work for me until I removed the "data" named volume.
## I don't know whether it is Windows vs Unix file system problem  
## or just the database data overridden by those on the volume.
docker volume ls
DRIVER    VOLUME NAME
...
local     data
...

docker volume rm data

docker run --name mongodb --rm -d --network goals-net -v data:/data/db -e MONGO_INITDB_ROOT_USERNAME=student -e MONGO_INITDB_ROOT_PASSWORD=secret mongo

docker run --name goals-backend --rm -d --network goals-net -p 80:80 goals-node
docker logs goals-backend
CONNECTED TO MONGODB


## Now we want to persist backend logs and be able to change the backend code.
## Remember, longer path precedes the shorter, so **app/logs** will survive if we also set **/app**
## And we also set anonymous volume for the **/app/node_modules** because we need it not to be overwritten by **/app** bind mount 
## it is sufficient to use named volume for the logs but I want to try to see changing logs.

docker run --name goals-backend \
-v $(pwd):/app -v /app/node_modules \
-v $(pwd)/logs:/app/logs \
--rm -d --network goals-net -p 80:80 goals-node
### and I can see the logs refreshing locally.

## To allow js code changes during the container is up and running we utilize the **nodemon**.

docker stop goals-backend
docker build -t goals-node .
docker run --name goals-backend \
-v $(pwd):/app -v /app/node_modules \
-v $(pwd)/logs:/app/logs \
--rm -d --network goals-net -p 80:80 goals-node

docker logs goals-backend
> backend@1.0.0 start
> nodemon app.js

[nodemon] 3.1.0
[nodemon] to restart at any time, enter `rs`
[nodemon] watching path(s): *.*
[nodemon] watching extensions: js,mjs,cjs,json
[nodemon] starting `node app.js`
CONNECTED TO MONGODB!!
TRYING TO FETCH GOALS
FETCHED GOALS
TRYING TO DELETE GOAL
DELETED GOAL
[nodemon] restarting due to changes...
[nodemon] starting `node app.js`
CONNECTED TO MONGODB
