---
title: Versiones de Azure Service Fabric
description: Notas de la versión de Azure Service Fabric. Incluyen información sobre las características y mejoras más recientes de Service Fabric.
ms.date: 04/13/2021
ms.topic: conceptual
hide_comments: true
hideEdit: true
ms.openlocfilehash: 0e5b4d17b0c0ccaa4c839338e0656c0a8eda3bd5
ms.sourcegitcommit: 3bb9f8cee51e3b9c711679b460ab7b7363a62e6b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112078031"
---
# <a name="service-fabric-releases"></a>Versiones de Service Fabric

En este artículo se proporciona más información sobre las versiones y actualizaciones más recientes para los SDK y el runtime de Service Fabric.

Los recursos siguientes también están disponibles:
- <a href="https://github.com/Azure/Service-Fabric-Troubleshooting-Guides" target="blank">Guías de solución de problemas</a> 
- <a href="https://github.com/Azure/service-fabric-issues" target="blank">Seguimiento de problemas</a> 
- <a href="/azure/service-fabric/service-fabric-support" target="blank">Opciones de soporte técnico</a> 
- <a href="/azure/service-fabric/service-fabric-versions" target="blank">Versiones compatibles</a> 
- <a href="https://azure.microsoft.com/resources/samples/?service=service-fabric&sort=0" target="blank">Ejemplos de código</a>


## <a name="service-fabric-80"></a>Service Fabric 8.0

Nos complace anunciar que la versión 8.0 del entorno de tiempo de ejecución de Service Fabric ha empezado a implementarse en las diversas regiones de Azure junto con las herramientas y las actualizaciones del SDK. Las actualizaciones del SDK de .NET, el SDK de Java y el entorno de ejecución de Service Fabric están disponibles mediante el Instalador de plataforma web, los paquetes de NuGet y los repositorios de Maven.

### <a name="key-announcements"></a>Anuncios clave

- **Disponibilidad general** del soporte técnico para .NET 5 para Windows.
- **Disponibilidad general** de [NodeTypes sin estado](./service-fabric-stateless-node-types.md).
- Capacidad de mover instancias de servicio sin estado.
- Capacidad de agregar DefaultLoad con parámetros en el manifiesto de aplicación.
- Para las actualizaciones de réplica de base de datos única: capacidad de definir parte de la configuración de nivel de clúster en el nivel de aplicación.
- Capacidad de selección de ubicación inteligente basada en etiquetas de nodo.
- Capacidad de definir el umbral de porcentaje de nodos incorrectos que afectan al estado del clúster.
- Capacidad de consultar los principales servicios cargados.
- Capacidad de agregar un nuevo intervalo para nuevos códigos de error.
- Capacidad de marcar la instancia de servicio como completada.
- Compatibilidad con el modelo de implementación basado en onda para las actualizaciones automáticas.
- Se ha agregado un sondeo de preparación para las aplicaciones en contenedores.
- Se ha habilitado UseSeparateSecondaryMoveCost en true de forma predeterminada.
- Se ha corregido StateManager para liberar la referencia tan pronto como sea seguro.
- Se ha bloqueado la eliminación del servicio secreto central al almacenar secretos de usuario.


