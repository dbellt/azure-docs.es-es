---
author: Juliako
ms.service: azure-video-analyzer
ms.topic: include
ms.date: 04/30/2021
ms.author: juliako
ms.openlocfilehash: d4a2c269c2dfe66dd0d3271b3eef8fc8de32e4a5
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110384126"
---
Los requisitos previos de este tutorial son los siguientes:


* Una cuenta de Azure que incluya una suscripción activa. [Cree una cuenta](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) gratuita, en caso de que aún no lo haya hecho.

    > [!NOTE]    
    > Necesitará una suscripción a Azure en la que tenga acceso a los roles [Colaborador](../../../../../role-based-access-control/built-in-roles.md#contributor) y [Administrador de acceso de usuario](../../../../../role-based-access-control/built-in-roles.md#user-access-administrator). Si no tiene los permisos adecuados, póngase en contacto con el administrador de la cuenta para que se los conceda.
* [Visual Studio Code](https://code.visualstudio.com/) con las siguientes extensiones:
    * [Herramientas de Azure IoT](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools)
    
    > [!TIP]
    > Al instalar Azure IoT Tools, es posible que se le pida que instale Docker. Si lo desea, ignore esta petición.    
    * [C#](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp)
* [SDK de .NET Core 3.1](https://dotnet.microsoft.com/download/dotnet-core/3.1). 

> [!Important]
> Este módulo de Custom Vision solo admite arquitecturas **Intel x86 y amd64**. Compruebe la arquitectura del dispositivo perimetral antes de continuar.

## <a name="set-up-azure-resources"></a>Configuración de los recursos de Azure

[![Implementación en Azure](https://aka.ms/deploytoazurebutton)](https://aka.ms/ava-click-to-deploy)

[!INCLUDE [resources](../../common-includes/azure-resources.md)]
