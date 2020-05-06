`ISE` > Prácticas > **Práctica 1.** Virtualización e instalación de SO

## Lección 1. Instalación de Ubuntu Server

> NOTA: esta lección está incompleta

###### ¿Desea instalar actualizaciones automáticamente?

* Ventajas e inconvenientes.

* **Configuración:** tener perfectamente definido todo lo que rodea al sistema: propiedades, librerías, usuarios, etc.
* **No**, ya que no es un sistema de uso personal.

### Semantic versioning

vA.B.C

* A. Versión principal: cambio sustancial, incompatibilidad.
* B. Versión secundaria: mejora.
* C. Versión terciaria: parche.

###### ¿Desea instalar cargador de grub?

* Sí, permite usar varios kernels de arranque.
* Instalamos arranque de grub en el primer disco.
  * Aunque sea RAID, grub no reconoce los dos discos --> si se rompe disco A el sistema no arranca, porque grub sólo está instalado en el A. Esto es porque estamos en RAID software.
  * Tenemos dos discos (RAID): Linux los nombra siguiendo una secuencia.
    * `/dev/sda`
    * `/dev/sdb`

### Cómo funciona arranque de ordenador (IBMPC)

(Por qué no ciframos la partición de `boot`)

* Cómo llega a la memoria principal del ordenador las instrucciones de arranque al encender el ordenador.
* En la BIOS hay un código almacenado que realiza el arranque (MBR: Master Boot Record): va al sector 0 del dispositivo configurado (ej. `sda`), tiene código de arranque muy pequeño en x86, no modificado desde los años 70.
  * Este código carga en memoria principal un código que averigua dónde está la tabla de partición primaria del disco y, a continuación, carga el kernel del SO (cargador del SO, empieza a arrancar el SO).
* El kernel lo tenemos en la partición `/boot`, si lo ciframos el código no puede acceder y no puede arrancar.

###### Nos pregunta las contraseñas de la semana pasada

###### En Ubuntu siempre hay que entrar con cuenta de usuario no privilegiado

* Kernel de Linux: código que carga el sistema de arranque para iniciar el SO: `/boot/vmlinuz*`.
* `initrd` contiene información de librerías de las que depende del arranque, que se monta como si fuese un sistema de ficheros real que reside en MP. _Técnica fundamental en sistemas Linux: parece que es un sistema de ficheros pero es otra cosa (ej. dispositivos, acceso a red, elementos de control del kernel)._

### Acceso a internet

La máquina virtual tiene acceso a internet con la siguiente forma de _networking_:

