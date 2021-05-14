---
title: Migración de NAS local a Azure File Sync
description: Obtenga información sobre cómo migrar archivos de una ubicación de almacenamiento conectado a la red (NAS) local a una implementación de nube híbrida con Azure File Sync y recursos compartidos de archivos de Azure.
author: fauhse
ms.service: storage
ms.topic: how-to
ms.date: 03/19/2020
ms.author: fauhse
ms.subservice: files
ms.openlocfilehash: 01289345ee6bebc0ab1a4608eb83cb8a2827e924
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "108745368"
---
# <a name="migrate-from-network-attached-storage-nas-to-a-hybrid-cloud-deployment-with-azure-file-sync"></a>Migración desde un almacenamiento conectado a la red (NAS) a una implementación de nube híbrida con Azure File Sync

Este artículo de migración es uno de varios que implican las palabras clave NAS y Azure File Sync. Compruebe si este artículo se aplica a su escenario:

> [!div class="checklist"]
> * Origen de datos: almacenamiento conectado a la red (NAS)
> * Ruta de migración: NAS &rArr; Windows Server &rArr; cargar y sincronizar con recursos compartidos de archivos de Azure
> * Almacenamiento en caché de archivos locales: sí, el objetivo final es una implementación de Azure File Sync.

Si el escenario es diferente, examine la [tabla de guías de migración](storage-files-migration-overview.md#migration-guides).

Azure File Sync funciona en ubicaciones de almacenamiento de conexión directa (DAS) y no admite la sincronización con ubicaciones de almacenamiento conectado a la red (NAS).
Es por esto que resulta necesario migrar los archivos y en este artículo encontrará una guía para planear y ejecutar dicha migración.

## <a name="migration-goals"></a>Objetivos de la migración

El objetivo es trasladar los recursos compartidos que tiene en el dispositivo NAS a una instancia de Windows Server. Después, usará Azure File Sync para una implementación de nube híbrida. En general, las migraciones se deben realizar de forma que garanticen la integridad de los datos de producción, así como su disponibilidad durante la migración. Esta última requiere que el tiempo de inactividad sea mínimo, para ajustarse o solo superar ligeramente las ventanas de mantenimiento regulares.

## <a name="migration-overview"></a>Información general sobre la migración

Como se mencionó en el [artículo de información general sobre la migración](storage-files-migration-overview.md), de Azure Files, es importante usar la herramienta de copia y el enfoque correctos. El dispositivo NAS expone los recursos compartidos de SMB directamente en la red local. RoboCopy, integrado en Windows Server, es la mejor manera de trasladar los archivos en este escenario de migración.

- Fase 1: [Identificación de cuántos recursos compartidos de archivos de Azure se necesitan](#phase-1-identify-how-many-azure-file-shares-you-need)
- Fase 2: [Aprovisionamiento de una instancia de Windows Server adecuada en el entorno local](#phase-2-provision-a-suitable-windows-server-on-premises)
- Fase 3: [Implementación del recurso de nube de Azure File Sync](#phase-3-deploy-the-azure-file-sync-cloud-resource)
- Fase 4: [Implementación de recursos de almacenamiento de Azure](#phase-4-deploy-azure-storage-resources)
- Fase 5: [Implementación del agente de Azure File Sync](#phase-5-deploy-the-azure-file-sync-agent)
- Fase 6: [Configuración de Azure File Sync en la instancia de Windows Server](#phase-6-configure-azure-file-sync-on-the-windows-server)
- Fase 7: [RoboCopy](#phase-7-robocopy)
- Fase 8: [Migración total de los usuarios](#phase-8-user-cut-over)

## <a name="phase-1-identify-how-many-azure-file-shares-you-need"></a>Fase 1: Identificación de cuántos recursos compartidos de archivos de Azure se necesitan

[!INCLUDE [storage-files-migration-namespace-mapping](../../../includes/storage-files-migration-namespace-mapping.md)]

## <a name="phase-2-provision-a-suitable-windows-server-on-premises"></a>Fase 2: Aprovisionamiento de una instancia de Windows Server adecuada en el entorno local

* Cree un servidor de Windows Server 2019 (como mínimo de la edición 2012 R2), como una máquina virtual o un servidor físico. También se admite un clúster de conmutación por error de Windows Server.
* Aprovisione o agregue almacenamiento de conexión directa (DAS frente a NAS, que no se admite).

    La cantidad de almacenamiento que aprovisione puede ser menor que la que usa actualmente en el dispositivo NAS. Esta opción de configuración requiere que use también la característica de [nube por niveles](../file-sync/file-sync-cloud-tiering-overview.md) de Azure File Sync.
    Sin embargo, cuando en una fase posterior copie los archivos del espacio de NAS más grande al volumen más pequeño de Windows Server, tendrá que trabajar en lotes:

    1. Mueva un conjunto de archivos que quepa en el disco.
    2. Deje que la sincronización de archivos y la nube por niveles interactúen.
    3. Cuando se cree más espacio disponible en el volumen, continúe con el siguiente lote de archivos. También puede revisar el comando RoboCopy en la próxima [sección RoboCopy](#phase-7-robocopy) para usar el nuevo modificador `/LFSM`. El uso de `/LFSM` puede simplificar significativamente los trabajos de RoboCopy, pero no es compatible con otros modificadores de RoboCopy de los que podría depender.
    
    Puede evitar este enfoque de procesamiento por lotes si aprovisiona el espacio equivalente en la instancia de Windows Server que ocupan los archivos en el dispositivo NAS. Considere la desduplicación en NAS/Windows. Si no quiere confirmar de manera permanente esta gran cantidad de almacenamiento en la instancia de Windows Server, puede reducir el tamaño del volumen después de la migración y antes de ajustar las directivas de nube por niveles. Esto crea una caché local más pequeña de los recursos compartidos de archivos de Azure.

La configuración de recursos (de proceso y RAM) de la instancia de Windows Server que implemente depende principalmente del número de elementos (archivos y carpetas) que se van a sincronizar. Se recomienda trabajar con una configuración de rendimiento más alto si tiene problemas.

[Aprenda a cambiar el tamaño de un servidor de Windows Server según el número de elementos (archivos y carpetas) que necesite sincronizar.](../file-sync/file-sync-planning.md#recommended-system-resources)

> [!NOTE]
> El artículo vinculado anteriormente presenta una tabla con un intervalo de memoria del servidor (RAM). Puede orientarse hacia el número más pequeño para el servidor, pero espere que la sincronización inicial tarde mucho más.

## <a name="phase-3-deploy-the-azure-file-sync-cloud-resource"></a>Fase 3: Implementación del recurso de nube de Azure File Sync

[!INCLUDE [storage-files-migration-deploy-afs-sss](../../../includes/storage-files-migration-deploy-azure-file-sync-storage-sync-service.md)]

## <a name="phase-4-deploy-azure-storage-resources"></a>Fase 4: Implementación de recursos de almacenamiento de Azure

En esta fase, consulte la tabla de asignación de la fase 1 y úsela para aprovisionar el número correcto de cuentas de almacenamiento de Azure y recursos compartidos de archivos que contienen.

[!INCLUDE [storage-files-migration-provision-azfs](../../../includes/storage-files-migration-provision-azure-file-share.md)]

## <a name="phase-5-deploy-the-azure-file-sync-agent"></a>Fase 5: Implementar el agente de Azure File Sync

[!INCLUDE [storage-files-migration-deploy-afs-agent](../../../includes/storage-files-migration-deploy-azure-file-sync-agent.md)]

## <a name="phase-6-configure-azure-file-sync-on-the-windows-server"></a>Fase 6: Configuración de Azure File Sync en el servidor de Windows Server

El servidor registrado de Windows Server debe estar preparado y conectado a Internet para este proceso.

[!INCLUDE [storage-files-migration-configure-sync](../../../includes/storage-files-migration-configure-sync.md)]

> [!IMPORTANT]
> La nube por niveles es la característica de Azure File Sync que permite al servidor local tener menos capacidad de almacenamiento de la que está almacenada en la nube y, a pesar de ello, disponer de todo el espacio de nombres. Los datos de interés local también se almacenan en caché para conseguir un rendimiento de acceso rápido. La nube por niveles es una característica opcional de cada "punto de conexión de servidor" de Azure File Sync.

> [!WARNING]
> Si aprovisionó menos almacenamiento en los volúmenes del servidor de Windows que los datos usados en el dispositivo NAS, la nube por niveles es obligatoria. Si no activa la nube por niveles, el servidor no liberará espacio para almacenar todos los archivos. De forma temporal para realizar la migración, establezca la directiva de almacenamiento por niveles en el 99 % de espacio disponible en el volumen. Asegúrese de volver a la configuración de nube por niveles una vez que se haya completado la migración y establézcala en un nivel de utilidad a más largo plazo.

Repita los pasos de creación de grupos de sincronización y adición de la carpeta de servidor correspondiente como un punto de conexión de servidor para todos los recursos compartidos de archivos de Azure o las ubicaciones del servidor, que deban configurarse para la sincronización.

Después de crear todos los puntos de conexión de servidor, la sincronización funciona. Puede crear un archivo de prueba y ver que se sincroniza desde la ubicación del servidor con el recurso compartido de archivos de Azure conectado (como se describe en el punto de conexión en la nube del grupo de sincronización).

En las dos ubicaciones, las carpetas del servidor y los recursos compartidos de archivos de Azure están vacíos y a la espera de datos. En el paso siguiente, comenzará a copiar archivos a la instancia de Windows Server para que Azure File Sync los transfiera a la nube. En caso de que haya habilitado la nube por niveles, el servidor comenzará a organizar los archivos en niveles, si se queda sin capacidad en los volúmenes locales.

## <a name="phase-7-robocopy"></a>Fase 7: RoboCopy

El enfoque de migración básico es una ejecución de RoboCopy del dispositivo NAS a Windows Server y de Azure File Sync a los recursos compartidos de archivos de Azure.

Ejecute la primera copia local en la carpeta de destino de Windows Server:

* Identifique la primera ubicación en el dispositivo NAS.
* Identifique la carpeta coincidente en el servidor de Windows Server, que ya tiene Azure File Sync configurado.
* Inicie la copia con RoboCopy.

El comando de RoboCopy siguiente copiará los archivos desde el almacenamiento NAS a la carpeta de destino de la instancia de Windows Server. Windows Server los sincronizará con los recursos compartidos de archivos de Azure. 

Si ha aprovisionado menos almacenamiento en la instancia de Windows Server que el que ocupan los archivos en el dispositivo NAS, ha configurado la nube por niveles. A medida que el volumen local de Windows Server se llena, la [nube por niveles](../file-sync/file-sync-cloud-tiering-overview.md) se iniciará y organizará por niveles los archivos que ya se han sincronizado correctamente. La nube por niveles generará espacio suficiente para continuar con la copia desde el dispositivo NAS. La nube por niveles realiza comprobaciones cada hora para averiguar qué se ha sincronizado y para liberar espacio en disco para alcanzar el 99 % de espacio libre del volumen.
Es posible que RoboCopy mueva los archivos más rápido de lo que se pueden sincronizar con la nube y el nivel localmente, y por tanto quedarse sin espacio en el disco local. Se producirá un error en RoboCopy. Se recomienda trabajar con los recursos compartidos en una secuencia que lo evite. Por ejemplo, no inicie trabajos de RoboCopy para todos los recursos compartidos al mismo tiempo, o bien transfiera solo los recursos compartidos que se ajusten a la cantidad actual de espacio disponible en la instancia de Windows Server, por mencionar algunas soluciones.

[!INCLUDE [storage-files-migration-robocopy](../../../includes/storage-files-migration-robocopy.md)]

## <a name="phase-8-user-cut-over"></a>Fase 8: Migración total de los usuarios

Al ejecutar por primera vez el comando RoboCopy, los usuarios y las aplicaciones siguen accediendo a los archivos en la ubicación de NAS y pueden modificarlos. Es posible que RoboCopy haya procesado un directorio, pase al siguiente y, después, un usuario en la ubicación de origen (NAS) agregue, cambie o elimine un archivo que no se procesará en esta ejecución actual de RoboCopy. Este comportamiento es normal.

La primera ejecución consiste en mover la mayor parte de los datos a la instancia de Windows Server y, después, a la nube a través de Azure File Sync. Esta primera copia puede tardar mucho tiempo, en función de lo siguiente:

* El ancho de banda de descarga.
* El ancho de banda de carga.
* La velocidad de la red local y el número de subprocesos de RoboCopy que coincidan de forma óptima con ella.
* El número de elementos (archivos y carpetas) que deben procesar RoboCopy y Azure File Sync.

Una vez completada la ejecución inicial, vuelva a ejecutar el comando.

La segunda vez se completará más rápido, ya que solo necesita transportar los cambios posteriores a la última ejecución. Durante esta segunda ejecución, todavía se pueden acumular nuevos cambios.

Repita este proceso hasta que considere que el tiempo que tarda en completarse una operación de RoboCopy para una ubicación concreta se encuentra dentro una ventana de inactividad aceptable.

Si considera que el tiempo de inactividad es aceptable, debe quitar el acceso de usuario a los recursos compartidos basados en NAS. Para ello, siga los pasos que impidan que los usuarios cambien el contenido y la estructura de archivos y carpetas. Un ejemplo es dirigir DFS-Namespace a una ubicación no existente o cambiar las ACL raíz del recurso compartido.

Ejecute una última ronda de RoboCopy. Recuperará todos los cambios que puedan haberse omitido.
El tiempo necesario para realizar este último paso dependerá de la velocidad del análisis de RoboCopy. Para realizar un cálculo estimado del tiempo (que equivale al tiempo de inactividad) averigüe cuánto tardó en realizarse la ejecución anterior.

Cree un recurso compartido en la carpeta de Windows Server y, eventualmente, ajuste esta carpeta como destino de la implementación de DFS-N. Asegúrese de establecer los mismos permisos de nivel de recurso compartido que en el recurso compartido de SMB de NAS. Si tuviera un NAS unido a un dominio de clase empresarial, los SID de usuario coincidirían automáticamente a medida que los usuarios se encuentren en Active Directory y RoboCopy copia los archivos y metadatos con total fidelidad. Si ha utilizado usuarios locales en la ubicación de NAS, debe volver a crearlos como usuarios locales de Windows Server y asignar los SID existentes que RoboCopy ha transferido a la instancia de Windows Server a los SID de los nuevos usuarios locales de Windows Server.

Ha terminado de migrar un recurso compartido o un grupo de recursos compartidos a un volumen o una raíz comunes. (Según la asignación de la fase 1)

Puede intentar ejecutar algunas de estas copias en paralelo. Se recomienda procesar el ámbito de un recurso compartido de archivos de Azure a la vez.

> [!WARNING]
> Cuando haya movido todos los datos de su ubicación de NAS a Windows Server y se haya completado la migración: vuelva a ***todos*** los grupos de sincronización de Azure Portal y ajuste el valor porcentual de espacio libre en el volumen de la nube por niveles a un valor más adecuado para el uso de la memoria caché, por ejemplo, un 20 %. 

La directiva de espacio libre en el volumen de la nube por niveles actúa en un nivel de volumen desde el que se pueden sincronizar varios puntos de conexión de servidor. Si olvida ajustar el espacio disponible en un punto de conexión del servidor, la sincronización seguirá aplicando la regla más restrictiva e intentará mantener un 99 % de espacio libre en disco, lo que hará que la memoria caché local no funcione según lo previsto. A menos que el objetivo sea tener solamente el espacio de nombres para un volumen que solo contiene datos de archivo a los que se accede con poca frecuencia y reserve el resto del espacio de almacenamiento para otro escenario.

## <a name="troubleshoot"></a>Solución de problemas

El problema que puede experimentar más probablemente, es que el comando de RoboCopy produzca el error *"Volumen lleno"* en el lado de Windows Server. La nube por niveles actúa una vez cada hora para evacuar el contenido del disco local de Windows Server, que se ha sincronizado. Su objetivo es alcanzar el 99 % de espacio libre en el volumen.

Permita que el progreso de la sincronización y la nube por niveles liberen espacio en disco. Puede observarlo en el Explorador de archivos en Windows Server.

Cuando Windows Server tenga capacidad suficiente disponible, el problema se resolverá al volver a ejecutar el comando. Si se da esta situación, no se interrumpe nada y puede avanzar con confianza. La única consecuencia es el inconveniente de tener que volver a ejecutar el comando.

Consulte el vínculo de la sección siguiente para solucionar problemas de Azure File Sync.

## <a name="next-steps"></a>Pasos siguientes

Hay más información sobre los recursos compartidos de archivos de Azure y Azure File Sync. Los artículos siguientes le ayudarán a comprender las opciones avanzadas, los procedimientos recomendados y también contienen ayuda para la solución de problemas. Estos artículos se vinculan a la [documentación de recursos compartidos de archivos de Azure](storage-files-introduction.md) según corresponda.

* [Introducción a Azure File Sync](../file-sync/file-sync-planning.md)
* [Implementación de Azure File Sync](../file-sync/file-sync-deployment-guide.md)
* [Solución de problemas de Azure File Sync](../file-sync/file-sync-troubleshoot.md)