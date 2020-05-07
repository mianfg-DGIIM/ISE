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

Probaremos `cpio` desde Ubuntu:

* Hacemos una copia de seguridad de los archivos de configuración:

  ~~~
  sudo ls /etc/*.conf | cpio -vo > ~/mianfg/conf_backup.cpio
  ~~~

  * `-v`: verbose
  * `-o`: output, toma una lista de nombres de fichero y los almacena en un archivador

* Simularemos que se rompe el archivo de configuración:

  ~~~
  mv /etc/yum.conf ~/yum.conf
  ~~~

* Descomprimimos los archivos guardados en el directorio original:

  ~~~
  cpio -iudv yum.conf < conf_backup.cpio
  ~~~

  * `-i`: input, lee una lista de nombres de archivos y los recupera del archivador
  * `-u`: unconditional, sobreescribe ficheros existentes
  * `-d`: directories, crea la estructura de directorios

* Vemos que la _backup_ ha funcionado:

  ~~~
  less /etc/yum.conf
  ~~~

* Para hacer `cpio` sobre toda una estructura de directorios, se puede hacer:

  ~~~
  find <base_dir> -t f | cpio -vo > <backup_file>
  ~~~

  * `find` va colocando los path de los archivos, podemos usar el comando ya visto para recuperar todo

### Copias de seguridad (II): `rsync`

Probamos `rsync` desde Ubuntu:

* Creamos una estructura de directorios para probar

  ~~~
  mkdir ise; touch archivo1; mkdir ise/p1 ise/p2; touch ise/p2/pruebas
  ~~~

* Copiamos la estructura en CentOS:

  ~~~
  rsync -a --delete <usuario>@centosise:/home/<usuario>
  ~~~

  * `-a`: mantener los metadatos de los archivos
  * `--delete`: para borrar, en otro caso se acumulan archivos
  * `-z`: comprimir el tráfico
  * `-P`: ver el progreso de la copia
  * Podríamos haber usado `-zaP`

### Control de versiones con `git`

#### Teoría previa

* En cada `git commit`:
  * Se calcula el SHA de todo el proyecto, este será el ID del _commit_.
    * El SHA-1 es de 40 caracteres.
    * Usando los 7 primeros suele ser suficiente.
  * Dentro de `.git` se guardan las diferencias con el directorio anterior.
    * Base de datos de la forma: clave primaria, diferencias, _commit_ padre.
      * Con clave primaria el SHA del proyecto
    * Al hacer un _branch_, generamos un _commit_ que contiene más de dos hijos.
  * Realmente cada rama es un tag que se le pone a un commit.
  * Es útil hacer ramas locales y mergearlas contra `master` en local, para hacer `push` de los cambios solo sobre `master`. El servidor no tendrá constancia de las ramas locales con las que hemos trabajado.
  * El último commit es el `HEAD`.
  * Podemos desplazarnos a partir del identificador (con `~` y `^`):
    * `HEAD~3==HEAD^^^==HEAD^3`
    * Se pueden combinar: `HEAD~3^2`
  * Hay un `log` de cómo nos desplazamos: `git reflog`
  * Los `..` hacen referencia a intervalos lineales de commits, y los `...` a intervalos no lineales.

* Objetos de `git`:
  * `blobs`: archivos (inodos y datos)
  * `trees`: directorios
  * `commits`

### Ubuntu

###### 1. Crear una estructura básica para el repositorio

~~~
mkdir isegim; cd isegim
~~~

###### 2. Iniciar el respositorio: `init`

~~~
git init
~~~

* Inicia un repositorio de _git_, creando un `.git` en el que se gestiona la historia y el contenido (BD de cambios de _git_).
* En `README.md` tenemos un archivo estándar que muestra la información del proyecto.
* Si hacemos `git log` obtenemos el error:
  * fatal: your current branch 'master' does not have any commits yet

###### 3. Ver el estado del proyecto

~~~
git status
~~~

###### 4. Pasar el archivo al `stage`

~~~
git add README.md
~~~

* No se suele hacer `stage`, pues tiene poca utilidad, directamente se pasa al `commit`.
* Si hacemos `git log` vemos que no tenemos cambios (el staging no vale para mucho).

###### 5. Configurar `git`

Podemos hacerlo editando `~/.gitconfig` o con `git config`:

~~~
git config --global user.name "username"
git config --global user.email "email"
~~~

Usando `--local` se guardará en `<repo>/.git/config` en lugar de en `~/.gitconfig`.

Copiamos la clave pública en GitHub.

###### 6. Hacer el commit: `commit`

~~~
git commit
~~~

De este modo guardamos el cambio hecho

###### 7. Añadir el repositorio remoto: `remote`

~~~
git remote add github git@github....
~~~

* Necesitamos copiar nuestra clave SSH en GitHub.
* Al principal se suele poner `origin`.

###### 8. Hacemos _push_: `push`

~~~
git push github --set-upstream github master
~~~

* `--set-upstream` configura la rama _upstream_.
  * Los futuros `git pull` harán _pull_ de la rama `master` (con la rama local hecha _checkout_).
* Nosotros somos la rama `origin`, podemos omitir esto.

#### CentOS

###### 1. Instalar y configurar `git`

~~~
yum install git
~~~

* Copiamos la clave pública en GitHub
* Realizamos la configuración de `user.name` y `user.email`.

###### 2. Clonar repositorio: `clone`

~~~
git clone git@github.com/...
~~~

###### 3. Creamos una rama: `branch`

~~~
git branch mianfg
~~~

* Con `git checkout -b mianfg` hacemos lo mismo, pero además nos movemos a la nueva rama.

#### Tarea

1. En Ubuntu y `master`:
   1. Ponemos nuestro nombre en un fichero `.csv` ubicado en la rama `master`.
   2. Registramos el cambio con `git commit -a`
2. En CentOS, tras haber creado la rama `mianfg`:
   1. Editamos el archivo `.csv`
   2. `git commit -a`: esto realiza `git add .; git commit` (es un _commit_ de todo, `-a`ll)
   3. `git log`: vemos que no se ha hecho el cambio
3. En Ubuntu y en la rama `master`:
   1. `git log`: vemos el cambio en paralelo
   2. `git push --set-upstream github master`: para colocar el _upstream_
4. Vemos los resultados:
   1. Vemos en GitHub que en `master` está todo como al principio.
5. En CentOS y en la rama `mianfg`:
   1. `git push origin mianfg`: hacemos _push_ sobre la rama alternativa
6. Vemos los resultados:
   1. En GitHub vemos que heos creado una nueva rama.
   2. Los cambios están sobre la rama nueva, la `master` sigue intacta con lo hecho desde Ubuntu.
7. En Centos y en la rama `mianfg`:
   1. `git checkout master`:  cambiamos a la rama `master`
   2. `git pull`: tomar los cambios hechos desde Ubuntu
   3. `git merge mianfg`: fusionamos la rama `mianfg` sobre la rama `master`
      1. Nos muestra un conflicto.
      2. Editamos quitando los comentarios del _merge conflict_.
8. Hacemos `git commit`: registramos que hemos resuelto el conflicto.
9. Hacemos `git push`: hacemos efectiva la resolución del cambio.

## Recursos útiles para aprender `git`

* https://www.atlassian.com/git/tutorials/
* http://marklodato.github.io/visual-git-guide/index-en.html
* https://rogerdudler.github.io/git-guide/index.es.html
* https://book.git-scm.com/docs/gittutorial