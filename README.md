# WordPress on Ubuntu EC2 — LAMP Stack Deployment

El laboratorio consiste en desplegar WordPress sobre una instancia Ubuntu en AWS EC2, utilizando Apache2 como servidor web, MariaDB como sistema de gestión de bases de datos y PHP como entorno de ejecución.

## 1. Objetivos

- Desplegar un servidor web Apache2 en Ubuntu.
- Instalar y verificar el servicio MariaDB.
- Instalar PHP y los módulos necesarios para WordPress.
- Crear una base de datos y un usuario para la aplicación.
- Descargar y desplegar WordPress en el directorio web.
- Configurar la propiedad y los permisos de los archivos.
- Completar el instalador web de WordPress y comprobar su funcionamiento.

## 2. Arquitectura

El proyecto utiliza una arquitectura LAMP:

| Componente | Función |
|---|---|
| Ubuntu Server | Sistema operativo de la instancia EC2. |
| AWS EC2 | Proporciona el servidor virtual en la nube. |
| Apache2 | Atiende las solicitudes HTTP y sirve el contenido web. |
| MariaDB | Almacena los datos de WordPress. |
| PHP | Ejecuta la lógica de la aplicación. |
| WordPress | Sistema de gestión de contenidos instalado. |

Los componentes se ejecutan en una misma instancia EC2. WordPress se conecta a MariaDB mediante el nombre de host `localhost`.

## 3. Requisitos previos

- Una cuenta de AWS.
- Una instancia EC2 con Ubuntu.
- Acceso administrativo a la instancia mediante SSH.
- Una llave privada SSH correspondiente a la instancia.
- Reglas de entrada adecuadas en el Security Group para SSH y HTTP.
- Acceso a los repositorios de paquetes de Ubuntu y al sitio oficial de WordPress.

La IP pública depende de la instancia utilizada. En los ejemplos se representa mediante `TU_IP_PUBLICA`.

## 4. Actualizar el sistema

Actualizar el índice de paquetes mediante APT:

```bash
sudo apt update
```

Este comando actualiza la información local sobre los paquetes disponibles en los repositorios configurados.

## 5. Instalar Apache2

Instalar Apache2:

```bash
sudo apt install apache2
```

Verificar el estado del servicio:

```bash
sudo systemctl status apache2
```

En la evidencia del laboratorio, Apache2 aparece en estado `active (running)`.

### Permitir tráfico HTTP

El laboratorio también incluye la siguiente regla de UFW:

```bash
sudo ufw allow "Apache"
```

Esta regla permite el tráfico correspondiente al perfil de aplicación Apache definido en UFW.

Para que el acceso HTTP funcione desde Internet, también deben permitirlo las reglas de entrada del Security Group de AWS.

## 6. Instalar MariaDB

Instalar el sistema de gestión de bases de datos:

```bash
sudo apt install mariadb-server -y
```

Comprobar su estado:

```bash
sudo systemctl status mariadb
```

La evidencia del laboratorio muestra MariaDB en estado `active (running)`.

## 7. Instalar PHP y sus módulos

Instalar PHP junto con los módulos utilizados durante el laboratorio:

```bash
apt install php libapache2-mod-php php-mysql php-cli php-curl php-gd php-intl php-mbstring php-soap php-xml php-zip -y
```

Los paquetes incluidos proporcionan el entorno PHP y sus extensiones, además de la integración con Apache2 y el controlador necesario para trabajar con MySQL/MariaDB.

### Verificar la versión de PHP

```bash
php -v
```

Durante el laboratorio, la terminal mostró PHP CLI 8.5.4.

### Reiniciar Apache2

```bash
systemctl restart apache2
```

Este comando reinicia el servicio para que cargue los cambios correspondientes al entorno PHP instalado.

## 8. Crear la base de datos y el usuario de WordPress

Entrar en el cliente de MariaDB:

```bash
mysql -u root -p
```

El comando solicita la contraseña del usuario root de MariaDB.

