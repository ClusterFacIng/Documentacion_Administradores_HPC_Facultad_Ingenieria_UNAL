# Configuración y administración de Slurm en el HPC

## 1. Objetivo

Este documento describe la forma en que Slurm debe configurarse y operar en un entorno HPC para cumplir con estándares habituales de administración, control de recursos, ejecución de trabajos y trazabilidad.

Está orientado a administradores del cluster, no a usuarios finales. El enfoque principal es definir una base operativa clara para nodos, particiones, colas, contabilidad y políticas de uso.

---

## 2. Rol de Slurm en el HPC

Slurm es el planificador de trabajos del clúster. Su función principal es:

- asignar recursos a trabajos según prioridad, políticas y disponibilidad;
- controlar qué nodos pueden ejecutar cada trabajo;
- gestionar particiones y colas de ejecución;
- mantener registro de utilización y contabilidad;
- facilitar la administración del cluster desde una sola capa de programación.

En un entorno HPC típico, Slurm se usa como la capa central entre:

```text
Usuarios / aplicaciones
        ↓
Slurm
        ↓
Nodos de cómputo
```

La administración correcta de Slurm es crítica porque impacta directamente:

- la utilización del clúster;
- la equidad entre usuarios;
- el tiempo de respuesta de los trabajos;
- la trazabilidad y auditoría;
- la estabilidad del sistema.

---

## 3. Principios de diseño recomendados

Para un HPC estándar, es recomendable seguir estas directrices:

1. Separar nodos de cómputo y nodos de servicios.
2. Definir particiones por tipo de trabajo y duración.
3. Usar memoria, CPUs y nodos con valores explícitos y consistentes.
4. Mantener contabilidad activa para auditoría y análisis.
5. Aplicar políticas de prioridad y límites por usuario o grupo.
6. Evitar sobreasignación de recursos.
7. Asegurar que cada trabajo tenga un tiempo límite y una partición adecuada.

---

## 4. Estructura de nodos recomendada

Un clúster HPC normalmente se compone de:

- nodos de control o login;
- nodos de cómputo;
- nodos de gestión o almacenamiento;
- nodos de servicios auxiliares.

En una arquitectura típica, Slurm se despliega con:

```text
ControlMachine: nodo principal de gestión
Compute Nodes: nodos de cómputo
Slurmctld: daemon de control
Slurmd: daemon en cada nodo de cómputo
```

Un ejemplo habitual de inventario es:

```text
cnode01
cnode02
cnode03
cnode04
```

Cada nodo debe tener una definición coherente en el archivo de configuración, incluyendo:

- nombre del nodo;
- número de CPUs;
- memoria total;
- particiones permitidas;
- estado operativo;
- recursos especializados, si aplica.

---

## 5. Configuración de particiones

Las particiones son uno de los elementos clave para un uso ordenado del clúster. Deben definirse de acuerdo con el tipo de trabajo, duración y prioridad.

Un esquema estándar es:

| Partición | Tiempo máximo | Propósito | Uso recomendado |
|---|---:|---|---|
| hpc-short | 1 hora | pruebas y trabajos cortos | validación rápida |
| hpc | 24 horas | trabajo general | producción normal |
| hpc-long | 72 horas | trabajos largos | simulaciones prolongadas |

Una definición típica en Slurm es:

```text
PartitionName=hpc-short Nodes=cnode[01-04] Default=YES MaxTime=01:00:00 State=UP
PartitionName=hpc Nodes=cnode[01-04] MaxTime=24:00:00 State=UP
PartitionName=hpc-long Nodes=cnode[01-04] MaxTime=72:00:00 State=UP
```

Recomendaciones:

- definir una partición default para trabajos estándar;
- reservar particiones específicas para trabajos largos;
- evitar que todos los usuarios envíen trabajos a la misma cola sin límites;
- mantener una política de tiempos clara y consistente.

---

## 6. Configuración básica de Slurm

El archivo principal suele ser slurm.conf. En él se definen parámetros globales del sistema. Algunos de los más relevantes son:

```text
ClusterName=HPCFACULTAD
ControlMachine=slurm-head
SlurmctldPort=6817
SlurmdPort=6818
AccountingStorageType=accounting_storage/slurmdbd
AccountingStorageHost=slurm-db
JobAcctGatherType=jobacct_gather/linux
TaskPlugin=task/affinity
SelectType=select/cons_res
SelectTypeParameters=CR_CPU_Memory
```

