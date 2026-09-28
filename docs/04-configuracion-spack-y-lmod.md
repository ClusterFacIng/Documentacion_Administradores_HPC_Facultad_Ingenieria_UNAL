# Guía de administración de la configuración Spack y Lmod

## 1. Propósito

Este documento describe la configuración actual de Spack y Lmod en el clúster, las decisiones adoptadas y las consideraciones que deben tenerse en cuenta para futuras instalaciones y mantenimientos.

La intención es que funcione como referencia técnica para administradores del HPC y no como un tutorial básico de uso.

## 2. Arquitectura general

La infraestructura se compone de tres elementos principales:

1. Spack, responsable de instalar y gestionar el software.
2. Lmod, responsable de exponer los módulos a los usuarios.
3. `MODULEPATH`, que define dónde busca Lmod los módulos disponibles.

El flujo general es:

```text
Spack
  │
  ├── Instala software
  │
  └── Genera módulos Lua para Lmod
          │
          ▼
/opt/ohpc/pub/moduledeps/spack/Core
          │
          ▼
        Lmod
          │
          ▼
      Usuarios
```

## 3. Versiones y ubicaciones

La configuración actual usa:

| Componente | Versión |
|---|---|
| Spack | 0.23.1 |
| Lmod | 8.7.59 |
| GCC principal | 14.2.0 |

Ubicaciones relevantes:

```text
/opt/ohpc/pub/apps/spack/0.23.1
/opt/ohpc/pub/apps/spack/0.23.1/bin/spack
/opt/ohpc/pub/apps/spack/local
/opt/ohpc/pub/moduledeps/spack
/opt/ohpc/pub/moduledeps/spack/Core
```

## 4. Configuración de módulos de Spack

La configuración principal vive en:

```text
/root/.spack/modules.yaml
```

La base de la configuración actual es:

```yaml
modules:
  default:
    roots:
      lmod: /opt/ohpc/pub/moduledeps/spack

    enable:
    - lmod

    arch_folder: false

    lmod:
      hide_implicits: true

      projections:
        all: '{name}/{version}'

      all:
        autoload: direct

      hierarchy:
      - mpi

      core_compilers:
      - gcc@=14.2.0
```

### Decisiones principales

- Solo se generan módulos Lmod.
- No se generan módulos Tcl.
- Los nombres de módulos se simplifican a `nombre/version`.
- Se conserva el hash de Spack para diferenciar instalaciones.
- Las dependencias directas se cargan automáticamente.
- Las dependencias implícitas se ocultan de `module avail`.
- Se mantiene la jerarquía MPI para compatibilidad futura.

## 5. Configuración de `MODULEPATH`

La ruta global expuesta por Lmod incluye:

```text
/opt/ohpc/pub/moduledeps/spack/Core
```

Esta ruta está declarada en:

```text
/opt/ohpc/admin/lmod/8.7.59/init/.modulespath
/opt/ohpc/admin/lmod/lmod/init/.modulespath
```

El punto clave es que la ruta de Spack/Core ya forma parte del `MODULEPATH` global, así que los usuarios no necesitan ejecutar `module use` manualmente.

## 6. Instalación de nuevo software

El flujo administrativo recomendado es:

```bash
module load spack
umask 022

spack list <paquete>
spack spec -Il <paquete>
spack install <paquete>
spack find <paquete>
module avail <paquete>
module load <paquete>/<version>-<hash>
```

Si la instalación modifica los módulos disponibles, puede regenerarse el árbol con:

```bash
spack module lmod refresh -y --delete-tree
```

## 7. Permisos recomendados

Los módulos son compartidos por todos los usuarios del clúster, así que conviene crear el árbol con permisos de lectura y ejecución para otros usuarios.

Antes de instalar o regenerar módulos, use:

```bash
umask 022
```

Esto ayuda a evitar problemas cuando el árbol se crea bajo una política de umask más restrictiva.

## 8. Verificaciones rápidas

Comandos útiles para validar la configuración:

```bash
spack config blame modules
module avail
module spider <paquete>
spack find
```

Si se detectan problemas de visibilidad, también conviene revisar los permisos del árbol de módulos:

```bash
find /opt/ohpc/pub/moduledeps/spack -type d ! -perm 755 -print
find /opt/ohpc/pub/moduledeps/spack -type f ! -perm 644 -print
```

## 9. Estado actual

La configuración actual permite que cualquier usuario cargue directamente los paquetes instalados por Spack mediante:

```bash
module load <paquete>/<versión-hash>
```

sin necesidad de cargar Spack para consumir el software ya publicado.

Las dependencias son gestionadas automáticamente por Lmod y los módulos se descubren a través de:

```text
/opt/ohpc/pub/moduledeps/spack/Core
```

que forma parte permanente del `MODULEPATH` global del sistema.