Dentro de la consola de MariaDB, ejecutar las siguientes instrucciones SQL:

```sql
CREATE DATABASE wordpress_db;

CREATE USER 'wp_user'@'localhost' IDENTIFIED BY 'CAMBIAR_POR_CLAVE_SEGURA';

GRANT ALL ON wordpress_db.* TO 'wp_user'@'localhost' IDENTIFIED BY 'CAMBIAR_POR_CLAVE_SEGURA';

FLUSH PRIVILEGES;

EXIT;
```

### Explicación

- `CREATE DATABASE` crea la base de datos `wordpress_db`.
- `CREATE USER` crea la cuenta `wp_user`, restringida al origen `localhost`.
- `GRANT ALL` concede al usuario los privilegios indicados sobre `wordpress_db`.
- `FLUSH PRIVILEGES` solicita a MariaDB que recargue las tablas de privilegios.
- `EXIT` cierra la consola de MariaDB.

**Credenciales:** en el informe original se utilizó una contraseña concreta que se ha sustituido por un marcador para publicar el procedimiento de forma segura. Al reproducir el laboratorio, reemplaza `CAMBIAR_POR_CLAVE_SEGURA` por una contraseña nueva, fuerte y exclusiva, utilizando el mismo valor en ambos comandos.

La sintaxis de asignación de contraseña dentro de `GRANT` puede depender de la versión de MariaDB. La secuencia anterior conserva la estructura documentada en el laboratorio; si una versión diferente la rechaza, consulta la documentación de esa versión antes de adaptar el procedimiento.

## 9. Descargar y desplegar WordPress

### Cambiar al directorio web

```bash
cd /var/www/html/
```

### Eliminar la página predeterminada de Apache

```bash
rm -f index.html
```

Este comando retira la página inicial predeterminada para que WordPress pueda ocupar la raíz del directorio web.

### Descargar WordPress

```bash
wget https://wordpress.org/latest.tar.gz
```

El archivo descargado contiene la distribución de WordPress.

### Extraer el archivo

```bash
tar -xvzf latest.tar.gz
```

La extracción genera el directorio `wordpress/` con los archivos de la aplicación.

### Mover los archivos a la raíz web

```bash
mv wordpress/* .
```

Este comando mueve el contenido del directorio `wordpress/` al directorio actual, `/var/www/html/`.

### Eliminar el archivo comprimido y el directorio ya utilizado

```bash
rm -rf latest.tar.gz wordpress
```

Se elimina el archivo comprimido y el directorio de origen una vez movidos los archivos.

**Precaución:** `rm -rf` elimina archivos y directorios sin solicitar confirmación. Antes de utilizarlo, verifica cuidadosamente la ruta y el contenido que deseas eliminar.

## 10. Configurar la propiedad y los permisos

Desde el directorio web, ejecutar:

```bash
cd /var/www/html/
chown -R www-data:www-data .
find . -type d -exec chmod 755 {} \;
find . -type f -exec chmod 644 {} \;
```

### Explicación

- `chown -R www-data:www-data .` asigna recursivamente la propiedad de los archivos y directorios al usuario y grupo `www-data`.
- `find . -type d -exec chmod 755 {} \;` establece modo `755` en los directorios.
- `find . -type f -exec chmod 644 {} \;` establece modo `644` en los archivos.

Estas instrucciones corresponden a los permisos utilizados durante el laboratorio. En un despliegue diferente, los permisos y la propiedad deben ajustarse a las necesidades reales de escritura de WordPress y a los requisitos de seguridad de la instalación.

## 11. Completar el instalador web de WordPress

Abrir la dirección pública de la instancia en un navegador:

```text
http://TU_IP_PUBLICA
```

El procedimiento documentado consiste en:

1. Seleccionar el idioma de instalación.
2. Acceder al formulario de configuración de la base de datos.
3. Introducir los datos de conexión.
4. Completar la información del sitio y crear la cuenta administrativa.
5. Finalizar la instalación y acceder al panel de WordPress.