### Parámetros clave

- ControlMachine: nodo que ejecuta el daemon de control.
- AccountingStorageType: habilita la contabilidad de trabajos.
- JobAcctGatherType: permite recolectar métricas del trabajo.
- SelectType: define cómo Slurm selecciona recursos.
- TaskPlugin: ayuda con la afinidad y el uso de CPUs.

---

## 7. Configuración de nodos

Cada nodo debe describirse con la información física y lógica necesaria para planificar trabajos. Un ejemplo estándar es:

```text
NodeName=cnode[01-04] CPUs=96 Sockets=2 CoresPerSocket=24 ThreadsPerCore=2 RealMemory=250000 State=UNKNOWN
```

Esto implica que el nodo tiene:

- 96 CPUs lógicas;
- 2 sockets;
- 24 núcleos por socket;
- 2 hilos por núcleo;
- 250 GB de memoria aproximadamente.

Desde el punto de vista administrativo, los nodos deben tener una política clara de:

- recursos disponibles;
- estado operativo;
- exclusividad o compartición;
- prevención de contención entre trabajos.

---

## 8. Políticas de recursos y exclusividad

En HPC, es habitual configurar la partición para que cada trabajo tenga acceso exclusivo a los recursos asignados. Una política común es:

```text
Oversubscribe=NO
```

o bien en configuraciones más estrictas:

```text
Oversubscribe=EXCLUSIVE
```

Esto evita que dos trabajos comparten el mismo recurso del mismo nodo y reduce problemas de rendimiento y estabilidad.

La recomendación general es:

- establecer exclusividad para particiones de cómputo intensivo;
- evitar sobreasignar tareas en los nodos de cómputo;
- documentar cualquier excepción explícitamente.

---

## 9. Contabilidad y auditoría

La contabilidad es una parte esencial de la administración de Slurm. Permite:

- revisar qué trabajos corrieron;
- calcular consumo de CPU, memoria y tiempo;
- detectar uso indebido o abusivo;
- justificar la utilización del clúster ante usuarios o líderes de proyecto;
- generar reportes de mantenimiento y planificación.

Los comandos más usados para administración son:

```bash
sacct
sacct -X
sacct -j 12345
scontrol show job 12345
sinfo
scontrol show partition
```

La contabilidad debería estar activa para:

- trabajos completados;
- trabajos fallidos;
- trabajos cancelados;
- métricas de recursos usados.

---

## 10. Administración de prioridades y cuotas

Las políticas de prioridad se pueden definir por:

- usuario;
- grupo;
- partición;
- tipo de trabajo;
- prioridad histórica;
- uso de colas premium o de investigación.

Lo habitual es implementar una política simple, clara y transparente:

- trabajos cortos tienen prioridad inmediata si son necesarios para pruebas;
- trabajos largos se encolan con tiempos de espera razonables;
- trabajos críticos del centro o de investigación se gestionan con prioridad explícita.

En la práctica, un administrador debe revisar:

- cuánto tiempo está esperando cada trabajo;
- si hay usuarios monopolizando recursos;
- si una partición está saturada;
- si existen trabajos de larga duración acumulando prioridad.

---

## 11. Reservas y colas especiales

Los administradores suelen crear reservas para:

- mantenimiento del clúster;
- nodos especiales para investigación o cursos;
- pruebas de software antes de publicarlo;
- trabajos de sistema o soporte.

Ejemplo de reserva:

```text
ReservationName=maintenance StartTime=2026-09-01T00:00:00 EndTime=2026-09-01T04:00:00 Nodes=cnode[01-04] Flags=MAINT
```

Esto permite aislar recursos para tareas operativas sin afectar el trabajo normal de los usuarios.

---

## 12. Monitoreo operativo

Un administrador debe revisar periódicamente:

- estado de nodos en sinfo;
- trabajos activos con squeue;
- trabajos terminados con sacct;
- ocupación por partición;
- errores de nodos o de ejecución.

Los comandos principales son:

```bash
sinfo -N
sinfo -l
squeue
squeue -l
scontrol show node cnode01
scontrol show partition hpc
```

También es recomendable registrar:

- nodos caídos;
- particiones saturadas;
- trabajos repetidos con errores;
- tiempos de espera promedio;
- historial de mantenimiento.

---

## 13. Buenas prácticas operativas

### 13.1. Mantener particiones claras

