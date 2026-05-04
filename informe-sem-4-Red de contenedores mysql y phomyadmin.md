# Práctica No. 4 — MySQL y phpMyAdmin con Docker usando redes personalizadas

## 1. Título

Despliegue y conexión de contenedores MySQL y phpMyAdmin mediante una red personalizada en Docker

---

## 2. Tiempo de duración

La práctica tomó aproximadamente **90 minutos** en completarse, incluyendo la creación de la red, configuración de los contenedores, verificación de la conectividad y creación de la base de datos de prueba.

---

## 3. Fundamentos

### Redes en Docker

Docker permite crear redes virtuales personalizadas para que los contenedores se comuniquen entre sí de forma aislada del resto del sistema. Cuando dos contenedores pertenecen a la misma red, pueden referenciarse directamente por nombre, sin necesidad de conocer sus direcciones IP. Esto es fundamental en entornos donde varios servicios deben colaborar, como una base de datos y una herramienta de administración.

El comando `docker network create` crea una red de tipo **bridge** por defecto. Una red bridge actúa como un switch virtual privado: todos los contenedores conectados a ella pueden comunicarse entre sí, pero están aislados de otras redes.

Cuando se inspecciona una red con `docker network inspect`, Docker muestra información detallada como la subred asignada, el gateway y la lista de contenedores conectados con sus respectivas IPs internas.

### MySQL en Docker

MySQL es uno de los sistemas de gestión de bases de datos relacionales más utilizados en el mundo. Al ejecutarlo en un contenedor Docker, se pueden definir credenciales y configuraciones iniciales mediante **variables de entorno** con la opción `-e`:

| Variable | Descripción |
|---|---|
| `MYSQL_ROOT_PASSWORD` | Contraseña del usuario root |
| `MYSQL_DATABASE` | Nombre de la base de datos inicial |
| `MYSQL_USER` | Usuario adicional |
| `MYSQL_PASSWORD` | Contraseña del usuario adicional |

Esto elimina la necesidad de configurar MySQL manualmente después de instalarlo: el contenedor arranca ya con la base de datos y las credenciales listas.

### phpMyAdmin en Docker

phpMyAdmin es una interfaz web que permite administrar bases de datos MySQL de forma visual, sin necesidad de usar la línea de comandos. Al conectarlo con el contenedor MySQL a través de la red personalizada, phpMyAdmin puede acceder al motor de base de datos usando el **nombre del contenedor** como host (`mysql_db`), ya que Docker resuelve ese nombre automáticamente dentro de la red.

Las variables de entorno clave para phpMyAdmin son:

| Variable | Descripción |
|---|---|
| `PMA_HOST` | Nombre del contenedor MySQL al que se conecta |
| `PMA_USER` | Usuario con el que inicia sesión |
| `PMA_PASSWORD` | Contraseña del usuario |

### Comunicación entre contenedores

La comunicación entre servicios en Docker depende de que ambos contenedores estén conectados a la misma red. Sin una red compartida, los contenedores no pueden verse entre sí aunque estén en el mismo equipo. Al adjuntar los dos contenedores a `red_mysql`, Docker asigna automáticamente IPs internas dentro de la subred `172.18.0.0/16` y permite la resolución de nombres por hostname.

---

## 4. Conocimientos previos

Para realizar esta práctica el estudiante necesita tener claros los siguientes temas:

* Comandos básicos de Docker: `docker run`, `docker ps`, `docker network`
* Concepto de base de datos relacional y sistema gestor (SGBD)
* Uso básico de variables de entorno en sistemas Linux
* Concepto de red y protocolo TCP/IP (puertos, subredes)
* Navegación web para acceder a interfaces de administración

---

## 5. Objetivos a alcanzar

* Crear una red personalizada en Docker de tipo bridge
* Desplegar un contenedor MySQL con credenciales configuradas mediante variables de entorno
* Desplegar un contenedor phpMyAdmin conectado al contenedor MySQL
* Verificar la conectividad entre ambos contenedores mediante `docker network inspect`
* Acceder a la interfaz de phpMyAdmin desde el navegador
* Crear una base de datos de prueba desde la interfaz web

---

## 6. Equipo necesario

* Computador con sistema operativo Windows con WSL (Windows Subsystem for Linux)
* Terminal de comandos (Bash / PowerShell)
* Docker versión 20.x o superior instalado y activo
* Navegador web (Chrome, Firefox, etc.)
* Conexión a internet para descargar las imágenes de MySQL y phpMyAdmin

---

## 7. Material de apoyo

* Documentación oficial de Docker: https://docs.docker.com
* Imagen oficial de MySQL en Docker Hub: https://hub.docker.com/_/mysql
* Imagen oficial de phpMyAdmin en Docker Hub: https://hub.docker.com/_/phpmyadmin
* Documentación de redes en Docker: https://docs.docker.com/network/
* Guía de la asignatura *Tendencias Tecnológicas*

---

## 8. Procedimiento

**Paso 1: Crear la red personalizada `red_mysql`**

Se creó una red de tipo bridge llamada `red_mysql` para que los dos contenedores puedan comunicarse entre sí por nombre:

```bash
docker network create red_mysql
```

Docker devolvió el ID de la red creada, confirmando la operación exitosa. Se verificó con:

```bash
docker network ls
```

La salida mostró la red `red_mysql` de tipo `bridge` con scope `local`, junto a las redes predeterminadas del sistema (`bridge`, `host`, `none`).

