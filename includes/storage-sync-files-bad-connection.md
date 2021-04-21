---
title: archivo de inclusión
description: archivo de inclusión
services: storage
author: roygara
ms.service: storage
ms.topic: include
ms.date: 07/08/2018
ms.author: rogarana
ms.custom: include file
ms.openlocfilehash: 4407ffb9ab16a720ef00a288d72b0fb4c2dbced1
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107774581"
---
Este error se puede producir cuando el servicio Azure File Sync no es accesible desde el servidor. Puede solucionar este error siguiendo estos pasos:

1. Compruebe que el servicio de Windows `FileSyncSvc.exe` no está bloqueado por el firewall.
2. Compruebe que el puerto 443 está abierto para las conexiones salientes al servicio Azure File Sync. Para ello, puede usar el cmdlet `Test-NetConnection`. La dirección URL para el marcador de posición `<azure-file-sync-endpoint>` a continuación se puede encontrar en el documento [Configuración del proxy y el firewall de Azure File Sync](../articles/storage/file-sync/file-sync-firewall-and-proxy.md#firewall). 

    ```powershell
    Test-NetConnection -ComputerName <azure-file-sync-endpoint> -Port 443
    ```

3. Asegúrese de establecer la configuración del proxy como estaba previsto. Esto puede hacerse con el cmdlet `Get-StorageSyncProxyConfiguration`. Puede encontrar más información sobre cómo configurar la configuración del proxy de Azure File Sync en [Configuración del proxy y el firewall de Azure File Sync](../articles/storage/file-sync/file-sync-firewall-and-proxy.md#firewall).

    ```powershell
    $agentPath = "C:\Program Files\Azure\StorageSyncAgent"
    Import-Module "$agentPath\StorageSync.Management.ServerCmdlets.dll"
    Get-StorageSyncProxyConfiguration
    ```
4. Use el cmdlet Test-StorageSyncNetworkConnectivity para comprobar la conectividad de red con los puntos de conexión de servicio. Para más información, vea [Prueba de la conectividad de red con los puntos de conexión de servicio](../articles/storage/file-sync/file-sync-firewall-and-proxy.md#test-network-connectivity-to-service-endpoints).    

5. Póngase en contacto con el administrador de red si necesita más ayuda para resolver problemas de conectividad de red.