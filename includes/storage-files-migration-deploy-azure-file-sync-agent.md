---
title: archivo de inclusión
description: archivo de inclusión
services: storage
author: fauhse
ms.service: storage
ms.topic: include
ms.date: 2/20/2020
ms.author: fauhse
ms.custom: include file
ms.openlocfilehash: 544bbaa358f0ca158c757524406ffd046ad43af7
ms.sourcegitcommit: ba8f0365b192f6f708eb8ce7aadb134ef8eda326
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2021
ms.locfileid: "109644882"
---
En esta sección se instala el agente de Azure File Sync en una instancia de Windows Server.

En la [guía de implementación](../articles/storage/file-sync/file-sync-deployment-guide.md) se explica que es preciso desactivar la **configuración de seguridad mejorada de Internet Explorer**. Esta medida de seguridad no se puede aplicar con Azure File Sync. Si la desactiva, puede autenticarse en Azure sin ningún problema.

Abra PowerShell. Instale los módulos de PowerShell necesarios mediante los siguientes comandos. Asegúrese de instalar el módulo completo y el proveedor de NuGet cuando se le solicite hacerlo.

```powershell
Install-Module -Name Az -AllowClobber
Install-Module -Name Az.StorageSync
```

Si tiene algún problema para conectarse a Internet desde el servidor, ahora es el momento de solucionarlo. Azure File Sync usa cualquier conexión de red disponible a Internet. También se admite la exigencia de un servidor proxy para tener acceso a Internet. Ya puede configurar un proxy en toda la máquina, o bien, durante la instalación del agente, especificar un proxy que solo va a usar Azure File Sync.

Si para configurar un proxy debe abrir los firewalls del servidor, es posible que ese enfoque le resulte aceptable. Al final de la instalación del servidor, después de haber completado el registro del servidor, un informe de conectividad de red le mostrará las direcciones URL exactas de los puntos de conexión en Azure, con las que Azure File Sync necesita comunicarse en la región que ha seleccionado. El informe también indica por qué se necesita la comunicación. Puede usar el informe para bloquear los firewalls del servidor en direcciones URL específicas.

También puede adoptar un enfoque más conservador y no abrir totalmente los firewalls. En su lugar puede limitar el servidor para que se comunique con espacios de nombres DNS de nivel superior. Para más información, consulte [Configuración del proxy y el firewall de Azure File Sync](../articles/storage/file-sync/file-sync-firewall-and-proxy.md). Siga sus propios procedimientos recomendados de redes.

Al final del Asistente para la instalación del servidor, se abrirá un Asistente para el registro del servidor. Registre el servidor en el recurso de Azure del servicio de sincronización de almacenamiento anterior.

Estos pasos se describen con más detalle en la guía de implementación, que incluye los módulos de PowerShell que se deben instalar primero: [Instalación de agente de Azure File Sync](../articles/storage/file-sync/file-sync-deployment-guide.md).

Use el agente más reciente. Puede descargarlo del Centro de descarga de Microsoft: [Agente de Azure File Sync](https://aka.ms/AFS/agent "Descarga del agente de Azure File Sync").

Después de una instalación y un registro del servidor correctos, puede confirmar que ha completado correctamente este paso. Vaya al recurso de Storage Sync Service en Azure Portal. En el menú de la izquierda, vaya a **Servidores registrados**. Verá que el servidor aparece en esa lista.
