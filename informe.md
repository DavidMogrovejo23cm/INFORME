# Práctica No. 1 – Gestión y Manipulación de Archivos y Directorios Mediante Comandos Básicos de Linux con Warp Terminal

**Estudiante:** David Mogrovejo
**Asignatura:** Sistemas Operativos
**Práctica No.:** 1
**Fecha:** Abril 2026

## 1. Título

Gestión y Manipulación de Archivos y Directorios Mediante Comandos Básicos de Linux.

## 2. Tiempo de Duración

Aproximadamente **50 minutos**.

## 3. Fundamentos

Linux es un sistema operativo de código abierto basado en Unix, ampliamente utilizado en servidores, estaciones de trabajo y entornos de desarrollo. A diferencia de otros sistemas operativos, Linux ofrece un control directo sobre sus recursos a través de la interfaz de línea de comandos (CLI). En lugar de interactuar con íconos y menús, el usuario escribe instrucciones en la terminal, lo que permite una comunicación más precisa, reproducible y automatizable con el sistema.

Para esta práctica se utilizó **Warp**, una terminal moderna desarrollada en Rust que transforma la experiencia tradicional de línea de comandos. Warp organiza cada par de entrada/salida en "bloques" visuales separados, lo que facilita la lectura de resultados, la edición de comandos anteriores y el seguimiento del historial de trabajo. Esta característica resultó especialmente útil al documentar el procedimiento paso a paso con mayor claridad.

### Estructura jerárquica del sistema de archivos

Linux organiza todos sus archivos en una estructura de árbol que parte desde la raíz (`/`). A partir de ahí se ramifican directorios del sistema como `/bin` (binarios ejecutables), `/etc` (archivos de configuración), `/home` (carpetas personales de usuarios), `/var` (datos variables como logs) y `/tmp` (archivos temporales). Comprender esta jerarquía es fundamental para navegar y gestionar archivos con precisión, ya sea usando **rutas absolutas** (desde `/`) o **rutas relativas** (desde la ubicación actual del usuario).

### Comandos esenciales de gestión

Los comandos utilizados en esta práctica forman el núcleo de la administración básica de archivos en Linux:

- `mkdir` Crea uno o varios directorios. Con `-p` crea rutas anidadas de una sola vez. |
- `cd` Cambia el directorio actual. Se puede usar con rutas absolutas, relativas o `~` para ir al home. |
- `ls` Lista el contenido de un directorio. Con `-la` muestra archivos ocultos y permisos. 
- `touch` Crea archivos vacíos o actualiza la marca de tiempo de un archivo existente. 
- `cp` Copia archivos o directorios. Con `-r` permite copiar carpetas enteras de forma recursiva. 
- `mv` Mueve o renombra archivos y directorios en una sola operación. 
- `rm` Elimina archivos. Con `-r` borra carpetas; con `-f` omite confirmaciones. Usar con precaución. 
- `cat` Muestra el contenido de un archivo o concatena múltiples archivos en la salida estándar. 
- `history` Muestra el historial de comandos ejecutados en la sesión actual de la terminal. 
- `>` Redirección de salida: envía el resultado de un comando a un archivo (sobrescribe). 
- `>>` Redirección de salida (append): agrega el resultado al final del archivo sin borrar el contenido existente. 
- `nano` / `nvim` Editores de texto en consola. `nano` es más sencillo; `nvim` (NeoVim) ofrece mayor potencia y extensibilidad. 

### Redirección de flujos y editores de consola

La redirección de flujos es una de las características más poderosas de la shell. Con `>` se guarda la salida de cualquier comando en un archivo, mientras que `>>` añade información al final sin sobrescribir lo existente. Por ejemplo, ejecutar `history >> historial.txt` genera automáticamente un registro de la sesión. Esta técnica, combinada con editores como `nano` y `nvim`, permite crear y editar archivos directamente desde la terminal sin necesidad de entorno gráfico.

NeoVim (`nvim`) es una versión modernizada y extensible de Vim, con soporte para plugins, resaltado de sintaxis y modos de edición avanzados (Normal, Insertar, Visual y Comando). Su curva de aprendizaje es mayor, pero su productividad una vez dominado supera ampliamente a la de editores más simples.

## 4. Conocimientos Previos

Para realizar esta práctica, el estudiante necesita tener claros los siguientes temas:

- Uso básico de un emulador de terminal moderno (en este caso, **Warp Terminal**).
- Navegación entre directorios usando `cd` y listado de contenidos con `ls`.
- Diferencia entre rutas absolutas (desde `/`) y rutas relativas (desde el directorio actual).
- Comandos elementales de gestión de archivos: `mkdir`, `touch`, `cp`, `mv` y `rm`.
- Uso básico de al menos un editor de texto en consola (`nano` o `nvim`).
- Comprensión del concepto de shell y sesión de terminal.

