# Wordpress with docker 

Create wordpress site and mysql database with nginx


**Stap1:** pull all needed images 
```bash
docker pull wordpress:latest
docker pull nginx:latest
docker pull mysql:5.7
```

**Stap2:** create network and check it 
```bash
docker network create --driver bridge --subnet=172.30.10.0/24 wp-net
docker network ls
docker inspect wp-net
```

**Stap3:** create volume and check it 
```bash
docker volume create --driver local --name wp-data
docker volume create --driver local --name db-data
docker volume ls
docker inspect wp-data
docker inspect db-data
```

**Stap4:** create nginx directory
```bash 
mkdir -p /home/ahmad/DockerMe/wp/nginx/conf.d
mkdir -p /home/ahmad/DockerMe/wp/nginx/cert
tree /home/ahmad/DockerMe/wp
```

**Stap5:** run mysql service and check it
```bash
docker run -itd --name mysql --hostname mysql \
--network=wp-net --network-alias=db --ip=172.30.10.10 \
--restart=always --memory=512m \
--mount=source=db-data,target=/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=EUEBmxTYtgrXdsdsnfHJJwE9V9fKK7Anha \
-e MYSQL_DATABASE=wordpress \
-e MYSQL_USER=wordpress \
-e MYSQL_PASSWORD=EUEBmxTYtgrXdsdsnfHJJwE9V9fKK7Anha \
mysql:5.7
```

**Stap6:** check mysql services
```bash
docker ps
docker stats mysql
docker exec -i mysql mysql -u root -pEUEBmxTYtgrXdsdsnfHJJwE9V9fKK7Anha  <<< "show databases"
```

**Stap7:** run wordpress service and check it
```bash
docker run -itd --name wordpress --hostname wordpress \
--network=wp-net --network-alias=wp --ip=172.30.10.20 \
--restart=always --memory=1024m \
--mount=source=wp-data,target=/var/www/html/ \
-e WORDPRESS_DB_PASSWORD=EUEBmxTYtgrXdsdsnfHJJwE9V9fKK7Anha \
-e WORDPRESS_DB_HOST=db:3306 \
--link mysql:db \
wordpress:latest
```

**Stap8:** check wordpress services
```bash
docker ps
docker stats --no-stream
docker logs -f wordpress 
curl -I -L 172.30.10.20
```

**Stap9:** create nginx config file for wordpress proxy pass
```bash
vim /home/ahmad/DockerMe/wp/nginx/conf.d/wordpress.conf
server {
  listen 80;
  server_name test.dockerme.ir;
    location / {
      proxy_pass            http://wordpress:80;
      proxy_set_header  Host              $http_host;   # required for docker client's sake
      proxy_set_header  X-Real-IP         $remote_addr; # pass on real client's IP
      proxy_set_header  X-Forwarded-For   $proxy_add_x_forwarded_for;
      proxy_set_header  X-Forwarded-Proto $scheme;
      add_header X-Powered-By "Ahmad Rafiee | DockerMe.ir";
        }
 }
```

**Stap10:** run nginx services and check it
```bash
docker run -itd --name nginx --hostname nginx \
--network=wp-net --network-alias=web --ip=172.30.10.30 \
--restart=always --memory=512m \
--volume=/home/ahmad/DockerMe/wp/nginx/conf.d:/etc/nginx/conf.d \
--volume=/home/ahmad/DockerMe/wp/nginx/cert:/etc/nginx/cert \
--publish=80:80 --publish=443:443 \
--link wordpress:wp \
nginx:latest
```

**Stap11:** check nginx services
```bash
docker ps
docker stats --no-stream
curl -I -L http://test.dockerme.ir
```

**Stap12:** backup databases
```bash
docker exec -i mysql mysqldump -u root -pEUEBmxTYtgrXdsdsnfHJJwE9V9fKK7Anha --all-databases --single-transaction --quick  > full-backup-$(date +%F).sql
```


## Short way
### Run mysql and wordpress without nginx. 
**Step1:** run mysql container
```bash
docker run -it -d --name db -e MYSQL_ROOT_PASSWORD=salamdonya -e MYSQL_DATABASE=wordpress -e MYSQL_USER=wordpress -e MYSQL_PASSWORD=salamdonya mysql:5.7
```
**Step2:** run wordpress container
```bash
docker run -it -d -p 80:80 --link db:db --name wordpress -e WORDPRESS_DB_HOST=db:3306 -e WORDPRESS_DB_PASSWORD=salamdonya wordpress:latest
```

# Run wordpress with docker-compose command.
**step1:** create compose file
```bash
vim docker-compose.yml
version: '3'
services:
  mysql:
    image: 'mysql:5.7'
    container_name: mysql
    restart: always
    environment:
      - MYSQL_ROOT_PASSWORD=EUEBmxTYtgrXdsdsnfHJJwE9V9fKK7Anha
      - MYSQL_DATABASE=wordpress
      - MYSQL_USER=wordpress
      - MYSQL_PASSWORD=EUEBmxTYtgrXdsdsnfHJJwE9V9fKK7Anha
  wordpress:
    image: 'wordpress:latest'      
    container_name: wordpress
    restart: always
    environment:
      - WORDPRESS_DB_HOST=mysql:3306
      - WORDPRESS_DB_USER=wordpress
      - WORDPRESS_DB_PASSWORD=EUEBmxTYtgrXdsdsnfHJJwE9V9fKK7Anha
      - WORDPRESS_DB_NAME=wordpress
        links:
          - 'mysql:db'
  nginx:
    image: 'nginx:latest'
    container_name: nginx
    restart: always
    volumes:
      - '/home/ahmad/DockerMe/wp/nginx/conf.d:/etc/nginx/conf.d'
      - '/home/ahmad/DockerMe/wp/nginx/cert:/etc/nginx/cert'
    ports:
      - '80:80'
      - '443:443'
    links:
      - 'wordpress:wp'
```
**Step2:** check compose file syntax
```bash
docker-compose config 
```

**Step3:** run compose file with docker-compose commands
```bash
docker-compose up -d 
```
**Step4:** check running services and services logs 
```bash
docker-compose ps
docker-compose logs -f --tail 10
```

## License
[DockerMe.ir](https://dockerme.ir)
