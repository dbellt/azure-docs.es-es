---
title: Opciones de restauración con el agente de Microsoft Azure Recovery Services (MARS)
description: Obtenga información sobre las opciones de restauración disponibles con el agente de Microsoft Azure Recovery Services (MARS).
ms.reviewer: mepand
ms.topic: conceptual
ms.date: 05/07/2021
ms.openlocfilehash: cebe13f6e72a9d8db346171384182756fb474ecd
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111970467"
---
# <a name="about-restore-using-the-microsoft-azure-recovery-services-mars-agent"></a>Acerca de la restauración con el agente de Microsoft Azure Recovery Services (MARS) 

En este artículo se describen las opciones de restauración disponibles con el agente de Microsoft Azure Recovery Services (MARS).

## <a name="before-you-begin"></a>Antes de empezar

- Asegúrese de que esté instalada la versión más reciente del [agente de MARS](https://aka.ms/azurebackup_agent).
- Asegúrese de que la [limitación de red](backup-windows-with-mars-agent.md#enable-network-throttling) está deshabilitada.
- Asegúrese de que está disponible el almacenamiento de alta velocidad con espacio suficiente para la [carpeta de caché del agente](/azure/backup/backup-azure-file-folder-backup-faq.yml#manage-the-backup-cache-folder).
- Supervise la memoria y los recursos de CPU, y asegúrese de que haya suficientes recursos disponibles para descomprimir y descifrar los datos.
- Al usar la característica **Restauración instantánea** para montar un punto de recuperación como disco, use **robocopy** con la opción de copia multiproceso (conmutador /MT) para copiar archivos de forma eficaz desde el punto de recuperación montado.

## <a name="restore-options"></a>Opciones de restauración

El agente de MARS ofrece varias opciones de restauración. Cada opción proporciona ventajas únicas que la hacen adecuada para determinados escenarios.

Con el agente de MARS puede:

- **[Restaurar la copia de seguridad del estado del sistema de Windows Server:](backup-azure-restore-system-state.md)** esta opción ayuda a restaurar el estado del sistema como archivos desde un punto de recuperación de Azure Backup y aplicarlos a Windows Server mediante la utilidad Copias de seguridad de Windows Server.  
- **[Restaurar todos los archivos de los que se ha creado una copia de seguridad en un volumen](restore-all-files-volume-mars.md):** esta opción recupera todos los datos de copia de seguridad de un volumen especificado desde el punto de recuperación de Azure Backup. Permite una velocidad de transferencia más rápida (hasta 40 MBPS).<br>Se recomienda usar esta opción para recuperar grandes cantidades de datos o volúmenes completos.
- **[Restaurar un conjunto específico de archivos y carpetas de los que se ha creado una copia de seguridad en un volumen mediante PowerShell](backup-client-automation.md#restore-data-from-azure-backup):** si se conocen las rutas de acceso a los archivos y carpetas relativos a la raíz del volumen, esta opción le permite restaurar el conjunto especificado de archivos y carpetas desde un punto de recuperación, con la velocidad de transferencia más rápida de la restauración completa del volumen. Sin embargo, esta opción no proporciona la comodidad de examinar archivos y carpetas en el punto de recuperación mediante la opción Restauración instantánea.
- **[Restaurar archivos y carpetas individuales mediante Restauración instantánea](backup-azure-restore-windows-server.md):** esta opción permite un acceso rápido a los datos de copia de seguridad montando el volumen en el punto de recuperación como una unidad. A continuación, puede examinar y copiar archivos y carpetas. Esta opción ofrece una velocidad de copia de hasta 6 MBPS, que es adecuada para recuperar archivos y carpetas individuales con un tamaño total inferior a 80 GB. Una vez copiados los archivos necesarios, puede desmontar el punto de recuperación.

## <a name="next-steps"></a>Pasos siguientes

- Para consultar las preguntas más frecuentes, vea [Preguntas más frecuentes sobre el agente de MARS](backup-azure-file-folder-backup-faq.yml).
- Para obtener información sobre los escenarios admitidos y las limitaciones, consulte [Matriz de compatibilidad para la copia de seguridad con el agente de Microsoft Azure Recovery Services (MARS)](backup-support-matrix-mars-agent.md).