## 5. Objetivos a Alcanzar

- Estructurar y organizar un árbol de directorios propio desde la línea de comandos.
- Crear, copiar, mover y eliminar archivos utilizando únicamente comandos de terminal.
- Aplicar redirecciones (`>` y `>>`) para transferir y acumular contenido entre archivos.
- Comprender y aprovechar la lógica de la shell para trabajar de forma eficiente.
- Documentar el proceso exportando el historial de comandos a un archivo de texto.
- Familiarizarse con las ventajas visuales y productivas de **Warp Terminal**.

## 6. Equipo Necesario

- Computadora con sistema operativo Linux, WSL2 o entorno compatible.
- Terminal **Warp** instalada (disponible en https://www.warp.dev/).
- Editor de texto **NeoVim** (`nvim`) instalado en el sistema.
- Conexión a internet y cuenta activa en GitHub (para respaldo opcional).

## 7. Material de Apoyo

- Guía oficial de la práctica proporcionada por el docente.
- Apuntes personales de clase sobre comandos de Linux.
- Cheat sheet de comandos de terminal (hoja de referencia rápida).
- Páginas del manual del sistema: `man mkdir`, `man cp`, `man mv`, `man rm`, `man history`.
- Documentación oficial de NeoVim: https://neovim.io/doc/
- Documentación oficial de Warp Terminal: https://docs.warp.dev/

## 8. Procedimiento

### Paso 1: Crear la estructura de carpetas principal

Se creó el directorio raíz del proyecto llamado `proyecto_comandos` y, dentro de él, las tres subcarpetas necesarias: `documentos`, `imagenes` y `scripts`. Se usó `ls` para verificar la estructura resultante.

```bash
mkdir proyecto_comandos
cd proyecto_comandos
mkdir documentos imagenes scripts
ls -la
```

### Paso 2: Crear archivos en cada subcarpeta

Se crearon archivos vacíos dentro de cada directorio usando `touch`, simulando la preparación de archivos antes de editarlos.

```bash
touch documentos/notas.txt documentos/resumen.txt
touch imagenes/foto1.png imagenes/foto2.png
touch scripts/tarea.sh scripts/backup.sh
```

### Paso 3: Editar un archivo con nvim

Se abrió `notas.txt` con NeoVim. Se presionó `i` para entrar al modo Insertar, se escribió el contenido y luego `Esc` seguido de `:wq` para guardar y salir.

```bash
nvim documentos/notas.txt
```

### Paso 4: Redirigir contenido con `>` y `>>`

Se copió el contenido de `notas.txt` hacia `resumen.txt` usando `>`, luego se agregó una línea adicional con `>>`. Se verificó el resultado con `cat`.

```bash
cat documentos/notas.txt > documentos/resumen.txt
echo "Línea adicional agregada" >> documentos/resumen.txt
cat documentos/resumen.txt
```

### Paso 5: Copiar y mover archivos

Se copió `resumen.txt` hacia la carpeta `scripts` con `cp`, y se renombró `backup.sh` usando `mv`.

```bash
cp documentos/resumen.txt scripts/resumen_copia.txt
mv scripts/backup.sh scripts/respaldo_final.sh
ls scripts/
```

### Paso 6: Eliminar archivos

Se eliminó `foto2.png` de la carpeta `imagenes` con `rm` y se verificó el resultado.

```bash
rm imagenes/foto2.png
ls imagenes/
```

### Paso 7: Exportar el historial de comandos

Al finalizar la sesión, se exportó el historial completo de comandos a un archivo de texto como evidencia documental de la práctica.

```bash
history >> documentos/tarea-s1-david_mogrovejo.txt
cat documentos/tarea-s1-david_mogrovejo.txt
```

## 9. Resultados Esperados

Al finalizar la práctica se obtuvo una estructura de directorios funcional dentro de `proyecto_comandos`, con archivos distribuidos en tres subcarpetas. El archivo `resumen.txt` contiene el texto original de `notas.txt` más la línea adicional agregada con `>>`. La carpeta `scripts/` muestra `resumen_copia.txt` y `respaldo_final.sh` como resultado de las operaciones de copia y renombrado. Finalmente, `tarea-s1-david_mogrovejo.txt` registra cronológicamente todos los comandos ejecutados durante la sesión, constituyendo la evidencia documental completa de la práctica.

## 10. Bibliografía

The Linux Documentation Project. (2023). *Linux System Administrator's Guide*. https://tldp.org/LDP/sag/html/

Warp Technologies Inc. (2024). *Warp Terminal Documentation*. https://docs.warp.dev/

Neovim contributors. (2024). *Neovim documentation*. https://neovim.io/doc/user/

Shotts, W. (2019). *The Linux Command Line* (2.a ed.). No Starch Press.

Canonical Ltd. (2024). *Ubuntu Server Guide – Command Line*. https://ubuntu.com/server/docs