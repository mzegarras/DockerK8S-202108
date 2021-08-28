

# Node

1. Compilar containers
```sh
cd ~/DockerK8S-202108/Lab08/03_node
docker build -t apinode:1.0.0 .
docker tag apinode:1.0.0 galaxy202108.azurecr.io/manzegarra/apinode:1.0.0
docker login galaxy202108.azurecr.io 
docker push galaxy202108.azurecr.io/manzegarra/apinode:1.0.0
```

2. Commandos

```sh
docker network create node-net
```

```sh
docker run --name mongodb \
    -e MONGO_INITDB_ROOT_USERNAME=root \
    -e MONGO_INITDB_ROOT_PASSWORD=pwd123456 \
    -e MONGO_INITDB_DATABASE=interfaces \
    --net node-net \
    -d mongo
```

```sh
docker run --name mongo-express \
-e ME_CONFIG_MONGODB_ADMINUSERNAME=root \
-e ME_CONFIG_MONGODB_ADMINPASSWORD=pwd123456 \
-e ME_CONFIG_MONGODB_ENABLE_ADMIN=true \
-e ME_CONFIG_MONGODB_SERVER=mongodb \
-p 9081:8081 \
--net node-net \
-d mongo-express
```

```sh
docker run --name backend \
    -e URL_DB='mongodb://mongodb:27017/interfaces' \
    -e URL_DB_USER=root \
    -e URL_DB_PWD=pwd123456 \
    --net node-net \
    -d galaxy202108.azurecr.io/manzegarra/apinode:1.0.0
```

```sh
docker run --name proxy \
    -v $PWD/nginx.conf:/etc/nginx/nginx.conf:ro \
    --net node-net \
    -p 9060:9060 -d nginx

```



# Config server

1. Compilar containers
```sh
cd /DockerK8S-202108/Lab08/04_config
docker build -t config-server:1.0.0 . --no-cache
docker tag config-server:1.0.0 galaxy202108.azurecr.io/manzegarra/config-server:1.0.0
docker login galaxy202108.azurecr.io 
docker push galaxy202108.azurecr.io/manzegarra/config-server:1.0.0
```


```sh
docker run \
    -e GIT_URI="https://github.com/mzegarras/ms-configuration.git" \
    -e GIT_USER="mzegarra" \
    -e GIT_PWD="" \
    -e KEYSTORE_PWD="YOU_KEYSTORE_PASSWORD" \
    -e KEYSTORE_ALIAS="YOU_CONFIG_SERVER_KEY" \
    -e KEYSTORE_SECRET="YOU_KEYSTORE_PASSWORD" \
    -v $PWD/config-server.jks:/config-server.jks:ro \
    -p 8888:8888 \
    galaxy202108.azurecr.io/ggonzales/config-server:1.0.0
    
```
