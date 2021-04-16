---
title: Lista de eventos de Azure Service Fabric
description: Lista completa de eventos proporcionados por Azure Service Fabric como ayuda para supervisar clústeres.
ms.topic: reference
ms.date: 2/25/2019
ms.openlocfilehash: a1ba21aaac831abb07544944420f2b77e1753955
ms.sourcegitcommit: 02bc06155692213ef031f049f5dcf4c418e9f509
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/03/2021
ms.locfileid: "106279789"
---
# <a name="list-of-service-fabric-events"></a>Lista de eventos de Service Fabric 

Service Fabric expone un conjunto principal de eventos de clúster para informarle sobre el estado del clúster como [eventos de Service Fabric](service-fabric-diagnostics-events.md). Estos se basan en las acciones realizadas por Service Fabric en los nodos y el clúster, o las decisiones de administración tomadas por un operador/propietario del clúster. Se puede acceder a estos eventos mediante numerosas configuraciones, por ejemplo, configurando [Registros de Azure Monitor con el clúster](service-fabric-diagnostics-oms-setup.md), o bien consultando [EventStore](service-fabric-diagnostics-eventstore.md). En máquinas Windows, estos eventos se introducen en el servicio EventLog, por lo que puede ver los eventos de Service Fabric en el Visor de eventos. 

Estas son algunas características de estos eventos:
* Cada evento está asociado a una entidad concreta en el clúster, por ejemplo, aplicación, servicio, nodo o réplica.
* Cada evento contiene un conjunto de campos comunes: EventInstanceId, EventName y Category.
* Cada evento contiene campos que vinculan el evento con la entidad a la que está asociada. Por ejemplo, el evento ApplicationCreated tendría campos que identifican el nombre de la aplicación creada.
* Los eventos están estructurados de tal manera que pueden utilizarse en diversas herramientas para realizar más análisis. Además, los detalles pertinentes de un evento se definen como propiedades independientes en lugar de una cadena larga. 
* Los eventos escritos por diferentes subsistemas en Service Fabric se identifican mediante la columna Origen (tarea) de abajo. Hay más información disponible sobre estos subsistemas en los artículos de [arquitectura de Service Fabric](service-fabric-architecture.md) y de [introducción técnica a Service Fabric](service-fabric-technical-overview.md).

A continuación, presentamos una lista de estos eventos de Service Fabric organizados por entidad.

## <a name="cluster-events"></a>Eventos de clúster

**Eventos de actualización de clústeres**

Se pueden encontrar más detalles sobre las actualizaciones de clústeres [en esta página](service-fabric-cluster-upgrade-windows-server.md).

| EventId | Nombre | Category | Descripción |Origen (tarea) | Nivel | 
| --- | --- | --- | --- | --- | --- | 
| 29627 | ClusterUpgradeStarted | Actualizar | Se ha iniciado una actualización de clúster | CM | Informativo |
| 29628 | ClusterUpgradeCompleted | Actualizar | Se ha terminado una actualización de clúster | CM | Informativo | 
| 29629 | ClusterUpgradeRollbackStarted | Actualizar | Se ha iniciado la reversión de una actualización de clúster  | CM | Advertencia | 
| 29630 | ClusterUpgradeRollbackCompleted | Actualizar | Se ha terminado la reversión de una actualización de clúster | CM | Advertencia | 
| 29631 | ClusterUpgradeDomainCompleted | Actualizar | Un dominio de actualización se ha terminado de actualizar durante una actualización de clúster | CM | Informativo | 

**Eventos de selección de ubicación**
| EventId | Nombre | Category | Descripción |Origen (tarea) | Nivel |
| --- | --- | ---| --- | --- | --- |
| 17616 | Decisión |StateTransition | La operación de selección de ubicación estaba programada para decidir la ubicación de nuevas réplicas. | CRM | Informativo |


## <a name="node-events"></a>Eventos de nodos

**Eventos del ciclo de vida de los nodos** 

