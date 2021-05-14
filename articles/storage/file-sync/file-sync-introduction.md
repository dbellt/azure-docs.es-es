---
title: Introducción a Azure File Sync | Microsoft Docs
description: Información general sobre Azure File Sync, un servicio que permite crear y utilizar recursos compartidos de archivos de red en la nube mediante el protocolo SMB estándar del sector.
author: roygara
ms.service: storage
ms.topic: overview
ms.date: 04/19/2021
ms.author: rogarana
ms.subservice: files
ms.openlocfilehash: da40783e3d299b0edf27c6d045dbc6dcfc5cf964
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107796262"
---
# <a name="what-is-azure-file-sync"></a>¿Qué es Azure File Sync?
Azure File Sync permite centralizar los recursos compartidos de archivos de la organización en Azure Files sin renunciar a la flexibilidad, el rendimiento y la compatibilidad de un servidor de archivos de Windows. Aunque algunos usuarios pueden optar por mantener una copia completa de sus datos localmente, Azure File Sync además ofrece la posibilidad de transformar Windows Server en una caché rápida del recurso compartido de archivos de Azure. Puede usar cualquier protocolo disponible en Windows Server para acceder a sus datos localmente, como SMB, NFS y FTPS. Puede tener todas las cachés que necesite en todo el mundo.   

## <a name="videos"></a>Vídeos
| Introducción a Azure File Sync | Azure Files con Sync (Ignite 2019)  |
|-|-|
| [![Presentación en pantalla del vídeo Introducing Azure File Sync, haga clic para reproducirlo.](../files/media/storage-files-introduction/azure-file-sync-video-snapshot.png)](https://www.youtube.com/watch?v=Zm2w8-TRn-o) | [![Presentación en pantalla de Azure Files with Sync, haga clic para reproducirlo.](../files/media/storage-files-introduction/ignite-2018-video.png)](https://www.youtube.com/embed/6E2p28XwovU) |

## <a name="benefits-of-azure-file-sync"></a>Ventajas de Azure File Sync

### <a name="cloud-tiering"></a>Niveles de nube
Con la nube por niveles habilitada, los archivos a los que se accede con más frecuencia se almacenan en caché en el servidor local y a los que se accede con menos frecuencia lo hacen por niveles en la nube. Puede controlar la cantidad de espacio en disco local que se usa para el almacenamiento en caché. Los archivos por niveles se pueden recuperar rápidamente a petición, lo que hace que la experiencia sea fluida y le permita reducir los costos, ya que solo necesita almacenar una fracción de los datos en el entorno local. Para más información sobre la nube por niveles, consulte [Introducción a la nube por niveles](file-sync-cloud-tiering-overview.md). 

### <a name="multi-site-access-and-sync"></a>Sincronización y acceso a varios sitios
Azure File Sync es muy adecuado en escenarios de acceso distribuido. Para cada una de las oficinas, puede aprovisionar una instancia local de Windows Server como parte de la implementación de Azure File Sync. Los cambios realizados en el servidor de una oficina se sincronizan automáticamente con los servidores de todas las demás oficinas.

### <a name="business-continuity-and-disaster-recovery"></a>Continuidad empresarial y recuperación ante desastres
Azure File Sync cuenta con el respaldo de Azure Files, que ofrece varias opciones de redundancia para el almacenamiento de alta disponibilidad. Dado que Azure contiene copias resistentes de los datos, el servidor local se convierte en un dispositivo de almacenamiento en caché descartable y la recuperación de un servidor con errores se puede realizar agregando un nuevo servidor a la implementación de Azure File Sync. En lugar de restaurar una copia de seguridad local, se aprovisiona otra instancia de Windows Server, se instala el agente de Azure File Sync en ella y, luego, se agrega a la implementación de Azure File Sync. Azure File Sync descarga el espacio de nombres de archivo antes de descargar los datos, de modo que el servidor pueda estar en funcionamiento lo antes posible. Para una recuperación más rápida si cabe, puede tener un estado de espera semiactiva por servidor como parte de la implementación, o bien puede usar Azure File Sync con la agrupación en clústeres de Windows.

## <a name="cloud-side-backup"></a>Copia de seguridad en la nube
Reduzca el gasto de copia de seguridad local mediante con copias de seguridad centralizadas en la nube con Azure Backup. Los recursos compartidos SMB de Azure Files tienen funcionalidades nativas de instantánea, y el proceso se puede automatizar mediante Azure Backup para programar las copias de seguridad y administrar su retención. Azure Backup también se integra con los servidores locales, por lo que, al realizar la restauración en la nube, estos cambios se descargan automáticamente en los servidores de Windows.  

## <a name="next-steps"></a>Pasos siguientes
* [Planeamiento de una implementación de Azure File Sync](file-sync-planning.md)
* [Introducción a la nube por niveles](file-sync-cloud-tiering-overview.md)
* [Supervisión de Azure File Sync](file-sync-monitoring.md)