### <a name="service-fabric-80-releases"></a>Versiones de Service Fabric 8.0
| Fecha de la versión | Release | Más información |
|---|---|---|
| 08 de abril de 2021 | [Azure Service Fabric 8.0](https://techcommunity.microsoft.com/t5/azure-service-fabric/azure-service-fabric-8-0-release/ba-p/2260016)  | [Notas de la versión](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service_Fabric_ReleaseNotes_80.md)|


## <a name="previous-versions"></a>Versiones anteriores

### <a name="service-fabric-72"></a>Service Fabric 7.2

#### <a name="key-announcements"></a>Anuncios clave

- **Versión preliminar**: Los [**clústeres administrados de Service Fabric**](https://techcommunity.microsoft.com/t5/azure-service-fabric/azure-service-fabric-managed-clusters-are-now-in-public-preview/ba-p/1721572) están ahora en versión preliminar pública. Los clústeres administrados de Service Fabric tienen por objeto simplificar la implementación y la administración de clústeres al encapsular los recursos subyacentes que componen un clúster de Service Fabric en un solo recurso de ARM. Para más información, consulte [Introducción a los clústeres administrados de Service Fabric](./overview-managed-cluster.md).
- **Versión preliminar**: [**Ahora, la compatibilidad con servicios sin estado con un número de instancias mayor que el número de nodos**](./service-fabric-cluster-resource-manager-advanced-placement-rules-placement-policies.md) se encuentra en versión preliminar pública. Una directiva de selección de ubicación permite la creación de varias instancias sin estado de una partición en un nodo.
- Ya está disponible [**FabricObserver (FO) 3.0**](https://aka.ms/sf/fabricobserver).
    - Ahora puede ejecutar FabricObserver en clústeres de Windows y Linux.
    - Ahora puede crear complementos de observador personalizados. Consulte el [archivo Léame de complementos](https://github.com/microsoft/service-fabric-observer/blob/master/Documentation/Plugins.md) y el [proyecto de complementos de ejemplo](https://github.com/microsoft/service-fabric-observer/tree/master/SampleObserverPlugin) para conocer los detalles y obtener el código.
    - Ahora puede cambiar cualquier valor del observador a través de la actualización de los parámetros de la aplicación. Esto significa que ya no necesita volver a implementar FO para modificar valores de configuración específicos del observador. Consulte el [ejemplo](https://github.com/microsoft/service-fabric-observer/blob/master/Documentation/Using.md#parameterUpdates).
- [**Compatibilidad con imágenes de contenedor de Ubuntu 18.04 OneBox**](https://hub.docker.com/_/microsoft-service-fabric-onebox).
- **Versión preliminar**: [**La referencia de KeyVault para aplicaciones de Service Fabric **SOLO admite secretos con versiones**. No se admiten secretos sin versiones.**](./service-fabric-keyvault-references.md)
- El SDK de SF requiere la última actualización de VS 2019 (16.7.6 o 16.8 Preview 4) para poder crear proyectos de .NET Framework sin estado, con estado o de actores. Si no tiene la última actualización de VS, después de crear el proyecto de servicio, use el administrador de paquetes para instalar Microsoft.ServiceFabric.Services (versión 4.2.x) para proyectos con estado y sin estado y Microsoft.ServiceFabric.Actors (versión 4.2.x) para proyectos de actores de nuget.org.
- **RunToCompletion**: Service Fabric admite el concepto de ejecución hasta finalización para los archivos ejecutables invitados. Con esta actualización, una vez que la réplica se ejecuta hasta su finalización, se liberan los recursos de clúster asignados a esta réplica.
- [**La compatibilidad con la gobernanza de recursos se ha mejorado**](./service-fabric-resource-governance.md), y permite solicitudes y especificaciones de límites de recursos de CPU y memoria.

#### <a name="service-fabric-72-releases"></a>Versiones de Service Fabric 7.2
| Fecha de la versión | Release | Más información |
|---|---|---|
| 21 de octubre de 2020 | [Azure Service Fabric 7.2](https://techcommunity.microsoft.com/t5/azure-service-fabric/azure-service-fabric-7-2-release/ba-p/1805653)  | [Notas de la versión](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service-Fabric-72-releasenotes.md)|
| 9 de noviembre de 2020 | [Segunda versión de actualización de Azure Service Fabric 7.2](https://techcommunity.microsoft.com/t5/azure-service-fabric/azure-service-fabric-7-2-second-refresh-release/ba-p/1874738) | [Notas de la versión](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service-Fabric-72CU2-releasenotes.md) |
| 10 de noviembre de 2020  | Tercera versión de actualización de Azure Service Fabric 7.2 | [Notas de la versión](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service-Fabric-72CU3-releasenotes.md) |
| 2 de diciembre de 2020 | [Cuarta versión de actualización de Azure Service Fabric 7.2](https://techcommunity.microsoft.com/t5/azure-service-fabric/azure-service-fabric-7-2-fourth-refresh-release/ba-p/1950584) | [Notas de la versión](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service-Fabric-72CU4.md)
| 25 de enero de 2021 | [Quinta versión de actualización de Azure Service Fabric 7.2](https://techcommunity.microsoft.com/t5/azure-service-fabric/azure-service-fabric-7-2-fifth-refresh-release/ba-p/2096575) | [Notas de la versión](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service-Fabric-72CU5-ReleaseNotes.md)
| 17 de febrero de 2021 | [Sexta versión de actualización de Azure Service Fabric 7.2](https://techcommunity.microsoft.com/t5/azure-service-fabric/azure-service-fabric-sixth-refresh-release/ba-p/2144685) | [Notas de la versión](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service-Fabric-72CU6-ReleaseNotes.md)
| 10 de marzo, 2021 | [Séptima versión de actualización de Azure Service Fabric 7.2](https://techcommunity.microsoft.com/t5/azure-service-fabric/azure-service-fabric-seventh-refresh-release/ba-p/2201100) | [Notas de la versión](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service-Fabric-72CU7-releasenotes.md)


### <a name="service-fabric-71"></a>Service Fabric 7.1

Debido a la actual crisis de la COVID-19 y teniendo en cuenta los desafíos a los que se enfrentan nuestros clientes, vamos a lanzar la versión 7.1, pero no se actualizarán automáticamente los clústeres configurados para recibir actualizaciones automáticas. Vamos a interrumpir las actualizaciones automáticas hasta próximo aviso para garantizar que los clientes pueden aplicar las actualizaciones cuando les resulte conveniente, con el fin de evitar interrupciones inesperadas.

Podrá actualizar a la versión 7.1 desde [Azure Portal](./service-fabric-cluster-upgrade-version-azure.md#manual-upgrades-with-azure-portal) o desde una [implementación de Azure Resource Manager](./service-fabric-cluster-upgrade-version-azure.md#resource-manager-template).

Los clústeres de Service Fabric con actualizaciones automáticas habilitadas comenzarán a recibir la actualización 7.1 automáticamente una vez que se reanude el procedimiento de lanzamiento estándar. Realizaremos otro anuncio antes de que comience el lanzamiento estándar en el [sitio de Tech Community de Service Fabric](https://techcommunity.microsoft.com/t5/azure-service-fabric/bg-p/Service-Fabric).
También hemos publicado [aquí](./service-fabric-versions.md) actualizaciones sobre la fecha de finalización del soporte para las versiones principales desde la 6.5 hasta la 7.1. 

#### <a name="key-announcements"></a>Anuncios clave

- **Disponibilidad general** de [**identidades administradas de Service Fabric para aplicaciones de Service Fabric**](./concepts-managed-identity.md)
- [**Compatibilidad con Ubuntu 18.04**](./service-fabric-tutorial-create-vnet-and-linux-cluster.md)
 - [**Versión preliminar: compatibilidad con discos de sistema operativo efímeros para conjuntos de escalado de máquinas virtuales**](./service-fabric-cluster-azure-deployment-preparation.md#use-ephemeral-os-disks-for-virtual-machine-scale-sets)**: Los discos del sistema operativo efímeros son almacenamiento creado en la máquina virtual local y no se guardan en la instancia remota de Azure Storage. Se recomiendan para todos los tipos de nodos de Service Fabric (principales y secundarios) ya que, comparados con los discos del SO persistentes tradicionales, los discos efímeros:
      -  Reducen la latencia de lectura y escritura en el disco del sistema operativo;
      -  Permiten realizar con mayor rapidez las operaciones de administración para restablecimiento y restablecimiento de imagen inicial de los nodos
      -  Reducen los costos generales (los discos son gratuitos y no hay ningún costo de almacenamiento adicional).
- Compatibilidad con la declaración de [**certificados de punto de conexión de servicio de aplicaciones de Service Fabric por nombre común del sujeto**](./service-fabric-service-manifest-resources.md).
- [**Compatibilidad con sondeos de estado para servicios en contenedores**](./probes-codepackage.md): Compatibilidad del mecanismo del sondeo de ejecución para aplicaciones en contenedores. El sondeo de ejecución ayuda a anunciar la vivacidad de aplicaciones en contenedores y, cuando no responden a tiempo, se producirá un reinicio. 
- [**Compatibilidad con paquetes de código del inicializador**](./initializer-codepackages.md) para [contenedores](./service-fabric-containers-overview.md) y aplicaciones [ejecutables invitadas](./service-fabric-guest-executables-introduction.md). Esto permite ejecutar paquetes de código (p.ej., contenedores), en un orden especificado, para realizar la inicialización de un paquete de servicio.
- **FabricObserver y ClusterObserver** son aplicaciones sin estado que capturan la telemetría de Service Fabric relacionada con diferentes aspectos de un clúster de Service Fabric. Las dos aplicaciones están listas para implementarse en clústeres de producción de Windows para capturar muchos datos de telemetría con compatibilidad implementada con ApplicationInsights, EventSource y LogAnalytics.
    - [**FabricObserver (FO) 2.0**](https://github.com/microsoft/service-fabric-observer): se ejecuta en todos los nodos, genera eventos de estado y emite datos de telemetría cuando se alcanzan los niveles de uso de recursos configurados por el usuario. Esta versión contiene varias mejoras en la supervisión, administración de datos, detalles del evento de estado y telemetría estructurada.
     - [**ClusterObserver (CO) 1.1**](https://github.com/microsoft/service-fabric-observer/tree/master/ClusterObserver): se ejecuta en un nodo, captura los datos de telemetría del estado del nivel de clúster. En esta versión, ClusterObserver también supervisa el estado del nodo y emite datos de telemetría cuando el nodo está inactivo/deshabilitándose/deshabilitado durante un periodo de tiempo mayor que el especificado por el usuario.

#### <a name="improve-application-life-cycle-experience"></a>Mejora de la experiencia del ciclo de vida de la aplicación

- **[Versión preliminar: solicitar purga](./service-fabric-application-upgrade-advanced.md#avoid-connection-drops-during-stateless-service-planned-downtime)** : durante un mantenimiento de servicio planeado, como las actualizaciones del servicio o la desactivación de nodos, desea permitir que los servicios purguen las conexiones. Esta característica agrega la duración del retraso del cierre de una instancia en la configuración del servicio. Durante las operaciones planeadas, Service Fabric eliminará la dirección del servicio de la detección y, después, esperará este tiempo antes de apagar el servicio.
- **[Equilibrado y detección automáticos de subclústeres](./cluster-resource-manager-subclustering.md)** : La agrupación en subclústeres se produce cuando los servicios con restricciones de selección de ubicación diferentes tienen una [métrica de carga](./service-fabric-cluster-resource-manager-metrics.md)común. Si la carga en los diferentes conjuntos de nodos difieren considerablemente, Cluster Resource Manager de Service Fabric cree que el clúster no está equilibrado, ni siquiera cuando tenga el mejor equilibrio posible debido a las restricciones de selección de ubicación. En consecuencia, intenta volver a equilibrar el clúster, lo que puede provocar movimientos innecesarios de servicios (dado que el "desequilibrio" no se puede mejorar sustancialmente). Si se empieza con esta versión, Cluster Resource Manager ahora intentará detectar automáticamente estos tipos de configuraciones y saber cuándo se puede solucionar el desequilibrio a través del movimiento, y cuándo hay que dejar todo tal cual está, ya que no de puede realizar ninguna mejora sustancial.  
- [**Diferente costo de movimiento para las réplicas secundarias**](./service-fabric-cluster-resource-manager-movement-cost.md): Hemos introducido el nuevo valor de costo de movimiento VeryHigh, que proporciona mayor flexibilidad en algunos escenarios para definir si se debe usar un costo de movimiento independiente para las réplicas secundarias.
- Mecanismo del [**sondeo de ejecución**](./probes-codepackage.md) para aplicaciones en contenedores. El sondeo de ejecución ayuda a anunciar la vivacidad de aplicaciones en contenedores y, cuando no responden a tiempo, se producirá un reinicio.
- [**Ejecutar los servicios hasta la finalización o una vez**](./run-to-completion.md)**

#### <a name="image-store-improvements"></a>Mejoras del Almacén de imágenes
 - Service Fabric 7.1 usa un **transporte personalizado para proteger la transferencia de archivos de manera predeterminada**. La dependencia del recurso compartido de archivos de SMB file se elimina de la versión 7.1. Los recursos compartidos de archivos de SMB protegidos siguen existiendo en los nodos que contienen la réplica del servicio de Almacén de imágenes para que el cliente tenga la posibilidad de elegir una opción que no sea la predeterminada y para poder actualizar y cambiar a una versión anterior.
       
 #### <a name="reliable-collections-improvements"></a>Mejoras en las colecciones de confianza

- [**En memoria almacenar solo soporte para servicios con estado mediante colecciones de colección de confianza**](./service-fabric-work-with-reliable-collections.md#volatile-reliable-collections): Las colecciones volátiles de confianza permiten que los datos se almacenen en disco para aumentar su durabilidad frente a las interrupciones a gran escala. Se pueden usar para las cargas de trabajo como la memoria caché replicada, por ejemplo, donde se puede tolerar la pérdida de datos ocasionales. En función de las [limitaciones y restricciones](./service-fabric-reliable-services-reliable-collections-guidelines.md#volatile-reliable-collections) de las colecciones volátiles de confianza, se recomiendan para las cargas de trabajo que no necesitan persistencia para los servicios que controlan los casos excepcionales de pérdida de cuórum.
- [**Versión preliminar: Explorador de Backup de Service Fabric**](https://github.com/microsoft/service-fabric-backup-explorer): para facilitar la administración de copias de seguridad de las colecciones de confianza para aplicaciones con estado de Service Fabric, el Explorador de Backup de Service Fabric permite a los usuarios
    - Auditar y revisar el contenido de las colecciones de confianza.
    - Actualizar el estado actual a una vista consistente.
    - Crear una copia de seguridad de la instantánea actual de las colecciones de confianza.
    - Reparar los daños en los datos.
                 
#### <a name="service-fabric-71-releases"></a>Versiones de Service Fabric 7.1
| Fecha de la versión | Release | Más información |
|---|---|---|
| 20 de abril de 2020 | [Azure Service Fabric 7.1](https://techcommunity.microsoft.com/t5/azure-service-fabric/azure-service-fabric-7-1-release/ba-p/1311373)  | [Notas de la versión](https://github.com/microsoft/service-fabric/tree/master/release_notes/Service-Fabric-71-releasenotes.md)|
| 16 de junio de 2020 | [Primera actualización de Microsoft Azure Service Fabric 7.1](https://techcommunity.microsoft.com/t5/azure-service-fabric/azure-service-fabric-7-1-first-refresh-release/ba-p/1466517) | [Notas de la versión](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service-Fabric-71CU1-releasenotes.md)
| 20 de julio de 2020 | [Segunda actualización de Microsoft Azure Service Fabric 7.1](https://techcommunity.microsoft.com/t5/azure-service-fabric/azure-service-fabric-7-1-second-refresh-release/ba-p/1534246) | [Notas de la versión](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service-Fabric-71CU2-releasenotes.md)
| 12 de agosto de 2020 | [Tercera actualización de Microsoft Azure Service Fabric 7.1](https://techcommunity.microsoft.com/t5/azure-service-fabric/azure-service-fabric-7-1-third-refresh-release/ba-p/1587586) | [Notas de la versión](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service-Fabric-71CU3-releasenotes.md)
| 10 de septiembre de 2020 | [Cuarta actualización de Microsoft Azure Service Fabric 7.1](https://techcommunity.microsoft.com/t5/azure-service-fabric/azure-service-fabric-7-1-fourth-refresh-release/ba-p/1653859)  | [Notas de la versión](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service-Fabric-71CU5-releasenotes.md)|
| 7 de octubre de 2020 | Sexta actualización de Microsoft Azure Service Fabric 7.1 | [Notas de la versión](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service-Fabric-71CU6-releasenotes.md)|
| 23 de noviembre de 2020 | Octava actualización de Microsoft Azure Service Fabric 7.1 | [Notas de la versión](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service-Fabric-71CU8-releasenotes.md)|


### <a name="service-fabric-70"></a>Service Fabric 7.0

Azure Service Fabric 7.0 ya está disponible. Podrá actualizar a la versión 7.0 mediante Azure Portal o una implementación de Azure Resource Manager. Debido a los comentarios de los clientes sobre las versiones en torno al período de vacaciones, los clústeres establecidos para recibir actualizaciones automáticas no se comenzarán a actualizar automáticamente hasta enero.
En enero, se reanudará el procedimiento de implementación estándar y los clústeres con actualizaciones automáticas habilitadas comenzarán a recibir la actualización 7.0 automáticamente. Antes de que comience la implementación, se proporcionará otro anuncio.
También se actualizarán las fechas de lanzamiento planeadas para indicar que esta directiva se debe tener en cuenta. Mire aquí para encontrar actualizaciones en nuestras futuras [programaciones de versiones](https://github.com/Microsoft/service-fabric/#service-fabric-release-schedule).

#### <a name="key-announcements"></a>Anuncios clave
 - [**Compatibilidad de KeyVaultReference con los secretos de aplicación (versión preliminar)**](./service-fabric-keyvault-references.md): las aplicaciones de Service Fabric que han habilitado [identidades administradas](./concepts-managed-identity.md) ahora pueden hacer referencia directamente a una dirección URL de secreto de Key Vault como una variable de entorno, un parámetro de aplicación o una credencial del repositorio de contenedores. Service Fabric resolverá automáticamente el secreto mediante la identidad administrada de la aplicación. 
     
- **Mejora de la seguridad de actualización en los servicios sin estado**: para garantizar la disponibilidad durante una actualización de la aplicación, se han introducido nuevas configuraciones para definir el [número mínimo de instancias de servicios sin estado](/dotnet/api/system.fabric.description.statelessservicedescription) que se considerarán disponibles. Anteriormente, este valor era 1 para todos los servicios y no se podía modificar. Con esta nueva comprobación de seguridad por servicio, puede estar seguro de que los servicios conservan un número mínimo de instancias durante las actualizaciones de la aplicación, las actualizaciones del clúster y otro mantenimiento que dependa de las comprobaciones de estado y seguridad de Service Fabric.
  
- [**Límites de recursos para los servicios de usuario**](./service-fabric-resource-governance.md#enforcing-the-resource-limits-for-user-services): los usuarios pueden configurar límites de recursos para los servicios de usuario de un nodo a fin de evitar escenarios como el agotamiento de recursos de los servicios del sistema de Service Fabric. 
  
- [**Costo muy alto de la migración de servicios**](./service-fabric-cluster-resource-manager-movement-cost.md) para un tipo de réplica. Las réplicas con un costo de migración muy alto solo se migrarán si hay una infracción de restricción en el clúster que no se pueda corregir de otra manera. Consulte el documento para leer información adicional sobre cuándo el uso de un costo de migración "muy alto" es razonable y para conocer otros aspectos adicionales, consulte los documentos.
  
-  **Comprobaciones adicionales de seguridad del clúster**: en esta versión, se ha introducido una comprobación de seguridad del cuórum de nodo de inicialización que se puede configurar. Esto le permite personalizar cuántos nodos raíz deben estar disponibles durante el ciclo de vida del clúster y los escenarios de administración. Las operaciones que aceptan el clúster por debajo del valor configurado están bloqueadas. Actualmente, el valor predeterminado es siempre un cuórum de los nodos raíz; por ejemplo, si tiene 7 nodos raíz, la operación que le lleve por debajo de 5 nodos raíz se bloquearán. Con este cambio, puede hacer que el valor seguro mínimo sea 6, lo que permitiría que solo un nodo raíz estuviera inactivo cada vez.
   
- Se ha agregado compatibilidad con la [**administración del servicio de copia de seguridad y restauración de Service Fabric Explorer**](./service-fabric-backuprestoreservice-quickstart-azurecluster.md). Como resultado, las siguientes actividades son posibles directamente desde SFX: detectar el servicio de copia de seguridad y restauración, crear directivas de copia de seguridad, habilitar copias de seguridad automáticas, realizar copias de seguridad ad hoc, desencadenar operaciones de restauración y examinar copias de seguridad existentes.

- Anuncio de disponibilidad de [**ReliableCollectionsMissingTypesTool**](https://github.com/hiadusum/ReliableCollectionsMissingTypesTool): esta herramienta ayuda a validar que los tipos que se usan en colecciones confiables son compatibles con versiones anteriores y posteriores durante una actualización gradual de la aplicación. Esto ayuda a evitar errores de actualización o pérdida de datos, así como daños en los datos debido a tipos que faltan o que no son compatibles.

- [**Habilitación de lecturas estables en las réplicas secundarias**](./service-fabric-reliable-services-configuration.md#configuration-names-1): las lecturas estables restringen las réplicas secundarias a devolver valores que se han confirmado en el cuórum.

Además, esta versión contiene otras características nuevas, correcciones de errores y mejoras de compatibilidad, confiabilidad y rendimiento. Para ver la lista completa de todos los cambios, consulte las [notas de la versión](https://github.com/Azure/service-fabric/blob/master/release_notes/Service_Fabric_ReleaseNotes_70.md).

#### <a name="service-fabric-70-releases"></a>Versiones de Service Fabric 7.0

| Fecha de la versión | Release | Más información |
|---|---|---|
| 18 de noviembre de 2019 | [Azure Service Fabric 7.0](https://techcommunity.microsoft.com/t5/Azure-Service-Fabric/Service-Fabric-7-0-Release/ba-p/1015482)  | [Notas de la versión](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service_Fabric_ReleaseNotes_70.md)|
| 30 de enero de 2020 | [Versión de actualización de Azure Service Fabric 7.0](https://techcommunity.microsoft.com/t5/azure-service-fabric/azure-service-fabric-7-0-second-refresh-release/ba-p/1137690)  | [Notas de la versión](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service-Fabric-70CU2-releasenotes.md)|
| 6 de febrero de 2020 | [Versión de actualización de Azure Service Fabric 7.0](https://techcommunity.microsoft.com/t5/azure-service-fabric/azure-service-fabric-7-0-third-refresh-release/ba-p/1156508)  | [Notas de la versión](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service-Fabric-70CU3-releasenotes.md)|
| 2 de marzo de 2020 | [Versión de actualización de Azure Service Fabric 7.0](https://techcommunity.microsoft.com/t5/azure-service-fabric/azure-service-fabric-7-0-fourth-refresh-release/ba-p/1205414)  | [Notas de la versión](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service-Fabric-70CU4-releasenotes.md)
| 6 de mayo de 2020 | [Sexta versión de actualización de Azure Service Fabric 7.0](https://techcommunity.microsoft.com/t5/azure-service-fabric/azure-service-fabric-7-0-sixth-refresh-release/ba-p/1365709) | [Notas de la versión](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service-Fabric-70CU6-releasenotes.md)|
| 9 de octubre de 2020 | Novena versión de actualización de Azure Service Fabric 7.0 | [Notas de la versión](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service-Fabric-70CU9-releasenotes.md)|

### <a name="service-fabric-65"></a>Service Fabric 6.5

Esta versión incluye mejoras de compatibilidad, confiabilidad y rendimiento, características nuevas, correcciones de errores y optimizaciones para facilitar la administración del ciclo de vida de los clústeres y las aplicaciones.

> [!IMPORTANT]
> Service Fabric 6.5 es la última versión con compatibilidad con las herramientas de Service Fabric en Visual Studio 2015. Se recomienda a los clientes cambiar a [Visual Studio 2019](https://visualstudio.microsoft.com/vs/) a partir de ahora.

Novedades de Service Fabric 6.5:

- Service Fabric Explorer incluye un [visor de almacén de imágenes](service-fabric-visualizing-your-cluster.md#image-store-viewer) para inspeccionar las aplicaciones que se han cargado en el almacén de imágenes.

- [Aplicación de orquestación de revisiones (POA)](service-fabric-patch-orchestration-application.md) versión [1.4.0](https://github.com/microsoft/Service-Fabric-POA/releases/tag/v1.4.0) incluye muchas mejoras de autodiagnóstico. Se recomienda a los clientes de POA cambiar a esta versión.

- [El servicio EventStore está habilitado de forma predeterminada](service-fabric-visualizing-your-cluster.md#event-store) para los clústeres de Service Fabric 6.5 a menos que lo rechace.

- Se han agregado [eventos de ciclo de vida de las réplicas](service-fabric-diagnostics-event-generation-operational.md#replica-events) para los servicios con estado.

- [Visibilidad mejorada del estado del nodo de inicialización](service-fabric-understand-and-troubleshoot-with-system-health-reports.md#seed-node-status), incluidas advertencias de nivel de clúster si un nodo raíz es incorrecto (*Inactivo*, *Quitado* o *Desconocido*).

- [La herramienta de recuperación ante desastres para aplicaciones de Service Fabric](https://github.com/Microsoft/Service-Fabric-AppDRTool) permite que los servicios con estado de Service Fabric se recuperen rápidamente cuando se produce un desastre en el clúster principal. Los datos del clúster principal se sincronizan continuamente en la aplicación en espera secundaria mediante operaciones periódicas de restauración y copia de seguridad.

- Compatibilidad de Visual Studio para [publicar aplicaciones .NET Core en clústeres basados en Linux](service-fabric-how-to-publish-linux-app-vs.md).

- Service Fabric 6.5 (y versiones posteriores) instalará [la CLI de Service Fabric (SFCTL)](./service-fabric-cli.md) de forma automática al actualizar o crear un clúster de Linux en Azure.

- [SFCTL](./service-fabric-cli.md) se instala de forma predeterminada en los clústeres OneBox de MacOS y Linux.

Para obtener más información, vea las [notas de la versión de Service Fabric 6.5](https://github.com/Azure/service-fabric/blob/master/release_notes/Service_Fabric_ReleaseNotes_65.pdf).

#### <a name="service-fabric-65-releases"></a>Versiones de Service Fabric 6.5

| Fecha de la versión | Release | Más información |
|---|---|---|
| 11 de junio de 2019 | [Azure Service Fabric 6.5](https://techcommunity.microsoft.com/t5/azure-service-fabric/bg-p/Service-Fabric)  | [Notas de la versión](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service_Fabric_ReleaseNotes_65.pdf)|
| 2 de julio de 2019 | [Versión de actualización de Azure Service Fabric 6.5](https://techcommunity.microsoft.com/t5/azure-service-fabric/bg-p/Service-Fabric)  | [Notas de la versión](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service_Fabric_ReleaseNotes_65CU1.pdf)  |
| 29 de julio de 2019 | [Versión de actualización de Azure Service Fabric 6.5](https://techcommunity.microsoft.com/t5/Azure-Service-Fabric/Azure-Service-Fabric-6-5-Second-Refresh-Release/ba-p/800523)  | [Notas de la versión](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service_Fabric_ReleaseNotes_65CU2.pdf)  |
| 23 de agosto de 2019 | [Versión de actualización de Azure Service Fabric 6.5](https://techcommunity.microsoft.com/t5/Azure-Service-Fabric/Azure-Service-Fabric-6-5-Third-Refresh-Release/ba-p/818599)  | [Notas de la versión](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service_Fabric_ReleaseNotes_65CU3.pdf)  |
| 14 de octubre de 2019 | [Versión de actualización de Azure Service Fabric 6.5](https://techcommunity.microsoft.com/t5/Azure-Service-Fabric/Azure-Service-Fabric-6-5-Fifth-Refresh-Release/ba-p/913296)  | [Notas de la versión](https://github.com/microsoft/service-fabric/blob/master/release_notes/Service_Fabric_ReleaseNotes_65CU5.md  |

### <a name="service-fabric-64-releases"></a>Versiones de Service Fabric 6.4

| Fecha de la versión | Release |
|---|---|
| 30 de noviembre de 2018 | [Azure Service Fabric 6.4](https://blogs.msdn.microsoft.com/azureservicefabric/2018/11/30/azure-service-fabric-6-4-release/) |
| 12 de diciembre de 2018 | [Versión de actualización de Azure Service Fabric 6.4 para clústeres de Windows](https://techcommunity.microsoft.com/t5/azure-service-fabric/bg-p/Service-Fabric) |
| 4 de febrero de 2019 | [Versión de actualización de Azure Service Fabric 6.4](https://techcommunity.microsoft.com/t5/azure-service-fabric/bg-p/Service-Fabric) |
| 4 de marzo de 2019 | [Versión de actualización de Azure Service Fabric 6.4](https://techcommunity.microsoft.com/t5/azure-service-fabric/bg-p/Service-Fabric) |
| 8 de abril de 2019 | [Versión de actualización de Azure Service Fabric 6.4](https://techcommunity.microsoft.com/t5/azure-service-fabric/bg-p/Service-Fabric) |
| 2 de mayo de 2019 | [Versión de actualización de Azure Service Fabric 6.4](https://techcommunity.microsoft.com/t5/azure-service-fabric/bg-p/Service-Fabric) |
| 28 de mayo de 2019 | [Versión de actualización de Azure Service Fabric 6.4](https://techcommunity.microsoft.com/t5/azure-service-fabric/bg-p/Service-Fabric) |
