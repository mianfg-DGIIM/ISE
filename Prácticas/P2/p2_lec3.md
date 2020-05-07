`ISE` > Prácticas > **Práctica 2.** Instalación y configuración de servicios

## Lección 3: Firewall + LAMP stack

## Stack LAMP

Es un framework o stack conformado por:

* Linux
* Apache
* MySQL (o MariaDB, MongoDB)
* PHP o Python

Más sobre las partes:

* **MariaDB** es el _fork_ open source de MySQL
  * Compatible a nivel de API (mismas llamadas)
  * Aurora AWS es otra base de datos compatible con MySQL desarrollada por Amazon
* **Apache HTTP**
  * Es uno de los servidores web más antiguos y más robustos, el más utilizado
  * Servidor alternativo: **Nginx**
    * Se utiliza mucho como proxy inverso (servidor frontend que tiene detrás servidores que son las que toman la carga que el frontend va distribuyendo)
    * Muy potente ejecutando reescrituras (distribuye bien la carga entre otros servidores)
    * Muy bueno accediendo a dispositivos (paralelismo en colas, que no sobrecargan la CPU)
    * Entradas / Salidas asíncronas
    * **Nginx** se utiliza como proxy inverso, línea de entrada a cloud. Potente ejecutando reglas de reescritura. Es muy bueno accediendo a dispositivos (paralelismo basado en colas similar a Javascript: no lanza hebras sino que usa una cola y E/S asíncrona de SO).

Algunas notas:

* **Node.js** no es concurrente, por lo que no es una buena herramienta para ciertas tareas.
* El proceso que sigue el profesor es en CentOS, pero deberemos replicarlo para Ubuntu Server.
* Realizaremos el proceso como usuario _root_: `sudo su`

### Apache Server

###### Instalar apache server

```
yum search httpd
yum install httpd (ó httpd-x86_64)
```

Está cargado pero no activo (`systemctl status httpd`).

Activamos el servicio, crea enlace simbólico para que ejecute servicio cada vez que arranca la máquina: `systemctl enable httpd`.

Ejecutamos `httpd`: `systemctl start httpd`. Engendra varios procesos con varias hebras cada uno (pero no tiene muy claro el profesor cómo va). Se puede controlar cuántos procesos se lanzan, cuántas peticiones puede obtener, límite de peticiones antes de acabar el proceso.

###### Configuración de Apache

```
cd /etc/httpd
```

La configuración está en `httpd.conf`. Pero está modularizado en `conf.d`.

###### Probar que Apache funciona

La forma más bruta sería usar un `telnet`: `telnet localhost 80`. Usamos una herramienta más común.

Usando `curl`, nos devuelve resultado HTML (con `-vv` nos da las llamadas del protocolo y la web). Esto nos dice que apache está funcionando.

```
curl http://192.168.56.20
```

Que nos devuelve el código HTML. Desde dentro si funciona (podemos hacer también `curl` a `localhost`), pero no desde fuera: quiere decir que el firewall está activado.

```
firewall-cmd --state
```

Vemos todas las reglas definidas:

```
firewall-cmd --list-all
```

Podemos definir grupos de reglas del firewall: **zonas de firewall**, podemos verlas con `firewall-cmd --list-all-zones`. La zona por defecto es la pública.

Los distintos servicios están en `/etc/services`.

###### Abrimos puerto 80

```
firewall-cmd --zone=public --add-service=http (o poner puerto)
```

Para que la regla se guarde de forma permanente añadimos la opción `--permanent`. Esto nos lo escribe en el archivo de configuración de firewall.

Tenemos que decirle que recargue para tener estos cambios:

```
firewall-cmd --reload
```

###### Más sobre la configuración

* `cd /etc/http/conf.d`
  * Este directorio tiene distintos archivos de configuración, en contraste al archivo monolítico de configuración que ya hemos visto
  * Así, es posible modularizar la configuración de Apache, para no tener un único archivo monstruoso de configuración
* Dentro de este directorio de configuraciones monolíticas `/etc/httpd/conf`, podemos ver la configuración de Apache
  * `vi httpd.conf`
    * `IncludeOptional conf.d/*`: hace que se modularice la configuración, como ya se ha visto
    * `DirectoryIndex`: muestra el archivo por defecto que carga Apache en la página web
* Para verificar las configuraciones de Apache, antes de causar errores innecesarios: `apachectl configtest` (análogo a `nginx -t`)

###### Logs de error

El log de Apache se encuentra en `/var/log/httpd`:

* `access_log`
  * Archivo muy largo, generan gigas, por lo que no suelen ser persistentes en el tiempo
  * Se tienen media hora, una hora... Depende del tráfico gestionado
  * Se pueden usar para tener estadísticas de usuarios, de uso por países...
