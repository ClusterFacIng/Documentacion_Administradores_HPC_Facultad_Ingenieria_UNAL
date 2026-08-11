# Automatización del proceso de solicitud de ingreso al HPC

## 1. Propósito del documento

Este documento describe la automatización implementada para gestionar las solicitudes de acceso al Clúster HPC.

Su objetivo es servir como guía técnica y operativa para los administradores actuales y futuros del sistema, permitiendo entender:

- Cómo está estructurado el proceso.
- Qué información almacena el formulario y la hoja de cálculo.
- Qué funciones de Google Apps Script intervienen.
- Cómo se gestionan los estados de una solicitud.
- Cómo se envían las notificaciones a Google Chat.
- Cómo se solicita el acceso VPN.
- Cómo se registra al usuario en el HPC.
- Cómo se confirma el acceso al usuario.
- Qué partes del proceso siguen siendo manuales.
- Qué aspectos deben tenerse en cuenta para realizar mantenimiento o modificaciones.

---

## 2. Arquitectura general

La automatización utiliza principalmente servicios de Google:

```text
Google Forms
     │
     ▼
Google Sheets
     │
     ▼
Google Apps Script
     │
     ├──────────────► Google Chat
     │
     ├──────────────► Correo Mesa de Ayuda
     │
     └──────────────► Correo Usuario
```

El proceso comienza cuando un usuario diligencia el formulario de solicitud.

La respuesta queda registrada automáticamente en Google Sheets. A partir de ese momento, Google Apps Script utiliza los cambios realizados en la hoja para ejecutar las diferentes etapas del proceso.

La hoja de cálculo funciona como el registro central de cada solicitud.

---

## 3. Flujo general de una solicitud

El proceso implementado utiliza cuatro estados principales:

1. `Nueva Solicitud`
2. `Solicitud VPN`
3. `Registro HPC`
4. `Acceso HPC`

El flujo general es:

```text
Formulario enviado
	│
	▼
Generar ID de solicitud
	│
	▼
Estado = Nueva Solicitud
	│
	▼
Notificación a Google Chat
	│
	▼
Administrador revisa solicitud
	│
	▼
Selecciona política VPN
	│
	▼
Estado = Solicitud VPN
	│
	▼
Correo a Mesa de Ayuda
	│
	▼
Estado = Registro HPC
	│
	▼
Notificación a administradores
	│
	▼
Administrador registra usuario en HPC
	│
	▼
Estado = Acceso HPC
	│
	▼
Correo de confirmación al usuario
	│
	▼
Notificación a Google Chat
```

No todas las etapas están automatizadas completamente. El registro efectivo del usuario dentro del HPC continúa requiriendo intervención del administrador.

---

## 4. Estructura de Google Sheets

La hoja de respuestas contiene inicialmente la información suministrada por el formulario y posteriormente las columnas administrativas.

La estructura actual es:

| Columna | Campo | Descripción |
|---|---|---|
| A | Marca temporal | Fecha y hora de envío del formulario |
| B | Nombres | Nombres del solicitante |
| C | Apellidos | Apellidos del solicitante |
| D | Correo electrónico institucional | Correo institucional |
| E | Vinculación a la Universidad | Tipo de vinculación |
| F | Nombre del proyecto de investigación o grupo | Proyecto o grupo asociado |
| G | Docente soportando solicitud uso clúster | Docente que respalda la solicitud |
| H | Justificación del uso del clúster | Justificación suministrada |
| I | Software o herramientas específicas requeridas | Software solicitado |
| J | Tiempo solicitado para cuenta | Tiempo solicitado en semestres |
| K | Espacio para carga de parte pública de llave RSA | Archivo de llave pública SSH |
| L | ESTADO | Estado actual de la solicitud |
| M | ID Solicitud | Identificador único |
| N | POLÍTICA VPN | Política VPN seleccionada |
| O | Fecha VPN | Fecha en que se procesó la solicitud VPN |
| P | Fecha Registro HPC | Fecha en que se procesó el registro HPC |
| Q | Usuario HPC | Nombre de usuario creado para el HPC |
| R | Fecha Acceso HPC | Fecha en que se confirmó el acceso |

