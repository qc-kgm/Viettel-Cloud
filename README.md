# Wordpress-Docker
Bài tập tuần 1 - Viettel cloud 

## Cài đặt môi trường docker trên VM
### Chạy các lệnh sau trên terminal để cài đặt docker
```sh
  $ sudo apt-get remove docker docker.io docker-engine
  $	sudo apt install docker.io
	$ sudo systemctl status docker(kiểm tra đã cài đặt thành công ? )
  $ sudo usermod -aG docker name_user_ubuntu
  $	su - name_user_ubuntu 
  $	sudo systemctl start docker 
  $	sudo systemctl enable docker 
```
## Cài đặt docker compose
```sh
  $ sudo apt install curl 
  $ sudo curl -L "https://github.com/docker/compose/releases/download/1.29.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
	$ sudo chmod +x /usr/local/bin/docker-compose 
```
## Tạo máy ảo bằng Virtualbox 
```sh
  $ sudo apt install virtualbox 
```
Dowload file iso của Ubuntu 20.04  <br />
Tạo máy ảo chạy Ubuntu bằng Virtualbox
## Practice 1 : Deploy Wordpress with command line 
### Bước 1 : Cài đặt Virtualbox và tạo máy ảo Ubuntu
### Bước 2 : Cài đặt docker trên máy ảo 
### Bước 3 : Deploy wordpress bằng command line
1.Tạo network để kết nối database với app <br />
```sh
  $ docker network create wordpress-network
```
2.Run MariaDB  <br />
Tạo volume cho mariadb <br />
```sh
   $ docker volume create --name mariadb_data
```
Run , connect MariaDB với mạng đã tạo 
```sh
   $ docker run -d --name mariadb \
   --env ALLOW_EMPTY_PASSWORD=yes \
   --env MARIADB_USER=bn_wordpress \
   --env MARIADB_PASSWORD=bitnami \
   --env MARIADB_DATABASE=bitnami_wordpress \
   --network wordpress-network \
   --volume mariadb_data:/bitnami/mariadb \
   bitnami/mariadb:latest
``` 
3.Run Wordpress <br />
Tạo volume cho Wordpress 
```sh
   $ docker volume create --name wordpress_data
``` 
Run,connect với network
```sh
   $docker run -d --name wordpress \
   -p 8080:8080 -p 8443:8443 \
   --env ALLOW_EMPTY_PASSWORD=yes \
   --env WORDPRESS_DATABASE_USER=bn_wordpress \
   --env WORDPRESS_DATABASE_PASSWORD=bitnami \
   --env WORDPRESS_DATABASE_NAME=bitnami_wordpress \
   --network wordpress-network \
   --volume wordpress_data:/bitnami/wordpress \
   bitnami/wordpress:latest
```
### Truy cập vào địa chỉ  https://localhost:8443 
## Practice 2 : Deploy Wordpress with Docker-compose
### Bước 1 : Cài đặt Virtualbox và tạo máy ảo Ubuntu
### Bước 2 : Cài đặt docker,docker-compose trên máy ảo 
### Bước 3 : Deploy wordpress bằng docker-compose
Chạy các lệnh sau trên terminal :
```sh 
   $ curl -sSL https://raw.githubusercontent.com/bitnami/bitnami-docker-wordpress/master/docker-compose.yml > docker-compose.yml
   $ docker-compose up -d
```
### Truy cập vào địa chỉ trang web :  https://localhost:8443 
## Practice 3: Deploy Wordpress và database trên 2 máy ảo riêng biệt 
### Bước 1 :Tạo 2 máy ảo Ubuntu , cài đặt docker trên mỗi máy 
### Bước 2 : 
#### Trên máy VM-1 : 
  
Tạo volume cho mariadb
```sh
    $ docker volume create --name mariadb_data
```
Run , connect MariaDB với mạng đã tạo
```sh
    $ docker run -d --name mariadb \
    --env ALLOW_EMPTY_PASSWORD=yes \
    --env MARIADB_USER=bn_wordpress \
    --env MARIADB_PASSWORD=bitnami \
    --env MARIADB_DATABASE=bitnami_wordpress \
    --network wordpress-network \
    --volume mariadb_data:/bitnami/mariadb \
    bitnami/mariadb:latest
```
### Trên máy VM-2:
Tạo volume cho Wordpress
```sh
      $ docker volume create --name wordpress_data
``` 
Run,connect với network 
```sh
     $docker run -d --name wordpress \
    -p 8080:8080 -p 8443:8443 \
    --env ALLOW_EMPTY_PASSWORD=yes \
    --env WORDPRESS_DATABASE_USER=bn_wordpress \
    --env WORDPRESS_DATABASE_PASSWORD=bitnami \
    --env WORDPRESS_DATABASE_NAME=bitnami_wordpress \
    --network wordpress-network \
    --volume wordpress_data:/bitnami/wordpress \
    bitnami/wordpress:latest
```
    

    
