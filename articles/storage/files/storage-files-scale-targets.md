---
title: Objetivos de escalabilidad y rendimiento de Azure Files
description: Obtenga información sobre los objetivos de escalabilidad y rendimiento de Azure Files, como la capacidad, la tasa de solicitudes, y los límites de ancho de banda entrante y saliente.
author: roygara
ms.service: storage
ms.topic: conceptual
ms.date: 05/28/2021
ms.author: rogarana
ms.subservice: files
ms.openlocfilehash: 78ecf275a9c607273aef16e6351224709f230959
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110690526"
---
# <a name="azure-files-scalability-and-performance-targets"></a>Objetivos de escalabilidad y rendimiento de Azure Files
[Azure Files](storage-files-introduction.md) ofrece recursos compartidos de archivos en la nube totalmente administrados a los que se puede acceder mediante los protocolos SMB y del sistema de archivos NFS. En este artículo se explican los objetivos de escalabilidad y rendimiento de Azure Files y Azure File Sync.

Los objetivos de escalabilidad y rendimiento que aparecen aquí son exigentes, pero pueden verse afectados por otras variables de la implementación. Por ejemplo, el rendimiento de un archivo puede verse limitado también por el ancho de banda de red disponible, no solo los servidores que hospedan los recursos compartidos de archivos de Azure. Se recomienda probar el patrón de uso para determinar si la escalabilidad y el rendimiento de Azure Files cumplen sus requisitos. También nos comprometemos a aumentar estos límites con el tiempo. 

## <a name="azure-files-scale-targets"></a>Objetivos de escalabilidad de Azure Files
Los recursos compartidos de archivos de Azure se implementan en cuentas de almacenamiento, que son objetos de nivel superior que representan un grupo compartido de almacenamiento. Este grupo de almacenamiento se puede usar para implementar varios recursos compartidos de archivos. Por tanto, existen tres categorías que se deben tener en cuenta: cuentas de almacenamiento, recursos compartidos de archivo de Azure y archivos.

### <a name="storage-account-scale-targets"></a>Objetivos de escalabilidad de la cuenta de almacenamiento
Azure admite varios tipos de cuentas de almacenamiento para los distintos escenarios de almacenamiento que pueden tener los clientes, pero hay dos tipos principales de cuentas de almacenamiento para Azure Files. El tipo de cuenta de almacenamiento que debe crear depende de si desea crear un recurso compartido de archivos estándar o un recurso compartido de archivos prémium: 

- **Cuentas de almacenamiento de uso general, versión 2 (GPv2)** : Las cuentas de almacenamiento de GPv2 permiten implementar recursos compartidos de archivos de Azure en hardware estándar o basado en disco duro (HDD). Además de almacenar recursos compartidos de archivos de Azure, las cuentas de almacenamiento de GPv2 pueden almacenar otros recursos de almacenamiento, como contenedores de blobs, colas o tablas. Los recursos compartidos de archivos se pueden implementar en los niveles de transacción optimizada (valor predeterminado), acceso frecuente o acceso esporádico.

- **Cuentas de almacenamiento FileStorage**: Las cuentas de almacenamiento FileStorage permiten implementar recursos compartidos de archivos de Azure en hardware prémium o basado en unidades de estado sólido (SSD). Las cuentas FileStorage solo se pueden usar para almacenar recursos compartidos de archivos de Azure. No se puede implementar ningún otro recurso de almacenamiento (contenedores de blobs, colas, tablas, etc.) en una cuenta FileStorage.