---

**Paso 2: Crear el contenedor MySQL (`mysql_db`)**

Se desplegó el contenedor de MySQL con las credenciales y la base de datos inicial configuradas mediante variables de entorno:

```bash
docker run -d \
  --name mysql_db \
  --network red_mysql \
  -e MYSQL_ROOT_PASSWORD=David1234 \
  -e MYSQL_DATABASE=db_prueba \
  -e MYSQL_USER=david \
  -e MYSQL_PASSWORD=davidpass \
  mysql:8.0
```

El contenedor arrancó en segundo plano y fue verificado con `docker ps`, mostrando estado `Up` y el puerto `3306/tcp` activo.

---

**Paso 3: Crear el contenedor phpMyAdmin (`phpmyadmin`)**

Se levantó el contenedor de phpMyAdmin conectado a la misma red, apuntando al contenedor MySQL por su nombre:

```bash
docker run -d \
  --name phpmyadmin \
  --network red_mysql \
  -e PMA_HOST=mysql_db \
  -e PMA_USER=root \
  -e PMA_PASSWORD=David1234 \
  -p 8080:80 \
  phpmyadmin
```

Con la opción `-p 8080:80` se expuso el puerto 80 del contenedor al puerto `8080` del host, permitiendo acceder a la interfaz web desde el navegador.

---

**Paso 4: Verificar que ambos contenedores están activos**

```bash
docker ps
```

La salida confirmó dos contenedores en estado `Up`:

| NAMES | IMAGE | PORTS | STATUS |
|---|---|---|---|
| phpmyadmin | phpmyadmin | 0.0.0.0:8080->80/tcp | Up |
| mysql_db | mysql:8.0 | 3306/tcp | Up |

---

**Paso 5: Inspeccionar la red para confirmar la conectividad**

```bash
docker network inspect red_mysql
```

La inspección reveló que ambos contenedores estaban correctamente conectados a la red `red_mysql` con la subred `172.18.0.0/16` y gateway `172.18.0.1`:

* `mysql_db` → IP interna `172.18.0.2`
* `phpmyadmin` → IP interna `172.18.0.3`

Esta asignación de IPs por parte de Docker confirma que los dos servicios pueden comunicarse dentro de la red virtual sin exponer credenciales al exterior.

---

**Paso 6: Acceder a phpMyAdmin desde el navegador**

Se abrió el navegador y se accedió a:

```
http://localhost:8080/index.php
```

La interfaz de phpMyAdmin cargó correctamente, mostrando la conexión establecida con el servidor `mysql_db`. En el panel izquierdo se visualizaron las bases de datos disponibles: `db_prueba`, `information_schema`, `mysql`, `performance_schema` y `sys`.

---

**Paso 7: Crear una base de datos de prueba (`bd_ejemplo`)**

Desde la interfaz de phpMyAdmin se navegó a la sección **Bases de datos** y se creó una nueva base de datos:

* **Nombre:** `bd_ejemplo`
* **Cotejamiento:** `utf8mb4_general_ci`

Se hizo clic en **Crear** y phpMyAdmin confirmó la operación con el mensaje:
*"Se creó la base de datos bd\_ejemplo"*

La nueva base de datos apareció en el listado junto a `db_prueba`.

---

**Paso 8: Verificar la base de datos creada**

Se accedió a `bd_ejemplo` desde el panel lateral. La interfaz mostró que la base de datos estaba vacía y lista para recibir tablas, con la opción **Crear tabla** disponible y el campo para definir el nombre y número de columnas.

---

## 9. Resultados esperados

Al finalizar la práctica se obtuvieron los siguientes resultados:

* Se creó exitosamente la red personalizada `red_mysql` de tipo bridge, permitiendo la comunicación directa entre contenedores por nombre de host.
* El contenedor `mysql_db` arrancó con la base de datos `db_prueba` y las credenciales configuradas sin intervención manual posterior.
* El contenedor `phpmyadmin` se conectó correctamente a `mysql_db` usando el nombre del contenedor como host, demostrando la resolución de nombres dentro de una red Docker.
* La inspección de la red confirmó que ambos contenedores recibieron IPs internas (`172.18.0.2` y `172.18.0.3`) dentro de la subred `172.18.0.0/16`.
* La interfaz de phpMyAdmin fue accesible en `http://localhost:8080` y permitió gestionar el servidor MySQL de forma visual.
* Se creó la base de datos `bd_ejemplo` con cotejamiento `utf8mb4_general_ci`, confirmando el funcionamiento completo del entorno.

Como resultado general, se comprendió cómo Docker permite orquestar múltiples servicios interconectados mediante redes personalizadas, replicando en un entorno local la arquitectura de sistemas reales donde la base de datos y su herramienta de administración corren como servicios independientes pero coordinados.

---

## 10. Bibliografía

* Docker Inc. (2024). *Docker Networking Overview*. Recuperado de https://docs.docker.com/network/
* Docker Hub. (2024). *mysql — Official Image*. Recuperado de https://hub.docker.com/_/mysql
* Docker Hub. (2024). *phpmyadmin — Official Image*. Recuperado de https://hub.docker.com/_/phpmyadmin
* Oracle Corporation. (2024). *MySQL 8.0 Reference Manual*. Recuperado de https://dev.mysql.com/doc/refman/8.0/en/