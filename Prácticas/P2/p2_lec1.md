`ISE` > Prácticas > **Práctica 2.** Instalación y configuración de servicios

# Lección 1. SSH + Firewall

## SSH

* **SSH** es un reemplazamiento de Telnet. Ambos son protocolos de aplicación de shell remotas (nos podemos conectar por red con un ordenador en cualquier ubicación, mostrándose un _shell prompt_ para poder ejecutar comandos). La principal diferencia es:
  * TELNET realiza la comunicación en abierto, sin cifrar. --> Es inseguro, y no se utiliza.
  * SSH se conecta sobre TLS, cifrando toda conexión TCP/IP. De este modo, toda la comunicación es cifrada. Con todo nos referimos al _handshaking_ (donde proporcionamos las credenciales de conexión), así como a todo el trabajo que hagamos con la máquina remota (permite confidencialidad de todas las operaciones y comandos que hagamos).
    * Esto se consigue mediante el **cifrado** de la comunicación.
* Tanto SSH como TELNET son protocolos de _shell_ remota: nos conectamos a la shell de un ordenador por red.

> Nota: mirar `fail2ban`, entra en el examen pero no lo ha explicado.

### Cifrado simétrico y asimétrico

* **Cifrado simétrico:** misma llave para cifrar y descifrar. Ej. de protocolos: DES, TDES (Triple-DES, es el estándar hoy en día).
  * Si queremos compartir información entre varios, podemos tener:
    * una sola llave: al tener cuatro copias puede que se robe una, comprometiendo la red;
    * llaves entre pares (todas las posibles combinaciones): más seguro, pero la gestión es más difícil.
* **Cifrado asimétrico:** tenemos dos llaves, una pública y otra privada.
  * Tenemos 2 llaves, cualquiera de ellas podría ser o la pública o la privada. Si ciframos con una de ella, desciframos con la otra, y viceversa.
  * Tomamos una llave como privada y otra como pública. La privada es privada y la pública es pública (_he, he_).
  * ¿Cómo nos envían un mensaje para que sólo podamos leerlo nosotros? Lo ciframos con la clave pública, sólo será descifrable con la clave privada.
  * Seguridad: si un usuario ve comprometida su llave privada, sólo su seguridad se ve comprometida, no la del resto.
  * Algoritmo más conocido: RSA.
  * **Enviar un mensaje a A:** encriptamos el mensaje con la pública de A, sólo A podrá descrifrarlo con su clave privada.
  * **Autenticar que un archivo es de A:** A cifra su documento con la llave privada, el resto de ordenadores pueden comprobar la autenticidad descifrando con la llave pública de A.
  * **A quiere identificar a B:**
    1. A le envía un mensaje que quiere B con su llave privada.
    2. A cifra el mensaje con su llave privada y lo envía a B.
    3. B descrifra con la clave pública de B y autentica a B
  * Otra aplicación: **firma digital**.
    * Si ciframos un archivo con la llave privada, éste sólo puede abrirse con la pública, pues solo puede haberlo escrito el que tenga la llave privada.
    * En el caso de los servidores, si el servidor tiene llave y el usuario no la tiene y quiere comunicarse:
      * CLI a SERV: cíframe X
      * SERV a CLI: cifrado
      * CLI a SERV: desciframos con clave pública del servidor
      * Vemos si coincide con lo que CLI ha enviado a SERV.
  * Computacionalmente costoso. Por eso normalmente la comunicación se hace intercambiando una contraseña compartida con cifrado asimétrico, y realizando la comunicación con una clave pública.

### Firma digital vs. cifrado digital

* Funcionan con **algoritmos hash**. El más conocido es el Standard Hash Algorithm, SHA.

  * Se usan para cifrar contraseñas. El protocolo **BCRYPT** funciona como sigue:

    1. Cogemos la contraseña.

       ```
       PASSWORD
       ```

    2. Añadimos un prefijo conocido.

       ```
       PASSWORDhola
       ```

    3. Hacemos hash.

    4. Almacenamos el hash junto con el prefijo (esto es lo que se almacena en un fichero `passwd` de Linux).

    5. Podemos comprobarlo volviendo a calcular el hash, y viendo si coincide el prefijo.

* **Firma digital**. El más extendido es DSA.

  1. Tomamos la información que queremos firmar, `I`.
  2. Calculamos el hash con un algoritmo como SHA: `H(I)`.
  3. Ciframos con nuestra llave privada, `P-`: `P-(H(I))`. Esto hace que no pueda modificarse, pues la modificación del documento implicaría un hash distinto.
  4. Para verificar si el documento es el firmado, desciframos con la llave pública, `P+`, y debemos de obtener el mismo hash que el hash del documento que queremos verificar.

