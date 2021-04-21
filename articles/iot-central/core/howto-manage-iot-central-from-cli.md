---
title: Administración de IoT Central desde la CLI de Azure | Microsoft Docs
description: En este artículo se describe cómo crear y administrar aplicaciones de IoT Central con la CLI. Puede ver, modificar y quitar la aplicación mediante la CLI.
services: iot-central
ms.service: iot-central
author: dominicbetts
ms.author: dobett
ms.date: 03/27/2020
ms.topic: how-to
ms.custom: devx-track-azurecli
manager: philmea
ms.openlocfilehash: c30781cb83436e15a217a1d43c0e39facae9f52d
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107770424"
---
# <a name="manage-iot-central-from-azure-cli"></a>Administración de IoT Central desde la CLI de Azure

[!INCLUDE [iot-central-selector-manage](../../../includes/iot-central-selector-manage.md)]

En lugar de crear y administrar aplicaciones de IoT Central desde el sitio web del [administrador de aplicaciones de Azure IoT Central](https://aka.ms/iotcentral), puede usar la [CLI de Azure](/cli/azure/) para administrar las aplicaciones.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

 - Si tiene que ejecutar los comandos de la CLI en otra suscripción a Azure, consulte [Cambio de la suscripción activa](/cli/azure/manage-azure-subscriptions-azure-cli#change-the-active-subscription).

## <a name="create-an-application"></a>Crear una aplicación

[!INCLUDE [Warning About Access Required](../../../includes/iot-central-warning-contribitorrequireaccess.md)]

Use el comando [az iot central app create](/cli/azure/iot/central/app#az_iot_central_app_create) para crear una aplicación de IoT Central en su suscripción de Azure. Por ejemplo:

```azurecli-interactive
# Create a resource group for the IoT Central application
az group create --location "East US" \
    --name "MyIoTCentralResourceGroup"
```

```azurecli-interactive
# Create an IoT Central application
az iot central app create \
  --resource-group "MyIoTCentralResourceGroup" \
  --name "myiotcentralapp" --subdomain "mysubdomain" \
  --sku ST1 --template "iotc-pnp-preview" \
  --display-name "My Custom Display Name"
```

Estos comandos crean en primer lugar un grupo de recursos en la región Este de EE. UU. para la aplicación. En la siguiente tabla se describen los parámetros utilizados con el comando **az iot central app create**:

| Parámetro         | Descripción |
| ----------------- | ----------- |
| resource-group    | Grupo de recursos que contiene a la aplicación. Este grupo de recursos ya debe existir en la suscripción. |
| ubicación          | Este comando usa de forma predeterminada la ubicación del grupo de recursos. Actualmente, se puede crear una aplicación de IoT Central en las zonas geográficas **Australia**, **Asia Pacífico**, **Europa**, **Estados Unidos**, **Reino Unido** y **Japón**. |
| name              | Nombre de la aplicación en Azure Portal. |
| subdomain         | Subdominio en la dirección URL de la aplicación. En el ejemplo, la dirección URL de la aplicación es `https://mysubdomain.azureiotcentral.com`. |
| sku               | Actualmente, puede usar **ST1** o **ST2**. Consulte [Precios de Azure IoT Central](https://azure.microsoft.com/pricing/details/iot-central/). |
| template          | Plantilla de aplicación que se va a usar. Para más información, vea la tabla siguiente. |
| display-name      | Nombre de la aplicación tal como se muestra en la interfaz de usuario. |

[!INCLUDE [iot-central-template-list](../../../includes/iot-central-template-list.md)]

## <a name="view-your-applications"></a>Visualización de sus aplicaciones

Use el comando [az iot central app list](/cli/azure/iot/central/app#az_iot_central_app_list) para enumerar las aplicaciones de IoT Central y ver los metadatos.

## <a name="modify-an-application"></a>Modificación de una aplicación

Use el comando [az iot central app update](/cli/azure/iot/central/app#az_iot_central_app_update) para actualizar los metadatos de una aplicación de IoT Central. Por ejemplo, para cambiar el nombre para mostrar de la aplicación, use:

```azurecli-interactive
az iot central app update --name myiotcentralapp \
  --resource-group MyIoTCentralResourceGroup \
  --set displayName="My new display name"
```

## <a name="remove-an-application"></a>Eliminación de una aplicación

Use el comando [az iot central app delete](/cli/azure/iot/central/app#az_iot_central_app_delete) para eliminar una aplicación de IoT Central. Por ejemplo:

```azurecli-interactive
az iot central app delete --name myiotcentralapp \
  --resource-group MyIoTCentralResourceGroup
```

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha aprendido a administrar aplicaciones de Azure IoT Central desde la CLI de Azure, le sugerimos el paso siguiente:

> [!div class="nextstepaction"]
> [Administrar su aplicación](howto-administer.md)
