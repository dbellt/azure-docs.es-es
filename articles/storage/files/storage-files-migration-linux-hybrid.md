---
title: Migración de Linux a Azure File Sync
description: Aprenda a migrar archivos desde una ubicación de servidor Linux a una implementación de nube híbrida con Azure File Sync y recursos compartidos de archivos de Azure.
author: fauhse
ms.service: storage
ms.topic: how-to
ms.date: 03/19/2020
ms.author: fauhse
ms.subservice: files
ms.openlocfilehash: 30a0269b5729516d8e8e378c700c493262e77f10
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "108756186"
---
# <a name="migrate-from-linux-to-a-hybrid-cloud-deployment-with-azure-file-sync"></a>Migración desde Linux a una implementación de nube híbrida con Azure File Sync

Este artículo de migración es uno de varios que implican las palabras clave NFS y Azure File Sync. Compruebe si este artículo se aplica a su escenario:

> [!div class="checklist"]
> * Origen de datos: almacenamiento conectado a la red (NAS)
> * Ruta de migración: servidor Linux con Samba &rArr; Windows Server 2012R2 o posterior &rArr; sincronización con recursos compartidos de archivos de Azure
> * Almacenamiento en caché de archivos locales: sí, el objetivo final es una implementación de Azure File Sync.

Si el escenario es diferente, examine la [tabla de guías de migración](storage-files-migration-overview.md#migration-guides).

Azure File Sync funciona en instancias de Windows Server con almacenamiento conectado directamente (DAS). No admite la sincronización hacia y desde clientes Linux, un recurso compartido de Bloque de mensajes del servidor (SMB) remoto o recursos compartidos de Network File System (NFS).

Como resultado, la transformación de los servicios de archivo en una implementación híbrida realiza una migración a una instancia de Windows Server necesaria. Este artículo le guía a través del planeamiento y la ejecución de este tipo de migración.

## <a name="migration-goals"></a>Objetivos de la migración

El objetivo es trasladar los recursos compartidos que tiene en el servidor Samba de Linux a una instancia de Windows Server. Después, usará Azure File Sync para una implementación de nube híbrida. Esta migración se debe realizar de forma que garantice la integridad de los datos de producción y la disponibilidad durante la migración. Esta última requiere que el tiempo de inactividad sea mínimo, para ajustarse o solo superar ligeramente las ventanas de mantenimiento regulares.

## <a name="migration-overview"></a>Información general sobre la migración

Como se ha mencionado en el [artículo de información general sobre la migración](storage-files-migration-overview.md), de Azure Files, es importante usar la herramienta de copia y el enfoque correctos. El servidor Samba de Linux expone recursos compartidos de SMB directamente en la red local. Robocopy, integrado en Windows Server, es la mejor manera de trasladar los archivos en este escenario de migración.

Si no ejecuta Samba en el servidor de Linux y prefiere migrar carpetas a una implementación híbrida en una instancia de Windows Server, puede usar las herramientas de copia de Linux en lugar de RoboCopy. Tenga en cuenta las capacidades de fidelidad de la herramienta de copia. Revise la [sección de aspectos básicos de la migración ](storage-files-migration-overview.md#migration-basics) en el artículo Información general sobre la migración para obtener información sobre lo que se debe buscar en una herramienta de copia.

## <a name="phase-1-identify-how-many-azure-file-shares-you-need"></a>Fase 1: Identificación de cuántos recursos compartidos de archivos de Azure se necesitan

[!INCLUDE [storage-files-migration-namespace-mapping](../../../includes/storage-files-migration-namespace-mapping.md)]

## <a name="phase-2-provision-a-suitable-windows-server-instance-on-premises"></a>Fase 2: Aprovisionamiento de una instancia de Windows Server adecuada en el entorno local

* Cree una instancia de Windows Server 2019 como una máquina virtual o un servidor físico. El requisito mínimo es Windows Server 2012 R2. También se admite un clúster de conmutación por error de Windows Server.
* Aprovisione o agregue almacenamiento de conexión directa (DAS). No se admite el almacenamiento conectado a la red (NAS).

  La cantidad de almacenamiento que aprovisione puede ser menor que la que usa actualmente en el servidor Samba de Linux, si usa la característica de [nube por niveles](../file-sync/file-sync-cloud-tiering-overview.md) de Azure File Sync. 

La cantidad de almacenamiento que aprovisione puede ser menor que la que usa actualmente en el servidor Samba de Linux. Esta opción de configuración requiere que use también la característica de [nube por niveles](../file-sync/file-sync-cloud-tiering-overview.md) de Azure File Sync. Sin embargo, cuando en una fase posterior copie los archivos del espacio del servidor Samba de Linux más grande al volumen más pequeño de Windows Server, tendrá que trabajar en lotes:

  1. Mueva un conjunto de archivos que quepa en el disco.
  2. Deje que la sincronización de archivos y la nube por niveles interactúen.
  3. Cuando se cree más espacio disponible en el volumen, continúe con el siguiente lote de archivos. También puede revisar el comando RoboCopy en la próxima [sección RoboCopy](#phase-7-robocopy) para usar el nuevo modificador `/LFSM`. El uso de `/LFSM` puede simplificar significativamente los trabajos de RoboCopy, pero no es compatible con otros modificadores de RoboCopy de los que podría depender.
    
  Puede evitar este enfoque de procesamiento por lotes si aprovisiona el espacio equivalente en la instancia de Windows Server que ocupan los archivos en el servidor Samba de Linux. Considere la posibilidad de habilitar la desduplicación en Windows. Si no quiere confirmar de manera permanente esta gran cantidad de almacenamiento en la instancia de Windows Server, puede reducir el tamaño del volumen después de la migración y antes de ajustar las directivas de nube por niveles. Esto crea una caché local más pequeña de los recursos compartidos de archivos de Azure.

La configuración de recursos (de proceso y RAM) de la instancia de Windows Server que implemente depende principalmente del número de elementos (archivos y carpetas) que se van a sincronizar. Se recomienda trabajar con una configuración de rendimiento más alto si tiene problemas.

[Aprenda a cambiar el tamaño de una instancia de Windows Server según el número de elementos (archivos y carpetas) que necesite sincronizar.](../file-sync/file-sync-planning.md#recommended-system-resources)

> [!NOTE]
> El artículo vinculado anteriormente presenta una tabla con un intervalo de memoria del servidor (RAM). Puede orientarse hacia el número más pequeño para el servidor, pero espere que la sincronización inicial tarde mucho más.

## <a name="phase-3-deploy-the-azure-file-sync-cloud-resource"></a>Fase 3: Implementación del recurso de nube de Azure File Sync

[!INCLUDE [storage-files-migration-deploy-afs-sss](../../../includes/storage-files-migration-deploy-azure-file-sync-storage-sync-service.md)]

## <a name="phase-4-deploy-azure-storage-resources"></a>Fase 4: Implementación de recursos de almacenamiento de Azure

En esta fase, consulte la tabla de asignación de la fase 1 y úsela para aprovisionar el número correcto de cuentas de almacenamiento de Azure y recursos compartidos de archivos que contienen.

[!INCLUDE [storage-files-migration-provision-azfs](../../../includes/storage-files-migration-provision-azure-file-share.md)]

## <a name="phase-5-deploy-the-azure-file-sync-agent"></a>Fase 5: Implementar el agente de Azure File Sync

[!INCLUDE [storage-files-migration-deploy-afs-agent](../../../includes/storage-files-migration-deploy-azure-file-sync-agent.md)]

## <a name="phase-6-configure-azure-file-sync-on-the-windows-server-deployment"></a>Fase 6: Configurar Azure File Sync en la implementación de Windows Server

La instancia registrada de Windows Server local debe estar preparada y conectada a Internet para este proceso.

[!INCLUDE [storage-files-migration-configure-sync](../../../includes/storage-files-migration-configure-sync.md)]

> [!IMPORTANT]
> La nube por niveles es la característica de Azure File Sync que permite al servidor local tener menos capacidad de almacenamiento de la que está almacenada en la nube y, a pesar de ello, disponer de todo el espacio de nombres. Los datos de interés local también se almacenan en caché para conseguir un rendimiento de acceso rápido. La nube por niveles es una característica opcional de cada punto de conexión de servidor de Azure File Sync.

> [!WARNING]
> Si ha aprovisionado menos almacenamiento en los volúmenes de Windows Server que los datos usados en el servidor Samba de Linux, la nube por niveles es obligatoria. Si no activa la nube por niveles, el servidor no liberará espacio para almacenar todos los archivos. De forma temporal para realizar la migración, establezca la directiva de almacenamiento por niveles en el 99 % de espacio disponible en el volumen. Asegúrese de volver a la configuración de nube por niveles una vez que se haya completado la migración y establezca la directiva en un nivel más útil a largo plazo.

Repita los pasos de creación de grupos de sincronización y adición de la carpeta de servidor correspondiente como un punto de conexión de servidor para todos los recursos compartidos de archivos de Azure y las ubicaciones del servidor, que deban configurarse para la sincronización.

Después de crear todos los puntos de conexión de servidor, la sincronización funciona. Puede crear un archivo de prueba y ver que se sincroniza desde la ubicación del servidor con el recurso compartido de archivos de Azure conectado (como se describe en el punto de conexión en la nube del grupo de sincronización).

En ambas ubicaciones, las carpetas del servidor y los recursos compartidos de archivos de Azure están vacíos y a la espera de datos. En el paso siguiente, comenzará a copiar archivos a la instancia de Windows Server para que Azure File Sync los transfiera a la nube. Si habilitó la nube por niveles, el servidor comenzará a organizar los archivos en niveles si se queda sin capacidad en los volúmenes locales.

## <a name="phase-7-robocopy"></a>Fase 7: Robocopy

El enfoque de migración básico es usar Robocopy para copiar archivos y usar Azure File Sync para realizar la sincronización.

Ejecute la primera copia local en la carpeta de destino de Windows Server:

1. Identifique la primera ubicación en el servidor Samba de Linux.
1. Identifique la carpeta coincidente en la instancia de Windows Server, que ya tiene Azure File Sync configurado.
1. Inicie la copia con Robocopy.

El comando de Robocopy siguiente copiará los archivos desde el almacenamiento de los servidores Samba de Linux a la carpeta de destino de la instancia de Windows Server. Windows Server los sincronizará con los recursos compartidos de archivos de Azure. 

Si ha aprovisionado menos almacenamiento en la instancia de Windows Server que el que ocupan los archivos en el servidor Samba de Linux, ha configurado la nube por niveles. A medida que el volumen local de Windows Server se llena, la [nube por niveles](../file-sync/file-sync-cloud-tiering-overview.md) se iniciará y organizará por niveles los archivos que ya se han sincronizado correctamente. La nube por niveles generará espacio suficiente para continuar con la copia desde el servidor Samba de Linux. La nube por niveles realiza comprobaciones cada hora para averiguar qué se ha sincronizado y para liberar espacio en disco para alcanzar la directiva del 99 % de espacio libre para un volumen.

Es posible que Robocopy mueva los archivos más rápido de lo que se pueden sincronizar con la nube y los organice por niveles de manera local, lo que hará que se quede sin espacio en el disco local. Robocopy generará un error. Se recomienda trabajar con los recursos compartidos en una secuencia que evite este problema. Por ejemplo, considere la posibilidad de no iniciar trabajos de Robocopy en todos los recursos compartidos al mismo tiempo. O bien considere la posibilidad de mover los recursos compartidos que se ajustan a la cantidad actual de espacio libre en la instancia de Windows Server. Si se produce un error en el trabajo de Robocopy, puede volver a ejecutar el comando siempre que use la opción de reflejo/purga siguiente:

[!INCLUDE [storage-files-migration-robocopy](../../../includes/storage-files-migration-robocopy.md)]

## <a name="phase-8-user-cut-over"></a>Fase 8: Migración total de los usuarios

Al ejecutar por primera vez el comando Robocopy, los usuarios y las aplicaciones siguen accediendo a los archivos en el servidor Samba de Linux y pueden modificarlos. Es posible que Robocopy haya procesado un directorio, pase al siguiente y, después, un usuario en la ubicación de origen (Linux) agregue, cambie o elimine un archivo que no se procesará en esta ejecución actual de Robocopy. Este comportamiento es normal.

La primera ejecución consiste en mover la mayor parte de los datos a la instancia de Windows Server y, después, a la nube a través de Azure File Sync. Esta primera copia puede tardar mucho tiempo, en función de lo siguiente:

* El ancho de banda de descarga.
* El ancho de banda de carga.
* La velocidad de la red local y el número de subprocesos de Robocopy que coincidan de forma óptima con ella.
* El número de elementos (archivos y carpetas) que deben procesar Robocopy y Azure File Sync.

Una vez completada la ejecución inicial, vuelva a ejecutar el comando.

La segunda vez se completará más rápido, ya que solo necesita transportar los cambios posteriores a la última ejecución. Durante esta segunda ejecución, de todos modos se pueden acumular cambios nuevos.

Repita este proceso hasta que considere que el tiempo que tarda en completarse una operación de Robocopy para una ubicación concreta se encuentra dentro una ventana de inactividad aceptable.

Cuando considere que el tiempo de inactividad es aceptable y esté preparado para dejar sin conexión la ubicación de Linux, puede cambiar las ACL en la raíz del recurso compartido para que los usuarios ya no puedan acceder a la ubicación. También puede realizar cualquier otro paso adecuado que impida que el contenido cambie en esta carpeta en el servidor Linux.

Ejecute una última ronda de Robocopy. Recuperará todos los cambios que puedan haberse omitido. El tiempo necesario para hacer este último paso depende de la velocidad del análisis de Robocopy. Para realizar un cálculo estimado del tiempo (que equivale al tiempo de inactividad) averigüe cuánto tardó en realizarse la ejecución anterior.

Cree un recurso compartido en la carpeta de Windows Server y, eventualmente, ajuste esta carpeta como destino de la implementación de DFS-N. Asegúrese de establecer los mismos permisos de nivel de recurso compartido que en el recurso compartido de SMB del servidor Samba de Linux. Si ha usado usuarios locales en el servidor Samba de Linux, debe volver a crearlos como usuarios locales de Windows Server. También debe asignar los SID existentes que Robocopy ha migrado a su instancia de Windows Server a los SID de los usuarios locales nuevos de Windows Server. Si ha usado cuentas y ACL de Active Directory, Robocopy las transferirá tal cual y no es necesario realizar ninguna otra acción.

Terminó de migrar un recurso compartido o un grupo de recursos compartidos a una raíz o un volumen común (en función de la asignación de la fase 1).

Puede intentar ejecutar algunas de estas copias en paralelo. Se recomienda procesar el ámbito de un recurso compartido de archivos de Azure a la vez.

> [!WARNING]
> Una vez que haya migrado todos los datos del servidor Samba de Linux a la instancia de Windows Server y se haya completado la migración, vuelva a *todos* los grupos de sincronización de Azure Portal. Ajuste el porcentaje de espacio disponible para el volumen de nube por niveles a un número más adecuado para el uso de la memoria caché, como el 20 por ciento. 

La directiva de espacio libre en el volumen de la nube por niveles actúa en un nivel de volumen desde el que se pueden sincronizar varios puntos de conexión de servidor. Si olvida ajustar el espacio disponible en un punto de conexión del servidor, la sincronización seguirá aplicando la regla más restrictiva e intentará mantener un 99 % de espacio libre en disco. Esto hará que la caché local no funcione según lo previsto. El rendimiento puede ser aceptable si el objetivo es tener el espacio de nombres de un volumen que contenga solo datos de archivo de acceso menos frecuente y se está reservando el resto del espacio de almacenamiento para otro escenario.

## <a name="troubleshoot"></a>Solución de problemas

El problema más común es que el comando de Robocopy genere el error de **Volumen lleno** en el lado de Windows Server. La nube por niveles actúa una vez cada hora para evacuar el contenido del disco local de Windows Server, que se ha sincronizado. Su objetivo es alcanzar un espacio libre del 99 por ciento en el volumen.

Permita que el progreso de la sincronización y la nube por niveles liberen espacio en disco. Puede observarlo en el Explorador de archivos en Windows Server.

Cuando la instancia de Windows Server tenga capacidad suficiente disponible, el problema se resolverá al volver a ejecutar el comando. Si se da esta situación, no se interrumpe nada y puede avanzar con confianza. La única consecuencia es el inconveniente de tener que volver a ejecutar el comando.

Consulte el vínculo de la sección siguiente para solucionar problemas de Azure File Sync.

## <a name="next-steps"></a>Pasos siguientes

Hay más información sobre los recursos compartidos de archivos de Azure y Azure File Sync. Los artículos siguientes contienen opciones avanzadas, procedimientos recomendados y ayuda para la solución de problemas. Estos artículos se vinculan a la [documentación de recursos compartidos de archivos de Azure](storage-files-introduction.md) según corresponda.

* [Introducción a Azure File Sync](../file-sync/file-sync-planning.md)
* [Implementación de Azure File Sync](../file-sync/file-sync-deployment-guide.md)
* [Solución de problemas de Azure File Sync](../file-sync/file-sync-troubleshoot.md)