* `error_log`
  * Errores de Apache

### PHP

```
yum search php
```

```
yum install php
```

Este es el modo largo: `<?php ... >`.

* `php` lanza el intérprete en línea
* `php -v` muestra la versión de PHP

El archivo de configuración de PHP es:

```
less /etc/php.ini
```

Ver `short_open_tag` que hace las descripciones de errores largos, porque por defecto está `Off`.

Todos los logs están en `/var/log`. Los errores de PHP van a los errores de Apache.

La raíz de documentos (`DOC_ROOT` o `DOCUMENT_ROOT`) está en `/var/www/html`. Aquí ponemos los ficheros de nuestras páginas web. Crearemos una página web (ver al final `index.html`). Por defecto, Apache busca `index.html`.

Creamos un archivo con código PHP, lo llamamos `index.php` (ver al final `index.php`). Colocando `<?php phpinfo(); ?>` no me lo interpreta: tengo PHP instalado pero no activo para Apache.

Hacemos `apachectl restart` (no basta `reload`) porque no teníamos el intérprete de PHP en memoria. `reload` cambia únicamente las configuraciones, mientras que `restart` carga todo de nuevo.

La configuración de PHP en Apache está en `/etc/http/conf.d/php.conf`.

### MariaDB

```
yum install mariadb-server
```

Todos los servicios configurados en CentOS:

```
systemctl list-unit-files
```

Habilitamos y empezamos servicio `mariadb` con `systemctl` (`start`, `enable`).

En producción, para obtener unas configuraciones más fuertes de seguridad, haríamos: `mysql_secure_installation`.

Entramos en la tabla `mysql` de mariadb que contiene metainformación con `mysql mysql`. Si funciona, funciona MariaDB.

```sql
show databases;	// muestra esquemas
show tables; // muestra las tablas, ta vacía
```

Creamos datos mySQL:

```sql
create database ise2020;
	// crear base de datos
grant all privileges on ise2020.* to 'alumno'@'localhost' identified by 'ise2020';
	// crear un usuario y darle todos los privilegios de la tabla
EXIT;
```

Comprobamos que hemos creado correctamente la configuración:

~~~
mysql -u <usuario> -p <clave> <nombre BD>
~~~

En nuestro caso: `mysql -u alumno -p ise2020 ise2020`.

Con `tail -f` se actualiza un archivo (perfecto para ver logs). Veremos el log de Apache para ver cómo nos vamos conectando.

Si ejecutamos el código PHP con

```php
$mysql = new mysqli('localhost', 'usuario', 'secreto','ise2020');
if ($mysql->connect_error) {
	die('No pudo conectarse: ' . $mysql->connected_error);
}
echo '<h1>Conectado satisfactoriamente</h1>';
```

En el log vemos `class 'mysqli' not found`. Vemos que no está `mysql` en los datos de la configuración del entorno PHP. Necesitamos un código intermedio entre el código que actúa como cliente y la BD.

###### Instalar driver MySQL para PHP

```
yum search php | grep -i mysql
```

Instalaremos `php-mysql`. Recargamos apache porque el intérprete PHP está corriendo en el espacio de memoria de Apache. Si corriésemos algo fuera de Apache, accedería a MySQL sin problema.

Queda crear una tabla `alumnos` en la base de datos `ise2020` e insertar datos.



#### Cosas que ha dicho por encima

Ver Hazelcast Vault para almacenamiento de secretos (almacenar contraseñas).

Repetir proceso en Ubuntu.

* Apache se instala y administra como `Apache2`
* Además de instalar php, se debe instalar el módulo `libapache2-mod-php`
* Además, para trabajar con MariaDB, instalamos `php-mysql`
* Para el firewall:
* `ufw allow http`
* `ufw restart`



### Archivos asociados a la práctica

###### `index.html`

~~~html
<html>
    <head>
        <title>Página de prueba</title>
    </head>
    <body>
        <h1>¡Hola, esto es una página de prueba!</h1>
    </body>
</html>
~~~

###### `index.php`

~~~php+HTML
<html>
    <head>
        <title>Página de prueba</title>
    </head>
    <body>
        <h1>Página de prueba</h1>
        <h2>Conexión MySQL</h2>
        <!-- Esto es para cuando hayamos configurado MySQL y hayamos creado un usuario en la BD -->
        <?php
			$mysql = new mysqli('localhost', 'alumno', 'ise2020', 'ise2020');
			if ( $mysql->connect_error ) {
				die('No pudo conectarse: ' . $mysql->connected_error);
			}
			echo '<p>Conectado satisfactoriamente</p>';
		?>
        <h2>Configuración PHP</h2>
        <?php
			phpinfo();
		?>
    </body>
</html>
~~~