Cada partición debe responder a una necesidad específica. No es recomendable crear demasiadas colas sin criterio claro.

### 13.2. Definir límites razonables

Los tiempos máximos y la memoria por trabajo deben ser proporcionales a los recursos reales del clúster.

### 13.3. Controlar el uso del cluster

Los trabajos que consuman demasiados recursos deben estar sujetos a políticas de revisión.

### 13.4. Registrar cambios de configuración

Cada cambio en Slurm debe documentarse para evitar degradación del servicio. Es útil mantener un historial de:

- cambios de particiones;
- ajustes de prioridad;
- nuevas reservas;
- cambios de nodos o memoria;
- actualizaciones de configuración.

### 13.5. Validar después de cambios

Cuando se actualiza slurm.conf o se altera la topología del cluster, se debe validar la configuración antes de recargar el servicio.

```bash
scontrol reconfigure
```

y revisar el estado general con:

```bash
sinfo
squeue
```

---

## 14. Validación de la configuración

Antes de poner una nueva configuración en producción, el administrador debe verificar lo siguiente:

- que el archivo de configuración no tiene errores sintácticos;
- que los nodos están definidos con los recursos correctos;
- que las particiones tienen tiempos sanos;
- que los nodos de cómputo aparecen correctamente en sinfo;
- que la contabilidad está funcionando;
- que la prioridad y los límites no bloquean trabajos válidos.

Comandos recomendados:

```bash
sacctmgr list cluster
sinfo -N
scontrol show partition
scontrol show node
```

---

## 15. Procedimiento operativo recomendado

Un administrador de HPC debería seguir este flujo para mantener Slurm saludable:

1. Revisar el estado del clúster.
2. Verificar nodos y particiones.
3. Confirmar carga de trabajo y tiempos de espera.
4. Revisar la contabilidad y posibles anomalías.
5. Ajustar políticas o reservas si es necesario.
6. Validar con sinfo y squeue.
7. Documentar cualquier cambio realizado.

---

## 16. Señales de alerta para el administrador

Debe prestarse atención a estas situaciones:

- partición completamente saturada;
- nodos marcados como down;
- trabajos pendientes durante tiempos excesivos;
- consumo de memoria muy elevado en nodos compartidos;
- trabajos que no terminan por errores recurrentes;
- fallas de contabilidad o de persistencia de trabajos.

---

## 17. Recomendación para el HPC de la Facultad

Para este entorno, el modelo más saneado es:

- definir particiones por duración y uso;
- mantener nodos homogéneos y bien documentados;
- activar contabilidad desde el inicio;
- usar políticas claras de prioridad y recursos;
- reservar capacidad para trabajo de soporte y validación;
- documentar cualquier modificación en la topología o en la configuración.

Un esquema recomendado para el clúster es:

```text
Nodos de cómputo: cnode[01-04]
Particiones: hpc-short, hpc, hpc-long
Contabilidad: activada
Prioridad: equilibrio por usuario y partición
Reservas: mantenimiento y pruebas
```

---

## 18. Resumen

Slurm debe configurarse como la base operacional del clúster. No es solo un sistema para enviar trabajos, sino la capa central de administración del recurso computacional.

Desde el punto de vista del administrador, lo esencial es:

- mantener una configuración coherente;
- definir particiones con sentido;
- controlar recursos y memoria;
- activar y revisar contabilidad;
- monitorear nodos y trabajos;
- documentar cambios y mantener políticas claras.

Una buena administración de Slurm permite un clúster más estable, más justo y más predecible para toda la comunidad de usuarios.

## 16. Ejemplo de trabajo paralelo

```bash
#!/bin/bash

#SBATCH --job-name=mpi_test
#SBATCH --partition=hpc
#SBATCH --time=02:00:00
#SBATCH --ntasks=8
#SBATCH --mem=8G

module purge
module load gnu14
module load openmpi5

module list

srun ./mi_programa
```

## 17. Ejemplo de trabajo multihilo

```bash
#!/bin/bash

#SBATCH --job-name=multihilo
#SBATCH --partition=hpc-short
#SBATCH --time=00:30:00
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=8
#SBATCH --mem=8G

module purge
module load <aplicacion>/<version>

export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK

srun ./mi_programa
```

## 18. Enviar un trabajo

```bash
sbatch trabajo.slurm
```

Slurm devolverá un identificador:

```text
Submitted batch job 12345
```

## 19. Consultar trabajos

