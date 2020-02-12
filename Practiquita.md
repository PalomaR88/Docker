## Creación de contendores
~~~
mkdir proyecto
cd proyecto/
mkdir public_html
cd public_html/
nano index.html
~~~

Configuración del Dockerfile:
~~~
FROM debian
RUN apt-get update -y && apt-get install -y \
    apache2 \
    && apt-get clean && rm -rf /var/lib/apt/lists/*
COPY ./public_html /var/www/html/
ENTRYPOINT ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
~~~

Creación de una imagen:
~~~
sudo docker build -t mi_pagina/aplicacionweb:v1 .
~~~

Y la nueva imagen aparece en mi lista de imágenes de docker:
~~~
sudo docker images
REPOSITORY                TAG                 IMAGE ID            CREATED             SIZE
mi_pagina/aplicacionweb   v1                  c201cc40a17e        14 seconds ago      226MB
debian                    latest              a8797652cfd9        4 days ago          114MB
~~~

## Cambiar la página
En este caso, por las opciones de la imagen que copia ./public_html, solo hay que poner el comando:
~~~
sudo docker build -t mi_pagina/aplicacionweb:v1 .
~~~

Y vuelve a crear la imagen, copiando el ./public_html:
~~~
paloma@coatlicue:~/Docker/proyecto$ sudo docker images
REPOSITORY                TAG                 IMAGE ID            CREATED              SIZE
mi_pagina/aplicacionweb   v1                  33200beba9e6        About a minute ago   226MB
<none>                    <none>              c201cc40a17e        5 minutes ago        226MB
debian                    latest              a8797652cfd9        4 days ago           114MB
~~~

Pero la antigua imágen sigue saliendo renombrado (<none>). Para borrarlo:
~~~
sudo docker images -f "dangling=true"
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
<none>              <none>              c201cc40a17e        6 minutes ago       226MB

sudo docker rmi $(sudo docker images -f "dangling=true" -q)
Deleted: sha256:c201cc40a17e4900ac1a6a8982a9df4754098e4bf51d926a7fdcfe4ac8deb020
Deleted: sha256:94a456da6bffef7a55ad1bedfde32e1f14e1dcada755b02294c7d1541d1fe387
Deleted: sha256:3fcfb2180348308e7d682eee3976b7bc7c2ee7517ed2313fe205cdbfda3511e2

sudo docker images
REPOSITORY                TAG                 IMAGE ID            CREATED             SIZE
mi_pagina/aplicacionweb   v1                  33200beba9e6        4 minutes ago       226MB
debian                    latest              a8797652cfd9        4 days ago          114MB
~~~

~~~
sudo docker run -d --name mipag -p 8080:80 mi_pagina/aplicacionweb:v1
~~~

## cración de una nueva versión
~~~
sudo docker build -t mi_pagina/aplicacionweb:v2 .
~~~

~~~
vagrant@servidor:~/Docker/proyecto$ sudo docker images
REPOSITORY                TAG                 IMAGE ID            CREATED             SIZE
mi_pagina/aplicacionweb   v1                  e6782b8f4f53        2 minutes ago       226MB
mi_pagina/aplicacionweb   v2                  e6782b8f4f53        2 minutes ago       226MB
debian                    latest              a8797652cfd9        4 days ago          114MB
~~~

Se va a crear una cuenta en Docker Hub para subir la imagen:
~~~
 docker login
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: palomar88
Password: 
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
~~~

Y se sube la imagen:
~~~
docker push palomar88/mi_pagina:v2
~~~

Y en el entorno de producción:
~~~
docker login
docker pull palomar88/mi_pagina:v2
~~~

## Docker con php
~~~
FROM debian
RUN apt-get update -y && apt-get install -y \
    apache2 php libapache2-mod-php \
    && apt-get clean && rm -rf /var/lib/apt/lists/* \
    && rm /var/www/html/index.html
COPY ./public_html /var/www/html/
ENTRYPOINT ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
~~~