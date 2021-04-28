---
author: mikben
ms.service: azure-communication-services
ms.topic: include
ms.date: 03/10/2021
ms.author: mikben
ms.openlocfilehash: 69857915620eada94586754a6c934edaf0b294a9
ms.sourcegitcommit: 2aeb2c41fd22a02552ff871479124b567fa4463c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107879673"
---
## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/dotnet/).
- Instalación de la [CLI de Azure](https://docs.microsoft.com/cli/azure/install-azure-cli-windows?tabs=azure-cli) 

## <a name="create-azure-communication-resource"></a>Creación de un recurso de Azure Communication Services

Para crear un recurso de Azure Communication Services, [inicie sesión en la CLI de Azure](/cli/azure/authenticate-azure-cli). Esto puede hacerlo mediante el terminal con el comando ```az login``` y proporcionando sus credenciales. Ejecute el siguiente comando para crear el recurso:

```azurecli
az communication create --name "<communicationName>" --location "Global" --data-location "United States" --resource-group "<resourceGroup>"
```

Si quiere seleccionar una suscripción específica, también puede especificar la marca ```--subscription``` y proporcionar el identificador de la suscripción.
```
az communication create --name "<communicationName>" --location "Global" --data-location "United States" --resource-group "<resourceGroup> --subscription "<subscriptionID>"
```

Puede configurar el recurso de Communication Services con las opciones siguientes:

* El grupo de recursos
* El nombre del recurso de Communication Services
* La geografía a la que se asociará el recurso

En el paso siguiente, puede asignar etiquetas al recurso. Las etiquetas se pueden usar para organizar los recursos de Azure. Para obtener más información sobre las etiquetas, consulte la [documentación sobre las etiquetas de recursos](../../../azure-resource-manager/management/tag-resources.md).

## <a name="manage-your-communication-services-resource"></a>Administración del recurso de Communication Services

Para agregar etiquetas al recurso de Communication Services, ejecute los siguientes comandos: También puede usar una suscripción específica.

```azurecli
az communication update --name "<communicationName>" --tags newTag="newVal1" --resource-group "<resourceGroup>"

az communication update --name "<communicationName>" --tags newTag="newVal2" --resource-group "<resourceGroup>" --subscription "<subscriptionID>"

az communication show --name "<communicationName>" --resource-group "<resourceGroup>"

az communication show --name "<communicationName>" --resource-group "<resourceGroup>" --subscription "<subscriptionID>"
```

Para información sobre comandos adicionales, consulte [az communication](/cli/azure/communication).
