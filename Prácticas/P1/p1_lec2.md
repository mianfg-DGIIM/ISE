`ISE` > Prácticas > **Práctica 1.** Virtualización e instalación de SO

## Lección 2: CentOS + particionado automático + LVM

1. Supongamos que se nos llena `/var`: añadimos discos y asignamos a `/var` (no hemos definido volumen lógico para `var`, partición automática).
2. Si se llena de nuevo, tenemos la ventaja de que ya está en un LVM y podemos hacer crecer el volumen lógico asignado a `/var`.

---

###### Creamos VM

* Todas las opciones por defecto.
* Sólo le damos un disco.
* Montamos CD-ROM.
* Arrancamos máquina y comenzamos instalación.

###### Por defecto la configuración de CentOS es gráfica. Forzamos a que arranque en modo texto en el arranque (aunque el profesor usa la versión gráfica)

###### Destino de la instalación: dejamos particionado automático

###### Creamos contraseña de root y usuario

Seleccionar "hacer que este usuario sea administrador" para poder hacer `sudo`.

###### Asignamos adaptador 2 HOST-ONLY en VirtualBox

Nos interesa tener ambas VMs en la misma red, asignamos el mismo adaptador.

###### Iniciamos la máquina y entramos como root

En CentOS podemos iniciar sesión como root directamente.

###### Hacemos `ip link show`

No tenemos acceso a internet. Por defecto, CentOS no configura el arranque automático de la red. Debemos configurarlo.

###### Configurar arranque automático de la red

Vamos a `/etc/sysconfig/network-scripts`. La configuración de la tarjeta `nat` está en `ifcfg-enp0s3` y modificamos el parámetro `ONBOOT=yes`.

Levantamos usando `ifup enp0s3`.

###### Vemos las direcciones con `ip addr`

Claramente, ha cogido la IP.

###### Configuramos HOST-ONLY

`cp ifcfg-enp0s3 ifcfg-enp0s8`

Dejamos `TYPE`, `NAME` y `DEVICE` (y los modificamos) y `ONBOOT`, añadimos antes de `ONBOOT`:

~~~
IPADDR = 192.168.56.20
MASK = 255.255.255.0
~~~

Hacemos `ifup enp0s8`.

Ahora podremos hacer _ping_ entre host y VM.

###### Vemos cómo está particionado el disco

Tenemos un disco duro y tenemos una partición únicamente para el `boot`. Como `boot` no va a crecer, no hace falta meterlo en la VM. Del mismo modo con `swap` (no cambia el tamaño de MP).

### Gestión de LVM en CentOS

Los volúmenes físicos (_physical volume_) tienen comandos comenzando por `pv`.

Con `pvdisplay` vemos la info del volumen físico.

En LVM también tenemos (_group volume_), `vg`, haciendo `vgdisplay` vemos la info del grupo de volúmenes. El espacio libre es 0, porque tenemos todo el espacio asignado a volúmenes lógicos.

Con `lvdisplay` tenemos información de volúmenes lógicos: `swap` y `root`.

LVM asigna nombres de la forma `/dev/[grupo de volumen]/[volumen lógico]`, y también de la forma `/dev/mapper/cl-*` que son enlaces simbólicos a los dispositivos.

> Supongamos que se nos ha llenado el `var`. En estos momentos, necesitamos meter un disco duro nuevo.

Podemos llenar el disco usando dispositivos que retornan valores de Linux: `zero`, `random`...

~~~
dd if=/dev/zero of=/var/log/zeros.raw bs=1024k count=1024
~~~

Copias de 1024 kbytes con un _count_ de 1024 (número de veces que repite la lectura).

~~~
df -h # nos da info de discos
~~~

###### Ver qué directorio está usando tu espacio en disco

~~~
du -sk *
~~~

###### Añadimos un nuevo disco en VirtualBox

Le damos 8GB, arrancamos.

###### Vemos discos en equipo

Haciendo `lsblk` vemos un nuevo disco.

Podemos crear un nuevo disco, pero nos crearía una partición para todo el disco (usando `pvcreate`).

Usamos `fdisk` para crear una partición:

1. `n` para crear.
2. `p` para partición primaria.
3. Por defecto sector más bajo.
4. Final sector más alto.
5. Imprimimos tabla de particiones con `p`, ocupa todo el disco.
6. `w` para escribir.

Con `lsblk` vemos que tenemos una partición. Hacemos `pvcreate /dev/sdb1` (el nuevo). Esto simplemente "etiqueta", dice que va a ser usado por LVM. Haciendo `pvdisplay` vemos que tenemos dos volúmenes físicos.

###### Añadimos volumen físico a grupo de volúmenes, para que lo podamos utilizar

~~~
vgextend cl /dev/sdb1  # cl es el nombre del grupo de volúmenes
~~~

Con `vgdisplay` vemos que tenemos 8GB libres.

###### Creamos un volumen lógico para llevarnos `/var`

~~~
lvcreate -L (ó --size) 4G -n nvar cl  # nuevo volumen lógico de 4GB y lo cogemos de cl
~~~

Haciendo `lvdisplay` vemos que tenemos un volumen lógico montado sobre `nvar`.

Ahora trasladamos la información de `var` a `nvar`. Tenemos que definir un sistema de ficheros antes de esto.

###### Creamos sistema de ficheros

~~~~
mkfs.ext4 /dev/cl/nvar
~~~~

###### Montamos el volumen lógico

Lo montamos en `/mnt/`

~~~
cd /mnt/
mkdir nvar
mount /dev/cl/nvar /mnt/nvar
~~~

Hemos montado un sistema de ficheros.

Conmutamos a modo de mantenimiento para que otros usuarios no modifiquen ni toquen `var`.

~~~
init 1
~~~

###### Movemos `var`

En modo mantenimiento, copiamos `/var` en `nvar`.

~~~
cp -a (mantener todos los derechos, copia íntegramente) /var/* /mnt/nvar/
~~~

###### Renombramos `var`

~~~
mv var old_var
umount /mnt/nvar
cd /etc
vi fstab
~~~

Añadimos:

~~~
/dev/cl/nvar	/var	ext4	defaults	0	0
~~~

Creamos el directorio `/var`:

~~~
mkdir /var
~~~

Con `mount -a` monta todo. Si vamos a `var` está todo.

###### Reiniciamos para comprobar

Si arranca está bien. Haciendo `lsblk` vemos el nuevo `var`.