### 4.1. Columnas administrativas

Las columnas `L` a `R` son utilizadas por la automatización.

Es importante no cambiar su posición sin actualizar las referencias de columna en Apps Script.

Actualmente:

```text
L = 12 → ESTADO
M = 13 → ID Solicitud
N = 14 → POLÍTICA VPN
O = 15 → Fecha VPN
P = 16 → Fecha Registro HPC
Q = 17 → Usuario HPC
R = 18 → Fecha Acceso HPC
```

Si se insertan o eliminan columnas antes de estas, las funciones deberán revisarse.

---

## 5. Estados de la solicitud

### 5.1. Nueva Solicitud

Es el estado inicial.

Se asigna automáticamente cuando el usuario envía el formulario.

Al generarse la solicitud:

1. Se obtiene la información del formulario.
2. Se genera un ID único.
3. Se establece el estado `Nueva Solicitud`.
4. Se envía una notificación al espacio de Google Chat de los administradores.

El administrador debe revisar la información antes de avanzar al siguiente estado.

---

### 5.2. Solicitud VPN

Representa una solicitud que ya fue revisada por un administrador y está lista para solicitar el acceso a la VPN.

Antes de cambiar el estado, el administrador debe seleccionar una de las políticas VPN disponibles:

```text
FAC_ING_CLUSTER_DUAL
FAC_ING_CLUSTER_PRINCIPAL
FAC_ING_CLUSTER_GNUM
FAC_ING_CLUSTER_ADMIN
```

Cuando el estado cambia a:

```text
Solicitud VPN
```

Apps Script:

1. Verifica que exista una política VPN válida.
2. Obtiene los datos del usuario.
3. Genera el correo para la Mesa de Ayuda.
4. Incluye la política VPN seleccionada.
5. Envía el correo.
6. Registra la fecha en `Fecha VPN`.
7. Envía una notificación al espacio de Google Chat.

#### Importante

Actualmente la política VPN **no se bloquea físicamente en Google Sheets**.

La columna `N` permanece editable.

Sin embargo, la automatización evita procesar nuevamente una solicitud cuando `Fecha VPN` ya tiene un valor:

```javascript
if (fechaVPN) return;
```

Esto evita que se envíen correos VPN duplicados por modificaciones posteriores.

La política utilizada queda registrada en:

- El correo enviado a la Mesa de Ayuda.
- El mensaje enviado a Google Chat.
- La columna `N` de la solicitud.

---

## 6. Políticas VPN

Las políticas actualmente soportadas son:

```text
FAC_ING_CLUSTER_DUAL
FAC_ING_CLUSTER_PRINCIPAL
FAC_ING_CLUSTER_GNUM
FAC_ING_CLUSTER_ADMIN
```

Estas políticas están definidas en el archivo de configuración.

Ejemplo:

```javascript
const POLITICA_VPN_DUAL =
  "FAC_ING_CLUSTER_DUAL";

const POLITICA_VPN_PRINCIPAL =
  "FAC_ING_CLUSTER_PRINCIPAL";

const POLITICA_VPN_GNUM =
  "FAC_ING_CLUSTER_GNUM";

const POLITICA_VPN_ADMIN =
  "FAC_ING_CLUSTER_ADMIN";
```

Para agregar, eliminar o cambiar una política, debe revisarse el archivo de configuración y la validación utilizada por `procesarSolicitudVPN()`.

---

## 7. Registro HPC

El estado:

```text
Registro HPC
```

indica que la solicitud VPN ya fue procesada y que el usuario está listo para ser registrado en el HPC.

En esta etapa el sistema genera el nombre de usuario HPC a partir del correo institucional.

Por ejemplo:

```text
Correo:
npiraquive@unal.edu.co

Usuario HPC:
npiraquive
```

La función utilizada para esto es:

```javascript
generarUsuarioHPC(correo)
```

El usuario generado se almacena en:

```text
Q = Usuario HPC
```

La fecha se almacena en:

