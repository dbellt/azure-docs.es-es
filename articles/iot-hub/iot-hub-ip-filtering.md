---
title: Filtros IP de IoT Hub Azure | Microsoft Docs
description: Aquí se explica cómo usar el filtrado IP para permitir conexiones desde direcciones IP específicas a Azure IoT Hub.
author: jlian
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 03/22/2021
ms.author: jlian
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 20edf9c0977db3b9557214220df1a76e7363e704
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110695700"
---
# <a name="use-ip-filters"></a>Uso de filtros IP

La seguridad es un aspecto importante de cualquier solución de IoT basada en Azure IoT Hub. En ciertas ocasiones necesitará especificar explícitamente las direcciones IP desde las que se pueden conectar los dispositivos como parte de la configuración de seguridad. La característica de *filtro IP* le permite configurar reglas para rechazar o aceptar tráfico de direcciones IPv4 específicas.

## <a name="when-to-use"></a>Cuándo se usa

Use un filtro IP para recibir tráfico únicamente de un intervalo especificado de direcciones IP y rechazar todo lo demás. Por ejemplo, está usando el centro de IoT con [Azure Express Route](../expressroute/expressroute-faqs.md#supported-services) para crear conexiones privadas entre un centro de IoT y la infraestructura local.

## <a name="default-setting"></a>Valor predeterminado

Para ir a la página Configuración de filtro IP, seleccione **Redes**, **Acceso público** y, a continuación, **elija rangos IP seleccionados**:

:::image type="content" source="media/iot-hub-ip-filtering/ip-filter-default.png" alt-text="Configuración predeterminada de filtro de direcciones IP de IoT Hub":::

De forma predeterminada, la cuadrícula de **filtro IP** del portal para un centro de IoT está vacía. Este valor predeterminado indica que el centro bloquea las conexiones desde todas las direcciones IP. Este valor predeterminado es equivalente a una regla que bloquee el intervalo de direcciones IP `0.0.0.0/0`.

## <a name="add-or-edit-an-ip-filter-rule"></a>Incorporación o edición de una regla de filtro IP

Para agregar una regla de filtro IP, seleccione **+ Agregar regla de filtro IP**. Para agregar rápidamente la dirección IP del equipo, haga clic en **Agregar la dirección IP del cliente**. 

:::image type="content" source="./media/iot-hub-ip-filtering/ip-filter-add-rule.png" alt-text="Incorporación de una regla de filtro IP a una instancia de IoT Hub":::

Después de seleccionar **Agregar regla de filtro IP**, rellene los campos. Estos campos se rellenan automáticamente si ha elegido agregar la dirección IP del cliente.

:::image type="content" source="./media/iot-hub-ip-filtering/ip-filter-after-selecting-add.png" alt-text="Después de seleccionar Agregar regla de filtro IP":::

* Escriba un **nombre** para la regla de filtro IP. Este nombre debe ser una cadena única de hasta 128 caracteres alfanuméricos que no distinga mayúsculas de minúsculas. Solo se aceptan los caracteres alfanuméricos de 7 bits ASCII más `{'-', ':', '/', '\', '.', '+', '%', '_', '#', '*', '?', '!', '(', ')', ',', '=', '@', ';', '''}`.

* Proporcione una única dirección IPv4 o un bloque de direcciones IP en la notación CIDR. Por ejemplo, en notación CIDR, 192.168.100.0/22 representa las direcciones IPv4 de 1024 de 192.168.100.0 a 192.168.103.255.

Después de rellenar los campos, seleccione **Guardar** para guardar la regla. Se mostrará una alerta que le informará de que la actualización está en curso.

:::image type="content" source="./media/iot-hub-ip-filtering/ip-filter-save-new-rule.png" alt-text="Notificación acerca de cómo guardar una regla de filtro IP":::

La opción **Agregar** está deshabilitada cuando se alcanza el máximo de 10 reglas de filtro IP.

Para editar una regla existente, seleccione los datos que quiere cambiar, realice el cambio y, a continuación, seleccione **Guardar** para guardar la edición.

## <a name="delete-an-ip-filter-rule"></a>Eliminación de una regla de filtro IP

Para eliminar una regla de filtro IP, seleccione el icono de la papelera de esa fila y, a continuación, seleccione **Guardar**. Se quita la regla y se guarda el cambio.

:::image type="content" source="./media/iot-hub-ip-filtering/ip-filter-delete-rule.png" alt-text="Eliminación de una regla de filtro IP de IoT Hub":::

## <a name="apply-ip-filter-rules-to-the-built-in-event-hub-compatible-endpoint"></a>Aplicación de reglas de filtro IP al punto de conexión integrado compatible con el centro de eventos

Para aplicar las reglas de filtro IP al punto de conexión integrado compatible con el centro de eventos, active la casilla situada junto a **Apply IP filters to the built-in endpoint?** (¿Aplicar filtros IP al punto de conexión integrado?) y seleccione **Guardar**.

:::image type="content" source="media/iot-hub-ip-filtering/ip-filter-built-in-endpoint.png" alt-text="Imagen que muestra el control de alternancia del punto de conexión integrado y Guardar":::

> [!NOTE]
> Esta opción no está disponible en los centros de IoT gratuitos (F1). Para aplicar reglas de filtro IP al punto de conexión integrado, use un centro de IoT de pago.

Al habilitar esta opción, las reglas de filtro IP se replican en el punto de conexión integrado, por lo que solo los intervalos de direcciones IP de confianza pueden acceder a él.

Si deshabilita esta opción, todas las direcciones IP pueden acceder al punto de conexión integrado. Este comportamiento puede ser útil si quiere leer desde el punto de conexión con servicios con direcciones IP cambiantes, como Azure Stream Analytics. 

## <a name="how-filter-rules-are-applied"></a>Cómo se aplican las reglas de filtro

Las reglas de filtro IP se aplican en el nivel de servicio de IoT Hub. Por lo tanto, las reglas de filtro IP se aplican a todas las conexiones de los dispositivos y aplicaciones de back-end mediante un protocolo admitido. Además, puede elegir si el [punto de conexión integrado compatible con el centro de eventos](iot-hub-devguide-messages-read-builtin.md) (no mediante la cadena de conexión de IoT Hub) está enlazado a estas reglas. 

Cualquier intento de conexión desde una dirección IP no permitida de forma explícita recibe un código de estado 401 no autorizado y la descripción. El mensaje de respuesta no menciona la regla IP. Rechazar direcciones de IP puede evitar que otros servicios de Azure (por ejemplo, Azure Stream Analytics, Azure Virtual Machines o el Explorador de dispositivos de Azure Portal) interactúen con el centro de IoT.

> [!NOTE]
> Si debe usar Azure Stream Analytics (ASA) para leer mensajes de un centro de eventos con el filtro IP habilitado, **deshabilite** la opción **Apply IP filters to the built-in endpoint** (Aplicar filtros IP al punto de conexión integrado) y luego use el nombre y el punto de conexión compatibles con el centro de eventos del centro de IoT para agregar manualmente una [entrada de streaming de Event Hubs](../stream-analytics/stream-analytics-define-inputs.md#stream-data-from-event-hubs) en ASA.

### <a name="ordering"></a>Ordenación

Las reglas de filtro IP son de tipo *permitir* y se aplican sin ordenación. Solo las direcciones IP que se agregan pueden conectarse a IoT Hub. 

Por ejemplo, si quiere aceptar direcciones del intervalo `192.168.100.0/22` y rechazar todo lo demás, solo tiene que agregar una regla a la cuadrícula con el intervalo de direcciones `192.168.100.0/22`.

### <a name="azure-portal"></a>Azure Portal 

También se aplican las reglas de filtro IP al utilizar IoT Hub a través de Azure Portal. Esto se debe a que las llamadas API al servicio IoT Hub se realizan directamente mediante el explorador con sus credenciales, lo cual es similar a lo que ocurre con otros servicios de Azure. Para obtener acceso a IoT Hub mediante Azure Portal cuando está habilitado el filtro IP, agregue la dirección IP del equipo a la lista de permitidos. 

## <a name="retrieve-and-update-ip-filters-using-azure-cli"></a>Recuperación y actualización de los filtros IP mediante la CLI de Azure

Los filtros IP de IoT Hub se pueden recuperar y actualizar con la [CLI de Azure](/cli/azure/).

Para recuperar los filtros IP actuales de IoT Hub, ejecute:

```azurecli-interactive
az resource show -n <iothubName> -g <resourceGroupName> --resource-type Microsoft.Devices/IotHubs
```

Esto devolverá un objeto JSON, donde se enumeran los filtros IP existentes bajo la clave `properties.networkRuleSets`:

```json
{
...
    "properties": {
        "networkRuleSets": {
            "defaultAction": "Deny",
            "applyToBuiltInEventHubEndpoint": true,
            "ipRules": [{
                    "filterName": "TrustedFactories",
                    "action": "Allow",
                    "ipMask": "1.2.3.4/5"
                },
                {
                    "filterName": "TrustedDevices",
                    "action": "Allow",
                    "ipMask": "1.1.1.1/1"
                }
            ]
        }
    }
}
```

Para agregar un nuevo filtro IP de IoT Hub, ejecute:

```azurecli-interactive
az resource update -n <iothubName> -g <resourceGroupName> --resource-type Microsoft.Devices/IotHubs --add properties.networkRuleSets.ipRules "{\"action\":\"Allow\",\"filterName\":\"TrustedIP\",\"ipMask\":\"192.168.0.1\"}"
```

Para quitar un filtro IP existente en IoT Hub, ejecute:

```azurecli-interactive
az resource update -n <iothubName> -g <resourceGroupName> --resource-type Microsoft.Devices/IotHubs --add properties.networkRuleSets.ipRules <ipFilterIndexToRemove>
```

Aquí, `<ipFilterIndexToRemove>` tiene que corresponder a la ordenación de filtros IP de `properties.networkRuleSets.ipRules` de IoT Hub.

## <a name="retrieve-and-update-ip-filters-using-azure-powershell"></a>Recuperación y actualización de los filtros IP mediante Azure PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Los filtros IP de IoT Hub se pueden recuperar y establecer con [Azure PowerShell](/powershell/azure/).

```powershell
# Get your IoT Hub resource using its name and its resource group name
$iothubResource = Get-AzResource -ResourceGroupName <resourceGroupNmae> -ResourceName <iotHubName> -ExpandProperties

# Access existing IP filter rules
$iothubResource.Properties.networkRuleSets.ipRules |% { Write-host $_ }

# Construct a new IP filter
$filter = @{'filterName'='TrustedIP'; 'action'='Allow'; 'ipMask'='192.168.0.1'}

# Add your new IP filter rule
$iothubResource.Properties.networkRuleSets.ipRules += $filter

# Remove an existing IP filter rule using its name, e.g., 'GoodIP'
$iothubResource.Properties.networkRuleSets.ipRules = @($iothubResource.Properties.networkRuleSets.ipRules | Where 'filterName' -ne 'GoodIP')

# Update your IoT Hub resource with your updated IP filters
$iothubResource | Set-AzResource -Force
```

## <a name="update-ip-filter-rules-using-rest"></a>Actualización de las reglas de filtro IP con REST


También puede recuperar y modificar el filtro IP de IoT Hub mediante el punto de conexión REST del proveedor de recursos de Azure. Consulte `properties.networkRuleSets` en [método createorupdate](/rest/api/iothub/iothubresource/createorupdate).

## <a name="ip-filter-classic-retirement"></a>Retirada de filtro IP (clásico)

Se ha retirado el filtro IP clásico. Para obtener más información, vea [Filtro IP clásico de IoT Hub y cómo actualizar](iot-hub-ip-filter-classic.md).

## <a name="next-steps"></a>Pasos siguientes

Para explorar aún más las funcionalidades de IoT Hub, consulte:

* [Métricas de IoT Hub](./monitor-iot-hub.md)
* [Compatibilidad de IoT Hub con redes virtuales mediante Private Link e identidad administrada](virtual-network-support.md)
* [Administración del acceso a la red pública para su instancia de IoT Hub](iot-hub-public-network-access.md)
* [Supervisión de IoT Hub](monitor-iot-hub.md)
