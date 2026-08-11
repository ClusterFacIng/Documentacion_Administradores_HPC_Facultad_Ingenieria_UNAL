# Guía general del proyecto MkDocs

Este documento explica de forma sencilla cómo funciona este proyecto con MkDocs, cómo está organizado, cómo se configura y cómo se despliega.

La idea es que sirva como una vista general, sin depender de un entorno virtual ni de conocimientos técnicos avanzados.

La documentación oficial de MkDocs está disponible en [https://www.mkdocs.org/](https://www.mkdocs.org/).

## ¿Qué es MkDocs?

MkDocs es una herramienta que toma archivos en formato Markdown y los convierte en un sitio web estático.

En este proyecto, la documentación se escribe en archivos `.md` y MkDocs la transforma en páginas web navegables.

## ¿Cómo funciona este proyecto?

El proyecto sigue una estructura simple:

1. Se escriben los contenidos en Markdown.
2. MkDocs lee esos archivos.
3. MkDocs usa un archivo de configuración para saber qué páginas mostrar y en qué orden.
4. Finalmente genera un sitio web listo para abrir en el navegador o publicar en un servidor.

En resumen, el contenido vive en archivos Markdown y la presentación web la construye MkDocs.

## Organización del proyecto

La estructura actual del proyecto está pensada para que sea fácil de mantener:

- `mkdocs.yml`: archivo principal de configuración.
- `docs/`: carpeta donde vive el contenido que se publica.
- `docs/index.md`: portada principal del sitio.
- `docs/guia-windows.md`: guía para Windows.
- `docs/guia-linux.md`: guía para Linux.
- `docs/assets/`: imágenes y recursos usados por las páginas.
- `exports/`: archivos generados para otros formatos, como PDF y LaTeX.

## ¿Qué hace cada parte?

### `mkdocs.yml`

Este archivo le dice a MkDocs:

- cuál es el nombre del sitio,
- qué páginas debe mostrar,
- desde qué carpeta debe leer el contenido,
- y qué tema visual usar.

Es el punto central de configuración del sitio.

### Carpeta `docs/`

Aquí está el contenido que realmente se publica.

MkDocs no toma cualquier archivo del proyecto; normalmente publica lo que está dentro de esta carpeta.

### Archivo `index.md`

Es la página inicial del sitio.

Sirve como portada y ayuda a orientar al usuario antes de entrar a las guías específicas.

### Archivos `guia-windows.md` y `guia-linux.md`

Son las dos páginas principales de la documentación.

Cada una explica el proceso de acceso al HPC desde un sistema operativo distinto.

### Carpeta `assets/`

Aquí se guardan las imágenes que usan las guías.

Esto permite que las capturas no queden sueltas en la raíz del proyecto y que las rutas sean más ordenadas.

## ¿Cómo se configura?

La configuración principal está en `mkdocs.yml`.

Ahí se define, entre otras cosas:

- el nombre del sitio,
- la descripción,
- la carpeta de documentación,
- la navegación,
- y el tema visual.

La navegación actual está organizada en tres páginas principales:

- Inicio
- Windows
- Linux

Eso permite que el usuario vea una estructura clara desde el navegador.

## ¿Cómo se despliega?

Desplegar significa convertir los archivos Markdown en un sitio web visible.

Hay dos formas comunes de hacerlo:

### 1. Vista local

MkDocs puede levantar un servidor local para revisar la documentación mientras se edita.

Esto sirve para comprobar:

- que los enlaces funcionen,
- que las imágenes carguen bien,
- y que el contenido se vea correctamente.

### 2. Publicación final

Cuando todo está listo, MkDocs genera el sitio final como archivos HTML estáticos.

Esos archivos se pueden subir a un servidor web, a GitHub Pages o a cualquier otro servicio de alojamiento estático.

## ¿Qué se necesita para usarlo?

Para trabajar con MkDocs solo se necesita:

- tener Python instalado,
- tener MkDocs disponible,
- y ejecutar los comandos desde la carpeta del proyecto.

En este entorno se usó una instalación local para no modificar el sistema, pero eso es solo una forma de trabajo. El proyecto, como idea, no depende de ese detalle.

## Comandos básicos

### Instalar MkDocs con entorno virtual

Esta forma mantiene las dependencias aisladas dentro del proyecto:

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install mkdocs
```

### Instalar MkDocs directamente en el sistema

Si el entorno lo permite, también se puede instalar de forma directa:

```bash
python3 -m pip install --user mkdocs
```

### Desplegar en local

Para ver el sitio en el navegador mientras se trabaja:

```bash
mkdocs serve -f mkdocs.yml -a 127.0.0.1:8000
```

Si se está usando un entorno virtual, el comando equivalente es:

```bash
.venv/bin/mkdocs serve -f mkdocs.yml -a 127.0.0.1:8000
```

### Compilar el sitio

Para generar el sitio estático final en la carpeta `site/`:

```bash
mkdocs build -f mkdocs.yml
```

Si se está usando un entorno virtual, el comando equivalente es:

```bash
.venv/bin/mkdocs build -f mkdocs.yml
```

## Flujo resumido del proyecto

1. Escribir o editar los archivos `.md` dentro de `docs/`.
2. Ajustar `mkdocs.yml` si cambia la estructura o la navegación.
3. Revisar el sitio en local.
4. Generar la versión final.
5. Publicar la documentación.

## Ventajas de esta organización

- El contenido está separado por tema y sistema operativo.
- Las imágenes quedan agrupadas en un solo lugar.
- La navegación es clara para el usuario.
- MkDocs permite mantener la documentación sin necesidad de escribir HTML manualmente.

## Resumen final

Este proyecto funciona como un sitio de documentación hecho con Markdown y publicado con MkDocs.

La parte importante es esta:

- los textos viven en `.md`,
- la configuración vive en `mkdocs.yml`,
- las imágenes viven en `docs/assets/`,
- y el resultado final es un sitio web estático fácil de leer y de publicar.

