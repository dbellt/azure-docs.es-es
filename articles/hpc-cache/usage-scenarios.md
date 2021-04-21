---
title: Escenarios de Azure HPC Cache
description: Describe cómo saber si un trabajo de informática funciona bien con Azure HPC Cache
author: ekpgh
ms.service: hpc-cache
ms.topic: how-to
ms.date: 03/29/2021
ms.author: v-erkel
ms.openlocfilehash: 36e0135102fbede5505e96fb1aa255588b2f2ae2
ms.sourcegitcommit: 20f8bf22d621a34df5374ddf0cd324d3a762d46d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/09/2021
ms.locfileid: "107259920"
---
# <a name="is-your-job-a-good-fit-for-azure-hpc-cache"></a>¿Azure HPC Cache es una opción adecuada para su trabajo?

Azure HPC Cache puede acelerar el acceso a los datos para los trabajos de informática de alto rendimiento en diversas materias. Pero no es perfecto para todo tipo de flujo de trabajo. En este artículo se dan instrucciones sobre cómo decidir si HPC Cache es una buena opción según sus necesidades.

En el artículo de [Información general](hpc-cache-overview.md) también se ofrece un breve resumen de cuándo usar Azure HPC Cache y algunos ejemplos de casos de uso.

Consulte también [este artículo](nfs-blob-considerations.md) acerca de cómo usar de manera eficaz el [almacenamiento de blobs montado en NFS](../storage/blobs/network-file-system-protocol-support.md), que se encuentra en versión preliminar.

## <a name="nfs-version-30-applications"></a>Aplicaciones NFS versión 3.0

Azure HPC Cache solo es compatible con los clientes NFS 3.0.

## <a name="high-read-to-write-ratio"></a>Proporción lectura-escritura elevada

Las cargas de trabajo en las que los clientes de proceso realizan más operaciones de lectura que de escritura suelen ser buenas candidatas para una caché. Por ejemplo, si la proporción lectura-escritura es 80/20 o 70/30, Azure HPC Cache puede ayudar al proporcionar los archivos solicitados con frecuencia desde la memoria caché, en lugar de que sea necesario recuperarlos desde el almacenamiento remoto una y otra vez.

Recuperar un archivo y almacenarlo en la memoria caché por primera vez tiene una pequeña latencia adicional en comparación con una solicitud de cliente normal directamente al almacenamiento, por lo que la mejora de la eficacia se producirá la próxima vez que un cliente solicite el mismo archivo. Esto sucede en particular con los archivos de gran tamaño. Si cada solicitud de cliente es única, el impacto de HPC Cache será limitado. Pero cuanto mayor sea el tamaño del archivo, mejor será el rendimiento con el tiempo después de ese primer acceso.

## <a name="file-based-analytic-workload"></a>Carga de trabajo analítica basada en archivos

Azure HPC Cache es la opción ideal para una canalización que usa datos basados en archivos y se ejecuta en un gran número de clientes de proceso, sobre todo si los clientes de proceso son máquinas virtuales de Azure. Puede ayudar a corregir un rendimiento lento o incoherente causado por los tiempos prolongados de acceso a archivos.

## <a name="remote-data-access"></a>Acceso a datos remotos

Azure HPC Cache puede ayudar a reducir la latencia si la carga de trabajo necesita acceder a datos remotos que no se pueden trasladar más cerca de los recursos de informática. Por ejemplo, los registros pueden estar en el extremo de un entorno WAN, en otra región de Azure o en un centro de datos del cliente. (A veces a esto se le denomina "expansión de archivos").

## <a name="heavy-request-load"></a>Sobrecarga de solicitudes

Si un gran número de clientes solicitan datos del origen al mismo tiempo, Azure HPC Cache agiliza el acceso a los archivos. Por ejemplo, cuando se usa con un clúster de informática de alto rendimiento, Azure HPC Cache ofrece escalabilidad para un gran número de solicitudes simultáneas a través de la caché.

## <a name="compute-resources-are-located-in-azure"></a>Los recursos de proceso se encuentran en Azure

Las máquinas virtuales de Azure son una solución escalable y rentable para las cargas de trabajo de informática de alto rendimiento. Azure HPC Cache acerca la información necesaria a las cargas, sobre todo si los datos originales se almacenan en un sistema remoto.

Si un cliente quiere ejecutar la canalización actual "tal cual" en máquinas virtuales de Azure, Azure HPC Cache puede proporcionarle una solución de almacenamiento compartido (o almacenamiento en caché) basada en POSIX para la escalabilidad.

Al usar Azure HPC Cache, no es necesario rediseñar la canalización de trabajo para hacer llamadas nativas a Azure Blob Storage. Puede acceder a los datos en su sistema original o usar HPC Cache para moverlos a un nuevo contenedor de blobs.

## <a name="next-steps"></a>Pasos siguientes

* Obtenga más información sobre cómo planear y configurar una caché en los artículos de [Información general](hpc-cache-overview.md) y [Requisitos previos](hpc-cache-prerequisites.md)
* Consulte las consideraciones sobre el uso de [Blob Storage habilitado para NFS](nfs-blob-considerations.md) (VERSIÓN PRELIMINAR) con Azure HPC Cache
