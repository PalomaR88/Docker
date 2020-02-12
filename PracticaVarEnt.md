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


