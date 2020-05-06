`ISE` > Prácticas > **Práctica 2.** Instalación y configuración de servicios

# Lección 2. Sistemas de backup y `git`

## Explicación teórica

### Copias de seguridad

#### `dd`: disk dump / disk destroyer

* `dd` permites copias _raw_ (a nivel de secuencias de bytes, nos saltamos completamente el sistema de ficheros) de archivos y dispositivos (en Linux ambos se gestionan igual).

* Es una utilidad muy antigua, ampliamente usada en la administración de sistemas.

* Borrar un archivo con `rm` es simplemente borrar una entrada en la tabla de inodos.

* `copy` baja al device a través del sistema de ficheros de archivos.

* `dd` se salta la parte del sistema de ficheros. Se usa para generar archivos artificiales.

* Un ejemplo:

  ~~~
  dd if=/dev/zeros of=~/pruebas.dat bs=1024MB count=2
  ~~~

  * `if`: indica el archivo de entrada
  * `of`: indica el archivo de salida
  * `bs`: indica el tamaño de bloque
  * `count`: indica el número de bloques de tamaño `bs` que se copian
  * Se puede poner `status=progress` para ver de forma interactiva cómo avanza el comando

* Ejemplo de a otro dispositivo:

  ~~~
  dd if=/dev/sda of=/deb/sdb
  ~~~

* Otro ejemplo:

  * Crear una imagen de `sda`:

    ~~~
    dd if=/dev/sda of=~/hdadisk.img
    ~~~

  * Recuperar imagen:

    ~~~
    dd if=hdadisk.img of=/dev/sdb
    ~~~

  * Se pueden especificar particiones: `sda1`, `sda2`

* Mucho cuidado con este comando: la escritura en _raw_ puede destruir todo un disco, una partición...

* Se suele usar bastante para hacer un _live USB_ sin necesitar una GUI.

#### `cpio`

* Originalmente creada para gestionar copias en cintas magnéticas, hoy en día es una herramienta empleada para empaquetar elementos del sistema de ficheros como archivos individuales o estructuras completas de directorios y subdirectorios. En este sentido es similar a `tar`.
* Es una utilidad para hacer backups en archivadores.
* Un archivador es un fichero que contiene varios ficheros a su vez.
* Crea un archivo único que contiene toda la información necesaria para recuperar total o parcialmente los archivos y carpetas almacenados.
* Así se emplea, por ejemplo, para crear los sistemas de ficheros temporales que emplea el kernel de Linux durante su arranque (`/boot/initrd.img-<version>`) para acceder a los recursos que necesita (módulos, utilidades, etc.).
* Funcionamiento:
  * Copia un archivo tras otro.
  * Pone marcas en medio para distinguirlos.
  * No usa, por defecto, compresión.
* Lo que buscamos es hacer un backup de varios archivos que se guarda en un solo fichero. Análogo a comprimirlos en un `.zip` pero con diferencias:
  * No estamos comprimiendo los archivos (aunque se puede indicar).
  * Podemos darle información a `cpio` para que al extraer los archivos, recree la estructura de directorios original.

* En CentOS, por ejemplo, `/boot/initrd-plymouth.img` está en formato `cpio`.

* Es muy antiguo, pero ciertas partes del sistema (como ya se ha comentado) lo siguen usando.

#### `tar`

Al igual que `cpio`, `tar` fue diseñado originalmente para gestionar copias en cinta magnética. Hoy en día, se usa junto con `gzip` como el formato más extendido para copiar y restaurar estructuras de directorios en Linux.

#### `rsync`

* `rsync` es una herramienta ampliamente utilizada para realizar copias de seguridad y mantener copias exactas de estructuras de ficheros (sincronizadas).
  * Nos centraremos en la segunda funcionalidad.
* Es un protocolo para sincronizar directorios, aunque estén en dos computadores distintos. Sincroniza dos: una fuente y un destino (incluso a través de SSH).
  * Copia enlaces, dispositivos, propietarios, grupos y permisos.
  * Permite excluir archivos.
  * Transfiere los bloques modificados de un archivo.
  * Puede agrupar todos los cambios de todos los archivos en un único archivo.
  * Puede borrar archivos.