```text
P = Fecha Registro HPC
```

### 7.1. Notificación a Google Chat

Cuando se procesa este estado se envía un mensaje a los administradores con información como:

- ID de solicitud.
- Nombre del usuario.
- Correo.
- Usuario HPC.
- Enlace directo a la fila de la solicitud.
- Acción sugerida para realizar el registro.

El mensaje permite al administrador localizar rápidamente la solicitud.

---

## 8. Registro manual en el HPC

Actualmente esta parte del proceso no está automatizada completamente.

El administrador debe conectarse al entorno del HPC y realizar el registro del usuario.

El procedimiento general es:

1. Crear el usuario.
2. Crear su directorio `home`, si corresponde al procedimiento configurado en el servidor.
3. Crear el directorio `.ssh`.
4. Configurar los permisos correspondientes.
5. Crear o modificar el archivo:

```text
authorized_keys
```

6. Copiar la llave pública SSH proporcionada por el usuario.
7. Verificar que el usuario pueda autenticarse mediante SSH.

El sistema automatizado genera el usuario y notifica al administrador, pero la ejecución de estos comandos sobre el servidor continúa siendo responsabilidad del administrador.

---

## 9. Llave pública SSH

El formulario permite al usuario proporcionar la parte pública de su llave SSH.

Actualmente la información puede quedar almacenada como un archivo en Google Drive asociado a la respuesta del formulario.

El archivo corresponde a la llave pública generada por el usuario, por ejemplo:

```text
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQ...
```

La automatización actual no utiliza directamente el contenido del archivo para crear el usuario en el HPC.

El administrador debe consultar la solicitud y utilizar la llave pública durante el registro del usuario.

### 9.1. Posible mejora futura

Una futura versión podría acceder automáticamente al archivo almacenado en Drive, leer su contenido y generar los comandos necesarios para configurar:

```text
~/.ssh/authorized_keys
```

Esto reduciría la intervención manual del administrador.

---

## 10. Acceso HPC

El estado:

```text
Acceso HPC
```

representa la etapa final.

Cuando el administrador cambia la solicitud a este estado, Apps Script:

1. Obtiene el usuario HPC.
2. Obtiene el correo institucional.
3. Envía un correo al usuario.
4. Informa que el acceso al HPC fue habilitado.
5. Incluye el nombre de usuario HPC.
6. Incluye el enlace a la documentación de conexión.
7. Registra la fecha en `Fecha Acceso HPC`.
8. Envía una notificación a Google Chat.

La fecha queda registrada en:

```text
R = Fecha Acceso HPC
```

---

## 11. Generación del ID de solicitud

Cada solicitud recibe un identificador único con el formato:

```text
HPC-AAAA-NNNN
```

Por ejemplo:

```text
HPC-2026-0001
HPC-2026-0002
HPC-2026-0003
```

El ID se almacena en:

```text
M = ID Solicitud
```

### 11.1. Consecutivo

El consecutivo se administra mediante:

```javascript
PropertiesService
```

utilizando una propiedad llamada:

```text
ULTIMO_ID
```

La generación utiliza un bloqueo:

```javascript
LockService.getScriptLock()
```

Esto evita que dos solicitudes procesadas simultáneamente obtengan el mismo número.

La función utilizada es:

```javascript
function generarIdSolicitud() {
	const lock = LockService.getScriptLock();

	lock.waitLock(30000);

	try {

		const props =
			PropertiesService.getScriptProperties();

		let contador =
			Number(props.getProperty("ULTIMO_ID")) || 0;

		contador++;

		props.setProperty(
			"ULTIMO_ID",
			contador.toString()
		);

		const anio =
			new Date().getFullYear();

		return `HPC-${anio}-${Utilities.formatString("%04d", contador)}`;

	} finally {

		lock.releaseLock();

	}
}
```

### 11.2. Consideración importante

El contador es independiente del número de filas de la hoja.

No se debe calcular el siguiente ID contando las filas existentes.

El valor persistente está en `PropertiesService`.

---

## 12. Google Chat

