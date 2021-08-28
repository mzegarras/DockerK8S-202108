# Network

1. Pasos previos en local
    ```bash
    sudo systemctl stop firewalld
    sudo systemctl disable firewalld
    sudo iptables -t filter -F
    sudo iptables -t filter -X
    systemctl restart docker
    ```

1. Limpiar system
    ```bash
    docker system prune
    ```

1. <span style="color:red">*Link containers*</span>
    ```bash

    docker run --name mysql01 -p 3306:3306 -e MYSQL_ROOT_PASSWORD=Password1234 -e WORDPRESS_DB_NAME=wordpress -d mysql:5.7.35

    docker run --name wordpress01 --link mysql01 -p 8080:80 -e WORDPRESS_DB_HOST=mysql01:3306 -e WORDPRESS_DB_USER=root -e WORDPRESS_DB_PASSWORD=Password1234 -e WORDPRESS_DB_NAME=wordpress -d wordpress
    ```

1. Listar network

* Bridge default

    * Todos los containers se pueden ver
    * La comunicación es por ip

    ```bash
    docker network ls
    docker run -d nginx
    docker inspect aad627|grep IPAddress
    docker run -it busybox sh
    wget -q -O - http://172.17.0.6:80/
    docker inspect bridge
    ```

* user-defined bridge network
    ```bash
    docker network ls
    docker network create galaxy-net

    docker run -d --net galaxy-net --name web01 -d nginx 
    docker run -it --net galaxy-net  busybox sh

    wget -q -O - http://web01:80/
    
    docker inspect galaxy-net

    docker network connect galaxy-net a10efce35e13

    docker network rm galaxy-net 

    ```
    
## Lab wordpress

1. Install bd
    ```bash
    docker network create wp-net

    docker run --name mysql01 \
        -e MYSQL_ROOT_PASSWORD=Password1234 \
        -e MYSQL_DATABASE=wordpress \
        -d \
        --net wp-net \
        mysql:5.7.35
    ``` 

1. Install wordpress
    ```bash
    
    docker rm -f wordpress01

    docker run --name wordpress01 \
        -e WORDPRESS_DB_HOST=mysql01 \
        -e WORDPRESS_DB_USER=root \
        -e WORDPRESS_DB_PASSWORD=Password1234 \
        -e WORDPRESS_DB_NAME=wordpress -d \
        --net wp-net \
        wordpress

    
    docker run --name wordpress01 \
        -e WORDPRESS_DB_HOST=mysql01 \
        -e WORDPRESS_DB_USER=root \
        -e WORDPRESS_DB_PASSWORD=Password1234 \
        -e WORDPRESS_DB_NAME=wordpress -d \
        --net wp-net \
        -p 8080:80 \
        wordpress

    docker run --name wordpress01 \
        -e WORDPRESS_DB_HOST=mysql01 \
        -e WORDPRESS_DB_USER=root \
        -e WORDPRESS_DB_PASSWORD=Password1234 \
        -e WORDPRESS_DB_NAME=wordpress -d \
        --net wp-net \
        -p 8080:80 \
        wordpress:php8.0
    ```

## Lab Java 01
1. cd 01_java

1. Network
    ```bash
    docker network create ms-net
    ```

1. Database
    ```bash
    docker run --name mongodb -e MONGO_INITDB_ROOT_USERNAME=root \
        -e MONGO_INITDB_ROOT_PASSWORD=pwd1234 \
        -e MONGO_INITDB_DATABASE=shop \
        --net ms-net \
        -d mongo
    
    docker run --name mongo-express \
    -e ME_CONFIG_MONGODB_ADMINUSERNAME=root \
    -e ME_CONFIG_MONGODB_ADMINPASSWORD=pwd1234 \
    -e ME_CONFIG_MONGODB_ENABLE_ADMIN=true \
    -e ME_CONFIG_MONGODB_SERVER=mongodb \
    -p 8081:8081 \
    --net ms-net \
    -d mongo-express
    ```