| Atributo | Cuentas de almacenamiento GPv2 (estándar) | Cuentas de almacenamiento FileStorage (prémium) |
|-|-|-|
| Número de cuentas de almacenamiento por suscripción y región | 250 | 250 |
| Capacidad máxima de la cuenta de almacenamiento | 5 PiB<sup>1</sup> | 100 TiB (aprovisionados) |
| Número máximo de recursos compartidos de archivos | Sin límite | Sin límite, el tamaño total aprovisionado de todos los recursos compartidos debe ser inferior al máximo que la capacidad máxima de la cuenta de almacenamiento. |
| Velocidad máxima de solicitudes simultáneas | 20 000 IOPS<sup>1</sup> | 100 000 IOPS |
| Entrada máxima | <ul><li>EE. UU./Europa: 9536 MiB/s<sup>1</sup></li><li>Otras regiones (LRS/ZRS): 9536 MiB/s<sup>1</sup></li><li>Otras regiones (GRS): 4768 MiB/s<sup>1</sup></li></ul> | 4136 MiB/s |
| Salida máxima | 47 683 MiB/s<sup>1</sup> | 6204 MiB/s |
| Número máximo de reglas de red virtual | 200 | 200 |
| Número máximo de reglas de dirección IP | 200 | 200 |
| Operaciones de lectura de administración | 800 por cada 5 minutos | 800 por cada 5 minutos |
| Operaciones de escritura de administración | 10 por segundo/1200 por hora | 10 por segundo/1200 por hora |
| Operaciones de lista de administración | 100 por cada 5 minutos | 100 por cada 5 minutos |