| EventId | Nombre | Category | Descripción |Origen (tarea) | Nivel |
| --- | --- | ---| --- | --- | --- | 
| 18602 | NodeDeactivateCompleted | StateTransition | Se ha completado la desactivación de un nodo | FM | Informativo | 
| 18603 | NodeUp | StateTransition | El clúster ha detectado que un nodo se ha iniciado | FM | Informativo | 
| 18604 | NodeDown | StateTransition | El clúster ha detectado que un nodo se ha apagado (durante un reinicio del nodo, verá un evento NodeDown seguido de un evento NodeUp) |  FM | Error | 
| 18605 | NodeAddedToCluster | StateTransition |  Se ha agregado un nuevo nodo al clúster y Service Fabric puede implementar aplicaciones en este nodo | FM | Informativo | 
| 18606 | NodeRemovedFromCluster | StateTransition |  Se ha eliminado un nodo del clúster (Service Fabric ya no implementará aplicaciones en este nodo) | FM | Informativo | 
| 18607 | NodeDeactivateStarted | StateTransition |  Se ha iniciado la desactivación de un nodo | FM | Informativo | 
| 25621 | NodeOpenSucceeded | StateTransition |  Un nodo se inició correctamente | FabricNode | Informativo | 
| 25622 | NodeOpenFailed | StateTransition |  No se ha podido iniciar un nodo y unir el anillo | FabricNode | Error | 
| 25624 | NodeClosed | StateTransition |  Un nodo se apagó correctamente | FabricNode | Informativo | 
| 25626 | NodeAborted | StateTransition |  Un nodo se ha apagado de manera brusca | FabricNode | Error | 

**Eventos de nodo del registrador de KTL** 

| EventId | Nombre | Category | Descripción |Origen (tarea) | Nivel |
| --- | --- | --- | --- | --- | --- | 
| 50187 | SharedLogWriteThrottled | Health | Las escrituras en el registro compartido están limitadas | KtlLoggerNode | Informativo | 
| 50188 | SharedLogWriteUnthrottled | Health | Las escrituras en el registro compartido no tienen límite | KtlLoggerNode | Informativo | 

## <a name="application-events"></a>Eventos de aplicación

**Eventos del ciclo de vida de aplicaciones**

| EventId | Nombre | Category | Descripción |Origen (tarea) | Nivel | 
| --- | --- | --- | --- | --- | --- | 
| 29620 | ApplicationCreated | LifeCycle | Se ha creado una nueva aplicación | CM | Informativo | 
| 29625 | ApplicationDeleted | LifeCycle | Se ha eliminado una aplicación existente | CM | Informativo | 
| 23083 | ApplicationProcessExited | LifeCycle | Un proceso de una aplicación ha salido | Hospedaje | Informativo | 

**Eventos de actualización de aplicaciones**

Pueden encontrar más detalles sobre las actualizaciones de aplicaciones [en esta página](service-fabric-application-upgrade.md).

| EventId | Nombre | Category | Descripción |Origen (tarea) | Nivel | 
| --- | --- | ---| --- | --- | --- | 
| 29621 | ApplicationUpgradeStarted | Actualizar | Se ha iniciado una actualización de aplicación | CM | Informativo | 
| 29622 | ApplicationUpgradeCompleted | Actualizar | Se ha terminado una actualización de aplicación | CM | Informativo | 
| 29623 | ApplicationUpgradeRollbackStarted | Actualizar | Se ha iniciado la reversión de una actualización de aplicación |CM | Advertencia | 
| 29624 | ApplicationUpgradeRollbackCompleted | Actualizar | Se ha terminado la reversión de una actualización de aplicación | CM | Advertencia | 
| 29626 | ApplicationUpgradeDomainCompleted | Actualizar | Un dominio de actualización se ha terminado de actualizar durante una actualización de aplicación | CM | Informativo | 

## <a name="service-events"></a>Eventos de servicios

**Eventos de ciclo de vida de servicios**