* **Certificados digitales.** Para evitar el _sniffing_, dice que una clave pública es de una persona determinada. Esto permite que no se falseen los datos, ¿quién dice que lo que se ha enviado es una persona? Para esto es necesario una _relación de confianza_, una tercera persona: las **autoridades de certificación**, que hacen los certificados digitales.

  * Consiste en:
    * Llave pública (puede tener además una serie de datos).
    * Firmada electrónicamente (con su hash) y cifrada por una entidad de confianza (autoridad certificadora), como la Fábrica de Moneda y Timbre.

## Instalación

### Prerequisitos

* Ubuntu y CentOS con instalación estándar, red NAT y Host-Only.

### Ubuntu

##### 1. Instalamos SSH

* Vemos si ya está instalado:

  ```
  systemctl status sshd
  ```

  Nos da el estado de un servicio que está en funcionamiento.

* También podemos preguntar:

  ```
  systemctl list-unit-files
  ```

  Nos da todos los servicios configurados, los lanzados y los deshabilitados. Podemos ver si está apagado haciendo:

  ```
  systemctl list-unit-files | ssh
  ```

* Instalación de SSH

  * con `tasksel`, una utilidad para configurar cosas del SO.

    ```
    tasksel
    ```

  * con línea de comandos, usando comandos de Debian:

    * Actualizar información de bases de datos de paquetes:

      ```
      apt update
      ```

      Baja las descripciones de los repositorios de Ubuntu.

    * Hacemos una búsqueda:

      ```
      apt-cache search sshd
      ```

      Usaremos `openssh-server`. Es una implementación abierta del estándar SSH.

    * Instalamos:

      ```
      apt-get install openssh-server
      ```

      La propia instalación configura SSH como si fuera un servicio, en algunos casos habrá que hacerlo a mano (lección 3).

    * Vemos si funciona:

      ```
      systemctl status sshd
      ```

##### 2. Nos conectamos desde el _host_ a Ubuntu

* Antes comprobamos que tenemos conectividad de red:

  ```
  $> ping 192.168.56.1
  %> ping 192.168.56.10
  ```

* Nos conectamos con SSH a nuestra máquina virtual.

  ```
  %> ssh <username>@<IP o dominio, se resuelve DNS>
   en este caso:
  %> ssh <username>@192.168.56.10
  ```

  * Si nos lanza un warning (_remote host identification has changed_), esto es porque el servidor ha devuelto su llave pública en la conexión, y teníamos una llave pública configurada ya con anterioridad (SSH las va almacenando para poder darnos cuenta de cuando un servidor cambie su llave pública, algo que sería cuanto menos sospechoso).

  * Podemos ver las claves almacenadas en `~/.ssh/known_hosts`. Su contenido en cada línea es del tipo:

    ```
    <servidor (IP, URL o nombre simbólico)> <clave>
    ```

  * Básicamente, buscamos la IP y borramos esa línea.

* Nos dice que la autenticidad del host no puede ser garantizada, nos lo creemos (en un entorno profesional sería un certificado digital). Decimos `yes`, y se almacenará la llave en nuestro equipo.

* Ya estamos conectados con SSH. La contraseña se ha enviado cifrado con su llave pública (la del servidor), evitando el _man-in-the-middle_.

* La comunicación se hará con una llave simétrica, que proporcionaría el cliente cifrada con la llave pública del servidor.

> NOTA: los comandos especificados con `%>` son los insertados en el SSH desde la consola del cliente (realmente es la terminal del servidor).

##### 3. Configuración del servidor (Ubuntu Server): cambio de puerto de SSH

Tenemos dos ficheros importantes en `/etc/ssh`:

* `sshd_config` se usa para la configuración del daemon SSH.
* `ssh_config` se usa para la configuración del cliente.

Realizaremos un cambio de puerto en SSH, modificando `sshd_config`. Cambiamos el puerto de `22` a `2232`. Reiniciamos el servidor para coger los cambios. De este modo matamos el servidor y arrancamos con la nueva configuración.

```
systemctl restart sshd
```

Intentamos `reload` en lugar de `restart` para ver si SSH es lo suficientemente inteligente como para no abortar la conexión SSH que ya hemos iniciado.

Con `systemctl status sshd` vemos que ya está arrancado en el nuevo puerto.

En otra consola deberíamos hacer

```
ssh <user>@<IP> -p <nuevo_puerto>
```

Esto no se usa para garantizar seguridad, un scan de puertos vería rápidamente que SSH está ahí. Esto suele hacerse para resolver conflictos entre puertos.

Volvemos al puerto `22`.

### Contenido de la carpeta `.ssh` (cliente)

* `~/.ssh/id_rsa`: tu clave privada.
* `~/.ssh/id_rsa.pub`: tu clave pública.
* `~/.ssh/authorized_keys`: contiene una lista de las claves públicas autorizadas para servidores. Cuando el cliente se conecta al servidor, este autentica al cliente comprobando la firma de su clave pública guardada en este archivo.
* `~/.ssh/known_hosts`: contiene claves DSA públicas de los servidores SSH accedidos por el usuario. Este archivo es muy importante para asegurar que el cliente SSH está conectando con el servidor SSH correcto.