* Como protocolo de comunicaciones, se basa en `ssh` para reducir las debilidades de seguridad a un punto.

* No se puede hacer `rsync` sobre dos máquinas remotas (ssh en una máquina y lanzamos `rsync`).
* Las sincronizaciones no se pueden programar temporalmente directamente usando `rsync`
  * Se recurren a herramientas externas como `crontab`.

### Copias de seguridad vs _backup_

COMPLETAR

### Sistemas de control de versiones

* SCM (_Software Change Management_) = VCS (_Version Control System_)
  
  * El primero es el término oficial
  
* Funciones:
  * Almacenamiento centralizado
  * Origen de verdad
  * Varios desarrolladores a la vez
  * Copias de seguridad de desarrolladores
  * Copias de seguridad del proyecto
  * Dispone de _changelog_ o registro de cambios de versiones
  * Manejan las diferencias sobre archivos de texto, no sobre binarios
    * Existen SCM para binarios
  
* `git`
  * Parte de un directorio que es la raíz del proyecto: `PROJECT_ROOT`.
  * Todas las carpetas que cuelgan de éste se guarda como unidad de versiones. Cualquier cambio debajo de ese punto se considera un cambio en el proyecto, y se genera una nueva versión completa.
  * Los cambios son a nivel de proyecto global, no de archivo.
  * Cada cambio en `git` se considera un _commit_:
    * Cada _commit_ es una fotografía entera del sistema y se almacena. Para ser eficientes en almacenamiento, se guardan solo las diferencias respecto a la versión anterior.
      * Esto considerando los tres niveles: _workdir_, _stage_ y repositorio.
    * Además se documenta el cambio
      * Automático: quién, cuándo
      * Para qué: descripción del desarrollador
    * Se puede hacer `blame` a nivel de fichero o de línea para ver quién hizo el cambio y en qué _commit_.
    * Es buena práctica tener ciertas reglas y directrices a la hora de hacer _commits_: ***meaningful commits***.
  * Permite el desarrollo en ramas.
    * Los equipos de desarrollo trabajan en paralelo sobre la misma base de código.
    * Se pueden hacer bifurcaciones de las versiones: ramas o _branches_.
    * Se suele tener una _branch_ `main`.
      * A parte del nombre, nada diferencia la rama `main` del resto de las ramas.
    * Da problemas a la hora de hacer `merge` porque se generan conflictos.
      * Este coste de _merge_ puede no merecer la pena. Se dan tres aproximaciones:
        * **Feature branches**
          * Equipos independientes en ramas diferentes.
          * Una rama para cada funcionalidad nueva.
          * No puede durar más de tres días una rama sin hacer _merge_.
        * **Git workflow**
        * **Trunk based development**
          * Solo una rama.
          * Es fundamental que los programadores estén todo el rato resubiendo los cambios (los cambios no deben romper el código, porque hace que otros no puedan seguir trabajando).
      * Por este cose nace la integración continua y _continuous delivery_.
    * El _branching_ es fundamental en todas las herramientas de desarrollo.
  
  * _subversion_ gestiona las versiones a nivel de directorio, lo que introduce muchos problemas.
  
* Tipos de servidores

  * Centralizados
    * La copia del software siempre está en un servidor compartido.
    * Favorece la integración continua.
    * No se puede trabajar sin estar conectado al servidor (falta de Internet).
    * El _commit_ automáticamente sube el cambio al servidor.
    * Se tienen que bloquear los archivos de directorios antes de trabajar con ellos.
  * Distribuidos (más modernos): `bazaar`, `mercurial`, `git`
    * No existe un servidor central.
    * Clono en el servidor local --> puedo seguir trabajando con todas las funcionalidades.
    * No bloqueamos a otros desarrolladores.
    * Permite _forks_: hacer una copia y seguir desarrollando otro proyecto partiendo de la base, ya no de código, sino de historia del proyecto.
    * El _open source_ se beneficia de esto.
    * Desde un repositorio se pueden registrar cambios contra el repo 'original'.

## Desarrollo práctico

### Copias de seguridad (I): `cpio`



### Copias de seguridad (II): `rsync`



### Control de versiones con `git`

