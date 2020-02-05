# Docker
Los contenedores son procesos que se van a ejecutar en la máquina. 

## Uso de LXC
- Pertenece a los denominados contenedores de sistemas.
- Los contenedores de aplicaciones están pensados para el despliegue de aplicaciones en arquitectura de microservidores.
- La gran diferencia con otros sistemas de contenedores es que tiene **systemd**, con todo lo que eso conlleva, pero no tiene kernel propio, que lo diferencia de KVM. 
- Los sistemas de ficheros es un directorio.
- se crean contenedores a partir de mecanismos conocidos como **debootstrap**.
- Para accedder al contenedor utilizamos **ssh**.
- Pensado para que tenga una larga vida.
- No compite con docker, porque sirven para otras cosas, sino con otros sistemas de virtualización como KVM.

## Contenedores de aplicaciones
- Contenedores centrados en la aplicación. Más alto nivel que los contenedores de sistemas porque no tiene kernel, tampoco systemd.
- Idealmente ejecuta un proceso.
- Muy relacionado con los microservicios. Pero no significa que solos e use con microservicios, porque además los microservicios dan problemas. 
- Centrado en el desarrollo y despliegue de aplicaciones.

## Cmabio de paradigma
- La aplicación no se isntala más.
- Se crea una imagen con la aplicación y sus dependencias.
- Se gestiona la imagen con control de versiones.
- Excepcionalmente se accede al contenedr en ejecuci´on.
- Lanzar, utilizar y monitorizar.

## Instalación de Docker
Tres conceptos:
- Docker daemon
- Docker client
- Registro imágenes docker - Docker Hub
Repositorio de imágenes de docker. 

Para bajar imágenes:
~~~
docker pull <imagen>
~~~

Las imágenes se dividen en capas. Si hay dos imagen con una capa igual, solo se guarda una capa, y la comparten. Cuando se crea un contenedor se crea una nueva capa de lectura y escritura, pero las capas de la imagen pueden compartirse entre contenedores. 

~~~
vagrant@servidor:~$ sudo apt install docker.io
~~~

Descargar una imagen:
~~~
root@servidor:/home/vagrant# docker pull debian
~~~

Ver imágenes:
~~~
root@servidor:/home/vagrant# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
debian              latest              a8797652cfd9        3 days ago          114MB
~~~

Crear un contenedor:
~~~
root@servidor:/home/vagrant# docker run ubuntu /bin/echo 'Hello world'
Unable to find image 'ubuntu:latest' locally
latest: Pulling from library/ubuntu
5c939e3a4d10: Pull complete 
c63719cdbe7a: Pull complete 
19a861ea6baf: Pull complete 
651c9d2d6c4f: Pull complete 
Digest: sha256:8d31dad0c58f552e890d68bbfb735588b6b820a46e459672d96e585871acc110
Status: Downloaded newer image for ubuntu:latest
Hello world
~~~
Si no tiene la imagen la descarga, y como los contenedores lo que hace son procesos, se le indica que es lo que va a hacer.


Ver contenedores:
~~~
root@servidor:/home/vagrant# docker ps --all
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                     PORTS               NAMES
0afc23b3c405        ubuntu              "/bin/echo hola mundo"   2 minutes ago       Exited (0) 2 minutes ago                       eloquent_nightingale
8e080a347e71        ubuntu              "/bin/echo 3+1"          2 minutes ago       Exited (0) 2 minutes ago                       xenodochial_brattain
b94465246a2b        ubuntu              "/bin/echo 'Hello wo…"   5 minutes ago       Exited (0) 5 minutes ago                       stoic_mendel
~~~

Borrar:
~~~
root@servidor:/home/vagrant# docker rm stoic_mendel
~~~

Contenedor con nombre:
~~~
root@servidor:/home/vagrant# docker run -it --name contenedor1 ubuntu /bin/bash
root@644f86d14cc0:/# 
~~~

Si sales, con exit, tienes que volver a iniciar el contenedor:
~~~
root@644f86d14cc0:/# exit
exit
root@servidor:/home/vagrant# docker start contenedor1
contenedor1
root@servidor:/home/vagrant# docker attach contenedor1
root@644f86d14cc0:/# 
~~~
Attach solo se conecta al contenedor si en ese contenedor se está ejecutando una bash. 

Ejecutar procesos en un contenedor desde fuera, el contenedor tiene que estar activado:
~~~
root@servidor:/home/vagrant# docker exec contenedor1 apt update
~~~

Con -d usa un demonio:
~~~
root@servidor:/home/vagrant# docker run -d --name contenecor2 ubuntu /bin/sh -c "while true; do echo hello word; sleep 1; done"
~~~

- docker top
- docker log

Hay imágenes con procesos por defecto, por ejemplo en este caso, se crea el contenedor con Apache2 que tiene por defecto el proceso que inicia Apache:
~~~
root@servidor:/home/vagrant# docker run -d --name my-apache-app -p 8080:80 httpd:2.4
~~~

Cuando se crea un contenedor se crea un bridge con una interfaz en el anfitrio que se llama docker0 y se crea una interfaz por cada contenedor. 


### Dockerfile
FROM -> la imagen
RUN -> lo que se va a ejecutar una vez
COPY -> copia algo del anfitrion
ENTRYPOINT -> Comando por defecto que se ejecuta cuando inicies el contenedor

docker build -t nombredelaminagen/<nombreimagen>:version .

Con esto se crean imagenes y se pueden crear contenedores a través de esta imagen. 

docker push nombredelaminagen/<nombreimagen>:version

Y se sube la imagen a docker.


