---
title: Migración de NAS local a recursos compartidos de archivos de Azure
description: Obtenga información sobre cómo migrar archivos de una ubicación de almacenamiento conectado a la red (NAS) local a los recursos compartidos de archivos de Azure a través de Azure Data Box.
author: fauhse
ms.service: storage
ms.topic: how-to
ms.date: 04/02/2021
ms.author: fauhse
ms.subservice: files
ms.openlocfilehash: eb3327ad84310e5dae55103171f7677d5b2c06d1
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108756168"
---
# <a name="use-databox-to-migrate-from-network-attached-storage-nas-to-azure-file-shares"></a>Uso de DataBox para migrar desde un almacenamiento conectado a la red (NAS) a los recursos compartidos de archivos de Azure

Este artículo de migración es uno de varios que implican las palabras clave NAS y Azure Data Box. Compruebe si este artículo se aplica a su escenario:

> [!div class="checklist"]
> * Origen de datos: almacenamiento conectado a la red (NAS)
> * Ruta de migración: NAS &rArr; DataBox &rArr; recurso compartido de archivos de Azure
> * Almacenamiento en caché de archivos locales: no, el objetivo es usar los recursos compartidos de archivos de Azure directamente en la nube. No hay planes para usar Azure File Sync.

Si el escenario es diferente, examine la [tabla de guías de migración](storage-files-migration-overview.md#migration-guides).

Este artículo le guía de forma detallada por el planeamiento, implementación y configuración de las redes necesarias para migrar del dispositivo NAS a recursos compartidos de archivos de Azure funcionales. En esta guía se usa Azure Data Box para el transporte de datos masivo (transporte de datos sin conexión).

## <a name="migration-goals"></a>Objetivos de la migración

El objetivo es migrar los recursos compartidos del dispositivo NAS a Azure y convertirlos en recursos compartidos de archivos de Azure nativos. Puede usar estos recursos compartidos sin necesidad de una instancia de Windows Server. Esta migración se debe realizar de forma que garantice la integridad de los datos de producción y la disponibilidad durante la migración. Esta última requiere que el tiempo de inactividad sea mínimo, para ajustarse o solo superar ligeramente las ventanas de mantenimiento regulares.

## <a name="migration-overview"></a>Información general sobre la migración

El proceso de migración consta de varias fases. Deberá implementar cuentas de almacenamiento y recursos compartidos de archivos de Azure y configurar las redes. A continuación, migrará los archivos mediante Azure DataBox y usará RoboCopy para ponerse al día con los cambios. Por último, hará la transición de los usuarios y las aplicaciones a los recursos compartidos de archivos de Azure recién creados. En las secciones siguientes se describen detalladamente las fases del proceso de migración.

> [!TIP]
> Si vuelve a este artículo, use la navegación del lado derecho para ir a la fase de migración en la que se quedó.

## <a name="phase-1-identify-how-many-azure-file-shares-you-need"></a>Fase 1: Identificación de cuántos recursos compartidos de archivos de Azure se necesitan

[!INCLUDE [storage-files-migration-namespace-mapping](../../../includes/storage-files-migration-namespace-mapping.md)]

## <a name="phase-2-deploy-azure-storage-resources"></a>Fase 2: Implementación de recursos de almacenamiento de Azure

En esta fase, consulte la tabla de asignación de la fase 1 y úsela para aprovisionar el número correcto de cuentas de almacenamiento de Azure y recursos compartidos de archivos que contienen.

[!INCLUDE [storage-files-migration-provision-azfs](../../../includes/storage-files-migration-provision-azure-file-share.md)]

## <a name="phase-3-determine-how-many-azure-databox-appliances-you-need"></a>Fase 3: Determinación del número de dispositivos de Azure DataBox que necesita

Inicie este paso solo cuando haya completado la fase anterior. Los recursos de almacenamiento de Azure Storage (cuentas de almacenamiento y recursos compartidos de archivos) se deben crear en este momento. Durante el pedido de DataBox, debe especificar a qué cuentas de almacenamiento se mueven los datos de DataBox.

En esta fase, debe asignar los resultados del plan de migración de la fase anterior a los límites de las opciones de DataBox disponibles. Estas consideraciones le ayudarán a crear un plan para las opciones de DataBox que debe elegir, así como cuántas necesitará para mover los recursos compartidos de NAS a recursos compartidos de archivos de Azure.

Para determinar el número de dispositivos que necesita de cada tipo, tenga en cuenta estos límites importantes:

* Cualquier instancia de Azure DataBox puede mover datos a un máximo de 10 cuentas de almacenamiento. 
* Cada opción de DataBox tiene su propia capacidad utilizable. Consulte [Opciones de DataBox](#databox-options).

Consulte el plan de migración para conocer el número de cuentas de almacenamiento que ha decidido crear y los recursos compartidos en cada una de ellas. A continuación, examine el tamaño de cada uno de los recursos compartidos de NAS. La combinación de esta información le permitirá optimizar y decidir qué dispositivo debe enviar datos a las cuentas de almacenamiento. Puede tener dos dispositivos de DataBox para trasladar los archivos a la misma cuenta de almacenamiento, pero no divida el contenido de un solo recurso compartido de archivos en dos instancias de DataBox.

### <a name="databox-options"></a>Opciones de DataBox

Para una migración estándar, se debe elegir una combinación de estas tres opciones de DataBox: 

* DataBox Disks: Microsoft le enviará de uno y a cinco discos SSD con una capacidad de 8 TiB cada uno, con un total de 40 TiB como máximo. La capacidad utilizable es aproximadamente un 20 % menos debido a la sobrecarga del sistema de archivos y el cifrado. Para obtener más información, consulte la [Documentación de DataBox Disks](../../databox/data-box-disk-overview.md).
* DataBox: esta es la opción más común. Se le enviará un dispositivo DataBox resistente, que funciona de forma similar a un NAS. Tiene una capacidad utilizable de 80 TiB. Para obtener más información, consulte la [Documentación de DataBox](../../databox/data-box-overview.md).
* DataBox Heavy: esta opción presenta un dispositivo DataBox resistente en ruedas, que funciona de forma similar a un NAS, con una capacidad de 1 PiB. La capacidad utilizable es aproximadamente un 20 % menos debido a la sobrecarga del sistema de archivos y el cifrado. Para obtener más información, consulte la [Documentación de DataBox Heavy](../../databox/data-box-heavy-overview.md).

## <a name="phase-4-provision-a-temporary-windows-server"></a>Fase 4: Aprovisionamiento de una instancia de Windows Server temporal

Mientras espera a que lleguen los dispositivos Azure Data Box, ya puede implementar una o varias instancias de Windows Server que necesitará para ejecutar trabajos de RoboCopy. 

- Estos servidores se usarán primero para copiar los archivos en el dispositivo Data Box.
- Después, se usarán para ponerse al día con los cambios que se han producido en el dispositivo NAS mientras el dispositivo Data Box estaba transportándose. Este enfoque reduce al mínimo el tiempo de inactividad en el lado de origen.

La velocidad con la que funcionan los trabajos de RoboCopy depende principalmente de los siguientes factores:

* El número de IOPS en el almacenamiento de origen y de destino.
* El ancho de banda de red disponible entre ambos. </br> Puede encontrar más detalles en la sección de solución de problemas: [Consideraciones de IOPS y ancho de banda](#iops-and-bandwidth-considerations)
* La capacidad de procesar rápidamente archivos y carpetas en un espacio de nombres. </br> Puede encontrar más detalles en la sección de solución de problemas: [Velocidad de procesamiento](#processing-speed).
* El número de cambios entre ejecuciones de RoboCopy. </br> Puede encontrar más detalles en la sección de solución problemas: [Evitación de trabajo innecesario](#avoid-unnecessary-work).

Es importante tener en cuenta los detalles a los que se hace referencia al elegir la RAM y el número de subprocesos que se proporcionarán a las instancias temporales de Windows Server.

## <a name="phase-5-preparing-to-use-azure-file-shares"></a>Fase 5: Preparación para el uso de recursos compartidos de archivos de Azure

Para ahorrar tiempo, debe continuar con esta fase mientras espera a que llegue el dispositivo Data Box. Con la información de esta fase, podrá decidir cómo se habilitarán los servidores y los usuarios dentro y fuera de Azure para usar los recursos compartidos de archivos de Azure. Las decisiones más críticas son las siguientes:

- **Redes:** habilite las redes para enrutar el tráfico SMB.
- **Autenticación:** configure cuentas de almacenamiento de Azure para la autenticación Kerberos. Unir su cuenta de almacenamiento a un dominio e instancia de AD Connect permitirá que las aplicaciones y los usuarios usen su identidad de AD para la autenticación
- **Autorización:** las ACL de nivel de recurso compartido para cada recurso compartido de archivos de Azure permitirán a los usuarios y grupos de AD acceder a un recurso compartido determinado. Además, dentro de un recurso compartido de archivos de Azure, las ACL nativas de NTFS asumirán el control. La autorización basada en ACL de archivos y carpetas funciona del mismo modo que en los recursos compartidos SMB locales.
- **Continuidad empresarial:** la integración de recursos compartidos de archivos de Azure en un entorno existente suele implicar la conservación de las direcciones de recursos compartidos existentes. Si aún no está usando los espacios de nombres DFS, considere la posibilidad de configurarlos en su entorno. De este modo, podría conservar sin cambios las direcciones de recursos compartidos que usan los usuarios y los scripts. DFS-N se usaría como servicio de enrutamiento de espacios de nombres para SMB, ya que redirigiría los destinos de espacio de nombres DFS a recursos compartidos de archivos de Azure después de la migración.

:::row:::
    :::column:::
        <iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/jd49W33DxkQ" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
    :::column-end:::
    :::column:::
        Este vídeo es una guía y demostración sobre cómo exponer de forma segura recursos compartidos de archivos de Azure directamente para las aplicaciones y trabajadores de la información en cinco sencillos pasos.</br>
        En el vídeo se hace referencia a documentación dedicada para algunos temas:

* [Introducción a las identidades](storage-files-active-directory-overview.md)
* [Unión a un dominio de una cuenta de almacenamiento](storage-files-identity-auth-active-directory-enable.md)
* [Información general sobre redes para recursos compartidos de archivos de Azure](storage-files-networking-overview.md)
* [Configuración de puntos de conexión públicos y privados](storage-files-networking-endpoints.md)
* [Configuración de una VPN S2S](storage-files-configure-s2s-vpn.md)
* [Configuración de una VPN de Windows P2S](storage-files-configure-p2s-vpn-windows.md)
* [Configuración de una VPN P2S de Linux](storage-files-configure-p2s-vpn-linux.md)
* [Configuración del reenvío de DNS](storage-files-networking-dns.md)
* [Configuración de DFS-N](files-manage-namespaces.md)
   :::column-end:::
:::row-end:::

## <a name="phase-6-copy-files-onto-your-databox"></a>Fase 6: Copia de archivos en el dispositivo Data Box

Cuando llegue el dispositivo Data Box, debe configurarlo en una línea de visión del dispositivo NAS. Siga la documentación de configuración del tipo de DataBox que solicitó.

* [Configuración de Data Box](../../databox/data-box-quickstart-portal.md)
* [Configuración de Data Box Disk](../../databox/data-box-disk-quickstart-portal.md)
* [Configuración de Data Box Heavy](../../databox/data-box-heavy-quickstart-portal.md)

En función del tipo de DataBox, puede que haya herramientas de copia de DataBox disponibles. En este momento, no se recomiendan para las migraciones a recursos compartidos de archivos de Azure, ya que no copian los archivos con plena fidelidad en el dispositivo DataBox. En su lugar, use RoboCopy.

Cuando llegue el dispositivo DataBox, tendrá recursos compartidos de SMB aprovisionados previamente disponibles para cada cuenta de almacenamiento que especificó en el momento del pedido.

* Si los archivos entran en un recurso compartido de archivos prémium de Azure, habrá un recurso compartido de SMB por cuenta de almacenamiento de "almacenamiento de archivos" prémium.
* Si los archivos entran en una cuenta de almacenamiento estándar, habrá tres recursos compartidos de SMB por cuenta de almacenamiento estándar (GPv1 y GPv2). Solo los recursos compartidos de archivos que terminan con `_AzFiles` son relevantes para la migración. Omita los recursos compartidos de blob en bloques y en páginas.

Siga los pasos descritos en la documentación de Azure DataBox:

1. [Conexión a un dispositivo Data Box](../../databox/data-box-deploy-copy-data.md)
1. Copia de datos a un dispositivo Data Box
1. [Preparación del dispositivo DataBox para su salida a Azure](../../databox/data-box-deploy-picked-up.md)

La documentación vinculada de DataBox especifica un comando RoboCopy. Sin embargo, el comando no es adecuado para conservar la fidelidad total de archivos y carpetas. En su lugar, use este comando:

```console
Robocopy /MT:32 /NP /NFL /NDL /B /MIR /IT /COPY:DATSO /DCOPY:DAT /UNILOG:<FilePathAndName> <SourcePath> <Dest.Path> 
```
* Para obtener más información acerca de los detalles de cada marca de RoboCopy, consulte la tabla en [sección de RoboCopy](#robocopy) más adelante.
* Para obtener más información acerca de cómo ajustar correctamente el número de subprocesos `/MT:n`, optimizar la velocidad de RoboCopy y hacer que RoboCopy funcione adecuadamente en el centro de datos, eche un vistazo a la [sección de solución de problemas de RoboCopy](#troubleshoot).


## <a name="phase-7-catch-up-robocopy-from-your-nas"></a>Fase 7: Puesta al día de RoboCopy a partir del NAS

Una vez que el dispositivo Data Box informe de que todos los archivos y carpetas se han colocado en los recursos compartidos de archivos de Azure planeados, puede continuar con esta fase.
Un comando RoboCopy de puesta al día solo es necesario si es posible que los datos del NAS hayan cambiado desde que se inició la copia de Data Box. En algunos escenarios en los que se usa un recurso compartido con fines de archivado, es posible que pueda detener los cambios en el recurso compartido del NAS hasta que se complete la migración. También podría cumplir los requisitos de su empresa al configurar recursos compartidos de NAS en modo solo lectura durante la migración.

En los casos en los que necesita que un recurso compartido sea de lectura y escritura durante la migración y solo pueda permitirse un pequeño período de inactividad, será importante que complete este paso de RoboCopy de puesta al día antes de la conmutación por error del acceso del usuario directamente al recurso compartido de archivos de Azure.

En este paso, ejecutará trabajos de RoboCopy para poner al día los recursos compartidos en la nube con los cambios más recientes del NAS desde el momento en que bifurcó los recursos compartidos en el dispositivo DataBox.
Esta puesta al día de RoboCopy puede finalizar rápidamente o tardar unos minutos en función de la cantidad de abandonos que haya ocurrido en los recursos compartidos de NAS.

Ejecute la primera copia local en la carpeta de destino de Windows Server:

1. Identifique la primera ubicación en el dispositivo NAS.
1. Identifique el recurso compartido de archivos de Azure correspondiente.
1. Monte el recurso compartido de archivos de Azure como una unidad de red local en la instancia temporal de Windows Server.
1. Inicie la copia con RoboCopy como se describió.

### <a name="mounting-an-azure-file-share"></a>Montaje de un recurso compartido de archivos de Azure

Antes de que pueda usar RoboCopy, debe hacer que el recurso compartido de archivos de Azure sea accesible a través de SMB. La manera más fácil consiste en montar el recurso compartido como una unidad de red local en la instancia de Windows Server que está planeando usar para RoboCopy. 

> [!IMPORTANT]
> Antes de que pueda montar correctamente un recurso compartido de archivos de Azure en una instancia local de Windows Server, debe haber completado la fase Preparación para el uso de recursos compartidos de archivos de Azure.

Una vez listo, revise el [artículo de procedimientos Uso de un recurso compartido de archivos de Azure con Windows](storage-how-to-use-files-windows.md) y monte el recurso compartido de archivos de Azure para el que quiere iniciar el comando RoboCopy de puesta al día del NAS.

### <a name="robocopy"></a>RoboCopy

El siguiente comando RoboCopy solo copiará las diferencias (archivos y carpetas actualizados) desde el almacenamiento NAS al recurso compartido de archivos de Azure. 

[!INCLUDE [storage-files-migration-robocopy](../../../includes/storage-files-migration-robocopy.md)]

> [!TIP]
> [Consulte la sección de solución de problemas](#troubleshoot) si RoboCopy está afectando a su entorno de producción, si notifica un gran número de errores o si no progresa tan rápido como se espera.

### <a name="user-cut-over"></a>Migración total de los usuarios

Al ejecutar por primera vez el comando RoboCopy, los usuarios y las aplicaciones siguen accediendo a los archivos en la ubicación de NAS y pueden modificarlos. Es posible que RoboCopy haya procesado un directorio, pase al siguiente y, después, un usuario en la ubicación de origen (NAS) agregue, cambie o elimine un archivo que no se procesará en esta ejecución actual de RoboCopy. Este comportamiento es normal.

La primera ejecución consiste en migrar la mayor parte de los datos renovados al recurso compartido de archivos de Azure. Esta primera copia puede tardar unos minutos. Consulte la [sección de solución de problemas](#troubleshoot) para obtener más información acerca de lo que puede afectar a la velocidad de RoboCopy.

Una vez completada la ejecución inicial, vuelva a ejecutar el comando.

La segunda vez que ejecute RoboCopy para el mismo recurso compartido se completará más rápido, ya que solo necesita transportar los cambios posteriores a la última ejecución. Puede ejecutar trabajos repetidos para el mismo recurso compartido.

Si considera que el tiempo de inactividad es aceptable, debe quitar el acceso de usuario a los recursos compartidos basados en NAS. Para ello, siga los pasos que impidan que los usuarios cambien el contenido y la estructura de archivos y carpetas. Un ejemplo es dirigir DFS-Namespace a una ubicación no existente o cambiar las ACL raíz del recurso compartido.

Ejecute una última ronda de RoboCopy. Recuperará todos los cambios que puedan haberse omitido.
El tiempo necesario para realizar este último paso dependerá de la velocidad del análisis de RoboCopy. Para realizar un cálculo estimado del tiempo (que equivale al tiempo de inactividad) averigüe cuánto tardó en realizarse la ejecución anterior.

Cree un recurso compartido en la carpeta de Windows Server y, eventualmente, ajuste esta carpeta como destino de la implementación de DFS-N. Asegúrese de establecer los mismos permisos de nivel de recurso compartido que en el recurso compartido de SMB de NAS. Si tuviera un NAS unido a un dominio de clase empresarial, los SID de usuario coincidirían automáticamente a medida que los usuarios se encuentren en Active Directory y RoboCopy copia los archivos y metadatos con total fidelidad. Si ha utilizado usuarios locales en la ubicación de NAS, debe volver a crearlos como usuarios locales de Windows Server y asignar los SID existentes que RoboCopy ha transferido a la instancia de Windows Server a los SID de los nuevos usuarios locales de Windows Server.

Ha terminado de migrar un recurso compartido o un grupo de recursos compartidos a un volumen o una raíz comunes. (Según la asignación de la fase 1)

Puede intentar ejecutar algunas de estas copias en paralelo. Se recomienda procesar el ámbito de un recurso compartido de archivos de Azure a la vez.

## <a name="troubleshoot"></a>Solución de problemas

[!INCLUDE [storage-files-migration-robocopy-optimize](../../../includes/storage-files-migration-robocopy-optimize.md)]

## <a name="next-steps"></a>Pasos siguientes

Hay más información sobre los recursos compartidos de archivos de Azure. Los artículos siguientes le ayudarán a comprender las opciones avanzadas, los procedimientos recomendados y también contienen ayuda para la solución de problemas. Estos artículos se vinculan a la [documentación de recursos compartidos de archivos de Azure](storage-files-introduction.md) según corresponda.

* [Información general acerca de la migración](storage-files-migration-overview.md)
* [Supervisión, diagnóstico y solución de problemas de Microsoft Azure Storage](../common/storage-monitoring-diagnosing-troubleshooting.md)
* [Consideraciones de red para el acceso directo](storage-files-networking-overview.md)
* [Copia de seguridad: Instantáneas de recursos compartidos de archivos de Azure](storage-snapshots-files.md)
