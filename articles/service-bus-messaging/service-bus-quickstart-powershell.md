---
title: Uso de Azure PowerShell para crear una cola de Service Bus
description: En este inicio rápido aprenderá a crear un espacio de nombres de Service Bus y una cola en ese mismo espacio mediante Azure PowerShell.
author: spelluru
ms.author: spelluru
ms.date: 08/12/2020
ms.topic: quickstart
ms.devlang: dotnet
ms.custom:
- devx-track-azurepowershell
- mode-api
ms.openlocfilehash: e850567f56cb89db135f9b6efeafb3d63dca151b
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110479012"
---
# <a name="use-azure-powershell-to-create-a-service-bus-namespace-and-a-queue"></a>Uso de Azure PowerShell para crear un espacio de nombres de Service Bus y una cola
En este inicio rápido se muestra cómo crear un espacio de nombres de Service Bus y una cola mediante Azure PowerShell. También se muestra cómo obtener credenciales de autorización que una aplicación cliente puede usar para enviar mensajes a la cola, o recibirlos de ella. 

[!INCLUDE [howto-service-bus-queues](../../includes/howto-service-bus-queues.md)]


## <a name="prerequisites"></a>Requisitos previos

Para completar este inicio rápido, asegúrese de que dispone de una suscripción de Azure. Si no tiene una suscripción a Azure, puede crear una [cuenta gratuita][] antes de empezar. 

En este inicio rápido, usará Azure Cloud Shell, que puede iniciar después de iniciar sesión en Azure Portal. Para más información sobre Azure Cloud Shell, consulte [Introducción a Azure Cloud Shell](../cloud-shell/overview.md). También puede [instalar](/powershell/azure/install-Az-ps) y usar Azure PowerShell en la máquina. 


## <a name="provision-resources"></a>Aprovisionamiento de recursos
1. Inicie sesión en el [Portal de Azure](https://portal.azure.com).
2. Para iniciar Azure Cloud Shell, seleccione l icono que se muestra en la siguiente imagen: 

    :::image type="content" source="./media/service-bus-quickstart-powershell/launch-cloud-shell.png" alt-text="Iniciar Cloud Shell":::
3. En la ventana inferior de Cloud Shell, cambie de **Bash** a **PowerShell**. 

    :::image type="content" source="./media/service-bus-quickstart-powershell/cloud-power-shell.png" alt-text="Cambio al modo de PowerShell":::    
4. Ejecute el siguiente comando para crear un grupo de recursos de Azure. Actualice el nombre del grupo de recursos y la ubicación si lo desea. 

    ```azurepowershell-interactive
    New-AzResourceGroup –Name ContosoRG –Location eastus
    ```
5. Ejecute el comando siguiente para crear un espacio de nombres de mensajería de Service Bus. En este ejemplo, `ContosoRG` es el grupo de recursos que creó en el paso anterior. `ContosoSBusNS` es el nombre del espacio de nombres de Service Bus creado en ese grupo de recursos. 

    ```azurepowershell-interactive
    New-AzServiceBusNamespace -ResourceGroupName ContosoRG -Name ContosoSBusNS -Location eastus
    ```
6. Ejecute lo siguiente para crear una cola en el espacio de nombres que creó en el paso anterior. 

    ```azurepowershell-interactive
    New-AzServiceBusQueue -ResourceGroupName ContosoRG -NamespaceName ContosoSBusNS -Name ContosoOrdersQueue 
    ```
7. Obtenga la cadena de conexión principal del espacio de nombres. Esta cadena de conexión se usará para conectarse a la cola y enviar y recibir mensajes. 

    ```azurepowershell-interactive    
    Get-AzServiceBusKey -ResourceGroupName ContosoRG -Namespace ContosoSBusNS -Name RootManageSharedAccessKey
    ```

    Anote la cadena de conexión y el nombre de la cola. Se usan para enviar y recibir mensajes. 


## <a name="next-steps"></a>Pasos siguientes
En este artículo, ha creado un espacio de nombres de Service Bus y una cola en dicho espacio de nombres. Para aprender a enviar mensajes a la cola y recibirlos de ella, consulte uno de los siguientes inicios rápido en la sección **Envío y recepción de mensajes**. 

- [.NET](service-bus-dotnet-get-started-with-queues.md)
- [Java](service-bus-java-how-to-use-queues.md)
- [JavaScript](service-bus-nodejs-how-to-use-queues.md)
- [Python](service-bus-python-how-to-use-queues.md)
- [PHP](service-bus-php-how-to-use-queues.md)

[cuenta gratuita]: https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio
