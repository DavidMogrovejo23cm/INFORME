# Práctica No. 3 — Persistencia de Datos con Volúmenes Docker y PostgreSQL

## 1. Título

Creación y gestión de volúmenes Docker para garantizar la persistencia de bases de datos PostgreSQL en entornos contenerizados.

---

## 2. Tiempo de duración

Aproximadamente 90 minutos.

---

## 3. Fundamentos

Uno de los conceptos más importantes que hay que entender cuando se trabaja con Docker en proyectos reales es la **persistencia de datos**. Docker fue diseñado para ejecutar aplicaciones en entornos completamente aislados llamados contenedores. Esta característica es muy útil porque permite tener entornos reproducibles, pero también trae un comportamiento que puede sorprender a quienes trabajan con bases de datos por primera vez: los contenedores son **efímeros por naturaleza**.

Lo que significa "efímero" en este contexto es que, cuando un contenedor es detenido y eliminado, todo el sistema de archivos que existía dentro de él desaparece con él. Si se tenía una base de datos almacenada en el interior del contenedor y el contenedor se elimina, esa base de datos se pierde de manera permanente. No existe ninguna forma de recuperarla, porque nunca salió del espacio de vida del contenedor.

Para resolver este problema, Docker ofrece un mecanismo llamado **volúmenes**. Un volumen es una unidad de almacenamiento que Docker gestiona de forma independiente al ciclo de vida de los contenedores. Los volúmenes se almacenan físicamente en el sistema anfitrión (en Linux, bajo la ruta `/var/lib/docker/volumes/`) y pueden ser montados dentro de uno o varios contenedores. La clave está en que, aunque el contenedor sea eliminado, el volumen y todo su contenido permanecen intactos. Cuando se crea un nuevo contenedor y se le asocia el mismo volumen, los datos siguen ahí, disponibles de inmediato.

**PostgreSQL** es un sistema gestor de bases de datos relacional de código abierto, reconocido por su robustez, su compatibilidad con el estándar SQL y su soporte para características avanzadas como transacciones ACID, integridad referencial mediante claves foráneas, vistas, índices compuestos y procedimientos almacenados. Es ampliamente utilizado tanto en entornos de desarrollo como en producción a nivel empresarial.

Cuando se ejecuta PostgreSQL dentro de un contenedor Docker, sus datos se almacenan en un directorio interno conocido como `PGDATA`. En las versiones anteriores de la imagen oficial de PostgreSQL, este directorio estaba ubicado en `/var/lib/postgresql/data`. Sin embargo, a partir de **PostgreSQL 18**, la imagen oficial reorganizó la estructura interna para facilitar actualizaciones usando `pg_upgrade`, por lo que el punto de montaje correcto para el volumen pasó a ser `/var/lib/postgresql` (sin el subdirectorio `/data`). Intentar montar el volumen en la ruta antigua provoca un error de inicialización.

Comprender la diferencia entre un contenedor sin volumen y uno con volumen es fundamental para cualquier proyecto donde la pérdida de datos no sea aceptable, que en la práctica incluye prácticamente cualquier aplicación real.

---

## 4. Conocimientos previos

Para llevar a cabo esta práctica es necesario tener claridad sobre los siguientes temas:

- Uso básico de la terminal de Linux o PowerShell/WSL en Windows.
- Conceptos fundamentales de Docker: imágenes, contenedores y puertos.
- Comandos básicos de Docker: `run`, `ps`, `stop`, `rm`, `volume create`, `volume ls`.
- Conceptos básicos de bases de datos relacionales: tablas, registros, tipos de datos y consultas SQL.
- Uso de un cliente PostgreSQL, ya sea `psql` en línea de comandos o una herramienta gráfica como DataGrip o TablePlus.

---

## 5. Objetivos a alcanzar

- Demostrar de forma práctica la pérdida de datos que ocurre en contenedores Docker sin volúmenes asignados.
- Crear y administrar volúmenes Docker a través de la terminal.
- Desplegar un contenedor PostgreSQL con un volumen correctamente montado.
- Crear bases de datos, tablas e insertar registros utilizando `psql`.
- Verificar que los datos persisten correctamente tras eliminar y recrear el contenedor.
- Identificar y resolver el error de compatibilidad del punto de montaje en PostgreSQL 18+.

---

## 6. Equipo necesario

- Computador con sistema operativo Linux, Windows con WSL2 habilitado, o macOS.
- Terminal de comandos (bash, zsh o PowerShell).
- Docker instalado, versión 20.x o superior.
- Cliente PostgreSQL: `psql` instalado localmente, o herramienta gráfica como DataGrip o TablePlus.
- Conexión a Internet para descargar la imagen de PostgreSQL desde Docker Hub.

---

## 7. Material de apoyo

- Documentación oficial de Docker Volumes: https://docs.docker.com/storage/volumes/
- Documentación oficial de PostgreSQL: https://www.postgresql.org/docs/
- Imagen oficial de PostgreSQL en Docker Hub: https://hub.docker.com/_/postgres
- Guía de la asignatura proporcionada por el docente.

---

## 8. Procedimiento

### 🔴 Parte 1: Base de datos SIN volumen

**Paso 1: Crear el contenedor PostgreSQL sin volumen**

Se ejecuta el siguiente comando para crear el contenedor llamado `server_db1`, sin asociarle ningún volumen:

```bash
docker run --name server_db1 \
  -e POSTGRES_PASSWORD=1234 \
  -p 5432:5432 \
  -d postgres
```

---

**Paso 2: Conectarse al contenedor con psql**

```bash
psql -h localhost -U postgres
```

---

**Paso 3: Crear la base de datos `test`**

```sql
CREATE DATABASE test;
```

---

**Paso 4: Conectarse a `test` y crear la tabla `customer`**

```sql
\c test

CREATE TABLE customer (
    id SERIAL PRIMARY KEY,
    fullname VARCHAR(100),
    status VARCHAR(50)
);
```

---

**Paso 5: Insertar un registro**

```sql
INSERT INTO customer (fullname, status)
VALUES ('David Mogrovejo', 'activo');
```

---

**Paso 6: Verificar el registro insertado**

```sql
SELECT * FROM customer;
```

Resultado obtenido:

```
 id | fullname        | status 
----+-----------------+--------
  1 | David Mogrovejo | activo
```

---

**Paso 7: Salir, detener y eliminar el contenedor**

```bash
\q
docker stop server_db1
docker rm server_db1
```

---

**Paso 8: Recrear el contenedor con el mismo nombre**

```bash
docker run --name server_db1 \
  -e POSTGRES_PASSWORD=1234 \
  -p 5432:5432 \
  -d postgres
```

*Figura 8-1. Detención, eliminación y recreación del contenedor server_db1.*

![Figura 8-1](fig_pasos7_8_stop_rm_run.png)

---

**Paso 9: Verificar que los datos ya no existen**

```bash
psql -h localhost -U postgres
\l
```

La base de datos `test` ya no aparece en el listado. Los datos se perdieron completamente al eliminar el contenedor, confirmando que sin volúmenes no existe ningún tipo de persistencia.

*Figura 8-2. Solo aparecen las bases de datos por defecto tras recrear el contenedor sin volumen.*

![Figura 8-2](fig_paso9_sin_volumen.png)

```bash
\q
docker stop server_db1
docker rm server_db1
```

---

### 🟢 Parte 2: Base de datos CON volumen

**Paso 10: Eliminar volumen previo si existe y crear uno nuevo**

```bash
docker volume rm pgdata 2>/dev/null
docker volume create pgdata
```

**Paso 11: Crear el contenedor PostgreSQL con volumen**

Para PostgreSQL 18+, el punto de montaje correcto es `/var/lib/postgresql`:

```bash
docker run --name server_db2 \
  -e POSTGRES_PASSWORD=1234 \
  -p 5432:5432 \
  -v pgdata:/var/lib/postgresql \
  -d postgres
```

*Figura 8-3. Creación del volumen pgdata y despliegue del contenedor server_db2.*

![Figura 8-3](fig_pasos10_11_volumen_create.png)

---

**Paso 12: Conectarse y crear la base de datos**

```bash
psql -h localhost -U postgres
```

```sql
CREATE DATABASE test;
\c test
```

---

**Paso 13: Crear la tabla `customer`**

```sql
CREATE TABLE customer (
    id SERIAL PRIMARY KEY,
    fullname VARCHAR(100),
    status VARCHAR(50)
);
```

---

**Paso 14: Insertar un registro**

```sql
INSERT INTO customer (fullname, status)
VALUES ('David Mogrovejo', 'activo');
```

---

**Paso 15: Verificar el registro**

```sql
SELECT * FROM customer;
```

Resultado:

```
 id | fullname        | status 
----+-----------------+--------
  1 | David Mogrovejo | activo
```

---

**Paso 16: Detener y eliminar el contenedor (manteniendo el volumen)**

```bash
\q
docker stop server_db2
docker rm server_db2
```

---

**Paso 17: Recrear el contenedor usando el mismo volumen**

```bash
docker run --name server_db2 \
  -e POSTGRES_PASSWORD=1234 \
  -p 5432:5432 \
  -v pgdata:/var/lib/postgresql \
  -d postgres
```

---

**Paso 18: Verificar que los datos persisten**

```bash
psql -h localhost -U postgres
\l
\c test
SELECT * FROM customer;
```

Resultado:

```
 id | fullname        | status 
----+-----------------+--------
  1 | David Mogrovejo | activo
```

La base de datos `test`, la tabla `customer` y el registro de David Mogrovejo siguen presentes e intactos. El volumen garantizó la persistencia correctamente.

*Figura 8-4. Verificación de persistencia: los datos sobreviven a la eliminación y recreación del contenedor.*

![Figura 8-4](fig_paso18_persistencia.png)

```bash
\q
```

---

### ⚠️ Nota sobre error común en PostgreSQL 18+

Al intentar montar el volumen en `/var/lib/postgresql/data`, la imagen oficial de PostgreSQL 18 genera el siguiente error:

```
Error: in 18+, these Docker images are configured to store database data in a
format which is compatible with "pg_ctlcluster"...
```

**Solución:** cambiar el punto de montaje de `/var/lib/postgresql/data` a `/var/lib/postgresql`:

```bash
# ❌ Incorrecto para PostgreSQL 18+
-v pgdata:/var/lib/postgresql/data

# ✅ Correcto para PostgreSQL 18+
-v pgdata:/var/lib/postgresql
```

---

## 9. Resultados esperados

Al finalizar la práctica se obtuvieron los siguientes resultados:

**Parte 1 — Sin volumen:** Se confirmó que al eliminar el contenedor `server_db1`, la base de datos `test` y todos los registros almacenados en la tabla `customer` desaparecieron sin posibilidad de recuperación. Al recrear el contenedor con el mismo nombre, la instancia de PostgreSQL estaba completamente vacía, demostrando que los contenedores Docker sin volúmenes no ofrecen ningún tipo de persistencia de datos.

**Parte 2 — Con volumen:** Se confirmó que al eliminar el contenedor `server_db2` y volver a crearlo usando el mismo volumen `pgdata`, la base de datos `test`, la tabla `customer` y el registro de David Mogrovejo (activo) seguían presentes e intactos. El volumen Docker actuó como una capa de almacenamiento independiente, sobreviviendo al ciclo de vida del contenedor.

**Error de compatibilidad:** Se identificó y resolvió el error que genera PostgreSQL 18+ al montar el volumen en el subdirectorio `/data`, adoptando como solución definitiva el uso del directorio padre `/var/lib/postgresql`.

Como conclusión general, se comprendió de forma práctica la diferencia crítica entre almacenamiento efímero y persistente en Docker. Esta distinción es fundamental al momento de desplegar cualquier base de datos en producción, donde la pérdida de datos no es una opción aceptable.

---

## 10. Bibliografía

Docker Inc. (2024). *Use volumes*. Docker Documentation. https://docs.docker.com/storage/volumes/

Docker Inc. (2024). *postgres - Official Image*. Docker Hub. https://hub.docker.com/_/postgres

The PostgreSQL Global Development Group. (2024). *PostgreSQL 16 Documentation*. https://www.postgresql.org/docs/16/

Merkel, D. (2014). Docker: Lightweight Linux containers for consistent development and deployment. *Linux Journal, 2014*(239), 2. https://www.linuxjournal.com/content/docker-lightweight-linux-containers-consistent-development-and-deployment

Turnbull, J. (2016). *The Docker Book: Containerization is the new virtualization*. James Turnbull.