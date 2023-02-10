# Introduction

Provision an ec2 Linux and attach the following user data

#!/bin/bash
sudo yum update -y
sudo amazon-linux-extras install docker -y
sudo service docker start
sudo yum install git -y
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
curl -L https://raw.githubusercontent.com/docker/compose-cli/main/scripts/install/install_linux.sh | sh
sudo usermod -a -G docker ec2-user
sudo service docker start
sudo chkconfig docker on

Here's how to deploy it on CentOS systems:

## Deploy Pre-Requisites

1. Install FirewallD

```
sudo yum install -y firewalld
sudo service firewalld start
sudo systemctl enable firewalld
```

## Deploy and Configure Database

1. Install MariaDB

```
sudo yum install -y mariadb-server
sudo vi /etc/my.cnf
sudo service mariadb start
sudo systemctl enable mariadb
```

2. Configure firewall for Database

```
sudo firewall-cmd --permanent --zone=public --add-port=3306/tcp
sudo firewall-cmd --reload
```

3. Configure Database

```
$ mysql
MariaDB > CREATE DATABASE ecomdb;
MariaDB > CREATE USER 'ecomuser'@'localhost' IDENTIFIED BY 'ecompassword';
MariaDB > GRANT ALL PRIVILEGES ON *.* TO 'ecomuser'@'localhost';
MariaDB > FLUSH PRIVILEGES;
```

> ON a multi-node setup remember to provide the IP address of the web server here: `'ecomuser'@'web-server-ip'`

4. Load Product Inventory Information to database

Create the db-load-script.sql

```
cat > db-load-script.sql <<-EOF
USE ecomdb;
CREATE TABLE products (id mediumint(8) unsigned NOT NULL auto_increment,Name varchar(255) default NULL,Price varchar(255) default NULL, ImageUrl varchar(255) default NULL,PRIMARY KEY (id)) AUTO_INCREMENT=1;

INSERT INTO products (Name,Price,ImageUrl) VALUES ("Laptop","100","c-1.png"),("Drone","200","c-2.png"),("VR","300","c-3.png"),("Tablet","50","c-5.png"),("Watch","90","c-6.png"),("Phone Covers","20","c-7.png"),("Phone","80","c-8.png"),("Laptop","150","c-4.png");

EOF
```

Run sql script

```

mysql < db-load-script.sql
```


## Deploy and Configure Web

1. Install required packages

```
sudo yum install -y httpd php php-mysql
sudo firewall-cmd --permanent --zone=public --add-port=80/tcp
sudo firewall-cmd --reload
```

2. Configure httpd

Change `DirectoryIndex index.html` to `DirectoryIndex index.php` to make the php page the default page

```
sudo sed -i 's/index.html/index.php/g' /etc/httpd/conf/httpd.conf
```

3. Start httpd

```
sudo service httpd start
sudo systemctl enable httpd
```

4. Download code

```
sudo yum install -y git
git clone https://github.com/kodekloudhub/learning-app-ecommerce.git /var/www/html/
```

5. Update index.php

Update [index.php](https://github.com/kodekloudhub/learning-app-ecommerce/blob/13b6e9ddc867eff30368c7e4f013164a85e2dccb/index.php#L107) file to connect to the right database server. In this case `localhost` since the database is on the same server.

```
sudo sed -i 's/172.20.1.101/localhost/g' /var/www/html/index.php

              <?php
                        $link = mysqli_connect('172.20.1.101', 'ecomuser', 'ecompassword', 'ecomdb');
                        if ($link) {
                        $res = mysqli_query($link, "select * from products;");
                        while ($row = mysqli_fetch_assoc($res)) { ?>
```

> ON a multi-node setup remember to provide the IP address of the database server here.
```
sudo sed -i 's/172.20.1.101/localhost/g' /var/www/html/index.php
```

6. Test

```
curl http://localhost
```
FROM php:8.0-apache
RUN docker-php-ext-install mysqli && docker-php-ext-enable mysqli
RUN apt-get update && apt-get upgrade -y
COPY ./ /var/www/html/

Docker guideline
------
docker image
------------
its template/snapshot for run a container/service

docker file (how to create docker file)
---------------------------------------
to create docker image we need to create docker file first. once we setup dokcer file we can create image by executing special commands.
dokcer build -t docker-file-name

docker file syntax 
------------------
docker file contain instruction and argument

FROM - base image
COPY - copy from local host to container
ADD - can copy friles, can take source from url, can extract from local and copy to container
WORKDIR - to set work dir
RUN - execute command while creatign docker file
CMD - aftre creating container, inside container will execute CMD command
EXPOSE - to expose port number
MAINTAINER - who is maintaining this image
ENV - to setup environment variable
ENTRYPOINT - wehnever you initiate container, this must be executed.

@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@

Create Flask Application
-------------------------
yum install python -y
yum install python-pip -y
pip install flask 

copy your python app to current directory
https://github.com/mohammedashiqu/docker-flask-python.git
FLASK_APP=app.py flask run --host=0.0.0.0 --port=100

dockerize this application
---------------------------
create docker file:
-------------------
https://github.com/mohammedashiqu/docker-flask-python.git

FROM amazonlinux
RUN yum install python -y
RUN yum install python-pip -y
RUN pip install flask 
COPY . /opt/
ENTRYPOINT FLASK_APP=/opt/app.py flask run --host=0.0.0.0 --port=8080

create image from this file:
---------------------------
docker build -t custom-name .

docker run -d -p 5001:8080 cont-id

@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@

how to push docker image from local to dokcer hub
-------------------------------------------------
name should be "ashiqummathoor/image-name"
docker login
docker push ashiqummathoor/image-name:tag

@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@

create image from application
-----------------------------
download any website template from free css and unzip and copy all files and folders in current directory create docker file

docker file
-----------
FROM httpd
COPY . /usr/local/apache2/htdocs/
RUN ls -l /usr/local/apache2/htdocs

docker build -t app1
docker run -d -p 1000:80 app1


come out from containerwithout exit
-------------------------------------
ctr+p and ctr+q

environmental variable
-----------------------
What is environmental variables and why?
Image result for what s environment variables
An environment variable is a variable whose value is set outside the program, typically through functionality built into the 
operating system or microservice. An environment variable is made up of a name/value pair, and any number may be created and available 
for reference at a point in time.

link
-----
mysql
------
docker run -d -p 3306:3306 --name mydatabase -h mydb mariadb
docker run -d -p 1000:80 --name phpmyadmin -h phpadmin --link mydatabase:db phpmyadmin

actualname:providername


mariadb
--------
docker run -d -p 3306:3306 -e MARIADB_ROOT_PASSWORD=root --name=mariadbashiq  mariadb
docker run -d -p 3000:80 --link mariadbashiq:db  phpmyadmin

syntax- local-name:atual-name

inspect envornmental variable
-------------------------------
docker inspect container/image id