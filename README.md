# Máquina: Kenobi

**Tryhackme: Kenobi**

Lo primero que haremos, será lanzar un NMAP para ver qué puertos tiene abiertos la máquina:

![KNOBI1]()
![KNOBI2]()
![KNOBI3]()
![KNOBI4]()

Como se observa en las imágenes anteriores, existen varios servicios que nos pueden interesar:

- Puerto 21 (FTP)
- Puerto 22 (SSH)
- Puerto 80 (HTTP)
- Puerto 111 (RPC)
- Puerto 139 (SMB)
- Puerto 445 (SMB)

A continuación, usaremos el módulo de escaneo de vulnerabilidades de NMAP para ver si este nos devuelve algo interesante sobre los puertos anteriormente encontrados:

![KNOBI5]()

En el "http-enum", veremos que existen 2 directorios que pueden ser de interés:

/admin.html: Posible directorio "admin"
/robots.txt

Nota: Con Dirbuster no encontraremos directorios más allá del "robots.txt".

En el "robots.txt", nos encontraremos como "disallow" el directorio que mencionamos anteriormente.

![KNOBI6]()

Nosotros aunque ponga "disallow", vamos a intentar acceder a dicho directorio para ver si nos encontramos con algo de interés.

![KNOBI7]()

Observando la imagen anterior, podremos deducir que no estamos yendo por el camino correcto para explotar la máquina objetivo.

Bien, si no podemos hacer mucho con el puerto 80, vamos a intentarlo con el 445, a ver si podemos sacar algo de él.

Lo primero que haremos, será listar las carpetas compartidas en el servicio para ver si hay alguna que nos llame la atención.

![KNOBI8]()

Como se observa en la imagen anterior, existe un directorio llamado "anonymous" que posiblemente contenga información que nos interese.

Para conectarnos, lo haremos como usuario "anonymous" y si hay suerte, conseguiremos ver el contenido de dicha carpeta.

![KNOBI9]()

Como se observa en la imagen anterior, existe un fichero ".txt" dentro de la carpeta compartida. Ahora lo que haremos será usar el comando "get" para poder traernos el fichero a nuestra máquina anfitriona.

![KNOBI10]()

Una vez descargado, lo abriremos y miraremos qué hay en su interior.

![KNOBI11]()