2. Java
    ```bash
    docker build -t reactivedemo .

    docker run -p 8082:8080 \
        --name reactiveapi \
        -v $PWD/application.yml:/application.yml \
        --net ms-net \
        -d reactivedemo:latest
    
    curl http://localhost:8082/listar
    curl http://localhost:8082/api/productos


    docker rm reactiveapi -f

    ```

1. Java
    ```bash

    docker run \
        --name reactiveapi \
        -v $PWD/application.yml:/application.yml \
        --net ms-net \
        -d reactivedemo:latest
    
    docker run --name proxyserver01 \
        -v $PWD/nginx.conf:/etc/nginx/nginx.conf:ro \
        --net ms-net \
        -p 8083:9060 -d nginx

    curl http://localhost:8083/listar
    curl http://localhost:8083/api/productos


    docker rm reactiveapi -f
    ```

* Host default (Sólo trabaja en linux)
    ```bash
    docker run -d --network host nginx
    ```

## Lab Java 02
1. cd 02_java

1. Network
    ```bash
    docker network create ms02-net
    ```

1. Database
    ```bash
    docker run --name mongodb \
        -e MONGO_INITDB_ROOT_USERNAME=root \
        -e MONGO_INITDB_ROOT_PASSWORD=pwd1234 \
        -e MONGO_INITDB_DATABASE=shop \
        --net ms02-net \
        -d mongo
    
    docker run --name mongo-express \
    -e ME_CONFIG_MONGODB_ADMINUSERNAME=root \
    -e ME_CONFIG_MONGODB_ADMINPASSWORD=pwd1234 \
    -e ME_CONFIG_MONGODB_ENABLE_ADMIN=true \
    -e ME_CONFIG_MONGODB_SERVER=mongodb \
    -p 9081:8081 \
    --net ms02-net \
    -d mongo-express
    ```

1. Java - Load balancer
    ```bash

    docker run \
        --name reactiveapi01 \
        -v $PWD/application.yml:/application.yml \
        --net ms02-net \
        -d reactivedemo:latest

    docker run \
        --name reactiveapi02 \
        -v $PWD/application.yml:/application.yml \
        --net ms02-net \
        -d reactivedemo:latest
     
    docker run --name proxyserver02 \
        -v $PWD/nginx.conf:/etc/nginx/nginx.conf:ro \
        --net ms02-net \
        -p 9083:9060 -d nginx

    curl http://localhost:9083/listar
    curl http://localhost:9083/api/productos    
    ```

## Reto node
1. Generar la imagen del proyecto 03_node con el nombre apinode:1.0.0
1. Ports: 3000:3000
1. Environment
    * PORT: 3000
    * URL_DB: 'mongodb://localhost:27017/interfaces'
    * URL_DB_USER:
    * URL_DB_PWD:
1. Usar ./03_node/Dockerfile
1. Crear la red node-net
1. Implementar el diseño proxy-reverse:9090->backend(proyecto previo)-->mongo
1. Para admins pudan usar mongo-express
1. Publicar las imágenes en repositorio
1. Puede probar las apis request.http

## Reto config-server
1. Generar la imagen del proyecto 04_config con el nombre config-server:1.0.0
1. Ports: 8888:8888
1. Volumes: ./config-server.jks:/config-server.jks
1. Environment
    - GIT_URI: "https://github.com/mzegarras/ms-configuration.git"
    - GIT_USER: "mzegarra"
    - GIT_PWD: ""
    - KEYSTORE_PWD: "YOU_KEYSTORE_PASSWORD"
    - KEYSTORE_ALIAS: "YOU_CONFIG_SERVER_KEY"
    - KEYSTORE_SECRET: "YOU_KEYSTORE_PASSWORD"
1. Test
```bash
    curl http://localhost:8888/ms-accounts/default
    curl http://localhost:8888/encrypt -H 'Content-Type: text/plain' -d 'password'
    curl http://localhost:8888/decrypt -H 'Content-Type: text/plain' -d '<<paso-previo>>'
```