##### 4. Utilidad SSH: lanzar comandos en servidores remotos

Podemos insertar en SSH comandos directamente haciendo:

```
ssh <user>@<IP> "<comando>"
```

##### Algunas observaciones

* `ssh` juega un papel fundamental en la automatización de sistemas (ej. Ansible)
* Normalmente, reducimos todo a un servicio `ssh` junto al servicio normal, para evitar puntos de ataque.

### CentOS

##### 1. Instalamos SSH

En CentOS viene instalado por defecto, lo comprobamos con:

```
$> systemctl status sshd
```

Comprobamos la conectividad con `ping` entre host>MV y MV>host.

##### 2. Nos conectamos desde el host por SSH

Nos podemos conectar con la cuenta de `root`. Normalmente esto no nos dejará, solemos tener un usuario que pueda convertirse en root.

```
%> ssh root@192.168.56.20
```

##### 3. Configuramos `sshd_config`

No permitiremos el login de root, con la opción:

```
PermitRootLogin no
```

Recargamos el servicio `sshd`:

```
$> systemctl reload sshd
```

Si intentamos iniciar sesión con root en el host, no nos dejará:

```
%> ssh root@192.168.56.20
```

Deberíamos entrar con un usuario no privilegiado.

##### 4. Cambiamos el hostname de la máquina

Cambiamos el fichero `/etc/hostname`, cambiamos la primera y única línea por el nombre que queramos.

##### 5. Creamos una clave SSH en Ubuntu

* `ssh-keygen`: crea la clave en `~/.ssh/id_rsa` y en `~/.ssh/id_rsa.pub`.
  * No ponemos passphrase por comodidad, en producción sí lo usaríamos.
  * Con `-c` podemos poner comentarios en la clave.
* Si perdemos la clave pública, la podemos recuperar en la privada.

##### 6. Copiamos la clave pública de Ubuntu en CentOS

```
ssh-copy-id <usuario>@<ip (CentOS)>
```

* Tenemos que introducir una clave para que CentOS nos autentifique
* Este comando hace el proceso automáticamente, pero podríamos hacerlo manualmente copiando la clave pública y colocándola en `~/.ssh/authorized_keys`.
* En CentOS, comprobamos la clave haciendo `less ~/.shh/authorized_keys`.
* De este modo podremos conectarnos sin necesidad de insertar contraseña.

##### 7. Vemos los mensajes que se transfieren en el protocolo SSH

~~~
ssh <usuario>@<ip> -vv
~~~

##### 8. Añadir hostname

De este modo, podremos hacer `ssh <usuario>@<hostname>` en lugar de `ssh <usuario>@<ip>`.

* A nivel de sistema:

  * Los hostnames se añaden editando `/etc/hosts` de la forma `ipaddr hostname`.
  * Editamos el archivo y ya podremos conectarnos.

* A nivel de usuario:

  * Editamos `~/.ssh/config` del siguiente modo:

    ~~~
    Host UbuntuServer
    	Hostname 192.168.56.101
    	user mianfg
    ~~~

  * Podemos especificar muchos más parámetros de este modo.

##### 9. Copiamos la clave de CentOS en Ubuntu a mano

* Copaimos la clave de CentOS del clipboard de nuestro sistema.
* Editamos en Ubuntu `~/.ssh/authorized_keys` y pegamos lo que tenemos en el clipboard del sistema.

##### 10. Usamos `ssh-agent` para trabajar con las claves de nuestro host

Descripción del problema:

* Imaginemos que necesitamos usar, desde nuestro servidor, un servicio configurado con las claves de nuestro host.
* Puede ser el caso de un repositorio `git`.
* Podemos configurar el servicio para que use las claves del servidor, pero esto puede hacer que los administradores del servidor nos roben la identidad en el servicio.
* Podemos copiar las claves del host en el servidor, pero esto puede hacer que los administradores del servidor nos roben la identidad en cualquier servicio.

Solución: usar `ssh-agent` que copia, en la sesión, las claves del host en el servidor.

Desde nuestro host:

* Iniciamos con `eval $(ssh-agent)`.
* `ssh-add .ssh/id_rsa`: copia en la sesión la clave privada.
* `ssh-add -l`: para mostrar las claves privadas que tenemos almacenadas en `ssh-agent`.
* `ssh -A usuario@ubuntuserver`: nos conectamos a Ubuntu con las claves de `ssh-agent`.

Dentro de Ubuntu:

* `ssh-add -l`: vemos que la clave ha llegado al servidor Ubuntu.
* Si editamos en nuestro host `~/.ssh/config` y añadimos `ForwardAgent yes` no tenemos que hacer `ssh -A ...`