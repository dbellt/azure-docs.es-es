---
title: Creación de una instancia de Azure IoT Hub mediante un cmdlet de PowerShell | Microsoft Docs
description: Aprenda a usar los cmdlets de PowerShell para crear un grupo de recursos y luego crear un centro de IoT en el grupo de recursos. También aprenda a quitar el centro.
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 08/29/2018
ms.author: robinsh
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: accfb6d4a5191d3c9729ffe075924d44891c0cb3
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110702345"
---
# <a name="create-an-iot-hub-using-the-new-aziothub-cmdlet"></a>Creación de una instancia de IoT Hub mediante el cmdlet New-AzlotHub

[!INCLUDE [iot-hub-resource-manager-selector](../../includes/iot-hub-resource-manager-selector.md)]

## <a name="introduction"></a>Introducción

Puede usar cmdlets de Azure PowerShell para crear y administrar instancias de Azure IoT Hub. En este tutorial se muestra cómo crear una instancia de IoT Hub con PowerShell.

Para completar este procedimiento, se necesita una suscripción de Azure. Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="connect-to-your-azure-subscription"></a>Conexión a su suscripción de Azure

Si usa Cloud Shell, ya ha iniciado sesión en su suscripción. Si, por el contrario, ejecuta PowerShell de forma local, escriba el siguiente comando para iniciar sesión en su suscripción de Azure:

```powershell
# Log into Azure account.
Login-AzAccount
```

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Necesita un grupo de recursos para implementar una instancia de IoT Hub. Puede usar un grupo de recursos existente o crear uno nuevo.

Para crear un grupo de recursos para su instancia de IoT Hub, use el comando [New-AzResourceGroup](/powershell/module/az.Resources/New-azResourceGroup). En este ejemplo se crea un grupo de recursos denominado **MyIoTRG1** en la región **Este de EE. UU.**:

```azurepowershell-interactive
New-AzResourceGroup -Name MyIoTRG1 -Location "East US"
```

## <a name="create-an-iot-hub"></a>Crear un centro de IoT

Para crear una instancia de IoT Hub en el grupo de recursos que creó en el paso anterior, use el comando [New-AzIotHub](/powershell/module/az.IotHub/New-azIotHub). Este ejemplo crea un centro **S1** llamado **MyTestIoTHub** en la región **Este de EE. UU.**:

```azurepowershell-interactive
New-AzIotHub `
    -ResourceGroupName MyIoTRG1 `
    -Name MyTestIoTHub `
    -SkuName S1 -Units 1 `
    -Location "East US"
```

El nombre de su instancia de IoT Hub debe único globalmente.

[!INCLUDE [iot-hub-pii-note-naming-hub](../../includes/iot-hub-pii-note-naming-hub.md)]

Para mostrar todas las instancias de IoT Hub de su suscripción, use el comando [Get-AzIotHub](/powershell/module/az.IotHub/Get-azIotHub):

```azurepowershell-interactive
Get-AzIotHub
```

En este ejemplo se muestra la instancia de IoT Hub estándar S1 que creó en el paso anterior.

Puede eliminar la instancia de IoT Hub con el comando [Remove-AzIotHub](/powershell/module/az.iothub/remove-aziothub):

```azurepowershell-interactive
Remove-AzIotHub `
    -ResourceGroupName MyIoTRG1 `
    -Name MyTestIoTHub
```

También puede quitar un grupo de recursos y todos los recursos que contiene con el comando [Remove-AzResourceGroup](/powershell/module/az.Resources/Remove-azResourceGroup):

```azurepowershell-interactive
Remove-AzResourceGroup -Name MyIoTRG1
```

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha implementado una instancia de IoT Hub mediante un cmdlet de PowerShell, si desea desee seguir explorando, lea los siguientes artículos:

* [Cmdlets de PowerShell para trabajar con una instancia de IoT Hub](/powershell/module/az.iothub/).

* [API REST del proveedor de recursos de IoT Hub](/rest/api/iothub/iothubresource).

Para obtener más información sobre cómo desarrollar para IoT Hub, consulte los siguientes artículos:

* [Introducción a SDK para C](iot-hub-device-sdk-c-intro.md)

* [SDK de IoT de Azure](iot-hub-devguide-sdks.md)

Para explorar aún más las funcionalidades de IoT Hub, consulte:

* [Implementación de IA en dispositivos perimetrales con Azure IoT Edge](../iot-edge/quickstart-linux.md)
