# Proyecto-CTF
Una pequeña guia de como hacer un servidor de Minecraft en docker que se autorespalde cada 12 horas

## Prerrequisitos
- Una computadora/servidor corriendo Docker y Docker compose

- Conocimiento basico de como usar linea de comandos

# Paso 0 - Iniciar el servidor
En este caso iniciare un servidor por medio de una pagina externa [digitalocean](https://www.digitalocean.com/) ya que esta tiene su prueba "gratuita"
![1](https://user-images.githubusercontent.com/62454146/205371888-c2c347e1-6419-41fa-ab65-5e90cff224d5.PNG)

Una vez iniciado el servidor ya que este es por Linux instalaremos las dos  weas que necesitaremos con el comando de

	apt install  docker-compose

Y tambien

	apt install docker.io
	

# Paso 1 - Crear el directorio base y el Volumen de docker
En tu Computadora/Servidor corriendo Docker creamos el directorio para nuestros archivos y plugins.
-       mkdir -p /home/Minecraft/plugins
Creamos el volumen de docker
-       docker volume create minecraftdata

minecraftdata es el nombre del volumen no necesariamente tiene que ser el mismo pero en caso de que cambie también tiene que cambiar el nombre en el docker-compose.yml.
![4 Crear el docker minecraft](https://user-images.githubusercontent.com/62454146/205373885-76d19fbf-11df-488d-9900-8f9cca9a0a58.PNG)


# Paso 2- Descargar/Editar el archivo docker-compose.yml
Descargar la plantilla [aqui](https://drive.google.com/file/d/1TsiBlYplkfruBLNtxW1JeVsp3Az3KF2N/view?usp=share_link)

Usar el editor de texto de tu elección para hacer las modificaciones

 -Pasar el volumen del paso 1 al directorio /data dentro del contenedor

-Pasar el volumen del paso 1 al directorio /plugins dentro del contenedor

Cambia las variables que se necesiten

-EULA - Este Si o Si necesita estar en “TRUE”

-TYPE    - El tipo de servidor que se va a correr (En la [documentacion](https://github.com/itzg/docker-minecraft-server/blob/master/README.md#server-types) viene toda la lista) 

-OPS - La lista de todos los jugadores que van a tener permisos de operador (separados por coma)

-ENABLE_WHITELIST - Permite a los OPS agregar usuarios a la whitelist

-ENFORCE_WHITELIST - No permite que se conecten jugadores que no esten en la whitelist

Asegurate de que la sección de volúmenes tenga el mismo nombre del volumen creado en el paso 1.

Transfiere el archivo docker-compose.yml de la máquina en la que lo estas editando al servidor.

Esto se hace accediendo a la ruta en la cual está el archivo 

         cd /ruta/de/tu/archivo

para transferirlo seria por el comando scp

        scp docker-compose.yml USUARIO@IPSERVIDOR:/home/Minecraft

![copiar yml al server](https://user-images.githubusercontent.com/62454146/205374279-f52369fc-1fb9-4d2d-8ad2-79488120d238.png)

Una vez ya copiado podemos confirmar que el archivo este donde se supone con el comando en el servidor

	ls
	
![6 Arhcivo ya en el server](https://user-images.githubusercontent.com/62454146/205374420-fd5113c6-59b1-49a7-9538-5468fbc34b8d.PNG)


# Paso 3 - Iniciar el contenedor

Accedemos a la ubicación donde guardamos el archivo .yml

        cd /home/Minecraft

Iniciamos el contenedor

        docker-compose up -d –force-recreate
								
![8 Servidor ya jalando](https://user-images.githubusercontent.com/62454146/205374824-512049cf-2411-4b16-9969-5158b1e608cf.PNG)


# Paso 4 (Opcional) - Cargar Plugins
En caso de que estés utilizando un tipo de servidor que soporte plugins pon los archivos .jar en el directorio que se creó en el paso 1

Para esto volveremos a utilizar el comando scp del paso 2 pero ahora en la carpeta de plugins

         scp ejemplo.jar USUARIO@IPSERVIDOR:/home/Minecraft/plugins


### A partir de este punto el servidor de Minecraft es accesible por medio de la IP del servidor y perfectamente jugable, los pasos 5 y 6 son para respaldar y restaurar el servidor respectivamente

# Paso 5 - Crear el script de respaldo

Descargar el script puedes hacerlo [aqui](https://drive.google.com/file/d/1tAC3f_sTcKSLxgg9iffMsl0ag1dunxsP/view?usp=share_link)

Creamos el directorio en el servidor para guardar el archivo

        mkdir /etc/scripts

Creamos el directorio para almacenar los archivos .tar respaldados

    mkdir /etc/scripts/Backups

Transferimos el archivo backup.sh a esta ubicacion

![12 copear el backup](https://user-images.githubusercontent.com/62454146/205374875-a68080e0-d2f6-425e-bf09-dbc21f2c50b7.PNG)

Creamos un cron job para correr el script de respaldo en tu volumen de minecraft

    crontab -e

Creamos un “cron entry” para correr el script con parametros. En este ejemplo se corre el script a mediodía y medianoche

    0 0,12 * * * bash /./etc/scripts/backup.sh -v minecraftdata -c minecraft_minecraft_1 -o /etc/scripts/Backups -r false -d 7

v = Nombre del volumen creado en el paso 1

c=  Nombre del contenedor

o=  Directorio en donde se van a guardar los respaldos 

r=   Si quieres que el directorio se limpie cada vez

d=  Cuantos dias quieres el servidor guarde los respaldos

![15 respaldo](https://user-images.githubusercontent.com/62454146/205374933-a8a4564a-5ee7-45e2-8c26-33c064170418.PNG)

    
Guardar y salir

# Paso 6 - Restaurar el respaldo

Paramos el contenedor

    docker stop minecraft_minecraft_1

Corremos el comando para restaurar el servidor

    docker run –rm –volumes-from minecraft_minecraft_1 -v /home/Minecraft:/backup bash -c “cd /data && tar xvf /backup/backupfile.tar –strip 1


Iniciamos el contenedor otra vez