| EventId | Nombre | Category | Descripción |Origen (tarea) | Nivel | 
| --- | --- | ---| --- | --- | --- |
| 18657 | ServiceCreated | LifeCycle | Se ha creado un nuevo servicio | FM | Informativo | 
| 18658 | ServiceDeleted | LifeCycle | Se ha eliminado un servicio existente | FM | Informativo | 

## <a name="partition-events"></a>Eventos de particiones

**Eventos de movimiento de particiones**

| EventId | Nombre | Category | Descripción |Origen (tarea) | Nivel | 
| --- | --- | ---| --- | --- | --- |
| 18940 | PartitionReconfigured | LifeCycle | Se ha terminado la reconfiguración de una partición | RA | Informativo | 

## <a name="replica-events"></a>Eventos de réplicas

**Eventos de ciclo de vida de las réplicas**

| EventId | Nombre | Category | Descripción |Origen (tarea) | Nivel |
| --- | --- | ---| --- | --- | --- |
| 61701 | ReliableDictionaryOpened | LifeCycle | El diccionario de confianza se ha abierto. | DistributedDictionary | Informativo |
| 61702 | ReliableDictionaryClosed | LifeCycle | El diccionario de confianza se ha cerrado. | DistributedDictionary | Informativo |
| 61703 | ReliableDictionaryCheckpointRecovered | LifeCycle | El diccionario de confianza ha recuperado su punto de control. | DistributedDictionary | Informativo |
| 61704 | ReliableDictionaryCheckpointFilesSent | LifeCycle | La réplica ha enviado los archivos del punto de control del diccionario de confianza. | DistributedDictionary | Informativo |
| 61705 | ReliableDictionaryCheckpointFilesReceived | LifeCycle | La réplica ha recibido los archivos del punto de control del diccionario de confianza. | DistributedDictionary | Informativo |
| 61963 | ReliableQueueOpened | LifeCycle | La cola de confianza se ha abierto. | DistributedQueue | Informativo |
| 61964 | ReliableQueueClosed | LifeCycle | La cola de confianza se ha cerrado. | DistributedQueue | Informativo |
| 61965 | ReliableQueueCheckpointRecovered | LifeCycle | La cola de confianza ha recuperado su punto de control. | DistributedQueue | Informativo |
| 61966 | ReliableQueueCheckpointFilesSent | LifeCycle | La réplica ha enviado los archivos del punto de control de la cola de confianza. | DistributedQueue | Informativo |
| 63647 | ReliableQueueCheckpointFilesReceived | LifeCycle | La réplica ha recibido los archivos del punto de control de la cola de confianza. | DistributedQueue | Informativo |
| 63648 | ReliableConcurrentQueueOpened | LifeCycle | La cola simultánea de confianza se ha abierto. | ReliableConcurrentQueue | Informativo |
| 63649 | ReliableConcurrentQueueClosed | LifeCycle | La cola simultánea de confianza se ha cerrado. | ReliableConcurrentQueue | Informativo |
| 63650 | ReliableConcurrentQueueCheckpointRecovered | LifeCycle | La cola simultánea de confianza ha recuperado su punto de control. | ReliableConcurrentQueue | Informativo |
| 61687 | TStoreError | Error | La colección de confianza ha recibido un error inesperado. | TStore | Error |
| 63831 | PrimaryFullCopyInitiated | LifeCycle | La réplica principal ha iniciado una copia completa. | TReplicator | Informativo |
| 63832 | PrimaryPartialCopyInitiated | LifeCycle | La réplica principal ha iniciado una copia parcial. | TReplicator | Informativo |
| 16831 | BuildIdleReplicaStarted | LifeCycle | La réplica principal ha iniciado la compilación de una réplica inactiva. | Replicación | Informativo |
| 16832 | BuildIdleReplicaCompleted | LifeCycle | La réplica principal ha completado la compilación de una réplica inactiva. | Replicación | Informativo |
| 16833 | BuildIdleReplicaFailed | LifeCycle | La réplica principal ha producido un erro al compilar una réplica inactiva. | Replicación | Advertencia |
| 16834 | PrimaryReplicationQueueFull | Health | La cola de replicación de la réplica principal está llena. | Replicación | Advertencia |
| 16835 | PrimaryReplicationQueueWarning | Health | La cola de replicación de la réplica principal está casi llena. | Replicación | Advertencia |
| 16836 | PrimaryReplicationQueueWarningMitigated | Health | La cola de replicación de la réplica principal es correcta. | Replicación | Informativo |
| 16837 | SecondaryReplicationQueueFull | Health | La cola de replicación de la réplica secundaria está llena. | Replicación | Advertencia |
| 16838 | SecondaryReplicationQueueWarning | Health | La cola de replicación de la réplica secundaria está casi llena. | Replicación | Advertencia |
| 16839 | SecondaryReplicationQueueWarningMitigated | Health | La cola de replicación de la réplica secundaria es correcta. | Replicación | Informativo |
| 16840 | PrimaryFaultedSlowSecondary | Health | La réplica principal ha producido un error en una réplica secundaria lenta. | Replicación | Advertencia |
| 16841 | ReplicatorFaulted | Health | La réplica ha producido un error. | Replicación | Advertencia |

