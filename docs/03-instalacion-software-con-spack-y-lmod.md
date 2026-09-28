# Tutorial de instalación de software con Spack y Lmod

## 1. Objetivo

Este tutorial describe el flujo operativo para instalar software con Spack y dejarlo disponible para los usuarios mediante módulos Lmod.

El objetivo es que el administrador pueda repetir el proceso de forma segura, consistente y con permisos adecuados para un entorno compartido.

## 2. Panorama general

El flujo recomendado es el siguiente:

```text
Administrador
  │
  ▼
module load spack
  │
  ▼
umask 022
  │
  ▼
spack spec -Il paquete
  │
  ▼
spack install paquete
  │
  ▼
Spack genera el módulo Lmod
  │
  ▼
module load paquete/version-hash
```

En este entorno, los módulos generados por Spack se publican para Lmod en el árbol compartido del clúster, por lo que los usuarios no necesitan reconstruir nada para usar el software.

## 3. Requisitos previos

Antes de instalar un paquete, verifique que Spack esté disponible:

```bash
module load spack
spack --version
```

En la configuración actual del clúster se utiliza:

```text
Spack 0.23.1
Lmod 8.7.59
gcc@14.2.0
```

## 4. Usar `umask 022`

Antes de instalar software compartido, establezca:

```bash
umask 022
```

Esto evita que los archivos y directorios se creen con permisos demasiado restrictivos para otros usuarios del clúster.

Con esta configuración, los permisos normales para un árbol compartido suelen quedar en:

- directorios: `755`
- archivos: `644`

¿Por qué?

`umask` determina los permisos iniciales con los que se crean archivos y directorios.

En este sistema existe una política global que establece:

```text
umask 0077
```

Este valor es apropiado para ciertos archivos personales porque restringe el acceso a otros usuarios.

Sin embargo, **no es adecuado para un árbol de software compartido mediante Lmod**.

Por ejemplo, con:

```text
umask 0077
```

un directorio creado con usuario root normalmente termina con permisos:

```text
700
```

y un archivo termina con permisos:

```text
600
```

Esto significa que solamente el propietario puede acceder a ellos.

Para software compartido, los módulos Lmod deben ser legibles y atravesables por los usuarios. Por ello, para las instalaciones manuales de software compartido utilizamos:

```bash
umask 022
```

Con `umask 022`, los permisos habituales resultantes son:

```text
directorios: 755
archivos:    644
```

Esto permite:

- al propietario: leer, escribir y ejecutar;
- a otros usuarios: leer y, en el caso de directorios, atravesarlos.

**Importante:** este procedimiento no requiere cambiar la política global del sistema. No se debe modificar `/etc/profile.d/systemWide_umask.sh` solamente para instalar software con Spack.

El `umask 022` se establece explícitamente en la sesión administrativa que realiza la instalación.

## 5. Buscar el paquete

Primero confirme que Spack conoce el paquete:

```bash
spack list cowsay
spack list <paquete>
```

Si el paquete ya existe en el árbol de Spack, puede revisarlo con:

```bash
spack find cowsay
```

## 6. Revisar la especificación

Antes de instalar, conviene ver qué versión, compilador y dependencias resolverá Spack:

```bash
spack spec -Il cowsay
spack spec -Il <paquete>
```

Esto ayuda a confirmar que la instalación se hará con el compilador y las variantes correctas.

## 7. Instalar el software

Con el `umask` ya configurado, instale el paquete:

```bash
spack install cowsay
spack install <paquete>
```

Spack descarga el código fuente, resuelve dependencias, compila el software y lo instala en el árbol compartido del clúster.

## 8. Verificar la instalación

Después de instalar, confirme que el paquete quedó registrado:

```bash
spack find cowsay
spack find -p cowsay
```

En este entorno, las instalaciones quedan bajo:

```text
/opt/ohpc/pub/apps/spack/local
```

## 9. Verificar el módulo Lmod

Spack genera el módulo Lmod automáticamente. Para comprobarlo:

```bash
module avail cowsay
module spider cowsay
```

El nombre del módulo sigue el patrón:

```text
nombre/version-hash
```

Por ejemplo:

```text
cowsay/3.04-iz6qek4
```

## 10. Cargar y probar el software

Cargue el módulo generado por Spack:

```bash
module load cowsay/3.04-iz6qek4
```

Luego verifique el ejecutable:

```bash
which cowsay
cowsay "Spack + Lmod funcionan"
```

Si el comando responde correctamente, el paquete quedó disponible para los usuarios del clúster.

## 11. Probar desde un entorno limpio

Para descartar dependencias accidentales de la sesión actual, pruebe desde un entorno limpio y desde un usuario no root:

```bash
module purge
module load cowsay/3.04-iz6qek4
which cowsay
cowsay "Prueba desde entorno limpio"
```

## 12. Resumen operativo

```bash
module load spack
umask 022

spack list <paquete>
spack spec -Il <paquete>
spack install <paquete>

spack find <paquete>
module avail <paquete>
module load <paquete>/<version>-<hash>
which <ejecutable>
```

Si la salida es correcta, el software queda listo para uso compartido mediante Lmod.