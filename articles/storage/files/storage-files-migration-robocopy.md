---
title: Migración a recursos compartidos de archivos de Azure con RoboCopy
description: Aprenda a migrar archivos desde recursos compartidos de archivos de Azure de distintas ubicaciones con RoboCopy.
author: fauhse
ms.service: storage
ms.topic: how-to
ms.date: 04/12/2021
ms.author: fauhse
ms.subservice: files
ms.openlocfilehash: 53c0ad42e51e8ffc562827e9a67e01b132dafd89
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108777048"
---
# <a name="use-robocopy-to-migrate-to-azure-file-shares"></a>Uso de RoboCopy para migrar a recursos compartidos de archivos de Azure

En este artículo de migración se describe el uso de RoboCopy para mover o migrar archivos a un recurso compartido de archivos de Azure. RoboCopy es una utilidad de copia de archivos de confianza y conocida con un conjunto de características que la hace adecuada para las migraciones. Usa el protocolo SMB, por lo que se puede aplicar de forma amplia a cualquier combinación de origen y destino compatible con SMB.

> [!div class="checklist"]
> * Orígenes de datos: cualquier origen que admita el protocolo SMB, como Almacenamiento conectado a la red (NAS), servidores Windows o Linux, otro recurso compartido de archivos de Azure y muchos más.
> * Ruta de migración: desde el almacenamiento de origen &rArr; máquina Windows con RoboCopy &rArr; recurso compartido de archivos.

