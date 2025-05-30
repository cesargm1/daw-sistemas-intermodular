---
---

![Linux](/images/picture.jpg)

# Guía instalación debian

Cesar Garcia

---

# Introduccion

## Introduccion

- Lista 1
- Lista 2
- Lista 3

**Esto es un texto** _para probar_ Markdown

---

# Desarrollo

---

# Desenlace

## Instalacion de servicio ssh

**Instalamos ssh con el siguente comando**

```bash
apt install openssh-server
```

---

Vamos a comprobar el estado del servidor con el siguente comando

```bash
systemctl status ssh
```

![ssh status](images/ssh/ssh-status.png)

---

### renvio puertos ssh

- nombre ssh

![renvio puertos ssh](images/ssh/renvio-puertos-ssh.png)

### Prueba de conexión SSH anfitrión-invitado

Usamos el siguiente comando para conectarnos a ssh remotamente desde mi windows a debian

```bash
ssh -p 10022 cesar@localhost
```

> Nota: Es recomendable que para conectarse a ssh utilizar una clave ssh para conectarse en vez de autenticarse con usuario y contraseña

![ssh conexion](/markdown/images/ssh/ssh-conexion.png)

## Instalación apache2

Instalaremos apache con el siguente comando

```bash
apt install apache2
```

### Comprobar status de apache

```bash
systemctl status apache2
```

![apache status](/markdown/images/apache/apache-status.png)

### renvio puertos apache2

![apache renvio puertos](/markdown/images/apache/renvio-puertos-apache.png)

## Accediendo a el servidor apache desde mi maquina en el puerto 8080

![apache default page](/markdown/images/apache/apache-default-page.png)

### Configuracion de apache2

1.  Creacion del sitio de prueba

```bash
mkdir -p /var/www/proyecto/public_html
```

2. Asignacion de permisos al usuario sobre la carpeta

```bash
chown -R $USER:$USER /var/www/proyecto
```

3.  Creando un html en nuestro sitio

```bash
vim /var/www/proyecto/public_html/index.html
```

4. Copia el archivo del Virtual Host por defecto:

```bash
cd /etc/apache2/sites-available/
cp 000-default.conf proyecto.conf
```

5. Editar archivo de virtual Host

```bash
vim /etc/apache2/sites-available/proyecto.conf
```

6. Habilitar el nuevo sitio y reiniciar Apache

```bash
/sbin/a2ensite proyecto.conf
systemctl reload apache2
```

> Nota: En el punto 6 tuve problemas ya que me seguia apareciendo la pagina de apache por defecto esto ocurre porque apache estaba cogiendo la configuracion por defecto del fichero /etc/ apache2/sites-available/proyecto.conf asi que tuve que desactivarlo con el siguiente comando

```bash
/sbin/a2dissite 000-default.conf
```

Luego reiniciar apache

```bash
systemctl reload apache2
```

En nuestro puerto 8080 veremos nuestra pagina

![apache test pahe](/markdown/images/apache/apache-test-page.png)

### Conexión segura con HTTPS

#### Crear un Virtual Host HTTPS

1. instalando el apaquete de opessl para certificados

```bash
apt update && apt install openssl
```

2. Generar la clave privada y el certificado

```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
-keyout /etc/ssl/private/proyecto.key \
-out /etc/ssl/certs/proyecto.crt
```

Nos dara la siguente salida y no hara unas preguntas

![Certificado ssl](/markdown/images/apache/https/certificado-clave-privada-publica.png)

3. Copia y el archivo del Virtual Host HTTP por defecto:

```bash
cd /etc/apache2/sites-available/
cp default-ssl.conf 000-proyecto-ssl.conf
```

4. Editar archivo virtual host https

```bash
vim /etc/apache2/sites-available/000-proyecto-ssl.conf
```

Configuremos de esta manera el archivo

![https configuracion virtual host](/markdown/images/apache/https/configuracion-fichero-clave-publica-privada.png)

5. Hsbilitamos el sitio y reiniciamos el apache

```bash
/sbin/a2ensite proyecto-ssl.conf
systemctl reload apache2
```

> Al hscer reload sale el siguente error por pantalla hay que instalar el siguente modulo a2enmod ssl

Tenemos que ir a esta ruta **/sbin/a2enmod**

Usar eeste comando **a2enmod ssl** y reiniciar apache
Despues he hacer un status todo funciona correctamente

![solucion mod ssl ](/markdown/images/apache/https/solution-mod-ssl.png)