## <a name="container-events"></a>Eventos de contenedores

**Eventos de ciclo de vida de contenedores** 

| EventId | Nombre | Descripción |Origen (tarea) | Nivel | Versión |
| --- | --- | ---| --- | --- | --- |
| 23074 | ContainerActivated | Se ha iniciado un contenedor | Hospedaje | Informativo | 1 |
| 23075 | ContainerDeactivated | Se ha detenido un contenedor | Hospedaje | Informativo | 1 |
| 23082 | ContainerExited | Ha salido un contenedor. Compruebe la marca UnexpectedTermination | Hospedaje | Informativo | 1 |

## <a name="health-reports"></a>Informes de mantenimiento

El [modelo de mantenimiento de Service Fabric](service-fabric-health-introduction.md) proporciona informes y datos de evaluación de mantenimiento completos, flexibles y extensibles. A partir de la versión 6.2 de Service Fabric, los datos de mantenimiento se escriben como eventos de plataforma para proporcionar registros históricos de mantenimiento. Para mantener el volumen de eventos de mantenimiento en un nivel reducido, solo se escribe lo siguiente como eventos de Service Fabric:

* Todos los informes de mantenimiento de `Error` o `Warning`.
* Los informes de mantenimiento de `Ok` durante las transiciones.
* Cuando un evento de mantenimiento de `Error` o `Warning` expira. Esto puede usarse para determinar durante cuánto tiempo una entidad no era correcta.

**Eventos de informe de estado de clústeres**

| EventId | Nombre | Descripción |Origen (tarea) | Nivel | Versión |
| --- | --- | --- | --- | --- | --- |
| 54428 | ClusterNewHealthReport | Hay un nuevo informe de estado de clúster disponible | HM | Informativo | 1 |
| 54437 | ClusterHealthReportExpired | Un informe de estado de clúster existente ha expirado | HM | Informativo | 1 |

**Eventos de informe de estado de nodos**

| EventId | Nombre | Descripción |Origen (tarea) | Nivel | Versión |
| --- | --- | ---| --- | --- | --- |
| 54423 | NodeNewHealthReport | Hay un nuevo informe de estado de nodo disponible | HM | Informativo | 1 |
| 54432 | NodeHealthReportExpired | Un informe de estado de nodo existente ha expirado | HM | Informativo | 1 |

**Eventos de informe de estado de aplicaciones**

| EventId | Nombre | Descripción |Origen (tarea) | Nivel | Versión |
| --- | --- | ---| --- | --- | --- |
| 54425 | ApplicationNewHealthReport | Se ha creado un informe de estado de aplicación. Para aplicaciones no implementadas. | HM | Informativo | 1 |
| 54426 | DeployedApplicationNewHealthReport | Se ha creado un informe de estado de aplicación implementada | HM | Informativo | 1 |
| 54427 | DeployedServicePackageNewHealthReport | Se ha creado un informe de estado de servicio implementado | HM | Informativo | 1 |
| 54434 | ApplicationHealthReportExpired | Un informe de estado de aplicación existente ha expirado | HM | Informativo | 1 |
| 54435 | DeployedApplicationHealthReportExpired | Un informe de estado de aplicación implementada existente ha expirado | HM | Informativo | 1 |
| 54436 | DeployedServicePackageHealthReportExpired | Un informe de estado de servicio implementado existente ha expirado | HM | Informativo | 1 |

