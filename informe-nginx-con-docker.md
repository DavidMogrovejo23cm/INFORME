# Práctica No. 1 — Servidores Web con Nginx usando Docker

## 1. Título

Creación y personalización de dos servidores web Nginx mediante contenedores Docker

---

## 2. Tiempo de duración

La práctica tomó aproximadamente **85 minutos** en completarse, incluyendo la descarga de la imagen, creación de los contenedores y edición de los archivos HTML.

---

## 3. Fundamentos

Docker es una plataforma que permite ejecutar aplicaciones dentro de **contenedores**: entornos aislados que comparten el kernel del sistema operativo anfitrión. A diferencia de las máquinas virtuales, los contenedores no cargan un sistema operativo completo, lo que los hace mucho más ligeros, rápidos de iniciar y eficientes en el uso de recursos.

El modelo de trabajo de Docker se basa en dos elementos principales:

**Imagen**: es una plantilla estática e inmutable que contiene todo lo necesario para ejecutar una aplicación: el sistema de archivos base, librerías, variables de entorno y el proceso de inicio. Las imágenes se almacenan en registros públicos como Docker Hub y se descargan con `docker pull`, o automáticamente cuando se usa `docker run` y la imagen no está en el sistema local.

**Contenedor**: es una instancia activa de una imagen. Cada contenedor opera de forma aislada, con su propio sistema de archivos en capas, red virtual y árbol de procesos. Puede comunicarse con el sistema exterior mediante la exposición de puertos.

**Nginx** (pronunciado *engine-x*) es un servidor web de alto rendimiento, ampliamente utilizado como servidor HTTP, proxy inverso y balanceador de carga. Su diseño orientado a eventos le permite manejar miles de conexiones simultáneas con un consumo mínimo de memoria. Dentro de su contenedor oficial, Nginx sirve archivos desde el directorio `/usr/share/nginx/html/` y escucha peticiones en el puerto **80** por defecto.

**Mapeo de puertos**: la opción `-p` en `docker run` crea una regla de redirección entre el host y el contenedor. Por ejemplo, `-p 8089:80` significa que cualquier petición al puerto `8089` de la máquina local se redirige al puerto `80` del contenedor. Esto permite levantar múltiples servidores en el mismo equipo, cada uno en un puerto externo distinto.

**`docker cp`**: comando que permite copiar archivos y directorios entre el sistema anfitrión y un contenedor activo, sin necesidad de abrir una sesión interactiva. Es la forma más práctica de actualizar el contenido web de un servidor Nginx sin detener ni reconstruir el contenedor.

Este enfoque de múltiples contenedores independientes es la base de las arquitecturas de **microservicios**, donde cada componente de una aplicación corre de forma aislada, escalable y fácil de reemplazar.

---

## 4. Conocimientos previos

Para realizar esta práctica el estudiante necesita tener claros los siguientes temas:

* Uso básico de la terminal de Linux (navegación de directorios, listado de archivos)
* Comandos esenciales: `ls`, `cd`, `cp`, `cat`
* Concepto de servidor web y protocolo HTTP (cliente-servidor, puertos)
* Manejo de un editor de texto en terminal (`nano` o `vi`)
* Estructura básica de un documento HTML
* Uso del navegador web para acceder a URLs locales (`localhost`)

---

## 5. Objetivos a alcanzar

* Desplegar contenedores Docker a partir de la imagen oficial de Nginx
* Configurar el mapeo de puertos para exponer cada servidor al navegador del host
* Utilizar `docker cp` para transferir archivos entre el host y los contenedores activos
* Personalizar el contenido HTML del primer servidor con información institucional
* Personalizar el contenido HTML del segundo servidor con información personal
* Verificar el funcionamiento simultáneo de ambos servidores en diferentes puertos

---

## 6. Equipo necesario

* Computador con sistema operativo Linux (Arch Linux)
* Terminal de comandos (Bash)
* Docker versión 20.x o superior instalado y activo
* Editor de texto en terminal (`nano` o `vi`)
* Navegador web (Firefox, Chrome, etc.)
* Conexión a internet para descargar la imagen de Nginx desde Docker Hub

---

## 7. Material de apoyo

* Documentación oficial de Docker: https://docs.docker.com
* Imagen oficial de Nginx en Docker Hub: https://hub.docker.com/_/nginx
* Wiki de Nginx en Arch Linux: https://wiki.archlinux.org/title/Nginx
* Guía de la asignatura *Tendencias Tecnológicas*
* Cheat sheet de comandos Docker y Linux

---

## 8. Procedimiento

**Paso 1: Verificar que Docker está activo**

Antes de crear los contenedores se comprobó el estado del daemon y si había contenedores previos en ejecución:

