---
title: 'Inicio rápido: Creación de una instancia de Azure API Management mediante la CLI'
description: Cree una instancia de servicio de Azure API Management mediante la CLI de Azure.
author: dlepow
ms.service: api-management
ms.topic: quickstart
ms.custom: ''
ms.date: 09/10/2020
ms.author: apimpm
ms.openlocfilehash: 19fc2e1629e7f67063e3cc3eec8cb3707b6dd2e4
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107775860"
---
# <a name="quickstart-create-a-new-azure-api-management-service-instance-by-using-the-azure-cli"></a>Inicio rápido: Creación de una instancia de servicio de Azure API Management mediante la CLI de Azure

Azure API Management (APIM) ayuda a las organizaciones a publicar API para desarrolladores externos, asociados e internos a fin de desbloquear el potencial de sus datos y servicios. API Management proporciona las competencias esenciales para garantizar un programa de API de éxito mediante compromisos con desarrolladores, información detallada empresarial, análisis, seguridad y protección. APIM le permite crear y administrar modernas puertas de enlace de API para los servicios back-end existentes hospedados en cualquier lugar. Para más información, consulte la [introducción](api-management-key-concepts.md).

Este inicio rápido describe los pasos que deben seguirse para crear una instancia de API Management mediante los comandos [az apim](/cli/azure/apim) de la CLI de Azure.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- En este artículo se necesita la versión 2.11.1 de la CLI de Azure, o cualquier versión posterior. Si usa Azure Cloud Shell, ya está instalada la versión más reciente.

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Las instancias de Azure API Management, al igual que otros recursos de Azure, se deben implementar en un grupo de recursos. Los grupos de recursos le permiten organizar y administrar los recursos relacionados de Azure.

En primer lugar, cree un grupo de recursos denominado *myResourceGroup* en la ubicación Centro de EE. UU. con el comando [az group create](/cli/azure/group#az_group_create) siguiente:

```azurecli-interactive
az group create --name myResourceGroup --location centralus
```

## <a name="create-a-new-service"></a>Creación de un nuevo servicio

Ahora que tiene un grupo de recursos, puede crear una instancia de servicio de API Management. Cree una mediante el comando [az apim create](/cli/azure/apim#az_apim_create) y proporcione un nombre de servicio y los detalles del editor. El nombre de servicio debe ser único en Azure. 

En el ejemplo siguiente, se utiliza *myapim* para el nombre del servicio. Actualice el nombre a un valor único. Actualice también el nombre de la organización del editor de la API y la dirección de correo electrónico para recibir notificaciones. 

```azurecli-interactive
az apim create --name myapim --resource-group myResourceGroup \
  --publisher-name Contoso --publisher-email admin@contoso.com \
  --no-wait
```

De forma predeterminada, el comando crea la instancia en el nivel Desarrollador, una opción económica para evaluar Azure API Management. Este nivel no puede utilizarse en producción. Para más información sobre el escalado de los niveles de API Management, consulte [Actualización y escalado](upgrade-and-scale.md). 

> [!TIP]
> Normalmente, se tarda entre 30 y 40 minutos en crear y activar una instancia del servicio API Management en este nivel. El comando anterior usa la opción `--no-wait` para que el comando se devuelva inmediatamente mientras se crea el servicio.

Compruebe el estado de la implementación mediante la ejecución del comando [az apim show](/cli/azure/apim#az_apim_show):

```azurecli-interactive
az apim show --name myapim --resource-group myResourceGroup --output table
```

Inicialmente, la salida será parecida a la siguiente, que muestra el estado `Activating`:

```console
NAME         RESOURCE GROUP    LOCATION    GATEWAY ADDR    PUBLIC IP    PRIVATE IP    STATUS      TIER       UNITS
-----------  ----------------  ----------  --------------  -----------  ------------  ----------  ---------  -------
myapim       myResourceGroup   Central US                                             Activating  Developer  1
```

Después de la activación, el estado es `Online` y la instancia de servicio tiene una dirección de puerta de enlace y una dirección IP pública. Por ahora, estas direcciones no exponen ningún contenido. Por ejemplo: ;;

```console
NAME         RESOURCE GROUP    LOCATION    GATEWAY ADDR                       PUBLIC IP     PRIVATE IP    STATUS    TIER       UNITS
-----------  ----------------  ----------  ---------------------------------  ------------  ------------  --------  ---------  -------
myapim       myResourceGroup   Central US  https://myapim.azure-api.net       203.0.113.1                 Online    Developer  1
```

Cuando la instancia de servicio de API Management está en línea, está listo para usarla. Comience con el tutorial [Importación y publicación de la primera API](import-and-publish.md).

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no lo necesite, puede usar el comando [az group delete](/cli/azure/group#az_group_delete) para eliminar el grupo de recursos y la instancia de servicio de API Management.

```azurecli-interactive
az group delete --name myResourceGroup
```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Importación y publicación de la primera API](import-and-publish.md)
