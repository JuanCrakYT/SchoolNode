# SchoolNode

**SchoolNode** es una plataforma de infraestructura para salas de informática que permite administrar los computadores del aula, facilitar la comunicación entre profesores y estudiantes, y aprovechar los recursos disponibles de los equipos como infraestructura distribuida.

El proyecto está diseñado para funcionar como una plataforma distribuida: cada computador ejecuta un nodo de `SchoolNode` y puede participar tanto en las funciones del aula como en las funciones del cluster.

---

## Objetivos

SchoolNode tiene cuatro responsabilidades principales:

1. **Controlar el aula**
2. **Compartir recursos y servicios entre computadores**
3. **Mantener la información sincronizada y recuperable**
4. **Supervisar el estado de los computadores**

La plataforma debe seguir funcionando aunque algunos computadores estén apagados, se desconecten o se reinicien.

---

## Arquitectura general

```text
SchoolNode
│
├── Classroom/
│   └── Funciones relacionadas con el aula
│
├── Cluster/
│   └── Infraestructura distribuida
│
├── Sync/
│   └── Sincronización y recuperación
│
└── Status/
    └── Estado y recursos del computador
```

Cada computador ejecuta su propio nodo:

```text
                    SchoolNode
                        │
          ┌─────────────┴─────────────┐
          │                           │
      Classroom                    Cluster
          │                           │
   Funciones del aula        Trabajo distribuido
```

Los nodos pueden comunicarse entre sí para compartir información, ejecutar trabajos y mantener el estado de la infraestructura.

---

# Funciones

SchoolNode está dividido actualmente en **17 áreas funcionales principales**.

## Classroom

Funciones directamente relacionadas con la administración del aula.

### RemoteControl

Permite ejecutar acciones sobre computadores remotos.

Puede incluir acciones como:

* Ejecución remota de comandos
* Administración de procesos
* Acciones sobre uno o varios computadores

### Screen

Permite visualizar las pantallas de los computadores desde la interfaz del profesor.

### Lock

Permite bloquear computadores cuando sea necesario.

### ScheduledActions

Permite ejecutar acciones automáticamente mediante:

* Horarios
* Fechas
* Calendarios de clases
* Acciones programadas

### TeacherCommunication

Permite enviar información e instrucciones desde el profesor hacia los estudiantes.

Los mensajes pueden utilizar diferentes templates dependiendo del tipo de comunicación.

---

# Cluster

El cluster permite aprovechar los computadores disponibles como infraestructura compartida.

Su funcionamiento debe ser principalmente automático y transparente para los usuarios.

```text
Cluster
│
├── Background
├── Teacher
├── TeacherUI
├── StudentsPrimary
└── Students
```

## Background

Motor principal del cluster.

Se encarga de tareas como:

* Administrar workers
* Distribuir trabajos
* Supervisar recursos
* Controlar la carga
* Mantener el funcionamiento del cluster en segundo plano

### Límite de recursos

El cluster debe evitar interferir significativamente con el uso normal del computador.

Como regla inicial, una tarea del cluster no debe provocar que el uso de CPU supere aproximadamente el **40 %** de capacidad utilizada por el cluster.

El sistema debe reducir o detener automáticamente la carga cuando se alcance el límite y reanudarla cuando haya recursos suficientes.

El objetivo es que el computador siga siendo principalmente un equipo de trabajo para el aula.

---

## Teacher

Funciones relacionadas con las capacidades del profesor que utilizan la infraestructura del cluster.

El profesor no necesita administrar manualmente los workers ni conocer cómo se distribuye internamente el trabajo.

---

## TeacherUI

Interfaz utilizada por el profesor para acceder a SchoolNode.

Debe poder utilizarse desde:

* Computadores
* Teléfonos
* Tablets

La interfaz está orientada a las funciones del aula y no necesita exponer los detalles internos del cluster.

### Diseño general

