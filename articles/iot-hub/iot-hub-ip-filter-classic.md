---
title: Filtro IP clásico de Azure IoT Hub (en desuso) | Microsoft Docs
description: Cómo actualizar el filtro IP clásico y cuáles son las ventajas
author: jlian
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 10/16/2020
ms.author: jlian
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 2840557728e7203aeffad0b0da4f631ec6e3837e
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110695723"
---
# <a name="iot-hub-classic-ip-filter-and-how-to-upgrade"></a>Filtro IP clásico de IoT Hub y cómo actualizarlo 

El filtro IP actualizado para IoT Hub protege el punto de conexión integrado y es seguro de forma predeterminada. Aunque nos esforzamos por no realizar nunca cambios importantes, el modelo de seguridad mejorado del filtro IP actualizado es incompatible con el filtro IP clásico, por lo que anunciamos su retirada. Para más información sobre el nuevo filtro IP actualizado, consulte [Novedades](#whats-new) y [Filtros IP de IoT Hub](iot-hub-ip-filtering.md).

Para evitar la interrupción del servicio, debe realizar la actualización guiada antes de la fecha límite de migración, momento en el que la actualización se realizará automáticamente. Para más información sobre la escala de tiempo de migración, consulte [Actualización de Azure](https://aka.ms/ipfilterv2azupdate).

## <a name="how-to-upgrade"></a>Procedimiento de actualización

1.  Visite Azure Portal.
2.  Vaya a su instancia de IoT Hub.
3.  Seleccione **Redes** en el menú de la izquierda.
4.  Verá un mensaje que le pide que actualice el filtro IP al nuevo modelo. Seleccione **Sí** para continuar.
    :::image type="content" source="media/iot-hub-ip-filter-classic/ip-filter-upgrade-banner.png" alt-text="Imagen que muestra el aviso para actualizar el filtro IP clásico":::
5.  Dado que el nuevo filtro IP bloquea todas las direcciones IP de forma predeterminada, la actualización elimina las reglas de denegación individuales, pero le ofrece la oportunidad de revisarlas antes de guardarlas. Revise cuidadosamente las reglas para asegurarse de que son adecuadas en su caso.
6.  Siga los avisos para finalizar la actualización.

## <a name="whats-new"></a>Novedades

### <a name="secure-by-default"></a>Seguro de forma predeterminada

El filtro IP clásico permite implícitamente que todas las direcciones IP se conecten a IoT Hub de forma predeterminada, lo que no concuerda del todo con los escenarios de seguridad de red más comunes. Normalmente, querrá que solo las direcciones IP de confianza puedan conectarse a su centro de IoT y rechazar todas las demás. Para lograr este objetivo con el filtro IP clásico, este es un proceso de varios pasos. Por ejemplo, si quiere aceptar únicamente el tráfico de `192.168.100.0/22`, debe hacer lo siguiente:

1. Configure una única regla de *permiso* para `192.168.100.0/22`.
1. Configure una regla de *bloqueo* diferente para `0.0.0.0/0` (la regla "bloquear todo").
1. Asegúrese de que las reglas estén ordenadas correctamente (la regla de permiso debe estar encima de la regla de bloqueo).

En la práctica, este proceso de varios pasos resulta confuso. Los usuarios no configuran la regla "bloquear todo" o no ordenan las reglas correctamente, lo que da lugar a una exposición imprevista. 

El nuevo filtro IP bloquea todas las direcciones IP de forma predeterminada. Solo los intervalos IP que se agregan explícitamente pueden conectarse a IoT Hub. En el ejemplo anterior, los pasos 2 y 3 ya no son necesarios. Este nuevo comportamiento simplifica la configuración y se rige por el [principio seguro de forma predeterminada](https://wikipedia.org/wiki/Secure_by_default).

### <a name="protect-the-built-in-event-hub-compatible-endpoint"></a>Protección del punto de conexión integrado compatible con el centro de eventos

No se puede aplicar el filtro IP clásico al punto de conexión integrado. Esta limitación significa que, si se configura el evento con una regla "bloquear todo" (bloquear `0.0.0.0/0`), el punto de conexión integrado sigue siendo accesible desde cualquier dirección IP.

El nuevo filtro IP proporciona una opción para aplicar reglas al punto de conexión integrado, lo que reduce la exposición a amenazas de seguridad de la red.

:::image type="content" source="media/iot-hub-ip-filter-classic/ip-filter-built-in-endpoint.png" alt-text="Imagen que muestra el botón de alternancia para aplicar o no los filtros al punto de conexión integrado":::

> [!NOTE]
> Esta opción no está disponible para los centros de IoT gratuitos (F1). Para aplicar reglas de filtro IP al punto de conexión integrado, use un centro de IoT de pago.

### <a name="api-impact"></a>Efecto de la API

El filtro IP actualizado está disponible en la API de recursos de IoT Hub desde la versión `2020-08-31` (así como en `2020-08-31-preview`) y en adelante. El filtro IP clásico todavía está disponible en todas las versiones de API, pero se quitará en una versión futura de la API cercana a la fecha límite de migración. Para más información sobre la escala de tiempo de migración, consulte [Actualización de Azure](https://aka.ms/ipfilterv2azupdate).

## <a name="tip-try-the-changes-before-they-apply"></a>Sugerencia: Pruebe los cambios antes de aplicarlos.

Dado que el nuevo filtro IP bloquea todas las direcciones IP de forma predeterminada, las reglas de bloqueo individuales ya no son compatibles. Por lo tanto, el proceso de actualización guiado quita estas reglas de bloqueo individuales. 

Para probar el cambio con el filtro IP clásico:

1. Vaya a la pestaña **Redes** de IoT Hub.
1. Anote la configuración del filtro IP existente (clásico), en caso de que desee revertirla.
1. Junto a las reglas con la opción **Bloquear**, seleccione el icono de la papelera para quitarlas.
1. Agregue otra regla en la parte inferior con `0.0.0.0/0` y elija **Bloquear**.
1. Seleccione **Guardar**.

Esta configuración imita el comportamiento del nuevo filtro IP después de actualizar desde la versión clásica. Una excepción es la protección del punto de conexión integrado, que no es posible probar con el filtro IP clásico. Sin embargo, esta característica es opcional, por lo que no tiene que usarla si cree que podría interrumpir algo.

## <a name="tip-check-diagnostic-logs-for-all-ip-connections-to-your-iot-hub"></a>Sugerencia: Compruebe los registros de diagnóstico de todas las conexiones IP a su centro de IoT.

Para garantizar una transición fluida, compruebe los registros de diagnóstico en la categoría Conexiones. Busque la propiedad `maskedIpAddress` para ver si los intervalos son los esperados. Recuerde: El nuevo filtro IP bloqueará todas las direcciones IP que no se hayan agregado explícitamente.

## <a name="iot-hub-classic-ip-filter-documentation-retired"></a>Documentación del filtro IP clásico de IoT Hub (retirada)

> [!IMPORTANT]
> A continuación, se muestra la documentación original del filtro IP clásico, que está en proceso de retirada.

La seguridad es un aspecto importante de cualquier solución de IoT basada en Azure IoT Hub. En ciertas ocasiones necesitará especificar explícitamente las direcciones IP desde las que se pueden conectar los dispositivos como parte de la configuración de seguridad. La característica de *filtro IP* le permite configurar reglas para rechazar o aceptar tráfico de direcciones IPv4 específicas.

### <a name="when-to-use"></a>Cuándo se usa

Hay dos casos específicos cuando resulta útil bloquear los puntos de conexión de IoT Hub para determinadas direcciones IP:

* IoT Hub debe recibir tráfico solo de un intervalo concreto de direcciones IP y rechazar todo lo demás. Por ejemplo, cuando se usa IoT Hub con [Azure ExpressRoute](../expressroute/expressroute-faqs.md#supported-services) para crear conexiones privadas entre una instancia de IoT Hub y la infraestructura local.

* Cuando necesite rechazar el tráfico de direcciones IP que el administrador de IoT Hub haya identificado como sospechosas.

### <a name="how-filter-rules-are-applied"></a>Cómo se aplican las reglas de filtro

Las reglas de filtro IP se aplican en el nivel de servicio de IoT Hub. Por lo tanto, las reglas de filtro IP se aplican a todas las conexiones de los dispositivos y aplicaciones de back-end mediante un protocolo admitido. Sin embargo, los clientes que leen directamente desde el [punto de conexión compatible con el centro de eventos integrado](iot-hub-devguide-messages-read-builtin.md) (no a través de la cadena de conexión IoT Hub) no están enlazados a las reglas de filtro IP. 

Cualquier intento de conexión desde una dirección IP que coincida con una regla IP de rechazo en su centro de IoT recibe un código de estado 401 no autorizado y la descripción. El mensaje de respuesta no menciona la regla IP. Rechazar direcciones de IP puede evitar que otros servicios de Azure (por ejemplo, Azure Stream Analytics, Azure Virtual Machines o el Explorador de dispositivos de Azure Portal) interactúen con el centro de IoT.

> [!NOTE]
> Si usa Azure Stream Analytics (ASA) para leer mensajes de una instancia de IoT Hub con el filtrado IP habilitado, use el punto de conexión y el nombre compatibles con el centro de eventos del centro de IoT para agregar manualmente una [entrada de flujo de datos de Event Hub](../stream-analytics/stream-analytics-define-inputs.md#stream-data-from-event-hubs) en ASA.

### <a name="default-setting"></a>Valor predeterminado

De forma predeterminada, la cuadrícula de **filtro IP** del portal para un centro de IoT está vacía. Este ajuste predeterminado indica que el centro acepta conexiones de cualquier dirección IP. Esta configuración predeterminada es equivalente a una regla que acepta el intervalo de direcciones IP 0.0.0.0/0.

Para ir a la página Configuración de filtro IP, seleccione **Redes**, **Acceso público** y, a continuación, **elija rangos IP seleccionados**:

:::image type="content" source="media/iot-hub-ip-filter-classic/ip-filter-default.png" alt-text="Configuración predeterminada de filtro de direcciones IP de IoT Hub":::

### <a name="add-or-edit-an-ip-filter-rule"></a>Incorporación o edición de una regla de filtro IP

Para agregar una regla de filtro IP, seleccione **+ Agregar regla de filtro IP**.

:::image type="content" source="./media/iot-hub-ip-filter-classic/ip-filter-add-rule.png" alt-text="Incorporación de una regla de filtro IP a una instancia de IoT Hub":::

Después de seleccionar **Agregar regla de filtro IP**, rellene los campos.

:::image type="content" source="./media/iot-hub-ip-filter-classic/ip-filter-after-selecting-add.png" alt-text="Después de seleccionar Agregar regla de filtro IP":::

* Escriba un **nombre** para la regla de filtro IP. Debe ser una cadena única de hasta 128 caracteres alfanuméricos que no distinga mayúsculas de minúsculas. Solo se aceptan los caracteres alfanuméricos de 7 bits ASCII más `{'-', ':', '/', '\', '.', '+', '%', '_', '#', '*', '?', '!', '(', ')', ',', '=', '@', ';', '''}`.

* Proporcione una única dirección IPv4 o un bloque de direcciones IP en la notación CIDR. Por ejemplo, en notación CIDR, 192.168.100.0/22 representa las direcciones IPv4 de 1024 de 192.168.100.0 a 192.168.103.255.

* Seleccione un **Permitir** o **Bloquear** como **acción** para la regla de filtro IP.

Después de rellenar los campos, seleccione **Guardar** para guardar la regla. Se mostrará una alerta que le informará de que la actualización está en curso.

:::image type="content" source="./media/iot-hub-ip-filter-classic/ip-filter-save-new-rule.png" alt-text="Notificación acerca de cómo guardar una regla de filtro IP":::

La opción **Agregar** está deshabilitada cuando se alcanza el máximo de 10 reglas de filtro IP.

Para editar una regla existente, seleccione los datos que quiere cambiar, realice el cambio y, a continuación, seleccione **Guardar** para guardar la edición.

### <a name="delete-an-ip-filter-rule"></a>Eliminación de una regla de filtro IP

Para eliminar una regla de filtro IP, seleccione el icono de la papelera de esa fila y, a continuación, seleccione **Guardar**. Se quita la regla y se guarda el cambio.

:::image type="content" source="./media/iot-hub-ip-filter-classic/ip-filter-delete-rule.png" alt-text="Eliminación de una regla de filtro IP de IoT Hub":::

### <a name="retrieve-and-update-ip-filters-using-azure-cli"></a>Recuperación y actualización de los filtros IP mediante la CLI de Azure

Los filtros IP de IoT Hub se pueden recuperar y actualizar con la [CLI de Azure](/cli/azure/).

Para recuperar los filtros IP actuales de IoT Hub, ejecute:

```azurecli-interactive
az resource show -n <iothubName> -g <resourceGroupName> --resource-type Microsoft.Devices/IotHubs
```

Esto devolverá un objeto JSON, donde se enumeran los filtros IP existentes bajo la clave `properties.ipFilterRules`:

```json
{
...
    "properties": {
        "ipFilterRules": [
        {
            "action": "Reject",
            "filterName": "MaliciousIP",
            "ipMask": "6.6.6.6/6"
        },
        {
            "action": "Allow",
            "filterName": "GoodIP",
            "ipMask": "131.107.160.200"
        },
        ...
        ],
    },
...
}
```

Para agregar un nuevo filtro IP de IoT Hub, ejecute:

```azurecli-interactive
az resource update -n <iothubName> -g <resourceGroupName> --resource-type Microsoft.Devices/IotHubs --add properties.ipFilterRules "{\"action\":\"Reject\",\"filterName\":\"MaliciousIP\",\"ipMask\":\"6.6.6.6/6\"}"
```

Para quitar un filtro IP existente en IoT Hub, ejecute:

```azurecli-interactive
az resource update -n <iothubName> -g <resourceGroupName> --resource-type Microsoft.Devices/IotHubs --add properties.ipFilterRules <ipFilterIndexToRemove>
```

Tenga en cuenta que `<ipFilterIndexToRemove>` tiene que corresponder al orden de filtros IP de `properties.ipFilterRules` de su instancia de IoT Hub.

### <a name="retrieve-and-update-ip-filters-using-azure-powershell"></a>Recuperación y actualización de los filtros IP mediante Azure PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Los filtros IP de IoT Hub se pueden recuperar y establecer con [Azure PowerShell](/powershell/azure/).

```powershell
# Get your IoT Hub resource using its name and its resource group name
$iothubResource = Get-AzResource -ResourceGroupName <resourceGroupNmae> -ResourceName <iotHubName> -ExpandProperties

# Access existing IP filter rules
$iothubResource.Properties.ipFilterRules |% { Write-host $_ }

# Construct a new IP filter
$filter = @{'filterName'='MaliciousIP'; 'action'='Reject'; 'ipMask'='6.6.6.6/6'}

# Add your new IP filter rule
$iothubResource.Properties.ipFilterRules += $filter

# Remove an existing IP filter rule using its name, e.g., 'GoodIP'
$iothubResource.Properties.ipFilterRules = @($iothubResource.Properties.ipFilterRules | Where 'filterName' -ne 'GoodIP')

# Update your IoT Hub resource with your updated IP filters
$iothubResource | Set-AzResource -Force
```

### <a name="update-ip-filter-rules-using-rest"></a>Actualización de las reglas de filtro IP con REST

También puede recuperar y modificar el filtro IP de IoT Hub mediante el punto de conexión REST del proveedor de recursos de Azure. Consulte `properties.ipFilterRules` en [método createorupdate](/rest/api/iothub/iothubresource/createorupdate).

### <a name="ip-filter-rule-evaluation"></a>Evaluación de las reglas de filtro IP

Las reglas de filtro IP se aplican en orden y la primera regla que coincida con la dirección IP determina la acción de aceptar o rechazar.

Por ejemplo, si desea aceptar las direcciones del intervalo 192.168.100.0/22 y rechazar todas las demás, la primera regla de la cuadrícula debe aceptar el intervalo de direcciones 192.168.100.0/22. La siguiente regla debe rechazar todas las direcciones mediante el intervalo 0.0.0.0/0.

Puede cambiar el orden de las reglas de filtro IP en la cuadrícula al hacer clic en los tres puntos verticales situados al principio de las filas y mediante el método arrastrar y colocar.

Para guardar el nuevo orden de reglas de filtro IP, haga clic en **Guardar**.

:::image type="content" source="media/iot-hub-ip-filter-classic/ip-filter-rule-order.png" alt-text="Cambio del orden de las reglas de filtro IP de IoT Hub":::

## <a name="next-steps"></a>Pasos siguientes

Para explorar aún más las funcionalidades de IoT Hub, consulte:

* [Uso de filtros IP](iot-hub-ip-filtering.md)
