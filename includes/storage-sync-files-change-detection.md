---
author: roygara
ms.service: storage
ms.topic: include
ms.date: 10/26/2018
ms.author: rogarana
ms.openlocfilehash: 1387933dc82c07e73b7715d6593238ea8c993e93
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "96005349"
---
Los cambios realizados en el recurso compartido de archivos de Azure mediante Azure Portal o SMB no se detectan y replican de forma inmediata como cambios en el punto de conexión del servidor. Azure Files aún no dispone de registros en diario o notificaciones, por lo que no hay manera de iniciar automáticamente una sesión de sincronización cuando se cambian los archivos. En Windows Server, Azure File Sync usa el [registro en diario de USN de Windows](/windows/win32/fileio/change-journals) para iniciar automáticamente una sesión de sincronización cuando cambian los archivos.

Para detectar cambios en el recurso compartido de archivos de Azure, Azure File Sync tiene un trabajo programado que se denomina *trabajo de detección de cambios*. Un trabajo de detección de cambios enumera todos los archivos del recurso compartido de archivos y, a continuación, los compara con la versión de sincronización correspondiente. Cuando el trabajo de detección de cambios determina qué archivos han cambiado, Azure File Sync inicia una sesión de sincronización. El trabajo de detección de cambios se inicia cada 24 horas. Dado que el trabajo de detección de cambios enumera todos los archivos del recurso compartido de archivos de Azure, la detección de cambios tarda más en los espacios de nombres más largos que los espacios de nombres más cortos. En el caso de los espacios de nombres largos, es posible que sea necesario determinar más de una vez cada 24 horas qué archivos han cambiado.

Para sincronizar inmediatamente los archivos que se modifican en el recurso compartido de archivos de Azure, se puede usar el cmdlet **Invoke-AzStorageSyncChangeDetection** de PowerShell para iniciar de forma manual la detección de cambios en el recurso compartido. Este cmdlet está pensado para escenarios en los que algún tipo de proceso automatizado está realizando cambios en el recurso compartido de archivos de Azure o en los que es el administrador el que efectúa los cambios (por ejemplo, al mover archivos y directorios al recurso compartido). En el caso de los cambios del usuario final, se recomienda instalar el agente de Azure File Sync en una máquina virtual de IaaS y hacer que los usuarios finales accedan al recurso compartido de archivos a través de ella. De este modo, todos los cambios se sincronizarán rápidamente con otros agentes sin necesidad de usar el cmdlet Invoke-AzStorageSyncChangeDetection. Para más información, consulte la documentación sobre [Invoke-AzStorageSyncChangeDetection](/powershell/module/az.storagesync/invoke-azstoragesyncchangedetection).

>[!NOTE]
>Tenga en cuenta que los cambios realizados en un recurso compartido de archivos de Azure con REST no actualizan la hora de la última modificación y no se verán como un cambio por la sincronización.

Estamos considerando agregar la detección de cambios para un recurso compartido de archivos de Azure similar al USN para volúmenes en Windows Server. Ayúdenos a priorizar el futuro desarrollo de esta característica votándola en [UserVoice de Azure Files](https://feedback.azure.com/forums/217298-storage/category/180670-files).