```text
SchoolNode // <PageName>

┌──────────┬───────────────────────────────────────────────┐
│  [Logo]  │ [<Page>] [<Page>] [<Page>] [<Page>] [ ... ] │
├──────────┼───────────────────────────────────────────────┤
│ [PGF]    │                                               │
│ [PGF]    │                                               │
│ [PGF]    │                 Page Content                   │
│ [PGF]    │                                               │
│ [PGF]    │                                               │
│ [PGF]    │                                               │
│  ...     │                                               │
└──────────┴───────────────────────────────────────────────┘
```

La interfaz se divide en:

* **Barra superior:** navegación entre páginas.
* **Panel lateral:** funciones rápidas de la página actual.
* **Área principal:** contenido de la página.
* **Título:** `SchoolNode // <PageName>`.

La barra superior debe poder adaptarse a diferentes tamaños de pantalla.

---

## StudentsPrimary

Contiene servicios compartidos destinados a proyectos de estudiantes o proyectos generales de la infraestructura.

Esta no es una función primordial de SchoolNode. La plataforma debe seguir siendo útil aunque esta parte no esté disponible.

Puede utilizarse para proyectos como:

```text
StudentsPrimary
│
├── Game Servers
├── Shared Projects
├── Compute Jobs
└── AI Services
```

### Servicios de IA distribuidos

Una posible utilización del cluster es distribuir trabajos relacionados con inteligencia artificial entre varios computadores.

Por ejemplo:

```text
              AI Service
                  │
          ┌───────┴───────┐
          ▼               ▼
        Node 1           Node 2
      procesamiento    procesamiento
          │               │
          └───────┬───────┘
                  ▼
             Coordinator
```

Esto no implica que todos los computadores deban funcionar como una única máquina con memoria y GPU compartidas.

La distribución puede realizarse mediante diferentes tipos de trabajos, como:

* Inferencia
* Procesamiento por lotes
* Embeddings
* Procesamiento de imágenes
* Otros trabajos distribuibles

---

## Students

Contiene las funciones destinadas al panel del estudiante y al acceso autorizado a archivos o tareas.

### Panel del estudiante

El panel puede utilizar navegación mediante teclado numérico:

```text
Numpad 7   Numpad 8   Numpad 9

Numpad 4   Numpad 5   Numpad 6

Numpad 1   Numpad 2   Numpad 3
```

Controles principales:

| Tecla                | Acción                       |
| -------------------- | ---------------------------- |
| `Numpad 5`           | Abrir/cerrar el panel        |
| `Numpad 4`           | Página anterior              |
| `Numpad 6`           | Página siguiente             |
| `Numpad 1/2/3/7/8/9` | Acciones de la página actual |

La navegación no debe saltar directamente de la primera página a la última ni viceversa.

El panel también debe poder utilizarse con el mouse.

Las acciones pueden mostrar una descripción mediante **hover** antes de ejecutarse.

---

# Sync

`Sync` permite que SchoolNode continúe funcionando correctamente cuando los computadores se desconectan, apagan o reinician.

```text
Sync
│
├── Queue
├── Time
└── Recovery
```

## Queue

Mantiene una cola de mensajes y operaciones pendientes.

Si un computador está apagado:

```text
PC A
 │
 └── tarea → PC B
              │
              └── OFFLINE
```

La tarea no debe perderse.

Cuando PC B vuelva:

```text
PC B
 │
 └── vuelve ONLINE
          │
          ▼
       Queue
          │
          ▼
       tarea
```

---

## Time

Mantiene una referencia temporal coherente entre los nodos.

La sincronización del reloj se realizará periódicamente, con un intervalo inicial de **2 minutos y 30 segundos**.

Las acciones programadas deben utilizar referencias temporales adecuadas y no depender únicamente de temporizadores locales.

---

## Recovery

Permite recuperar el funcionamiento después de:

* Apagados
* Reinicios
* Desconexiones
* Pérdidas temporales de conexión

El sistema debe detectar nuevamente el nodo y sincronizar su estado con la infraestructura.

Las operaciones pendientes deben poder continuar cuando corresponda.

---

# Status

Proporciona información sobre el estado de cada computador.

```text
Status
│
├── CPU
├── RAM
├── Storage
└── Connectivity
```

## CPU

Proporciona información sobre:

