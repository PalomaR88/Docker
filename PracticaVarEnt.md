## Configuración de contenedores docker con variables de entorno
### Configuración de apache2 en un contenedor docker.
Se va a crear una imagen docker que ofrece Apache2 y, además, configurar el servicio. En este caso, se quiere modificar el parámetro **ServerName** de la configuración del virtualhost.

Configuración del fichero Dockerfile:
~~~
FROM debian
MAINTAINER Paloma R. Garcia "palomagarciacampon08@gmail.com"

RUN apt-get update && apt-get install -y apache2 && apt-get clean && rm -rf /var/lib/apt/lists/*

ENV APACHE_SERVER_NAME www.example.com

EXPOSE 80
ADD ["index.html","/var/www/html/"]

ADD script.sh /usr/local/bin/script.sh
RUN chmod +x /usr/local/bin/script.sh

CMD ["/usr/local/bin/script.sh"]
~~~

- FROM: imagen que se va a usar.
- RUN: acciones que se van a realizar la primera vez que se inicie el contenedor.
- ENV: creación de la vvariable de entorno. APACHE_SERVER_NAME es el nombre de la variable que se va a crear y, a continuación, el valor de la variable. 
- EXPOSE: indica el contenedor por donde se va a escuchar.
- ADD: como un COPY pero permite acciones que COPY no (como descomprimir ficheros introduciendolo comprimido, introducir URL, etc.)
- CMD: es como ENTRYPOINT, pero si se coloca ENTRYPOINT no permite que a la hora de crear el contenedor se introduzcan otras acciones a ejecutar y modifique lo que manda a hacer el CMD.

Antes de lanzar el contenedor se crea el index.html y el fichero script.sh con el siguiente contenido:
~~~
#!/bin/bash
sed -i "s/#ServerName www.example.com/ServerName ${APACHE_SERVER_NAME}/g" /etc/apache2/sites-available/000-default.conf
apache2ctl -D FOREGROUND
~~~

Se crea la imagen:
~~~
sudo docker build -t palomar88/myapache2:v1 .
~~~

Se crea el contenedor:
~~~
sudo docker run -d --name prueba -p 8081:80 palomar88/myapache2:v1
~~~

Se comprueba que se ha creado:
~~~
vagrant@servidor:~/variablesEnt$ sudo docker exec prueba env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=85e969fda9ba
APACHE_SERVER_NAME=www.example.com
HOME=/root
~~~

Y cómo se ha creado el virtualhost:
~~~
vagrant@servidor:~/variablesEnt$ sudo docker exec prueba cat /etc/apache2/sites-available/000-default.conf
...
	ServerName www.example.com
...
~~~

### Configurando nuestro contenedor con variables de entorno
Se va a crear un contenedor desde una imagen de mariadb que tenga un usuario, **usuario**, y una contraseña **asdasd**, con una base de datos llamada **mibasededatos**.
~~~
docker run -d --name servidor_mysql -e MYSQL_DATABASE=db_wp -e MYSQL_USER=user_wp -e MYSQL_PASSWORD=asdasd -e MYSQL_ROOT_PASSWORD=asdasd mariadb
~~~



> Ampliación:
Se inicia un contenedor que se borre cuando haga lo que se vaya a hacer. Y lo que se va a hacer es enlazarse al servidor de mysql que se ha creado anteriormente y entrar en una bash.
~~~
docker run -it --rm --name prueba --link servidor_mysql:mysql debian /bin/bash
~~~

## Enlazando contenedores docker
### Instalación de wordpress en docker
Se utiliza el contenedor de mysql que se ha creado antes:
~~~
docker run --name servidor_wp -p 80:80 --link servidor_mysql:mysql -d wordpress
~~~

Las variables del nuevo contenedor son:
~~~
vagrant@servidor:~$ sudo docker exec servidor_wp env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=0f203e1e4ea1
MYSQL_PORT=tcp://172.17.0.2:3306
MYSQL_PORT_3306_TCP=tcp://172.17.0.2:3306
MYSQL_PORT_3306_TCP_ADDR=172.17.0.2
MYSQL_PORT_3306_TCP_PORT=3306
MYSQL_PORT_3306_TCP_PROTO=tcp
MYSQL_NAME=/servidor_wp/mysql
MYSQL_ENV_MYSQL_DATABASE=db_wp
MYSQL_ENV_MYSQL_USER=user_wp
MYSQL_ENV_MYSQL_PASSWORD=asdasd
MYSQL_ENV_MYSQL_ROOT_PASSWORD=asdasd
MYSQL_ENV_GOSU_VERSION=1.10
MYSQL_ENV_GPG_KEYS=177F4010FE56CA3336300305F1656F24C74CD1D8
MYSQL_ENV_MARIADB_MAJOR=10.4
MYSQL_ENV_MARIADB_VERSION=1:10.4.12+maria~bionic
PHPIZE_DEPS=autoconf 		dpkg-dev 		file 		g++ 		gcc 		libc-dev 		make 		pkg-config 		re2c
PHP_INI_DIR=/usr/local/etc/php
APACHE_CONFDIR=/etc/apache2
APACHE_ENVVARS=/etc/apache2/envvars
PHP_EXTRA_BUILD_DEPS=apache2-dev
PHP_EXTRA_CONFIGURE_ARGS=--with-apxs2 --disable-cgi
PHP_CFLAGS=-fstack-protector-strong -fpic -fpie -O2 -D_LARGEFILE_SOURCE -D_FILE_OFFSET_BITS=64
PHP_CPPFLAGS=-fstack-protector-strong -fpic -fpie -O2 -D_LARGEFILE_SOURCE -D_FILE_OFFSET_BITS=64
PHP_LDFLAGS=-Wl,-O1 -Wl,--hash-style=both -pie
GPG_KEYS=CBAF69F173A0FEA4B537F470D66C9593118BCCB6 F38252826ACD957EF380D39F2F7956BC5DA04B5D
PHP_VERSION=7.3.14
PHP_URL=https://www.php.net/get/php-7.3.14.tar.xz/from/this/mirror
PHP_ASC_URL=https://www.php.net/get/php-7.3.14.tar.xz.asc/from/this/mirror
PHP_SHA256=cc05dd373ca5d36652800762f65c10e828a17de35aaf246262e3efa99d00cdb0
PHP_MD5=
WORDPRESS_VERSION=5.3.2
WORDPRESS_SHA1=fded476f112dbab14e3b5acddd2bcfa550e7b01b
HOME=/root
~~~

Y se peude ver el contenido del fichero wp-config.php:
~~~
vagrant@servidor:~$ sudo docker exec servidor_wp cat wp-config.php
<?php
/**
 * The base configuration for WordPress
 *
 * The wp-config.php creation script uses this file during the
 * installation. You don't have to use the web site, you can
 * copy this file to "wp-config.php" and fill in the values.
 *
 * This file contains the following configurations:
 *
 * * MySQL settings
 * * Secret keys
 * * Database table prefix
 * * ABSPATH
...
~~~

> Esta forma de usar link para interconectar contenedores está en desuso y no es recomendable.

### Creación de redes docker
Si no se indica, por defecto, se crea un bridge:
~~~
docker network create mired
~~~

Para ver las opciones de crear redes:
~~~
docker network
~~~

Opciones para conectar al contenedor a una red concreta:
~~~
--network <nombreRed>
~~~

> El ejercicio anteriores, correstamente, sería:
Se crea una red:
~~~
vagrant@servidor:~$ sudo docker network create red_wp
c7609efbfd737d871ca9c0dceb94eac69cfde54677ac269e0d97ad62838a58aa
~~~

Se crea la máquina para la base de datos:
~~~
vagrant@servidor:~$ sudo docker run -d --name servidor_mysql --network red_wp -e MYSQL_DATABASE=bd_wp -e MYSQL_USER=user_wp -e MYSQL_PASSWORD=asdasd -e MYSQL_ROOT_PASSWORD=asdasd mariadb
f5a793b942d4ee6d241159927c6b9bcc310641a118b28cdd097debb8a0e07023
~~~

Se crea la máquina con la aplicación wordpress:
~~~
vagrant@servidor:~$ sudo docker run -d --name servidor_wp --network red_wp -e WORDPRESS_DB_HOST=servidor_mysql -e WORDPRESS_DB_USER=user_wp -e WORDPRESS_DB_PASSWORD=asdasd -e WORDPRESS_DB_NAME=bd_wp -p 80:80  wordpress
f576478a3a0a396cc272cacaf0d7b6670c518589abe49bb7db4900804e0b5df3
~~~

### Almacenamiento persistente en los contenedores
#### Contenedor mariadb con almacenamiento persistente
Se guarda la información de la base de datos en un volumen persistente:
~~~
vagrant@servidor:~$ sudo docker run --name some-mysql -v /opt/mysql:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=asdasd -d mariadb
3958f0c03913a2fb9c4cfd36870917931b88bee70a8b31b2a8956618997f2e3e
~~~

Se comprueba que se ha creado correctamente:
~~~
vagrant@servidor:~$ ls /opt/mysql/
aria_log.00000001  ibdata1      ibtmp1             performance_schema
aria_log_control   ib_logfile0  multi-master.info
ib_buffer_pool     ib_logfile1  mysql
~~~

Se crea una base de datos:
~~~
vagrant@servidor:~$ sudo docker exec -it some-mysql bash
root@3958f0c03913:/# mysql -u root -p -h localhost
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 8
Server version: 10.4.12-MariaDB-1:10.4.12+maria~bionic mariadb.org binary distribution

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> create database dbtest;
Query OK, 1 row affected (0.004 sec)
~~~

Otra forma de conectarse, se crea un contenedor que se borre tras probar lo que se quiera probar con --rm:
~~~
vagrant@servidor:~$ sudo docker run -it --rm --link some-mysql:mysql mariadb mysql -hmysql -uroot -p
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 9
Server version: 10.4.12-MariaDB-1:10.4.12+maria~bionic mariadb.org binary distribution

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> 
~~~

- Comprobación:
Se simula un fallo:
~~~
vagrant@servidor:~$ sudo docker rm -f some-mysql 
some-mysql
~~~

Y se crea otro contenedor y se comprueba que la base de datos sigue existiendo:
~~~
vagrant@servidor:~$ sudo docker run --name some-mysql2 -v /opt/mysql:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=asdasd -d mariadb
dec236ee5b7b2b10a56f678ccb2ecc1e944f67eb7d1148e11d9827766587909a

vagrant@servidor:~$ docker exec -it some-mysql2 bash
Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get http://%2Fvar%2Frun%2Fdocker.sock/v1.39/containers/some-mysql2/json: dial unix /var/run/docker.sock: connect: permission denied
vagrant@servidor:~$ sudo docker exec -it some-mysql2 bash
root@dec236ee5b7b:/# mysql -u root -p -h localhost
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 8
Server version: 10.4.12-MariaDB-1:10.4.12+maria~bionic mariadb.org binary distribution

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| dbtest             |
| information_schema |
| mysql              |
| performance_schema |
+--------------------+
4 rows in set (0.004 sec)

MariaDB [(none)]> 
~~~



### Instalación de wordpress de forma persistente
Al instalar el wordpress en el ejercicio anterior, si eliminamos el contenedor todos los datos del wordpress se perdían (directorio wp-content). Por lo tanto vamos a crear el contenedor de wordpress con un volumen asociado:
~~~
docker run -d --name servidor_mysql --network red_wp -v /opt/mysql_wp:/var/lib/mysql -e MYSQL_DATABASE=bd_wp -e MYSQL_USER=user_wp -e MYSQL_PASSWORD=asdasd -e MYSQL_ROOT_PASSWORD=asdasd mariadb

docker run -d --name servidor_wp --network red_wp -v /opt/wordpress:/var/www/html/wp-content -e WORDPRESS_DB_HOST=servidor_mysql -e WORDPRESS_DB_USER=user_wp -e WORDPRESS_DB_PASSWORD=asdasd -e WORDPRESS_DB_NAME=bd_wp -p 80:80  wordpress
~~~


### Creación de imágenes con volúmenes desde Dockerfile
Cuando creamos una imagen desde Dockerfile podemos indicar la creación de volúmenes: Gestionando el almacenamiento docker con Dockerfile.

## Docker-compose
Para gestionar contenedores.

### Intralación 
~~~
vagrant@servidor:~$ sudo apt install docker-compose
~~~

También se puede con pip en un entorno virtual:
~~~
python3 -m venv docker-compose
source docker-compose/bin/activate
(docker-compose) ~# pip install docker-compose
~~~

### Configuración de docker-compose
El fichero **docker-compose.yml** se define el escenario. Docker-compose se debe ejecutar en el direcotorio donde este este fichero. Para la ejecución de wordpress persistente podríamos tener el sigueinte contenido:

En el fichero docker-compose.yml vamos a definir el escenario. El programa docker-compose se debe ejecutar en el directorio donde este ese fichero. Por ejemplo para la ejecución de wordpress persistente podríamos tener un fichero con el siguiente contenido:
~~~
version: '3.1'

services:

  wordpress:
    container_name: servidor_wp
    image: wordpress
    restart: always
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: user_wp
      WORDPRESS_DB_PASSWORD: asdasd
      WORDPRESS_DB_NAME: bd_wp
    ports:
      - 80:80
    volumes:
      - /opt/wordpress:/var/www/html/wp-content

  db:
    container_name: servidor_mysql
    image: mariadb
    restart: always
    environment:
      MYSQL_DATABASE: bd_wp
      MYSQL_USER: user_wp
      MYSQL_PASSWORD: asdasd
      MYSQL_ROOT_PASSWORD: asdasd
    volumes:
      - /opt/mysql_wp:/var/lib/mysql
~~~

Con docker-compose se crea una nueva red definida por el usuario donde se contectan los contenedores, por lo tanto se están enlazando. Pero no comparten las variables de entorno. 

~~~
vagrant@servidor:~/compose$ sudo docker-compose up -d
Creating servidor_mysql ... done
Creating servidor_wp    ... done
~~~

Listar:
~~~
vagrant@servidor:~/compose$ sudo docker-compose ps
     Name                  Command              State         Ports       
--------------------------------------------------------------------------
servidor_mysql   docker-entrypoint.sh mysqld    Up      3306/tcp          
servidor_wp      docker-entrypoint.sh apach     Up      0.0.0.0:80->80/tcp
                 ...         
~~~

Parar:
~~~
vagrant@servidor:~/compose$ sudo docker-compose stop 
Stopping servidor_wp    ... done
Stopping servidor_mysql ... done
~~~

Borrar:
~~~
vagrant@servidor:~/compose$ sudo docker-compose rm
Going to remove servidor_wp, servidor_mysql
Are you sure? [yN] y
Removing servidor_wp    ... done
Removing servidor_mysql ... done
~~~