<sup>1</sup> Las cuentas de almacenamiento de uso general, versión 2, admiten límites más altos de capacidad y límites más altos de entrada por solicitud. Para solicitar un aumento en los límites de cuenta, póngase en contacto con el [soporte técnico de Azure](https://azure.microsoft.com/support/faq/).

### <a name="azure-file-share-scale-targets"></a>Objetivos de escala de recursos compartidos de archivos de Azure
| Atributo | Recursos compartidos de archivos estándar<sup>1</sup> | Recursos compartidos de archivos Prémium |
|-|-|-|
| Tamaño máximo de un recurso compartido de archivos | Sin mínimo | 100 GiB (aprovisionados) |
| Unidad de aumento/reducción de tamaño aprovisionada | N/D | 1 GiB |
| Tamaño máximo de un recurso compartido de archivos | <ul><li>100 TiB, con la característica de recurso compartido de archivos grande habilitada<sup>2</sup></li><li>5 TiB, valor predeterminado</li></ul> | 100 TiB |
| Número máximo de archivos en un recurso compartido de archivos | Sin límite | Sin límite |
| Velocidad máxima de solicitud (IOPS máx.) | <ul><li>10 000, con la característica de recurso compartido de archivos grandes habilitada<sup>2</sup></li><li>1000 o 100 solicitudes por 100 ms, valor predeterminado</li></ul> | <ul><li>IOPS base: 400 + 1 IOPS por GiB hasta 100 000</li><li>Expansión de IOPS: Máx. (4000, 3 x IOPS por GiB), hasta 100 000</li></ul> |
| Entrada máxima para un único recurso compartido de archivos | <ul><li>Hasta 300 MiB/s, con la característica de recurso compartido de archivos grandes habilitada<sup>2</sup></li><li>Hasta 60 MiB/s, valor predeterminado</li></ul> | 40 MiB/s + 0,04 * GiB aprovisionado |
| Salida máxima para un único recurso compartido de archivos | <ul><li>Hasta 300 MiB/s, con la característica de recurso compartido de archivos grandes habilitada<sup>2</sup></li><li>Hasta 60 MiB/s, valor predeterminado</li></ul> | 60 MiB/s + 0,06 * GiB aprovisionado |
| Número máximo de instantáneas de recurso compartido | 200 instantáneas | 200 instantáneas |
| Longitud máxima del nombre de objeto (archivos y directorios) | 2048 caracteres | 2048 caracteres |
| Número máximo de componentes de la ruta de acceso (en la ruta de acceso \A\B\C\D, cada letra es un componente) | 255 caracteres | 255 caracteres |
| Límite de vínculo físico (solo NFS) | N/D | 178 |
| Número máximo de canales de SMB multicanal | N/D | 4 |
| Número máximo de directivas de acceso almacenadas por recurso compartido de archivos | 5 | 5 |

<sup>1</sup> Los límites de los recursos compartidos de archivos estándar se aplican a los tres niveles disponibles para dichos recursos: optimizado para transacciones, acceso frecuente y acceso esporádico.

<sup>2</sup> El valor predeterminado en los recursos compartidos de archivos estándar es 5 TiB. En el artículo [Habilitación y creación de recursos compartidos de archivos grandes](./storage-files-how-to-create-large-file-share.md) encontrará más información sobre cómo aumentar el escalado vertical de los recursos compartidos de archivos estándar hasta 100 TiB.

### <a name="file-scale-targets"></a>Objetivos de escalabilidad de archivos
| Atributo | Archivos en recursos compartidos de archivos estándar  | Archivos en recursos compartidos de archivos prémium  |
|-|-|-|
| Tamaño de archivo máximo | 4 TiB | 4 TiB |
| Velocidad máxima de solicitudes simultáneas | 1000 IOPS | Hasta 8000<sup>1</sup> |
| Entrada máxima de un archivo | 60 MiB/s | 200 MiB/s (hasta 1 GiB/s con la versión preliminar de SMB multicanal)<sup>2</sup>|
| Salida máxima de un archivo | 60 MiB/s | 300 MiB/s (hasta 1 GiB/s con la versión preliminar de SMB multicanal)<sup>2</sup> |
| Identificadores simultáneos máximos | 2000 identificadores | 2000 identificadores  |

<sup>1 Se aplica a entradas y salidas de lectura y escritura (normalmente tamaños de E/S menores o iguales que 64 KiB). Las operaciones de metadatos, que no sean lecturas ni escrituras, pueden ser más lentas.</sup>
<sup>2 En función de los límites de red de la máquina, el ancho de banda disponible, los tamaños de E/S, la profundidad de la cola y otros factores. Para más información, vea [Rendimiento de SMB multicanal](./storage-files-smb-multichannel-performance.md).</sup>

## <a name="azure-file-sync-scale-targets"></a>Objetivos de escalabilidad de Azure File Sync
La tabla siguiente indica los límites de las pruebas de Microsoft y también indica qué objetivos son límites estrictos:

| Recurso | Destino | Límite máximo |
|----------|--------------|------------|
| Servicios de sincronización de almacenamiento por región | 100 servicios de sincronización de Storage | Sí |
| Grupos de sincronización por servicio de sincronización de almacenamiento | 200 grupos de sincronización | Sí |
| Servidores registrados por servicio de sincronización de almacenamiento | 99 servidores | Sí |
| Puntos de conexión en la nube por grupo de sincronización | 1 punto de conexión en la nube | Sí |
| Puntos de conexión del servidor por grupo de sincronización | 100 puntos de conexión de servidor | Sí |
| Puntos de conexión del servidor por servidor | 30 puntos de conexión de servidor | Sí |
| Objetos del sistema de archivos (archivos y directorios) por grupo de sincronización | 100 millones de objetos | No |
| Número máximo de objetos del sistema de archivos (archivos y directorios) en un directorio | 5 millones de objetos | Sí |
| Tamaño máximo del descriptor de seguridad de (archivos y directorios) del objeto | 64 KiB | Sí |
| Tamaño de archivo | 100 GiB | No |
| Tamaño mínimo de un archivo que se va a organizar en niveles | V9 y más recientes: basado en el tamaño del clúster del sistema de archivos (tamaño del clúster del sistema de archivos doble). Por ejemplo, si el tamaño del clúster del sistema de archivos es 4 KiB, el tamaño mínimo del archivo será de 8 KiB.<br> V8 y anteriores: 64 KiB  | Sí |

> [!Note]  
> Un punto de conexión de Azure File Sync puede escalar verticalmente hasta el tamaño de un recurso compartido de archivos de Azure. Si se alcanza el límite de tamaño de recurso compartido de archivos de Azure, la sincronización no podrá funcionar.

### <a name="azure-file-sync-performance-metrics"></a>Métricas de rendimiento de Azure File Sync
Dado que el agente de Azure File Sync se ejecuta en una máquina con Windows Server que se conecta a los recursos compartidos de archivos de Azure, el rendimiento de sincronización efectivo depende de una serie de factores de su infraestructura: Windows Server y la configuración del disco subyacente, el ancho de banda de la red entre el servidor y Azure Storage, el tamaño del archivo, el tamaño total del conjunto de datos y la actividad en el conjunto de datos. Dado que Azure File Sync funciona en el nivel de archivos, las características de rendimiento de una solución basada en Azure File Sync se mide mejor en el número de objetos (archivos y directorios) que se procesan por segundo.

En el caso de Azure File Sync, el rendimiento es fundamental en dos fases:

1. **Aprovisionamiento inicial que se realiza una sola vez**: para optimizar el rendimiento del aprovisionamiento inicial, consulte [Incorporación con Azure File Sync](../file-sync/file-sync-deployment-guide.md#onboarding-with-azure-file-sync), donde obtendrá los detalles de una implementación óptima.
2. **Sincronización en curso**: después de que los datos se inicializan en los recursos compartidos de archivos de Azure, Azure File Sync mantiene varios puntos de conexión sincronizados.

Para ayudarle a planear la implementación de cada una de las fases, a continuación encontrará los resultados observados durante las pruebas internas en un sistema con una configuración

| Configuración del sistema | Detalles |
|-|-|
| CPU | 64 núcleos virtuales con una memoria caché L3 de 64 MiB |
| Memoria | 128 GB |
| Disco | Discos SAS con RAID 10 con caché con respaldo de batería |
| Red | Red de 1 Gbps |
| Carga de trabajo | Servidor de archivos de uso general|

| Aprovisionamiento inicial que se realiza una sola vez  | Detalles |
|-|-|
| Número de objetos | 25 millones de objetos |
| Tamaño del conjunto de datos| ~4,7 TiB |
| Tamaño de archivo medio | ~200 KiB (archivo más grande: 100 GiB) |
| Enumeración inicial de cambios de nube | 20 objetos por segundo  |
| Rendimiento de carga | 20 objetos por segundo por grupo de sincronización |
| Rendimiento de descarga de espacio de nombres | 400 objetos por segundo |

### <a name="initial-one-time-provisioning"></a>Aprovisionamiento inicial que se realiza una sola vez

**Enumeración inicial de cambios de nube**: Cuando se crea un nuevo grupo de sincronización, la enumeración inicial de cambios en la nube es el primer paso que se ejecutará. En este proceso, el sistema enumerará todos los elementos del recurso compartido de archivos de Azure. Durante este proceso, no habrá ninguna actividad de sincronización; es decir, no se descargará ningún elemento del punto de conexión de la nube al punto de conexión del servidor, y no se cargará ningún elemento desde el punto de conexión del servidor al punto de conexión en la nube. La actividad de sincronización se reanudará una vez que se complete la enumeración inicial de cambios en la nube.
La tasa de rendimiento es de 20 objetos por segundo. Para calcular el tiempo que se tarda en completar la enumeración inicial de cambios en la nube, los clientes pueden calcular el número de elementos del recurso compartido de nube y usar la siguiente fórmula para obtener el tiempo en días. 

   **Tiempo (en días) para la enumeración inicial en la nube = (número de objetos en el punto de conexión de nube)/(20*60*60*24)**

**Sincronización inicial de datos de Windows Server con un recurso compartido de archivos de Azure**: muchas implementaciones de Azure File Sync comienzan con un recurso compartido de archivos de Azure vacío porque todos los datos están en el servidor de Windows. En estos casos, la enumeración inicial de cambios en la nube es rápida y la mayor parte del tiempo se dedica a sincronizar los cambios de Windows Server con los recursos compartidos de archivos de Azure. 

Mientras la sincronización carga los datos en el recurso compartido de archivos de Azure, no hay tiempo de inactividad en el servidor de archivos local y los administradores pueden [configurar los límites de red](../file-sync/file-sync-server-registration.md#set-azure-file-sync-network-limits) para restringir la cantidad de ancho de banda que se usa para la carga de datos en segundo plano.

La sincronización inicial suele estar limitada por la velocidad de carga inicial de 20 archivos por segundo/por grupo de sincronización. Los clientes pueden calcular el tiempo de carga de todos sus datos en Azure con las siguientes fórmulas para obtener el tiempo en días:  

   **Tiempo (en días) para la carga de archivos en un grupo de sincronización = (número de objetos en el punto de conexión del servidor)/(20*60*60*24)**

Dividir los datos en varios puntos de conexión de servidor y grupos de sincronización puede acelerar esta carga de datos inicial, ya que la operación puede realizarse en paralelo con varios grupos de sincronización a una velocidad de 20 elementos por segundo. Por lo tanto, se ejecutarían dos grupos de sincronización con una tasa combinada de 40 elementos por segundo. El tiempo total para terminar la operación sería el tiempo estimado del grupo de sincronización con el mayor número de archivos que se van a sincronizar.

**Rendimiento de descarga de espacio de nombres** Cuando se agrega un nuevo punto de conexión de servidor a un grupo de sincronización existente, el agente de Azure File Sync no descarga ningún contenido de archivo del punto de conexión en la nube. En primer lugar sincroniza el espacio de nombres completo y, después, desencadena la recuperación en segundo plano para descargar los archivos, ya sea en su totalidad o, si está habilitada la organización en niveles en la nube, la directiva de niveles en la nube establecida en el punto de conexión del servidor.

| Sincronización en curso  | Detalles  |
|-|--|
| Número de objetos sincronizados| 125 000 objetos (renovación ~ 1 %) |
| Tamaño del conjunto de datos| 50 GiB |
| Tamaño de archivo medio | ~500 KiB |
| Rendimiento de carga | 20 objetos por segundo por grupo de sincronización |
| Rendimiento de descarga completa* | 60 objetos por segundo |

*Si están habilitados los niveles de la nube, es probable que observe un rendimiento mejor, ya que solo se descargan algunos de los datos de los archivos. Azure File Sync solo descarga los datos de los archivos almacenados en la memoria caché cuando cambian en cualquiera de los puntos de conexión. En el caso de los archivos en niveles o recién creados, el agente no descarga los datos de los archivos, solo sincroniza el espacio de nombres en todos los puntos de conexión del servidor. El agente también admite descargas parciales de archivos en capas cuando el usuario accede a ellos. 

> [!Note]  
> Los números anteriores no indican el rendimiento que experimentará. El rendimiento real dependerá de varios factores, como se ha indicado al principio de esta sección.

Como guía general para la implementación, debería tener varios factores en cuenta:

- El rendimiento de los objetos se escala aproximadamente en proporción al número de grupos de sincronización del servidor. La división de los datos en varios grupos de sincronización en un servidor mejora el rendimiento, que también está limitado por el servidor y la red.
- El rendimiento de los objetos es inversamente proporcional al rendimiento de MiB por segundo. En archivos pequeños, el rendimiento será mayor en cuanto al número de objetos procesados por segundo, pero el rendimiento en MiB por segundo será inferior. Por el contrario, en archivos grandes, se procesarán menos objetos por segundo, pero el rendimiento en MiB por segundo será superior. El rendimiento en MiB por segundo está limitado por los destinos del escalado de Azure Files.

## <a name="see-also"></a>Consulte también
- [Planeamiento de una implementación de Azure Files](storage-files-planning.md)
- [Planeamiento de una implementación de Azure File Sync](../file-sync/file-sync-planning.md)