* Uso de CPU
* Recursos disponibles
* Carga utilizada por el cluster

Esta información también permite que `Cluster/Background` respete los límites de utilización establecidos.

## RAM

Proporciona información sobre la memoria disponible y utilizada.

## Storage

Supervisa el almacenamiento disponible y permite administrar las operaciones relacionadas con archivos cuando corresponda.

## Connectivity

Indica si un nodo está:

```text
ONLINE
```

o

```text
OFFLINE
```

También permite detectar reconexiones.

---

# Actualizaciones

SchoolNode utilizará un sistema de actualización separado del ejecutable principal.

La estructura está pensada para permitir actualizaciones seguras y recuperación de versiones anteriores.

```text
SchoolNode/
├── exe/
│   └── SchoolNode.exe
│
├── Pipeline/
│
└── Versions/
    ├── version-001/
    ├── version-002/
    └── ...
```

`SchoolNode.exe` actúa como componente estable de inicio, mientras que las versiones de `Pipeline` pueden actualizarse independientemente.

Una actualización puede incluir información como:

```json
{
    "UpdateName": "Nueva interfaz estudiantil",
    "UpdatePriority": "Medium",
    "UpdateCode": "studentpanelredesign",
    "Version": "0.4.2",
    "ReleaseDate": "...",
    "Description": "...",
    "Download": "...",
    "Checksum": "...",
    "MinimumVersion": "...",
    "RequiresRestart": true
}
```

`UpdateCode` identifica la actualización mediante un código textual en minúsculas y sin espacios.

Las actualizaciones podrán utilizar diferentes niveles de prioridad:

```text
Low
Medium
High
Urgent
```

El sistema debe verificar la integridad de una versión antes de activarla y permitir volver a una versión anterior si una actualización falla.

---

# Estructura del proyecto

La estructura inicial del proyecto es:

```text
SchoolNode/
├── exe/
│   └── SchoolNode.exe
│
├── Pipeline/
│   │
│   ├── Classroom/
│   │   ├── RemoteControl/
│   │   ├── Screen/
│   │   ├── Lock/
│   │   ├── ScheduledActions/
│   │   └── TeacherCommunication/
│   │
│   ├── Cluster/
│   │   ├── Background/
│   │   ├── Teacher/
│   │   ├── TeacherUI/
│   │   ├── StudentsPrimary/
│   │   └── Students/
│   │
│   ├── Sync/
│   │   ├── Queue/
│   │   ├── Time/
│   │   └── Recovery/
│   │
│   └── Status/
│       ├── CPU/
│       ├── RAM/
│       ├── Storage/
│       └── Connectivity/
│
├── Versions/
│
└── docs/
```

El `README.md` principal se encuentra en la **raíz del proyecto**, no dentro de `docs/`.

`docs/` está reservado para documentación más específica y extensa de cada componente.

---

# Principios del proyecto

SchoolNode seguirá estos principios:

### Distribuido

Los computadores forman una infraestructura cooperativa en lugar de depender de una única máquina.

### Resistente a fallos

Un computador apagado o desconectado no debe provocar la pérdida de operaciones que puedan recuperarse posteriormente.

### Automático

El cluster debe administrar recursos y distribución de trabajos sin requerir intervención constante.

### No intrusivo

Las tareas del cluster deben utilizar únicamente los recursos que el computador pueda proporcionar sin perjudicar significativamente su uso normal.

### Separación de funciones

Las funciones del profesor y del estudiante deben mantenerse separadas dentro de sus respectivas interfaces.

### Modular

Las diferentes áreas de SchoolNode deben poder desarrollarse y actualizarse independientemente.

### Recuperable

Las actualizaciones y operaciones importantes deben poder verificarse y recuperarse cuando sea necesario.

---

# Estado del proyecto

> **SchoolNode se encuentra actualmente en fase de diseño y arquitectura.**

La estructura, los protocolos de comunicación, los formatos de datos y los componentes internos todavía pueden cambiar durante el desarrollo.

Las decisiones documentadas aquí representan la arquitectura actual del proyecto y sirven como referencia para futuras implementaciones.
