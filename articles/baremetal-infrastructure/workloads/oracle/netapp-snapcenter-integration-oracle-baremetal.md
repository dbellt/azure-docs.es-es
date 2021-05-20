---
title: La integración de SnapCenter para Oracle en BareMetal Infrastructure
description: Aprenda a usar tecnologías de instantáneas de Oracle y NetApp para crear copias de seguridad de recuperación operativa para bases de datos de Oracle en BareMetal Infrastructure.
ms.topic: how-to
ms.subservice: workloads
ms.date: 05/05/2021
ms.openlocfilehash: 4182a5fe97e356152b6d589fc6788b922e5c8493
ms.sourcegitcommit: 38d81c4afd3fec0c56cc9c032ae5169e500f345d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109520670"
---
# <a name="snapcenter-integration-for-oracle-on-baremetal-infrastructure"></a>La integración de SnapCenter para Oracle en BareMetal Infrastructure

Esta guía paso a paso sobre las tecnologías de instantáneas de Oracle y NetApp le ayudan a crear copias de seguridad de recuperación operativa para bases de datos de Oracle. El uso de instantáneas es solo uno de los distintos enfoques de protección de datos de Oracle. Las instantáneas pueden mitigar el tiempo de inactividad y la pérdida de datos al ejecutar una base de datos de Oracle en BareMetal Infrastructure. 

>[!IMPORTANT]
>SnapCenter es compatible con Oracle para las aplicaciones SAP, pero no proporciona la integración de SAP BR\*Tools.

Aunque Oracle ofrece dos métodos de copia de seguridad diferentes para instantáneas, SnapCenter de NetApp solo es compatible con un método: el modo de copia de seguridad en caliente. El modo de copia de seguridad en caliente es la versión tradicional de realizar copias de seguridad y de crear instantáneas de Oracle. Requiere la interacción con el host de Oracle para colocar temporalmente la base de datos en un modo de copia de seguridad en caliente y catalogar los registros de archivo correctamente. El modo de copia de seguridad activa también permite una mayor interacción con la base de datos de RMAN para realizar un seguimiento más cercano de las instantáneas disponibles para la recuperación. 

Esta guía le mostrará cómo crear un marco de recuperación operativa y recuperación ante desastres para Oracle en modo de copia de seguridad en caliente. La recuperación ante desastres es la recuperación de una base de datos, o parte de una base de datos, después de un desastre. Por ejemplo, una unidad defectuosa o una base de datos dañada. La recuperación operativa es la recuperación de algo que no sea un desastre. Un ejemplo podría ser la pérdida de algunas filas de datos que no obstaculizan el funcionamiento de su negocio.

## <a name="snapcenter-database-organization"></a>Organización de base de datos de SnapCenter
SnapCenter organiza las bases de datos en grupos de recursos. Un grupo de recursos puede consistir en una o varias bases de datos con la misma directiva de protección. Por lo tanto, no tiene que seleccionar volúmenes individuales que forman parte de la copia de seguridad.

:::image type="content" source="media/netapp-snapcenter-integration-oracle-baremetal/snapcenter-database-resource-group.png" alt-text="Diagrama que muestra cómo SnapCenter organiza las bases de datos en grupos de recursos" border="false":::

## <a name="oracle-disaster-recovery-architecture"></a>Arquitectura de recuperación ante desastres de Oracle

Las copias de seguridad en caliente de Oracle se dividen en dos tipos de copia de seguridad distintas para ponerse al día mediante registros de archivo después de restaurar los archivos de datos. La protección de los archivos de datos y los archivos de control se lleva a cabo "cada hora", pero se acepta dejar un intervalo de tiempo más largo entre copias de seguridad. Cuanto más largos sean los intervalos entre copias de seguridad, mayor será el tiempo de recuperación.  

SnapCenter crea instantáneas locales de la base de datos en las ubicaciones predeterminadas del archivo de datos (nfs01). Las instantáneas se crean en los sistemas de archivos que hospedan archivos de datos o archivos de control. Estas copias de seguridad garantizan la recuperación rápida de la base de datos. No protege los datos en caso de error de varios discos o erro de sitio. 

El número de instantáneas "por hora" depende de las directivas de copia de seguridad que se establezcan. Una base de datos de Oracle tiene entre 2 y 5 días de capacidad de recuperación operativa a partir de instantáneas.
 
Deben existir suficientes registros de archivo y es necesario que pongan al día la base de datos de Oracle desde el punto de recuperación de archivos de datos más viable y reciente. Use instantáneas de registros de archivo para reducir el objetivo de punto de recuperación (RPO) para una base de datos de Oracle. Cuanto menos frecuencia haya entre las instantáneas en los registros de archivo, menor será el RPO. El intervalo de instantáneas recomendado para los registros de archivo es de 5 o 15 minutos. El intervalo más corto, el de 5 minutos, tiene el RPO más corto.  El intervalo más corto aumenta la complejidad, ya que se deben gestoinar más instantáneas como parte del proceso de recuperación.

## <a name="next-steps"></a>Pasos siguientes

Aprenda a configurar SnapCenter de NetApp para enrutar el tráfico de Azure a servidores de Oracle BareMetal Infrastructure.

> [!div class="nextstepaction"]
> [Configuración de SnapCenter para enrutar el tráfico](set-up-snapcenter-to-route-traffic.md)
