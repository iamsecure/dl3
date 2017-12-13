### How To Install Wordpress With Docker On Ubuntu 16.04 LTS And Backup Data With Bitbucket?

#### Introduction

This article provides a real-world example of using Docker Compose to install WordPress with Docker and Nginx and MariaDB. WordPress normally runs on a LAMP stack, which means Linux, Apache, MySQL/MariaDB, and PHP. The official WordPress Docker image includes Apache and PHP, so the only part we have to install is MariaDB.

WordPress is a free and open-source content management system (CMS) based on PHP and MySQL.

#### Prerequisites

To follow this article, you will need the following:
* Deploy a new VC2 instance On [vultr](https://my.vultr.com/deploy/) and running Ubuntu 16.04 LTS (memory >= 1G)
* Shell acces to the server using SSH remote console. We use [Item2 ssh-client](https://www.iterm2.com/) on Macos.

#### Step1 ssh ubuntu server
```
ssh root@server_ip
```
For a new ubuntu server, you may need to update source.
```
apt-get update
```

#### Step2 Install docker
```
wget -qO- https://get.docker.com/ | sh
docker ps
```
The output will be similar to this:
```
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS      NAMES
```

Install docker success.

#### Step3 Get Mariadb docker images
```
docker pull mariadb
```
The output will be similar to this:
```
Using default tag: latest
latest: Pulling from library/mariadb
f49cf87b52c1: Pull complete
78032de49d65: Pull complete
84229c396ea3: Pull complete
c992bbf20f4e: Pull complete
380e932be3aa: Pull complete
8003c6d68fcd: Pull complete
603f60c36160: Pull complete
0ca5691f5d0e: Pull complete
0d278e76dd60: Pull complete
c4b6848ec164: Pull complete
5f0f8875ccf3: Pull complete
Digest: sha256:e2b1ed9c3cdca35c0b4d8e5e919626c1c9ccf0482fe20ae7dd58944bebf8546b
Status: Downloaded newer image for mariadb:latest
```
#### Step4 Get Wordpress docker images
```
docker pull wordpress:latest
```
The output will be similar to this:
```
latest: Pulling from library/wordpress
e7bb522d92ff: Pull complete
2580aa42d1b6: Pull complete
1a8abb84c6a3: Pull complete
7bff113cc3d6: Pull complete
dd1a4dcbc4ec: Pull complete
ab893328ce84: Pull complete
13367be12a3f: Pull complete
892f2a2b8c32: Pull complete
f17d3f3dc1af: Pull complete
b6e71bdb2060: Pull complete
36ee5f422b46: Pull complete
f57ac824c32c: Pull complete
f62afcfdde73: Pull complete
c315d6df5bb1: Pull complete
21b0fa595035: Pull complete
176a9ac56651: Pull complete
26d4680488be: Pull complete
07a88cd98fd5: Pull complete
Digest: sha256:2d907dbc2aac81fcfcbae13c7ed4597f57d82142ca7653cc28102f087c5be4d5
Status: Downloaded newer image for wordpress:latest
```
Then
```
docker images
```
The output will be similar to this:
```
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
wordpress           latest              f345336550a0        19 hours ago        410MB
mariadb             latest              b1fe0881b739        26 hours ago        398MB
```
Get Mariadb and Wordpress docker mages sucess.

#### Step5 Make dir for database and wordpress files
```
mkdir /var/www/
mkdir /var/www/tor9/ 
mkdir /var/www/tor9/database 
mkdir /var/www/tor9/html
```
#### Step6 Run Mariadb
```
docker run -e MYSQL_ROOT_PASSWORD=root123456 \
-e MYSQL_USER=wpuser \
-e MYSQL_PASSWORD=user123456 \
-e MYSQL_DATABASE=wp_tor9 \
-v /var/www/tor9/database:/var/lib/mysql \
-p 3306:3306 \
--name tor9_db \
-d mariadb
```
Then
```
docker ps
```
The output will be similar to this:
```
CONTAINER ID     IMAGE      COMMAND              CREATED            STATUS                    PORTS               NAMES
5b89da142af0     mariadb    "docker-entrypoint.s…"   5 seconds ago       Up 4 seconds        3306/tcp            tor9_db
```
Docker run Mariadb success. We can access database with port(3306) and user(root/root123456 or wpuser/user123456). The database wp_tor9 has been created for wordpress and data is in the dir of var/www/tor9/database.

The docker run command is used to run a process in a container. It is the go-to command for launching new containers. Its options allow users to configure how the Docker image is run, override Dockerfile settings, configure networking, and set privileges and resources for the container.

-v /var/www/tor9/database:/var/lib/mysql<br>
The directory of /var/www/tor9/database will mount on /var/lib/mysql in docker container.

-p 3306:3306<br>
We can use mysql client with port 3306 to access this database docker container. And you can use '-p 3307:3306' or other port for this port map. It maps port 3307 on ubuntu server with 3306 on docker container.

--name tor9_db<br>
docker container name

-d mariadb<br>
docker image name

#### Step7 Run Wordpress with Mariadb
```
docker run -e WORDPRESS_DB_USER=wpuser \
-e WORDPRESS_DB_PASSWORD=user123456 \
-e WORDPRESS_DB_NAME=wp_tor9 \
-p 8081:80 \
-v /var/www/tor9/html:/var/www/html \
--link tor9_db:mysql --name tor9_wp -d wordpress
```
Then
```
Notice:
```
--link tor9_db:mysql 
```

docker ps
```
The output will be similar to this:
```
ONTAINER ID     IMAGE        COMMAND                  CREATED             STATUS       PORTS                NAMES
49ded2d248a2    wordpress   "docker-entrypoint.s…"   3 seconds ago    Up 3 seconds  0.0.0.0:8081->80/tcp     tor9_wp
d4ba34d117f0    mariadb     "docker-entrypoint.s…"   6 minutes ago    Up 6 minutes  0.0.0.0:3306->3306/tcp   tor9_db
```
Docker run Wordpress success.

#### Step8 Install Nginx
```
apt-get update && apt-get install vim nginx -y
```
config nginx for domain 
```
vim /etc/nginx/conf.d/tor9.conf
```
tor9.conf
```
server {
listen 80;
server_name tor9.xxx.com;

location / {
    proxy_pass http://localhost:8081;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
}
}
```
restart nginx
```
service nginx restart
```

#### Step9 Config your domain 
You need to config your domain(CName) with Server Ip.

#### Step10 Access your wordpress website

![](https://github.com/peimin/pic/blob/master/20171213%20wordpress1.png?raw=true)

Auto backup wordpress and database data to bitbucket

#### Step11 Create a free acount on bitbucket
```
https://bitbucket.org
```

#### Step12 Create a repository on bitbucket 

![](https://github.com/peimin/pic/blob/master/20171213%20bit1.png?raw=true)<br>
![](https://github.com/peimin/pic/blob/master/20171213%20%20bit2.png?raw=true)<br>
![](https://github.com/peimin/pic/blob/master/20171213%20%20bit3.png?raw=true)<br>

#### Step13 Add ssh key to bitbucket

```
ssh-keygen -t rsa
cat /root/.ssh/id_rsa.pub
```

#### Step14 Init Git dir on ubuntu server 
```
cd /var/www/tor9/
git init
git add .
git commit -a m 'backup'
git remote add origin git@bitbucket.org:tor9/tor9.git
git push -u origin master
```
The output will be similar to this:
```
Counting objects: 1760, done.
Compressing objects: 100% (1731/1731), done.
Writing objects: 100% (1760/1760), 11.43 MiB | 1.51 MiB/s, done.
Total 1760 (delta 220), reused 0 (delta 0)
To git@bitbucket.org:tor9/tor9.git
 * [new branch]      master -> master
Branch master set up to track remote branch master from origin.
```

#### Step15 Auto backup with crontab

```
crontab -e
```
add 
```
*/5  * * * * cd /var/www/tor9 && git add . && git commit -a -m 'backup' && git pull origin master && git push origin master
```
It will be backup to bitbucket by every 5 minute.
```
crontab -l
```

It is all. Enjoy!

#### References:
* https://en.wikipedia.org/wiki/WordPress
* https://hub.docker.com/_/wordpress/
