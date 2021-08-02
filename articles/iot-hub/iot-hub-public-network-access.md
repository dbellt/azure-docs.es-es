---
title: Administración del acceso a una red pública para Azure IoT Hub
description: Documentación sobre cómo deshabilitar y habilitar el acceso a la red pública para IoT Hub
author: jlian
ms.author: jlian
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 03/22/2021
ms.openlocfilehash: ece547ac7032e4629a2df48c34b0412ecdc15f54
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110072125"
---
# <a name="managing-public-network-access-for-your-iot-hub"></a>Administración del acceso a la red pública para su instancia de IoT Hub

Para restringir el acceso solo al [punto de conexión privado para su instancia de IoT Hub en su red virtual](virtual-network-support.md), deshabilite el acceso a la red pública. Para ello, use Azure Portal o la API de `publicNetworkAccess`. También puede permitir el acceso público mediante el portal o la API de `publicNetworkAccess`.

## <a name="turn-off-public-network-access-using-azure-portal"></a>Desactivación del acceso a la red pública mediante Azure Portal

1. Visite [Azure Portal](https://portal.azure.com).
2. Vaya a su instancia de IoT Hub. Vaya a **Grupos de recursos**, elija el grupo adecuado y seleccione el IoT Hub.
3. Seleccione **Redes** en el menú de la izquierda.
4. En "Allow public network access to" (Permitir acceso a la red pública para), seleccione **Deshabilitado**.
5. Seleccione **Guardar**.

:::image type="content" source="media/iot-hub-publicnetworkaccess/turn-off-public-network-access.png" alt-text="Imagen que muestra en Azure Portal dónde desactivar el acceso a la red pública" lightbox="media/iot-hub-publicnetworkaccess/turn-off-public-network-access.png":::

Para activar el acceso a la red pública, seleccione **Todas las redes** y **Guardar**.

### <a name="accessing-the-iot-hub-after-disabling-public-network-access"></a>Obtención de acceso al servicio IoT Hub después de la deshabilitación del acceso a la red pública

Una vez deshabilitado el acceso a la red pública, el servicio IoT Hub solo es accesible a través de [su punto de conexión privado de red virtual mediante el vínculo privado de Azure](virtual-network-support.md). Esta restricción incluye el acceso a través de Azure Portal, ya que las llamadas API al servicio IoT Hub se realizan directamente mediante el explorador con sus credenciales.

### <a name="iot-hub-endpoint-ip-address-and-ports-after-disabling-public-network-access"></a>El punto de conexión de IoT Hub, la dirección IP y los puertos después de deshabilitar el acceso a la red pública

IoT Hub es una plataforma como servicio (PaaS) multiinquilino, por lo que diferentes clientes comparten el mismo grupo de recursos de proceso, red y hardware de almacenamiento. Los nombres de host de IoT Hub se asignan a un punto de conexión público con una dirección IP enrutable públicamente a través de Internet. Diferentes clientes comparten este punto de conexión público de IoT Hub, que es accesible para todos los dispositivos de IoT a través de redes de área extensa y de redes locales. 

La deshabilitación del acceso a la red pública se aplica sobre un recurso de IoT Hub específico, lo que garantiza el aislamiento. Para mantener el servicio activo para otros recursos del cliente usando la ruta de acceso pública, su punto de conexión público aún se puede resolver, las direcciones IP aún se pueden detectar y los puertos permanecen abiertos. Esto no es motivo de preocupación, ya que Microsoft integra varias capas de seguridad para garantizar el aislamiento completo entre los inquilinos. Para más información, consulte [Aislamiento en la nube pública de Azure](../security/fundamentals/isolation-choices.md#tenant-level-isolation).

### <a name="ip-filter"></a>Filtro IP

Si el acceso a la red pública está deshabilitado, todas las reglas de [Filtro IP](iot-hub-ip-filtering.md) se omiten. Esto se debe a que se bloquean todas las direcciones IP de la red pública de Internet. Para usar el filtro IP, utilice la opción **Selected IP ranges** (Intervalos IP seleccionados).

### <a name="bug-fix-with-built-in-event-hub-compatible-endpoint"></a>Corrección de errores con el punto de conexión compatible con el centro de eventos integrado

Hay un error con IoT Hub en el que el [punto de conexión compatible con el centro de eventos integrado](iot-hub-devguide-messages-read-builtin.md) sigue siendo accesible a través de la red pública de Internet cuando se deshabilita el acceso a la red pública para IoT Hub. Para obtener más información y ponerse en contacto con nosotros en relación con este error, consulte [Al deshabilitar el acceso de la red pública a Azure IoT Hub, se deshabilitará el acceso al punto de conexión integrado del centro de eventos integrado](https://azure.microsoft.com/updates/iot-hub-public-network-access-bug-fix).

## <a name="turn-on-network-access-using-azure-portal"></a>Activación del acceso a la red mediante Azure Portal

1. Visite [Azure Portal](https://portal.azure.com).
2. Vaya a su instancia de IoT Hub. Vaya a **Grupos de recursos**, elija el grupo adecuado y seleccione el centro de conectividad.
3. Seleccione **Redes** en el menú de la izquierda.
4. En "Permitir acceso a la red pública para", seleccione **Intervalos IP seleccionados**.
5. En el cuadro de diálogo **Filtro IP** que se abre, seleccione **Agregar la dirección IP del cliente** y escriba un nombre y un intervalo de direcciones.
6. Seleccione **Guardar**. Si el botón está atenuado, asegúrese de que la dirección IP del cliente ya se ha agregado como filtro IP.

:::image type="content" source="media/iot-hub-publicnetworkaccess/turn-on-public-network-access.png" alt-text="Imagen que muestra en Azure Portal dónde activar el acceso a la red pública":::

### <a name="turn-on-all-network-ranges"></a>Activación de todos los intervalos de red

1. Vaya a su instancia de IoT Hub. Vaya a **Grupos de recursos**, elija el grupo adecuado y seleccione el centro de conectividad.
1. Seleccione **Redes** en el menú de la izquierda.
1. En "Permitir acceso a la red pública para", seleccione **Todas las redes**.
1. Seleccione **Guardar**.

### <a name="check-iot-hub-access-using-cloud-shell"></a>Comprobación del acceso a IoT Hub mediante Cloud Shell

Puede comprobar el acceso a IoT Hub mediante Azure Cloud Shell. Asegúrese de que ha activado todos los intervalos de red y, a continuación, emita los siguientes comandos. Reemplace "SubscriptionName" por el nombre de la suscripción y "MyIoTHub" por el nombre del centro de conectividad.

```azurecli
  az account set -s "SubscriptionName"
  az iot hub device-identity list --hub-name "MyIoTHub"
```

```azurepowershell
  Set-AzContext -Name "SubscriptionName"
  Get-AzIoTHubDevice -IotHubName "MyIoTHub"
```
### <a name="troubleshooting"></a>Solución de problemas

Si tiene problemas para acceder a IoT Hub, la configuración de red podría ser el problema. Por ejemplo, si ve el siguiente mensaje de error al intentar acceder a la página dispositivos IoT, compruebe la página **Redes** para ver si el acceso a la red pública está deshabilitado o restringido a los intervalos IP seleccionados.

```
  Unable to retrieve devices. Please ensure that your network connection is online and network settings allow connections from your IP address.
```

Para obtener acceso a IoT Hub, solicite permiso al administrador de TI para agregar la dirección IP en el intervalo de direcciones IP o para habilitar el acceso de red pública a todas las redes. Si no se puede resolver el problema, compruebe la configuración de red local o póngase en contacto con el administrador de red local para corregir la conectividad con IoT Hub. Por ejemplo, a veces un proxy de la red local puede interferir con el acceso a IoT Hub.

Si los comandos anteriores no funcionan o no puede activar todos los intervalos de red, póngase en contacto con el soporte técnico de Microsoft.