Google Chat se utiliza como mecanismo de notificación para los administradores.

Actualmente se utiliza un **webhook de Google Chat**, no una Google Chat App.

El webhook está almacenado en el archivo de configuración:

```javascript
const CHAT_WEBHOOK = "...";
```

### 12.1. Mensajes enviados

Dependiendo del estado, se pueden enviar notificaciones como:

#### Nueva solicitud

```text
🆕 NUEVA SOLICITUD

ID:
HPC-2026-0001

Usuario:
...

Correo:
...

Abrir solicitud:
...
```

#### Solicitud VPN

```text
📨 SOLICITUD VPN ENVIADA

ID:
HPC-2026-0001

Usuario:
...

Correo:
...

Política VPN:
FAC_ING_CLUSTER_PRINCIPAL

Abrir solicitud:
...
```

#### Registro HPC

```text
🖥️ REGISTRO HPC

ID:
HPC-2026-0001

Usuario HPC:
...

Abrir solicitud:
...
```

#### Acceso HPC

```text
✅ ACCESO HPC HABILITADO

ID:
HPC-2026-0001

Usuario HPC:
...

Abrir solicitud:
...
```

---

## 13. Enlaces directos a la solicitud

Las notificaciones incluyen un enlace directo a la fila correspondiente en Google Sheets.

El enlace se construye utilizando:

```javascript
const spreadsheetId =
	SpreadsheetApp
		.getActiveSpreadsheet()
		.getId();

const gid =
	sheet.getSheetId();
```

Y posteriormente:

```javascript
const linkFila =
	`https://docs.google.com/spreadsheets/d/${spreadsheetId}/edit#gid=${gid}&range=A${row}:R${row}`;
```

Esto permite que el administrador pueda pasar directamente desde Google Chat a la solicitud correspondiente.

---

## 14. Organización del proyecto Apps Script

El proyecto se encuentra dividido en archivos `.gs` para facilitar el mantenimiento.

Una estructura recomendada es:

```text
Apps Script
│
├── Codigo.gs
├── Estados.gs
└── Configuracion.gs
```

La división por archivos no modifica la forma en que Apps Script ejecuta el proyecto. Todos los archivos forman parte del mismo proyecto.

---

## 15. Archivo `Configuracion.gs`

Este archivo contiene valores que pueden cambiar sin necesidad de modificar la lógica principal.

Ejemplo:

```javascript
// ===== GOOGLE CHAT =====

const CHAT_WEBHOOK =
	"...";

// ===== CORREOS =====

const CORREO_MESA_AYUDA =
	"npiraquive@unal.edu.co";

// ===== DOCUMENTACIÓN =====

const URL_DOCUMENTACION_HPC =
	"...";

// ===== ESTADOS =====

const ESTADO_NUEVA_SOLICITUD =
	"Nueva Solicitud";

const ESTADO_SOLICITUD_VPN =
	"Solicitud VPN";

const ESTADO_REGISTRO_HPC =
	"Registro HPC";

const ESTADO_ACCESO_HPC =
	"Acceso HPC";

// ===== POLÍTICAS VPN =====

const POLITICA_VPN_DUAL =
	"FAC_ING_CLUSTER_DUAL";

const POLITICA_VPN_PRINCIPAL =
	"FAC_ING_CLUSTER_PRINCIPAL";

const POLITICA_VPN_GNUM =
	"FAC_ING_CLUSTER_GNUM";

const POLITICA_VPN_ADMIN =
	"FAC_ING_CLUSTER_ADMIN";
```

La idea es evitar valores repetidos dentro de las funciones.

Por ejemplo, es preferible:

```javascript
to: CORREO_MESA_AYUDA
```

que:

```javascript
to: "npiraquive@unal.edu.co"
```

---

## 16. `onFormSubmit(e)`

La función `onFormSubmit(e)` se ejecuta cuando llega una nueva respuesta del formulario.

Su responsabilidad principal es iniciar el proceso.

Conceptualmente realiza:

```text
Nueva respuesta
			↓
Obtener fila
			↓
Generar ID
			↓