Hay muchas rutas de migración diferentes para diversas combinaciones de origen e implementación. Busque en la [tabla de guías de migración](storage-files-migration-overview.md#migration-guides) la migración que mejor se adapte a sus necesidades.

## <a name="azcopy-vs-robocopy"></a>AzCopy frente a RoboCopy
AzCopy y RoboCopy son dos herramientas de copia de archivos radicalmente diferentes. RoboCopy usa cualquier versión del protocolo SMB. AzCopy es una herramienta nativa de la nube que se puede usar para mover datos siempre que el destino esté en Azure Storage. AzCopy depende de un protocolo REST.

RoboCopy, como herramienta de copia basada en Windows de confianza, tiene la ventaja de copiar archivos con plena fidelidad. RoboCopy admite muchos escenarios de migración debido a su amplio conjunto de características y a la capacidad de copiar archivos y carpetas con plena fidelidad. Consulte la sección [fidelidad de archivos del artículo de información general sobre la migración](storage-files-migration-overview.md#migration-basics) para obtener más información sobre la importancia de copiar archivos con la máxima fidelidad posible.

AzCopy, por otro lado, se ha expandido recientemente para admitir la copia de archivos con cierta fidelidad y ha agregado las primeras características necesarias para considerarse como una herramienta de migración. Sin embargo, todavía hay lagunas y pueden surgir malentendidos con facilidad en relación con la funcionalidad al comparar las marcas de AzCopy con las de RoboCopy.

Un ejemplo: *RoboCopy /MIR* reflejará el origen en el destino, lo que significa que se tienen en cuenta los archivos agregados, modificados y eliminados. Una diferencia importante de *AzCopy -sync* es que los archivos eliminados en el origen no se quitarán del destino. Por ello, el conjunto de características de copia diferencial es incompleto. AzCopy continuará evolucionando. En este momento, AzCopy no es una herramienta recomendada para escenarios de migración con recursos compartidos de archivos de Azure como destino. 

## <a name="migration-goals"></a>Objetivos de la migración

El objetivo es mover los datos de ubicaciones de recursos compartidos de archivos existentes a Azure. En Azure, almacenará los datos en recursos compartidos de archivos nativos de Azure que puede usar sin necesidad de Windows Server. Esta migración se debe realizar de forma que garantice la integridad de los datos de producción y la disponibilidad durante la migración. Esta última requiere que el tiempo de inactividad sea mínimo, para ajustarse o solo superar ligeramente las ventanas de mantenimiento regulares.

## <a name="migration-overview"></a>Información general sobre la migración

El proceso de migración consta de varias fases. Deberá implementar cuentas de almacenamiento y recursos compartidos de archivos de Azure. Además, configurará las redes, considerará una implementación de espacio de nombres DFS (DFS-N) o actualizará la existente. Una vez que haya llegado el momento de la copia de datos real, deberá tener en cuenta las ejecuciones diferenciales de RoboCopy repetidas para minimizar el tiempo de inactividad y, por último, deberá migrar a los usuarios a los recursos compartidos de archivos de Azure recién creados. En las secciones siguientes se describen detalladamente las fases del proceso de migración.

> [!TIP]
> Si vuelve a este artículo, use la navegación del lado derecho para ir a la fase de migración en la que se quedó.

## <a name="phase-1-identify-how-many-azure-file-shares-you-need"></a>Fase 1: Identificación de cuántos recursos compartidos de archivos de Azure se necesitan

[!INCLUDE [storage-files-migration-namespace-mapping](../../../includes/storage-files-migration-namespace-mapping.md)]

## <a name="phase-2-deploy-azure-storage-resources"></a>Fase 2: Implementación de recursos de almacenamiento de Azure

En esta fase, consulte la tabla de asignación de la fase 1 y úsela para aprovisionar el número correcto de cuentas de almacenamiento de Azure y recursos compartidos de archivos que contienen.

[!INCLUDE [storage-files-migration-provision-azfs](../../../includes/storage-files-migration-provision-azure-file-share.md)]

## <a name="phase-3-preparing-to-use-azure-file-shares"></a>Fase 3: Preparación para el uso de recursos compartidos de archivos de Azure

Con la información de esta fase, podrá decidir cómo se habilitarán los servidores y los usuarios dentro y fuera de Azure para usar los recursos compartidos de archivos de Azure. Las decisiones más críticas son las siguientes:

- **Redes:** habilite las redes para enrutar el tráfico SMB.
- **Autenticación:** configure cuentas de almacenamiento de Azure para la autenticación Kerberos. Unir su cuenta de almacenamiento a un dominio e instancia de AD Connect permitirá que las aplicaciones y los usuarios usen su identidad de AD para la autenticación
- **Autorización:** las ACL de nivel de recurso compartido para cada recurso compartido de archivos de Azure permitirán a los usuarios y grupos de AD acceder a un recurso compartido determinado. Además, dentro de un recurso compartido de archivos de Azure, las ACL nativas de NTFS asumirán el control. La autorización basada en ACL de archivos y carpetas funciona del mismo modo que en los recursos compartidos SMB locales.
- **Continuidad empresarial:** la integración de recursos compartidos de archivos de Azure en un entorno existente suele implicar la conservación de las direcciones de recursos compartidos existentes. Si aún no está usando los [espacios de nombres DFS](files-manage-namespaces.md), considere la posibilidad de configurarlos en su entorno. De este modo, podría conservar sin cambios las direcciones de recursos compartidos que usan los usuarios y los scripts. DFS-N proporciona un servicio de enrutamiento de espacios de nombres para SMB mediante el redireccionamiento de los clientes a los recursos compartidos de archivos de Azure.

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

### <a name="mounting-an-azure-file-share"></a>Montaje de un recurso compartido de archivos de Azure

Antes de que pueda usar RoboCopy, debe hacer que el recurso compartido de archivos de Azure sea accesible a través de SMB. La manera más fácil consiste en montar el recurso compartido como una unidad de red local en la instancia de Windows Server que está planeando usar para RoboCopy. 

> [!IMPORTANT]
> Antes de que pueda montar correctamente un recurso compartido de archivos de Azure en una instancia local de Windows Server, debe haber completado la fase 3: Preparación para el uso de recursos compartidos de archivos de Azure.

Una vez listo, revise el [artículo de procedimientos Uso de un recurso compartido de archivos de Azure con Windows](storage-how-to-use-files-windows.md). A continuación, monte el recurso compartido de archivos de Azure para el que desea iniciar RoboCopy.

## <a name="phase-4-robocopy"></a>Fase 4: RoboCopy

El siguiente comando RoboCopy solo copiará las diferencias (archivos y carpetas actualizados) desde el almacenamiento de origen al recurso compartido de archivos de Azure.

[!INCLUDE [storage-files-migration-robocopy](../../../includes/storage-files-migration-robocopy.md)]

> [!TIP]
> [Consulte la sección de solución de problemas](#troubleshoot-and-optimize) si RoboCopy está afectando a su entorno de producción, si notifica un gran número de errores o si no progresa tan rápido como se espera.

## <a name="phase-5-user-cut-over"></a>Fase 5: Migración total de los usuarios

Al ejecutar por primera vez el comando RoboCopy, los usuarios y las aplicaciones siguen accediendo a los archivos en el origen de la migración y pueden modificarlos. Es posible que RoboCopy haya procesado un directorio, pase al siguiente y, después, un usuario en la ubicación de origen agregue, cambie o elimine un archivo que no se procesará en esta ejecución actual de RoboCopy. Este comportamiento es normal.

La primera ejecución consiste en migrar la mayor parte de los datos renovados al recurso compartido de archivos de Azure. Esta primera copia puede tardar unos minutos. Consulte la [sección de solución de problemas](#troubleshoot-and-optimize) para obtener más información acerca de lo que puede afectar a la velocidad de RoboCopy.

Una vez completada la ejecución inicial, vuelva a ejecutar el comando.

La segunda vez que ejecute RoboCopy para el mismo recurso compartido se completará más rápido, ya que solo necesita transportar los cambios posteriores a la última ejecución. Puede ejecutar trabajos repetidos para el mismo recurso compartido.

Si considera que el tiempo de inactividad es aceptable, debe quitar el acceso de usuario a los recursos compartidos de origen. Para ello, siga los pasos que impidan que los usuarios cambien el contenido y la estructura de archivos y carpetas. Un ejemplo es dirigir DFS-Namespace a una ubicación no existente o cambiar las ACL de cada recurso compartido.

Ejecute una última ronda de RoboCopy. Recuperará todos los cambios que puedan haberse omitido.
El tiempo necesario para hacer este último paso, que depende de la velocidad del análisis de RoboCopy. Para realizar un cálculo estimado del tiempo (que equivale al tiempo de inactividad) averigüe cuánto tardó en realizarse la ejecución anterior.

En una sección anterior, ha configurado los usuarios para [acceder al recurso compartido con su identidad](#phase-3-preparing-to-use-azure-file-shares) y debe haber establecido una estrategia para que los usuarios [usen rutas de acceso establecidas a los nuevos recursos compartidos de archivos de Azure (DFS-N)](files-manage-namespaces.md).

Puede intentar ejecutar algunas de estas copias entre distintos recursos compartidos de origen y destino en paralelo. Al hacerlo, tenga en cuenta el rendimiento de la red y la relación de número de subprocesos y núcleos para no sobrecargar el sistema.

## <a name="troubleshoot-and-optimize"></a>Solución de problemas y optimización

[!INCLUDE [storage-files-migration-robocopy-optimize](../../../includes/storage-files-migration-robocopy-optimize.md)]

## <a name="next-steps"></a>Pasos siguientes

Hay más información sobre los recursos compartidos de archivos de Azure. Los artículos siguientes le ayudarán a comprender las opciones avanzadas, los procedimientos recomendados y también contienen ayuda para la solución de problemas. Estos artículos se vinculan a la [documentación de recursos compartidos de archivos de Azure](storage-files-introduction.md) según corresponda.

* [Información general acerca de la migración](storage-files-migration-overview.md)
* [Copia de seguridad: Instantáneas de recursos compartidos de archivos de Azure](storage-snapshots-files.md)
* [Uso de Espacios de nombres DFS con Azure Files](files-manage-namespaces.md)