[Solucion mod ssl activar ](https://stackoverflow.com/questions/5257974/how-to-install-mod-ssl-for-apache-httpd)

6. Renvio de puertos https

![Renvio puertos https configuracion](/markdown/images/apache/https/https-renvio-puertos.png)

Para acceder a la veb escribimos esta url con el puerto 8443
https://localhost:8443/

Si entramos por http nos dara el siguente error

![http error](/markdown/images/apache/https/http-bad-request-put-https.png)

tendremos que acceder con esta direccion y este puerto https://localhost:8443/

### Sitios web dinámicos con PHP

1. instalacion y configuracion de php mediante el driver de apache

```bash
apt install php libapache2-mod-php
systemctl restart apache2
```

2. checkear ls instalacion

```bash
php -v
```

este comando nos arrojara esta salida

PHP 8.2.28 (cli) (built: Mar 13 2025 18:21:38) (NTS)
Copyright (c) The PHP Group
Zend Engine v4.2.28, Copyright (c) Zend Technologies
with Zend OPcache v8.2.28, Copyright (c), by Zend Technologies

3. Prueba en el sitio web

Añadamos un fichero php a nuestra carpeta raiz del sitio para ver si funciona

```bash
vim /var/www/proyecto/public_html
```

Agregamos este contenido para ver la informacion del servidor php

```php
<?php
phpinfo();
```

4. Comprobar el log de errores aqui apareceran todos los errores del servidor

```bash
tail /var/log/apache2/proyecto_error.log
```

## Servidor de base de datos

### introduccion

1. MariaDB es un sistema de gestión de bases de datos relaciona

2. MariaDB utiliza por defecto el puerto 3306

#### Instalacion

1. instalacion de mariadb

```bash
apt update
apt install mariadb-server mariadb-client -y
```

2. comprobar estado del servidor

```bash
systemctl status mariadb
```

3. Transferir el archivo SQL al servidor de destino

Ya tenia un dump realizado asi que lo vamos a copiar a la maquina debian con scp s vierne de ssh

```bash
 scp -P 10022 dump-Intermodular.sql cesar@localhost:/home/cesar/
```

- Con la opcion -P indicamos el numero de puerto del ssh veamos si se exporto correctamente

![scp resultado](/markdown/images/mariadb/resultado-scp.png)

4. Nos conectamos a mysql con este comando

```bash
 mysql -u root
```

5. Creamos la base de datos

```bash
CREATE DATABASE peliculasIntermodular;
```

5.5. Verificar si la base de datos se creo bien

```bash
SHOW DATABASES;
```

6. Creamos al usuario administrador

```bash
CREATE USER 'peflix_admin'@'%' IDENTIFIED BY 'peflixAdmin!';
```

7. Checkear si el usuario esta creado

```bash
SELECT User FROM mysql.user;
```

8. inportar dump
   ocurrio un error ya que mysql no es compatible con maria db habia que cambiar 2 parametros

```bash
 cat dump-Intermodular.sql | sed -e s/utf8mb4_0900_ai_ci/utf8_general_ci/g | sed -e s/utf8mb4/utf8/g > dump-Intermodular-changed.sql
```

![sed command](/markdown/images/mariadb/sed-command.png)

8.1. Documentacion
[change params mariaDB](https://stackoverflow.com/questions/3031412/how-to-export-a-mysql-database-using-command-prompt)

9. Damos los permisos

```bash
GRANT ALL PRIVILEGES  ON peliculasIntermodular.* TO 'peflix_admin'@'%';
FLUSH PRIVILEGES;
```

### Instalamos el módulo php-mysql

```bash
apt install php-mysql -y
systemctl restart apache2
```

## Instalacion GIT

```bash
apt install git
```

1. Configurar usuario y correo

```bash
git config --global user.name "cesargm1"
git config --global user.name "cg.gtav@gmail.com"
```

### Instalacion de github cli

```bash
apt install gh
```

**login**

```bash
gh auth login
```

Cambiamos el documentRoot de los ficheros

```bash
vim  /etc/apache2/sites-available/proyecto.conf
vim  /etc/apache2/sites-available/proyecto.conf
vim  /etc/apache2/sites-available/000-proyecto-ssl.conf

```

usaremos esta ruta nueva

```bash
var/www/proyecto/public_html/public
```

2. Creamos el .env con usuario y contraseña que creamos

[.env](/markdown/images/proyecto/env.png)

### Instalamos composer

```bash
 apt-get install composer
 composer install
```

3. resultado final

![proyecto](/markdown/images/proyecto/proyecto.png)