* **Tarjeta de red virtual:** la ha creado VirtualBox. Tenemos una tarjeta de tipo NAT en el adaptador 1. Es la configuración por defecto de VirtualBox.
  * NAT: te deja acceder a Internet de manera directa. Accedemos a través de nuestro host a Internet (red donde está el host).
    * Lo hace implementando un servidor de NAT en el host (NAT: Network Address Translation).
    * Se usa porque en IPv4 hay un número limitado de IP. Solución: rango de IP como IP privadas, el router implementa NAT. NAT mantiene una lista de las peticiones locales, hace las peticiones en el nombre de los de nivel inferior (IP externa) y luego lo traduces a la IP privada, realizando la traducción a través de la tabla NAT.
    * Podemos abrir conexiones hacia internet, pero no en sentido contrario, porque nadie desde internet puede direccionar las IP privadas que subyacen el NAT. Sólo pueden direccionar al router que tenga una IP pública.
    * Muy seguro, nadie puede atacar las MVs.
    * No podemos publicar servicios accesibles desde el exterior dentro de los servidores. Algunos routers permiten hacer operaciones (ej. que todo el tráfico del puerto 80 vaya a una IP, de ese modo publicamos un servicio web). Debemos abrir los puertos explícitamente (_whitelist_).
    * Problema de NAT en VirtualBox: si tenemos dos MVs, cada una de ellas se conecta al exterior, pero no pueden conectarse entre sí. La opción NAT Network de VirtualBox hace que las MVs compartan una red privada y puedan verse, pero no usaremos esa opción.
  * BRIDGE:
    * Al crear una máquina virtual creamos una tarjeta virtual (con una MAC virtual).
    * Al implementar un BRIDGE, la capa de software de acceso al medio (1-3 OSI) está implementado en la tarjeta real del anfitrión: todos los paquetes de la tarjeta de red se ven en la virtual. Esto implica que con la tarjeta BRIDGE tenemos acceso a internet inmediato, tenemos los mismos recursos que el anfitrión (¡es la misma tarjeta!). Del mismo modo, cualquier acceso exterior accede a tu VM, es visible al exterior (problema de seguridad). No ocurre en NAT, ya que hay enrutamiento.
    * Las máquinas en BRIDGE se ven entre ellas.
    * Solución cuando queremos acceso muy eficiente, físico real. Ej. varios servidores de misma tarjeta de red. En Linux podemos asignarle varias IP a la misma tarjeta de red.
  * HOST-ONLY:
    * Creamos una tarjeta virtual a la máquina virtual. En el host creamos una tarjeta virtual distinta de la tarjeta física, y se podrán comunicar entre sí.
    * Si tenemos varias MVs se comunicarán todas ellas entre sí: se ven y se ven con el host. De este modo, las máquinas virtuales se ven entre sí.
    * Tenemos en el host dos tarjetas: ¿cómo pasa la información de virtual a real y viceversa? Hace falta definir un router explícitamente (y lo explícito es seguro).
    * Modo de operadores de Cloud.

###### Configuramos la red de la VM Ubuntu

* Crearemos una tarjeta HOST-ONLY, ya tenemos la de red. VirtualBox ya ha creado una tarjeta virtual.
* Usaremos la HOST-ONLY para hacer prácticas de _networking_.
* Con NAT no podemos llegar a la MV (no nos permite hacer `ping` ni `ssh`).

1. Apagamos la VM para configurar una nueva tarjeta de red.

2. Vemos las tarjetas virtuales que tengamos configuradas. Si no hay una tarjeta de red, la configuramos.

3. Si hacemos en el host `ifconfig -a` vemos la tarjeta de red virtual.

4. Dejamos el adaptador 1, y creamos el adaptador 2, HOST-ONLY con la tarjeta virtual del host.

5. Volvemos a arrancar la máquina.

6. Hacemos `ifconfig` en la VM. Tenemos las interfaces anteriores y la local. Para habilitar las tarjetas de red, hacemos `ifconfig -a` para ver las que no están habilitadas también. Veremos la nueva.

7. Vamos a `/etc/network`, y modificamos el fichero `interfaces`.

   * `auto`: lo levanta el sistema en el arranque de forma automática.
   * No vamos a dar DHCP en HOST-ONLY, asignaremos las IP manualmente para controlar el networking. Esto nos permite saber manualmente las IP de las máquinas virtuales.

   ~~~
   # Nueva tarjeta HostOnly
   auto enp0s8
   iface enp0s8 inet static # asignación IP estática, enp0s8 aquí es nombre simbólico
   address 192.168.56.10 # en el rango de la tarjeta
   netmask 255.255.255.0
   ~~~

8. Si hacemos `ifconfig` en la máquina local, ya veremos que tiene una IP asignada.

9. `ifup enp0s8` (comando de CentOS, funciona en Ubuntu)

   * Forma alternativa: ir al script `/etc/init.d/networking restart`
     * Se desconectaría si estuviésemos trabajando en red.

10. Probamos si tenemos comunicación con máquina anfitriona con `ping`.

11. Probar que la máquina anfitriona tiene comunicación con la invitada.

###### Qué pedirá al final de la lección

* Árbol de particiones, explicar por qué está hecho así y qué significa cada uno.
* Comunicación anterior

###### Ver que está bien particionado

Hacemos `lsblk`.