**Eventos de informe de estado de servicios**

| EventId | Nombre | Descripción |Origen (tarea) | Nivel | Versión |
| --- | --- | ---| --- | --- | --- |
| 54424 | ServiceNewHealthReport | Se ha creado un informe de estado de servicio | HM | Informativo | 1 |
| 54433 | ServiceHealthReportExpired | Un informe de estado de servicio existente ha expirado | HM | Informativo | 1 |

**Eventos de informe de estado de particiones**

| EventId | Nombre | Descripción |Origen (tarea) | Nivel | Versión |
| --- | --- | ---| --- | --- | --- |
| 54422 | PartitionNewHealthReport | Se ha creado un informe de estado de partición | HM | Informativo | 1 |
| 54431 | PartitionHealthReportExpired | Un informe de estado de partición existente ha expirado | HM | Informativo | 1 |

**Eventos de informe de estado de réplicas**

| EventId | Nombre | Descripción |Origen (tarea) | Nivel | Versión |
| --- | --- | ---| --- | --- | --- |
| 54429 | StatefulReplicaNewHealthReport | Se ha creado un informe de estado de réplica con estado | HM | Informativo | 1 |
| 54430 | StatelessInstanceNewHealthReport | Se ha creado un informe de estado de instancia sin estado | HM | Informativo | 1 |
| 54438 | StatefulReplicaHealthReportExpired | Un informe de estado de réplica con estado existente ha expirado | HM | Informativo | 1 |
| 54439 | StatelessInstanceHealthReportExpired | Un informe de estado de instancia sin estado existente ha expirado | HM | Informativo | 1 |

## <a name="chaos-testing-events"></a>Eventos de pruebas de caos 

**Eventos de sesión de caos**

| EventId | Nombre | Descripción |Origen (tarea) | Nivel | Versión |
| --- | --- | ---| --- | --- | --- |
| 50021 | ChaosStarted | Se ha iniciado una sesión de pruebas de caos | Capacidad de prueba | Informativo | 1 |
| 50023 | ChaosStopped | Se ha detenido una sesión de pruebas de caos | Capacidad de prueba | Informativo | 1 |

**Eventos de nodos de Chaos**

| EventId | Nombre | Descripción |Origen (tarea) | Nivel | Versión |
| --- | --- | ---| --- | --- | --- |
| 50033 | ChaosNodeRestartScheduled | Se ha programado el reinicio de un nodo como parte de una sesión de pruebas de caos | Capacidad de prueba | Informativo | 1 |
| 50087 | ChaosNodeRestartCompleted | Un nodo ha terminado el reinicio como parte de una sesión de pruebas de caos | Capacidad de prueba | Informativo | 1 |

**Eventos de aplicación Chaos**

| EventId | Nombre | Descripción |Origen (tarea) | Nivel | Versión |
| --- | --- | ---| --- | --- | --- |
| 50053 | ChaosCodePackageRestartScheduled | Se ha programado un reinicio del paquete de código durante una sesión de pruebas de caos | Capacidad de prueba | Informativo | 1 |
| 50101 | ChaosCodePackageRestartCompleted | Se ha completado el reinicio del paquete de código durante una sesión de pruebas de caos | Capacidad de prueba | Informativo | 1 |

**Eventos de particiones de Chaos**

