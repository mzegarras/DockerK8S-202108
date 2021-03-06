
1. Actualizar centos (opcional)
    ```bash
    sudo dnf update -y
    ```

1. Agregar Docker-CE repositorio
    ```console
    sudo dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
    ```

1. Instalar última versión docker
    ```console
    sudo dnf install -y docker-ce --nobest
    ```
1. Iniciar docker cuando iniciar el SO
    ```console
    sudo systemctl start docker
    sudo systemctl enable docker
    ```
1. Verificar versión
    ```console
    docker --version
    ```
1. Instalar docker-compose
    ```console
    sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
    ```

1. Agregar permisos de ejecución
    ```console
    sudo chmod +x /usr/local/bin/docker-compose
    ```

1. Agregar acceso directo
    ```console
    sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
    ```
1. Verificar versión
    ```console
    docker-compose --version
    ```

1. Agregar acceso directo
    ```console
    sudo docker run -p 8080:80 nginx
    ```

1. Agregar acceso directo
    ```console
    sudo groupadd docker
    sudo chmod 666 /var/run/docker.sock
    sudo service docker restart
    docker run -p 8080:80 nginx
    ```

