# Máquina: Kenobi

**Tryhackme: Kenobi**

Lo primero que haremos, será lanzar un NMAP para ver qué puertos tiene abiertos la máquina:

![KNOBI1](https://github.com/AntonioPC94/Kenobi/blob/a9c234f800c409ab541d68e9e950e320c16804e8/img/KNOBI1.png)
![KNOBI2](https://github.com/AntonioPC94/Kenobi/blob/a9c234f800c409ab541d68e9e950e320c16804e8/img/KNOBI2.png)
![KNOBI3](https://github.com/AntonioPC94/Kenobi/blob/a9c234f800c409ab541d68e9e950e320c16804e8/img/KNOBI3.png)
![KNOBI4](https://github.com/AntonioPC94/Kenobi/blob/a9c234f800c409ab541d68e9e950e320c16804e8/img/KNOBI4.png)

Como se observa en las imágenes anteriores, existen varios servicios que nos pueden interesar:

- Puerto 21 (FTP)
- Puerto 22 (SSH)
- Puerto 80 (HTTP)
- Puerto 111 (RPC)
- Puerto 139 (SMB)
- Puerto 445 (SMB)

A continuación, usaremos el módulo de escaneo de vulnerabilidades de NMAP para ver si este nos devuelve algo interesante sobre los puertos anteriormente encontrados:

![KNOBI5](https://github.com/AntonioPC94/Kenobi/blob/a9c234f800c409ab541d68e9e950e320c16804e8/img/KNOBI5.png)

En el "http-enum", veremos que existen 2 directorios que pueden ser de interés:

/admin.html: Posible directorio "admin"
/robots.txt

Nota: Con Dirbuster no encontraremos directorios más allá del "robots.txt".

En el "robots.txt", nos encontraremos como "disallow" el directorio que mencionamos anteriormente.

![KNOBI6](https://github.com/AntonioPC94/Kenobi/blob/a9c234f800c409ab541d68e9e950e320c16804e8/img/KNOBI6.png)

Nosotros aunque ponga "disallow", vamos a intentar acceder a dicho directorio para ver si nos encontramos con algo de interés.

![KNOBI7](https://github.com/AntonioPC94/Kenobi/blob/a9c234f800c409ab541d68e9e950e320c16804e8/img/KNOBI7.png)

Observando la imagen anterior, podremos deducir que no estamos yendo por el camino correcto para explotar la máquina objetivo.

Bien, si no podemos hacer mucho con el puerto 80, vamos a intentarlo con el 445, a ver si podemos sacar algo de él.

Lo primero que haremos, será listar las carpetas compartidas en el servicio para ver si hay alguna que nos llame la atención.

![KNOBI8](https://github.com/AntonioPC94/Kenobi/blob/a9c234f800c409ab541d68e9e950e320c16804e8/img/KNOBI8.png)

Como se observa en la imagen anterior, existe un directorio llamado "anonymous" que posiblemente contenga información que nos interese.

Para conectarnos, lo haremos como usuario "anonymous" y si hay suerte, conseguiremos ver el contenido de dicha carpeta.

![KNOBI9](https://github.com/AntonioPC94/Kenobi/blob/a9c234f800c409ab541d68e9e950e320c16804e8/img/KNOBI9.png)

Como se observa en la imagen anterior, existe un fichero ".txt" dentro de la carpeta compartida. Ahora lo que haremos será usar el comando "get" para poder traernos el fichero a nuestra máquina anfitriona.

![KNOBI10](https://github.com/AntonioPC94/Kenobi/blob/a9c234f800c409ab541d68e9e950e320c16804e8/img/KNOBI10.png)

Una vez descargado, lo abriremos y miraremos qué hay en su interior.

![KNOBI11](https://github.com/AntonioPC94/Kenobi/blob/a9c234f800c409ab541d68e9e950e320c16804e8/img/KNOBI11.png)

Como se observa en la imagen anterior, un usuario ha creado un par de claves RSA para poder autenticarse en el sistema a través del servicio SSH.

Aunque no podamos hacer nada ahora mismo con dicho servicio, tenemos un usuario del sistema objetivo identificado.

Ahora vamos a irnos al servicio RPCBind, el cual se encuentra en el puerto 111. Dicho servicio es interesante porque lo que hace es que cuando un programa cliente necesita comunicarse con un programa servidor a través de RPC, consulta a RPCBind para obtener el número de puerto asociado con el programa servidor en cuestión. De esta manera, RPCBind ayuda a los programas clientes a localizar los servicios RPC en el sistema.

Bien, entonces ahora vamos a lanzar una petición al servidor para obtener información sobre los servicios NFS o directorios que podrían estar disponibles en la máquina objetivo, como listar archivos (nfs-ls), estadísticas de espacio en disco (nfs-statfs) y exportaciones montadas (nfs-showmount).

Para ello, usaremos la siguiente orden:

![KNOBI12](https://github.com/AntonioPC94/Kenobi/blob/a9c234f800c409ab541d68e9e950e320c16804e8/img/KNOBI12.png)

Como se observa en la imagen anterior, el directorio "/var" estaría listo en la máquina objetivo para ser montado en una carpeta de nuestro sistema.

Entonces lo que vamos a hacer ahora, es montar dicha carpeta en otra que crearemos a continuación. Para ello, realizaremos el siguiente proceso:

- Creamos una carpeta en "/mnt" llamada "var".
- Una vez hayamos creado la carpeta, montaremos el directorio compartido del servidor utilizando el siguiente comando: sudo mount -t nfs (DirecciónIPMáquinaObjetivo):/var /mnt/var

Ahora si nos vamos a "/mnt/var" podremos ver el contenido del recurso que nos hemos traído anteriormente.

![KNOBI13](https://github.com/AntonioPC94/Kenobi/blob/a9c234f800c409ab541d68e9e950e320c16804e8/img/KNOBI13.png)

Revisando la ruta "/mnt/var/lib/dhcp/" encontraremos que la máquina objetivo tiene dos tarjetas de red.

![KNOBI14](https://github.com/AntonioPC94/Kenobi/blob/a9c234f800c409ab541d68e9e950e320c16804e8/img/KNOBI14.png)

![KNOBI15](https://github.com/AntonioPC94/Kenobi/blob/a9c234f800c409ab541d68e9e950e320c16804e8/img/KNOBI15.png)

Al tener dos tarjetas, es posible que la máquina objetivo esté en una red con más equipos, así que es posible que más adelante podremos realizar "Pivoting" entre ellos.

Aparte de la información sacada anteriormente, en el directorio "log" encontraremos varios archivos ".log", pero los que nos interesarán a nosotros, serán el "auth.log" y el "syslog".

Revisando el "auth.log" nos encontraremos con que, tanto el usuario "kenobi", como el usuario "root", han cambiado sus respectivas contraseñas.

![KNOBI16](https://github.com/AntonioPC94/Kenobi/blob/a9c234f800c409ab541d68e9e950e320c16804e8/img/KNOBI16.png)

Bien, ahora vamos a cambiar de servicio, concretamente al FTP. A ver qué conseguimos sacar de él.

Lo primero que haremos, será verificar la versión de FTP instalada en la máquina objetivo. Para ello realizaremos una conexión al servicio y, aunque no introduzcamos ninguna credencial, el sistema nos mostrará la versión de dicho servicio.

![KNOBI17](https://github.com/AntonioPC94/Kenobi/blob/a9c234f800c409ab541d68e9e950e320c16804e8/img/KNOBI17.png)

Ahora usaremos "searchsploit" para buscar exploits que puedan servirnos para explotar de alguna manera la versión del servicio indicada anteriormente.

![KNOBI18](https://github.com/AntonioPC94/Kenobi/blob/a9c234f800c409ab541d68e9e950e320c16804e8/img/KNOBI18.png)

Como se observa en la imagen anterior, tenemos 4 exploits disponibles. El módulo "mod_copy" que aparece en la imagen, implementa los comandos "SITE CPFR" y "SITE CPTO", que se pueden usar para copiar archivos/directorios de un lugar a otro en el servidor.

Es decir, cualquier cliente no autenticado puede aprovechar dichos comandos para copiar archivos desde cualquier parte del sistema de archivos a un destino elegido.

Entonces lo que vamos a hacer nosotros, es copiar la clave privada del usuario encontrado para poder acceder al sistema suplantando su identidad.

Para ello, realizaremos los siguientes pasos:

- Usamos "nc" para realizar una conexión al servicio FTP.

![KNOBI19](https://github.com/AntonioPC94/Kenobi/blob/a9c234f800c409ab541d68e9e950e320c16804e8/img/KNOBI19.png)

- Usamos "SITE CPFR" para indicarle al servidor el fichero que queremos copiar.

![KNOBI20](https://github.com/AntonioPC94/Kenobi/blob/a9c234f800c409ab541d68e9e950e320c16804e8/img/KNOBI20.png)

-  Usamos "SITE CPTO" para indicarle al servidor el directorio destino donde queremos copiar el fichero indicado anteriormente. En este caso, lo vamos a copiar en "/var", ya que este directorio lo podemos montar de manera local en nuestra máquina y ver todo su contenido.

![KNOBI21](https://github.com/AntonioPC94/Kenobi/blob/a9c234f800c409ab541d68e9e950e320c16804e8/img/KNOBI21.png)

A continuación desmontaremos y montaremos el directorio "/var" en la carpeta que creamos anteriormente y nos iremos al directorio "/tmp" para obtener la clave que hemos copiado anteriormente.

![KNOBI22](https://github.com/AntonioPC94/Kenobi/blob/a9c234f800c409ab541d68e9e950e320c16804e8/img/KNOBI22.png)

Nota: Copiar la clave en el directorio "/home/kali/Desktop" para futuras conexiones.

Bien, ahora accederemos al sistema objetivo usando dicha clave privada.

![KNOBI23](https://github.com/AntonioPC94/Kenobi/blob/a9c234f800c409ab541d68e9e950e320c16804e8/img/KNOBI23.png)

Ya estaríamos dentro del sistema objetivo.





