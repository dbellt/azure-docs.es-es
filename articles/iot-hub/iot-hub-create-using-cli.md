---
title: Creación de un centro de IoT mediante la CLI de Azure | Microsoft Docs
description: Aprenda a usar los comandos de la CLI de Azure para crear un grupo de recursos y, luego, crear en él un centro de IoT. También aprenda a quitar el centro.
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 08/23/2018
ms.author: robinsh
ms.openlocfilehash: eff0085a4a739e0831b25b1bd28cd234fdbcde3d
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107766468"
---
# <a name="create-an-iot-hub-using-the-azure-cli"></a>Creación de una instancia de IoT Hub mediante la CLI de Azure

[!INCLUDE [iot-hub-resource-manager-selector](../../includes/iot-hub-resource-manager-selector.md)]

En este artículo se muestra cómo utilizar la CLI de Azure para crear una instancia de IoT Hub.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

## <a name="create-an-iot-hub"></a>Creación de un IoT Hub

Use la CLI de Azure para crear un grupo de recursos y, luego, agregue una instancia de IoT Hub.

1. Cuando crea una instancia de IoT Hub, debe crearla en un grupo de recursos. Use un grupo de recursos existente o ejecute el [comando siguiente para crear un grupo de recursos](/cli/azure/resource):
    
   ```azurecli-interactive
   az group create --name {your resource group name} --location westus
   ```

   > [!TIP]
   > El ejemplo anterior crea el grupo de recursos en la ubicación del oeste de EE. UU. Puede ejecutar el siguiente comando para ver una lista de las ubicaciones disponibles: 
   >
   > ```azurecli-interactive
   > az account list-locations -o table
   > ```
   >

2. Ejecute el [comando siguiente para crear una instancia de IoT Hub](/cli/azure/iot/hub#az_iot_hub_create) en el grupo de recursos, con un nombre único global para la instancia de IoT Hub:
    
   ```azurecli-interactive
   az iot hub create --name {your iot hub name} \
      --resource-group {your resource group name} --sku S1
   ```

   [!INCLUDE [iot-hub-pii-note-naming-hub](../../includes/iot-hub-pii-note-naming-hub.md)]


El comando anterior crea una instancia de IoT Hub en el plan de tarifa S1 según el que factura. Para obtener más información, vea el artículo sobre [precios de IoT Hub](https://azure.microsoft.com/pricing/details/iot-hub/).

## <a name="remove-an-iot-hub"></a>Eliminación de una instancia de IoT Hub

Puede usar la CLI de Azure para [eliminar un recurso individual](/cli/azure/resource), como una instancia de IoT Hub, o para eliminar un grupo de recursos y todos sus recursos, incluida cualquier instancia de IoT Hub.

Ejecute el comando siguiente para [eliminar una instancia de IoT Hub](/cli/azure/iot/hub#az_iot_hub_delete):

```azurecli-interactive
az iot hub delete --name {your iot hub name} -\
  -resource-group {your resource group name}
```

Ejecute el comando siguiente para [eliminar un grupo de recursos](/cli/azure/group#az_group_delete) y todos sus recursos:

```azurecli-interactive
az group delete --name {your resource group name}
```

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre cómo usar una instancia de IoT Hub, consulte los siguientes artículos:

* [Guía para desarrolladores de IoT Hub](iot-hub-devguide.md)
* [Uso de Azure Portal para administrar IoT Hub](iot-hub-create-through-portal.md)