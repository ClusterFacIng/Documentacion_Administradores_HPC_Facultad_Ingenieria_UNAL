# Administración de Software HPC con Lmod

## Objetivo

Este documento describe las estrategias recomendadas para instalar, publicar y mantener aplicaciones científicas dentro del HPC utilizando **Lmod** como sistema de gestión de módulos.

Está dirigido a administradores del cluster responsables de instalar y mantener el software disponible para los usuarios.

---

## 1. Introducción

En un entorno HPC, las aplicaciones no suelen instalarse directamente en los directorios personales de los usuarios.

En su lugar, los administradores instalan y mantienen versiones controladas del software, mientras que los usuarios acceden a ellas mediante comandos como:

```bash
module load <aplicacion>
```

Lmod permite:

- Gestionar múltiples versiones de una misma aplicación.
- Controlar dependencias entre aplicaciones.
- Configurar automáticamente variables de entorno.
- Mantener un entorno de trabajo reproducible.

---

## 2. Arquitectura General

El flujo general es:

```text
Administrador
	│
	▼
Instala software
	│
	▼
Configura Lmod
	│
	▼
Publica módulos
	│
	▼
Usuarios
	│
	▼
module load aplicacion
```

Dependiendo de la estrategia utilizada, la instalación del software puede realizarse manualmente o mediante herramientas especializadas.

---

## 3. Estrategias de Instalación

Existen tres enfoques principales para administrar software en un HPC:

1. Instalación Manual + Lmod
2. Spack + Lmod
3. EasyBuild + Lmod

---

### Método 1: Instalación Manual + Lmod

#### Descripción

Es el método más simple y directo.

El administrador instala manualmente cada aplicación y posteriormente crea un módulo Lmod para que los usuarios puedan utilizarla.

---

#### Arquitectura

```text
Administrador
	│
	▼
Instala software
	│
	▼
/apps/software
	│
	▼
Crea modulefile Lua
	│
	▼
Usuarios ejecutan:
module load
```

---

#### Ejemplo de Instalación

Crear directorio para OpenFOAM:

```bash
mkdir -p /apps/openfoam
cd /apps/openfoam
```

Instalar la aplicación:

```text
/apps/openfoam/2506
```

---

#### Crear un Modulefile

Crear estructura de módulos:

```bash
mkdir -p /apps/modulefiles/openfoam
```

Crear el archivo:

```bash
nano /apps/modulefiles/openfoam/2506.lua
```

Ejemplo:

```lua
help([[
OpenFOAM 2506
]])

whatis("OpenFOAM CFD Toolkit")

prepend_path("PATH",
"/apps/openfoam/2506/bin")

prepend_path("LD_LIBRARY_PATH",
"/apps/openfoam/2506/lib")

setenv("FOAM_VERSION","2506")
```

---

#### Registrar el Directorio de Módulos

Temporalmente:

```bash
module use /apps/modulefiles
```

Permanentemente:

```bash
echo 'module use /apps/modulefiles' \
>> /etc/profile.d/modules.sh
```

---

#### Validación

Verificar que Lmod detecta la aplicación:

```bash
module spider openfoam
```

Cargar el módulo:

```bash
module load openfoam/2506
```

Verificar la instalación:

```bash
which foamRun
```

---

#### Ventajas

- Fácil de comprender.
- Control total sobre la instalación.
- Ideal para clusters pequeños.

---

#### Desventajas

- Administración completamente manual.
- Dependencias difíciles de mantener.
- Actualizaciones más complejas.
- Escalabilidad limitada.

---

### Método 2: Spack + Lmod

#### Descripción

Actualmente es una de las soluciones más utilizadas en HPC modernos.

Spack automatiza:

- Descarga de software.
- Resolución de dependencias.
- Compilación.
- Gestión de versiones.
- Generación de módulos Lmod.

---

#### Arquitectura

```text
Administrador
	│
	▼
Spack
	│
	├── Instala software
	├── Resuelve dependencias
	└── Genera módulos Lmod
	│
	▼
Usuarios
```

---

#### Instalación de Spack

Clonar el repositorio:

```bash
git clone https://github.com/spack/spack.git
```

Ejemplo de ubicación:

```text
/opt/spack
```

Activar el entorno:

```bash
source /opt/spack/share/spack/setup-env.sh
```

---

#### Instalar Software

Ejemplo:

```bash
spack install openfoam
```

Spack instalará automáticamente todas las dependencias necesarias.

Por ejemplo:

```text
gcc
openmpi
hdf5
scotch
metis
openfoam
```

---

#### Generar Módulos Lmod

Actualizar módulos:

```bash
spack module lmod refresh -y
```

---

#### Ubicación Típica de los Módulos

```text
/opt/spack/share/spack/lmod
```

---

#### Registrar los Módulos

```bash
module use /opt/spack/share/spack/lmod
```

---

#### Uso por Parte del Usuario

Buscar:

```bash
module spider openfoam
```

Cargar:

```bash
module load openfoam
```

---

#### Instalar una Versión Específica

```bash
spack install openfoam@2506
```

---

#### Consultar Instalaciones

```bash
spack find
```

---

#### Consultar Dependencias

```bash
spack spec openfoam
```

---

#### Actualizar Módulos Después de Nuevas Instalaciones

```bash
spack module lmod refresh -y
```

---

#### Ventajas

- Gestión automática de dependencias.
- Soporte para múltiples compiladores.
- Soporte para múltiples implementaciones MPI.
- Escalable para grandes catálogos de software.
- Excelente integración con Lmod.

---

#### Desventajas

- Curva de aprendizaje inicial.
- Configuración más compleja.

---

### Método 3: EasyBuild + Lmod

#### Descripción

EasyBuild es una plataforma orientada específicamente a entornos científicos y académicos.

Utiliza archivos de configuración llamados **EasyConfig** para automatizar la instalación de aplicaciones.

---

#### Arquitectura

```text
EasyConfig
	│
	▼
EasyBuild
	│
	▼
Compila software
	│
	▼
Genera módulos
```

---

#### Instalación de EasyBuild

Ejemplo mediante Python:

```bash
pip install easybuild
```

También puede instalarse mediante paquetes de la distribución.

---

#### Buscar Software Disponible

```bash
eb -S OpenFOAM
```

---

#### Instalar una Aplicación

```bash
eb OpenFOAM-2506.eb
```

EasyBuild realizará automáticamente:

```text
Descarga
Compilación
Instalación
Generación de módulos
```

---

#### Ubicación Típica de los Módulos

```text
/apps/modules
```

o

```text
/easybuild/modules
```

---

#### Verificar Disponibilidad

```bash
module spider openfoam
```

---

#### Ventajas

- Instalaciones altamente reproducibles.
- Amplio catálogo científico.
- Menor esfuerzo administrativo.

---

#### Desventajas

- Menor flexibilidad que Spack.
- Comunidad más pequeña.
- Menor adopción en algunos entornos HPC modernos.

---

## Comparación de Métodos

| Característica | Manual + Lmod | Spack + Lmod | EasyBuild + Lmod |
|------------|------------|------------|------------|
| Facilidad inicial | Alta | Media | Media |
| Escalabilidad | Baja | Alta | Alta |
| Dependencias automáticas | No | Sí | Sí |
| Control total | Alto | Medio | Medio |
| Curva de aprendizaje | Baja | Media | Media |
| Mantenimiento | Alto | Bajo | Bajo |
| Recomendado para grandes catálogos | No | Sí | Sí |

---

