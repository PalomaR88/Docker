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