Guardar ID
			↓
Establecer estado inicial
			↓
Enviar mensaje Google Chat
```

Esta función no debe utilizarse para procesar manualmente estados posteriores.

---

## 17. `onEdit(e)`

La función `onEdit(e)` es la encargada de detectar cambios realizados en la columna `ESTADO`.

La columna de estado actualmente es:

```text
L = 12
```

Conceptualmente:

```javascript
if (col !== 12) return;
```

Posteriormente identifica el nuevo estado:

```javascript
const nuevoEstado = e.value;
```

Y ejecuta la función correspondiente.

Por ejemplo:

```text
Solicitud VPN
			↓
procesarSolicitudVPN()

Registro HPC
			↓
procesarRegistroHPC()

Acceso HPC
			↓
procesarAccesoHPC()
```

Esto permite mantener separada la lógica de cada etapa.

---

## 18. Funciones principales

### `generarIdSolicitud()`

Genera el identificador único de cada solicitud.

Utiliza:

- `PropertiesService`
- `LockService`

### `generarUsuarioHPC(correo)`

Obtiene el nombre de usuario HPC a partir del correo institucional.

Ejemplo:

```text
npiraquive@unal.edu.co
					↓
npiraquive
```

### `procesarSolicitudVPN(sheet, row)`

Responsable del estado:

```text
Solicitud VPN
```

Realiza:

- Validación de política VPN.
- Generación del correo.
- Envío a Mesa de Ayuda.
- Registro de fecha VPN.
- Notificación a Google Chat.

### `procesarRegistroHPC(sheet, row)`

Responsable del estado:

```text
Registro HPC
```

Realiza:

- Generación del usuario HPC.
- Registro del usuario en la hoja.
- Registro de fecha.
- Notificación al administrador mediante Google Chat.

El registro efectivo dentro del servidor HPC continúa siendo manual.

### `procesarAccesoHPC(sheet, row)`

Responsable del estado:

```text
Acceso HPC
```

Realiza:

- Validación del usuario HPC.
- Envío del correo al usuario.
- Registro de fecha de acceso.
- Notificación a Google Chat.

---

## 19. Control de ejecuciones duplicadas

Cada etapa utiliza una columna de fecha como indicador de que ya fue procesada.

Por ejemplo, VPN:

```javascript
const fechaVPN =
	sheet.getRange(row, 15).getValue();

if (fechaVPN) return;
```

Registro HPC:

```javascript
const fechaRegistro =
	sheet.getRange(row, 16).getValue();

if (fechaRegistro) return;
```

Acceso HPC:

```javascript
const fechaAcceso =
	sheet.getRange(row, 18).getValue();

if (fechaAcceso) return;
```

Esto es importante porque un administrador puede modificar varias veces el estado de una solicitud.

Las fechas funcionan como una barrera para evitar:

- Correos duplicados.
- Creación repetida de usuarios.
- Notificaciones repetidas.

---

## 20. Manejo de errores

Los servicios externos utilizados por Apps Script pueden generar errores.

Entre ellos:

- `MailApp`
- `UrlFetchApp`
- Google Sheets
- Google Chat
- Google Drive

Cuando se utiliza `MailApp.sendEmail()`, un error de permisos puede detener la ejecución.

Por ejemplo, Apps Script puede solicitar el permiso:

```text
https://www.googleapis.com/auth/script.send_mail
```

Después de agregar nuevas funciones que utilizan servicios adicionales, puede ser necesario volver a autorizar el proyecto.

### Recomendación

Ante un error:

1. Abrir el proyecto de Apps Script.
2. Ir a **Ejecuciones**.
3. Revisar la ejecución correspondiente.
4. Identificar la función y línea que produjo el error.
5. Corregir el problema.
6. Repetir la prueba con una solicitud de prueba.

---

## 21. Pruebas

Para realizar pruebas se recomienda utilizar solicitudes ficticias.

No se deben realizar pruebas iniciales con solicitudes reales si pueden generar:

- Correos reales a Mesa de Ayuda.
- Creación de cuentas reales.