```bash
docker --version
docker ps
```

**Paso 2: Crear el primer contenedor nginx1**

Se levantó el primer servidor Nginx con el nombre `nginx1`, exponiendo el puerto `8089` del host al puerto `80` del contenedor:

```bash
docker run -d --name nginx1 -p 8089:80 nginx
```

| Parámetro | Descripción |
|---|---|
| `-d` | Ejecuta el contenedor en segundo plano |
| `--name nginx1` | Asigna el nombre `nginx1` al contenedor |
| `-p 8089:80` | Redirige el puerto 8089 del host al 80 del contenedor |
| `nginx` | Imagen oficial descargada desde Docker Hub |

**Paso 3: Crear el segundo contenedor nginx2**

Se repitió el proceso para el segundo servidor, usando el puerto `8090`:

```bash
docker run -d --name nginx2 -p 8090:80 nginx
```

**Paso 4: Confirmar que ambos contenedores están activos**

```bash
docker ps
```

Los dos contenedores deben aparecer con estado `Up` en la columna `STATUS`.

**Paso 5: Extraer el index.html de nginx1 al sistema anfitrión**

Se copió el archivo de bienvenida por defecto de Nginx hacia el directorio de trabajo local:

```bash
docker cp nginx1:/usr/share/nginx/html/index.html ./index1.html
```

**Paso 6: Editar index1.html con información institucional**

Se abrió el archivo con el editor de texto y se reemplazó el contenido original:

```bash
nano index1.html
```

Contenido del archivo editado:

```html
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Instituto Tecnológico Sudamericano</title>
</head>
<body>
  <h1>Instituto Tecnológico Sudamericano</h1>
  <p>Carrera: Desarrollo de Software</p>
  <p>Materia: Tendencias Tecnológicas</p>
  <p>Período académico: 2025 - 2026</p>
</body>
</html>
```

**Paso 7: Cargar el archivo editado de vuelta a nginx1**

Una vez guardado el archivo, se copió de regreso al contenedor:

```bash
docker cp index1.html nginx1:/usr/share/nginx/html/index.html
```

El servidor refleja el cambio de inmediato sin necesidad de reiniciarlo.

**Paso 8: Extraer el index.html de nginx2 al sistema anfitrión**

Se repitió el proceso de extracción para el segundo contenedor:

```bash
docker cp nginx2:/usr/share/nginx/html/index.html ./index2.html
```

**Paso 9: Editar index2.html con información personal**

```bash
nano index2.html
```

Contenido del archivo editado:

```html
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>David Mogrovejo</title>
</head>
<body>
  <h1>David Mogrovejo</h1>
  <p>Estudiante de Desarrollo de Software</p>
  <p>Ciudad: Cuenca, Ecuador</p>
  <p>Materia: Tendencias Tecnológicas</p>
</body>
</html>
```

**Paso 10: Cargar el archivo editado de vuelta a nginx2**

```bash
docker cp index2.html nginx2:/usr/share/nginx/html/index.html
```

**Paso 11: Verificar los resultados en el navegador**

Se accedió a las siguientes URLs desde el navegador:

| Contenedor | URL | Contenido |
|---|---|---|
| `nginx1` | http://localhost:8089 | Información institucional |
| `nginx2` | http://localhost:8090 | Información personal del estudiante |

---

## 9. Resultados esperados

Al finalizar la práctica se obtuvieron los siguientes resultados:

- Dos contenedores Nginx corriendo simultáneamente en el mismo equipo, cada uno sirviendo contenido distinto sin interferencia entre ellos.
- El servidor `nginx1` en `http://localhost:8089` muestra la página con información del Instituto Tecnológico Sudamericano.
- El servidor `nginx2` en `http://localhost:8090` muestra la página de presentación personal de David Mogrovejo.
- Se comprobó que `docker cp` permite actualizar el contenido web de un contenedor activo de forma directa, sin detener el servicio ni reconstruir la imagen.
- La edición del `index.html` y su carga inmediata al contenedor confirmó que Nginx sirve archivos estáticos en tiempo real.

Como resultado general, se logró comprender y aplicar el flujo completo de trabajo con Docker: descarga de imagen → creación de contenedor → exposición de puertos → transferencia de archivos → personalización → verificación en navegador.

---

## 10. Bibliografía

* Docker Inc. (2024). *Docker Documentation*. Recuperado de https://docs.docker.com
* Docker Hub. (2024). *nginx — Official Image*. Recuperado de https://hub.docker.com/_/nginx
* Arch Linux Wiki. (2024). *Nginx*. Recuperado de https://wiki.archlinux.org/title/Nginx
* Reitz, K. & Schlusser, T. (2016). *The Hitchhiker's Guide to Python*. O'Reilly Media.