| EventId | Nombre | Descripción |Origen (tarea) | Nivel | Versión |
| --- | --- | ---| --- | --- | --- |
| 50069 | ChaosPartitionPrimaryMoveScheduled | Se ha programado la migración de una partición principal como parte de una sesión de pruebas de caos | Capacidad de prueba | Informativo | 1 |
| 50077 | ChaosPartitionSecondaryMoveScheduled | Se ha programado la migración de una partición secundaria como parte de una sesión de pruebas de caos | Capacidad de prueba | Informativo | 1 |
| 65003 | PartitionPrimaryMoveAnalysis | El análisis en profundidad de la migración de la partición principal está disponible | Capacidad de prueba | Informativo | 1 |

**Eventos de réplicas de Chaos**

| EventId | Nombre | Descripción |Origen (tarea) | Nivel | Versión |
| --- | --- | ---| --- | --- | --- |
| 50047 | ChaosReplicaRestartScheduled | Se ha programado un reinicio de una réplica como parte de una sesión de pruebas de caos | Capacidad de prueba | Informativo | 1 |
| 50051 | ChaosReplicaRemovalScheduled | Se ha programado una eliminación de réplica como parte de una sesión de pruebas de caos | Capacidad de prueba | Informativo | 1 |
| 50093 | ChaosReplicaRemovalCompleted | Se ha terminado una eliminación de réplica como parte de una sesión de pruebas de caos | Capacidad de prueba | Informativo | 1 |

## <a name="other-events"></a>Otros eventos

**Eventos de correlaciones**

| EventId | Nombre | Descripción |Origen (tarea) | Nivel | Versión |
| --- | --- | ---| --- | --- | --- |
| 65011 | CorrelationOperational | Se ha detectado una correlación | Capacidad de prueba | Informativo | 1 |

## <a name="events-prior-to-version-62"></a>Eventos anteriores a la versión 6.2

A continuación figura una lista completa de los eventos proporcionados por Service Fabric antes de la versión 6.2.

| EventId | Nombre | Origen (tarea) | Nivel |
| --- | --- | --- | --- |
| 25620 | NodeOpening | FabricNode | Informativo |
| 25621 | NodeOpenedSuccess | FabricNode | Informativo |
| 25622 | NodeOpenedFailed | FabricNode | Informativo |
| 25623 | NodeClosing | FabricNode | Informativo |
| 25624 | NodeClosed | FabricNode | Informativo |
| 25625 | NodeAborting | FabricNode | Informativo |
| 25626 | NodeAborted | FabricNode | Informativo |
| 29627 | ClusterUpgradeStart | CM | Informativo |
| 29628 | ClusterUpgradeComplete | CM | Informativo |
| 29629 | ClusterUpgradeRollback | CM | Informativo |
| 29630 | ClusterUpgradeRollbackComplete | CM | Informativo |
| 29631 | ClusterUpgradeDomainComplete | CM | Informativo |
| 23074 | ContainerActivated | Hospedaje | Informativo |
| 23075 | ContainerDeactivated | Hospedaje | Informativo |
| 29620 | ApplicationCreated | CM | Informativo |
| 29621 | ApplicationUpgradeStart | CM | Informativo |
| 29622 | ApplicationUpgradeComplete | CM | Informativo |
| 29623 | ApplicationUpgradeRollback | CM | Informativo |
| 29624 | ApplicationUpgradeRollbackComplete | CM | Informativo |
| 29625 | ApplicationDeleted | CM | Informativo |
| 29626 | ApplicationUpgradeDomainComplete | CM | Informativo |
| 18566 | ServiceCreated | FM | Informativo |
| 18567 | ServiceDeleted | FM | Informativo |

## <a name="next-steps"></a>Pasos siguientes

* Consulte una introducción a los [diagnósticos de Service Fabric](service-fabric-diagnostics-overview.md).
* Obtenga más información sobre EventStore en [Información general del servicio EventStore](service-fabric-diagnostics-eventstore.md).
* Modificación de la configuración de [Azure Diagnostics](service-fabric-diagnostics-event-aggregation-wad.md) para recopilar más registros
* [Configuración de Application Insights](service-fabric-diagnostics-event-analysis-appinsights.md) para ver los registros de canal operativo