Los valores utilizados para la conexión a la base de datos durante el laboratorio fueron:

| Campo | Valor |
|---|---|
| Base de datos | `wordpress_db` |
| Usuario de base de datos | `wp_user` |
| Contraseña | La definida localmente durante la configuración |
| Servidor de base de datos | `localhost` |
| Prefijo de tablas | `wp_` |

El título del sitio, el usuario administrador, su contraseña y el correo electrónico deben establecerse durante el proceso de instalación.

No publiques las contraseñas utilizadas ni las capturas que las puedan revelar.

## 12. Resultados documentados

El informe académico muestra las siguientes comprobaciones:

| Comprobación | Resultado |
|---|---|
| Instalación de Apache2 | Servicio activo y en ejecución. |
| Acceso web inicial | Página predeterminada de Apache visible desde el navegador. |
| Instalación de MariaDB | Servicio activo y en ejecución. |
| Instalación de PHP | Versión de PHP CLI visible en la terminal. |
| Creación de la base de datos | Consulta `CREATE DATABASE` ejecutada correctamente. |
| Creación y permisos del usuario de la base de datos | Consultas SQL aceptadas en la consola de MariaDB. |
| Despliegue de WordPress | Archivos colocados en `/var/www/html`. |
| Configuración de permisos | Propiedad `www-data:www-data`, directorios `755` y archivos `644`. |
| Instalación final | Panel administrativo de WordPress accesible desde el navegador. |

Estos resultados se basan en las evidencias incorporadas al informe académico. No implican que el procedimiento se haya repetido desde una instancia nueva mediante este README.

## 13. Seguridad y limitaciones

- No publiques contraseñas de MariaDB, credenciales administrativas ni llaves privadas.
- La IP utilizada en el laboratorio se sustituye por `TU_IP_PUBLICA`.
- No se documenta una configuración HTTPS para este laboratorio; el acceso mostrado se realiza mediante HTTP.
- La regla UFW para Apache no sustituye la configuración del Security Group de AWS.
- Los permisos `755` en directorios y `644` en archivos son los establecidos durante el laboratorio, pero no garantizan por sí solos una configuración óptima para todos los despliegues WordPress.
- La configuración de acceso a la base de datos se documenta para un entorno de laboratorio. En otros entornos debe revisarse el principio de mínimo privilegio.
- La descarga de `latest.tar.gz` utiliza la distribución disponible al momento de la descarga. Por ello, la versión de WordPress puede cambiar con el tiempo.
- El laboratorio no constituye una auditoría de seguridad ni documenta una configuración completa de hardening para producción.

## 14. Aprendizajes

La práctica permitió integrar conocimientos sobre:

- Administración de Ubuntu mediante terminal.
- Instalación y supervisión de servicios con APT y systemd.
- Despliegue de Apache2 y aplicaciones PHP.
- Administración de bases de datos con MariaDB.
- Creación de bases de datos, usuarios y privilegios.
- Despliegue de un sistema de gestión de contenidos.
- Administración de propiedad y permisos de archivos en Linux.
- Comprensión de la interacción entre servidor web, aplicación y base de datos.

## 15. Referencias

- Apache Software Foundation. *Apache HTTP Server Documentation*.  
  https://httpd.apache.org/docs/

- Canonical. *Ubuntu Server Documentation*.  
  https://ubuntu.com/server/docs

- MariaDB Foundation. *MariaDB Server Documentation*.  
  https://mariadb.com/docs/

- WordPress Foundation. *WordPress Documentation*.  
  https://wordpress.org/documentation/

- WordPress Developer Resources. *Installing WordPress*.  
  https://developer.wordpress.org/advanced-administration/before-install/howto-install/

- Yadava, J. A. (2024). *How to Install WordPress on Ubuntu: A Step-by-Step Guide*. Medium.  
  https://medium.com/@jayaprakashaadhira/how-to-install-wordpress-on-ubuntu-a-step-by-step-guide-a6bd3c40b36f
