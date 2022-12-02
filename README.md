# Proyecto-CTF
Una pequeña guia de mi servidor hecho con docker
# Paso 1 - Crear el directorio base y el Volumen de docker
En tu Computadora/Servidor corriendo Docker creamos el directorio para nuestros archivos y plugins.
-     mkdir -p /home/Minecraft/plugins
Creamos el volumen de docker
-     docker volume create minecraftdata

minecraftdata es el nombre del volumen no necesariamente tiene que ser el mismo pero en caso de que cambie también tiene que cambiar el nombre en el docker-compose.yml.

# Paso 2- Descargar/Editar el archivo docker-compose.yml
Descargar la plantilla [aqui](https://www.google.com)

Usar el editor de texto de tu elección para hacer las modificaciones

 -Pasar el volumen del paso 1 al directorio /data dentro del contenedor

-Pasar el volumen del paso 1 al directorio /plugins dentro del contenedor


Cambia las variables que se necesiten

-EULA - Este Si o Si necesita estar en “TRUE”

-TYPE    - El tipo de servidor que se va a correr (En la documentacion viene toda la lista) 

-OPS - La lista de todos los jugadores que van a tener permisos de operador (separados por coma)

-ENABLE_WHITELIST - Permite a los OPS agregar usuarios a la whitelist

-ENFORCE_WHITELIST - No permite que se conecten jugadores que no esten en la whitelist

Asegurate de que la sección de volúmenes tenga el mismo nombre del volumen creado en el paso 1.

Transfiere el archivo docker-compose.yml de la máquina en la que lo estas editando al servidor.

Esto se hace accediendo a la ruta en la cual está el archivo 

         cd /ruta/de/tu/archivo

para transferirlo seria por el comando scp

        scp docker-compose.yml USUARIO@IPSERVIDOR:/home/Minecraft


# Paso 3 - Iniciar el contenedor

Accedemos a la ubicación donde guardamos el archivo .yml

        cd /home/Minecraft

Iniciamos el contenedor

        docker-compose up -d –force-recreate


# Paso 4 (Opcional) - Cargar Plugins
En caso de que estés utilizando un tipo de servidor que soporte plugins pon los archivos .jar en el directorio que se creó en el paso 1

Para esto volveremos a utilizar el comando scp del paso 2 pero ahora en la carpeta de plugins

         scp ejemplo.jar USUARIO@IPSERVIDOR:/home/Minecraft/plugins


# Paso 5 - Crear el script de respaldo

Descargar el script puedes hacerlo aqui

Creamos el directorio en el servidor para guardar el archivo

        mkdir /etc/scripts

Creamos el directorio para almacenar los archivos .tar respaldados

    mkdir /etc/scripts/Backups

Transferimos el archivo backup.sh a esta ubicacion

Creamos un cron job para correr el script de respaldo en tu volumen de minecraft

    crontab -e

Creamos un “cron entry” para correr el script con parametros. En este ejemplo se corre el script a mediodía y medianoche

    0 0,12 * * * bash /./etc/scripts/backup.sh -v minecraftdata -c minecraft_minecraft_1 -o /etc/scripts/Backups -r false -d 7

v = Nombre del volumen creado en el paso 1

c=  Nombre del contenedor

o=  Directorio en donde se van a guardar los respaldos 

r=   Si quieres que el directorio se limpie cada vez

d=  Cuantos dias quieres el servidor guarde los respaldos
    
Guardar y salir

# Paso 6 - Restaurar el respaldo

Paramos el contenedor

    docker stop minecraft_minecraft_1

Corremos el comando para restaurar el servidor

    docker run –rm –volumes-from minecraft_minecraft_1 -v /home/Minecraft:/backup bash -c “cd /data && tar xvf /backup/backupfile.tar –strip 1

- Nombre del contenedor
- Ubicacion del respaldo
- Nombre del archivo de respaldo

Iniciamos el contenedor otra vez