```bash
squeue -u $USER
```

Trabajo específico:

```bash
squeue -j <jobid>
```

## 20. Estados comunes

| Estado | Significado |
|---|---|
| `PD` | Pendiente |
| `R` | Ejecutándose |
| `CG` | Finalizando |
| `CD` | Completado |
| `F` | Fallido |
| `CA` | Cancelado |

## 21. Cancelar un trabajo

```bash
scancel <jobid>
```

Ejemplo:

```bash
scancel 12345
```

Para cancelar todos los trabajos propios:

```bash
scancel -u $USER
```

Utilice esta opción con cuidado.

## 22. Consultar las particiones

```bash
sinfo
```

Vista resumida:

```bash
sinfo -o "%P %a %l %D %C"
```

## 23. Consultar información de un trabajo

```bash
scontrol show job <jobid>
```

## 24. Consultar trabajos terminados

```bash
sacct
```

Trabajo específico:

```bash
sacct -j <jobid>
```

También:

```bash
sacct -j <jobid> --format=JobID,JobName,Partition,State,Elapsed,AllocCPUS,MaxRSS
```

## 25. Revisar el uso de recursos

Después de ejecutar un trabajo:

```bash
sacct -j <jobid> --format=JobID,State,Elapsed,AllocCPUS,MaxRSS
```

Esto ayuda a comprobar si los recursos solicitados fueron adecuados.

## 26. Variables útiles de Slurm

| Variable | Información |
|---|---|
| `$SLURM_JOB_ID` | Identificador del trabajo |
| `$SLURM_JOB_NAME` | Nombre del trabajo |
| `$SLURM_JOB_NODELIST` | Nodos asignados |
| `$SLURM_NTASKS` | Número de tareas |
| `$SLURM_CPUS_PER_TASK` | CPU por tarea |
| `$SLURM_JOB_PARTITION` | Partición utilizada |

Ejemplo:

```bash
echo "Job ID: $SLURM_JOB_ID"
echo "Partición: $SLURM_JOB_PARTITION"
echo "Tareas: $SLURM_NTASKS"
```

## 27. Buenas prácticas

- Solicitar únicamente los recursos necesarios.
- Elegir la partición según la duración real del trabajo.
- Solicitar un tiempo razonable.
- Cargar los módulos dentro del script.
- Utilizar `srun` para ejecutar el programa con los recursos asignados.
- Revisar el consumo con `sacct`.
- No seleccionar manualmente un nodo para trabajos normales.

## 28. Flujo recomendado

```text
Preparar script
      ↓
Solicitar recursos
      ↓
Cargar módulos
      ↓
Enviar con sbatch
      ↓
Consultar con squeue
      ↓
Ejecutar
      ↓
Revisar resultados
      ↓
Consultar consumo con sacct
```

## 29. Ejemplo completo

```bash
#!/bin/bash

#SBATCH --job-name=simulacion
#SBATCH --partition=hpc
#SBATCH --time=04:00:00
#SBATCH --ntasks=8
#SBATCH --mem=16G
#SBATCH --output=simulacion_%j.out
#SBATCH --error=simulacion_%j.err

module purge
module load gnu14
module load openmpi5

echo "Job ID: $SLURM_JOB_ID"
echo "Partición: $SLURM_JOB_PARTITION"
echo "Tareas: $SLURM_NTASKS"

module list

cd ~/proyecto

srun ./mi_programa
```

Enviar:

```bash
sbatch simulacion.slurm
```

Consultar:

```bash
squeue -u $USER
```

Cuando termine:

```bash
sacct -j <jobid>
```

## 30. Referencia rápida

### Enviar

```bash
sbatch trabajo.slurm
```

### Consultar

```bash
squeue -u $USER
```

### Ver información

```bash
scontrol show job <jobid>
```

### Cancelar

```bash
scancel <jobid>
```

### Revisar un trabajo terminado

```bash
sacct -j <jobid>
```

### Ver particiones

```bash
sinfo
```

## 31. Importante

- Los trabajos de cómputo deben enviarse mediante Slurm.
- No se debe seleccionar manualmente un nodo para un trabajo normal.
- Se debe elegir la partición de acuerdo con la duración del trabajo.
- No se deben solicitar más CPU, memoria o tiempo del necesario.
- Los módulos deben cargarse dentro del script.
- La aplicación debe ejecutarse utilizando los recursos solicitados.
