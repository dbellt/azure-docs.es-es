---
title: 'Escalado vertical y automático de las unidades de procesamiento: Azure Event Hubs | Microsoft Docs'
description: Habilite el inflado automático en un espacio de nombres para escalar verticalmente las unidades de procesamiento.
ms.topic: article
ms.date: 06/23/2020
ms.openlocfilehash: e4e54282b7f455f661238b0129dd1f8f9c70d9d6
ms.sourcegitcommit: eda26a142f1d3b5a9253176e16b5cbaefe3e31b3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2021
ms.locfileid: "109738111"
---
# <a name="automatically-scale-up-azure-event-hubs-throughput-units"></a>Escalado vertical y automático de las unidades de procesamiento de Azure Event Hubs
Azure Event Hubs es una plataforma de streaming de datos muy escalable. Por lo tanto, el uso de Event Hubs suele aumentar después de empezar a utilizar el servicio. Para esta utilización, es necesario incrementar las [unidades de procesamiento](event-hubs-scalability.md#throughput-units) predeterminadas con el objetivo de escalar Event Hubs y controlar velocidades de transferencia más elevadas. La característica de **inflado automático** de Event Hubs escala verticalmente y de forma automática mediante el aumento del número de unidades de procesamiento para responder a las necesidades de utilización. Al aumentar las unidades de rendimiento, se evitan escenarios de limitación en los que nos encontramos con:

* Velocidades de entrada de datos que superan las unidades de rendimiento establecidas.
* Velocidades de solicitud de salida de datos que superan las unidades de rendimiento establecidas.

El servicio Event Hubs aumenta el rendimiento cuando la carga aumenta más allá del umbral mínimo, sin que se produzca ningún problema de las solicitudes con errores de ServerBusy.

## <a name="how-auto-inflate-works"></a>Funcionamiento del inflado automático

El tráfico de los Event Hubs lo controlan las [unidades de procesamiento](event-hubs-scalability.md#throughput-units). Una sola unidad de procesamiento permite una entrada de 1 MB por segundo y una salida que duplica esa cifra. Event Hubs estándar se puede configurar con un número de unidades de procesamiento de uno a veinte. El inflado automático permite empezar poco a poco con las unidades de procesamiento mínimas que se elijan. Después, la característica escala automáticamente la cantidad hasta el límite máximo de unidades de procesamiento que necesite, según el aumento del tráfico. El inflado automático proporciona las siguientes ventajas:

- Un mecanismo de escalado eficaz para empezar poco a poco y escalar verticalmente a medida que aumente el tráfico.
- Escalado automático hasta el límite superior especificado sin problemas de limitación.
- Más control sobre el escalado, ya que se puede controlar el momento y la cantidad que se escala.

## <a name="enable-auto-inflate-on-a-namespace"></a>Habilitación del inflado automático en un espacio de nombres

Puede habilitar o deshabilitar aumentar el inflado automático en un espacio de nombres de Event Hubs del nivel Estándar mediante cualquiera de los métodos siguientes:

- [Azure Portal](https://portal.azure.com)
- Una [Plantilla de Azure Resource Manager](https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.eventhub/eventhubs-create-namespace-and-enable-inflate).

> [!NOTE]
> Los espacios de nombres de Event Hubs de nivel Básico no admiten el inflado automático.

### <a name="enable-auto-inflate-through-the-portal"></a>Habilitación del inflado automático mediante Azure Portal


#### <a name="enable-at-the-time-of-creation"></a>Habilitación en el momento de creación
Puede habilitar la característica de inflado automático **al crear un espacio de nombres de Event Hubs**:

![Habilitación del inflado automático en el momento de creación de centro de eventos](./media/event-hubs-auto-inflate/event-hubs-auto-inflate1.png)

Con esta opción habilitada, puede empezar poco a poco con las unidades de procesamiento y escalarlas verticalmente a medida que sus necesidades de utilización sean más exigentes. El límite superior del inflado no afecta inmediatamente al precio, que depende del número de unidades de procesamiento utilizadas por hora.

#### <a name="enable-auto-inflate-for-an-existing-event-hub"></a>Habilitación del inflado automático para un centro de eventos existente
También puede habilitar la característica de inflado automático y modificar su configuración mediante las siguientes instrucciones:

1. En la página **Espacio de nombres de Event Hubs**, seleccione **Deshabilitado** en **Unidades de procesamiento de inflado automático**.

    ![Seleccione las unidades de procesamiento en la página Espacio de nombres de Event Hubs.](./media/event-hubs-auto-inflate/select-throughput-units.png)
2. En la página **Configuración de escalado**, seleccione la casilla de verificación **Habilitar** (si no se ha habilitado la característica de escalabilidad automática).

    ![Seleccione Habilitar.](./media/event-hubs-auto-inflate/scale-settings.png)
3. Escriba el número **máximo** de unidades de rendimiento o use la barra de desplazamiento para establecer el valor.
4. (opcional) Actualice el número **mínimo** de unidades de procesamiento en la parte superior de esta página.


> [!NOTE]
> Al aplicar la configuración de inflado automático para aumentar las unidades de procesamiento, el servicio Event Hubs emite los registros de diagnóstico que le proporcionan información acerca de por qué y cuándo aumentó el rendimiento. Para habilitar el registro de diagnóstico para un centro de eventos, seleccione **configuración de diagnóstico** en el menú izquierdo de la página del centro de eventos en Azure Portal. Para más información, vea [Configuración de registros de diagnóstico de Azure Event Hubs](event-hubs-diagnostic-logs.md).

### <a name="enable-auto-inflate-using-an-azure-resource-manager-template"></a>Habilitación del inflado automático mediante una plantilla de Azure Resource Manager

Puede habilitar el inflado automático durante la implementación de una plantilla de Azure Resource Manager. Por ejemplo, establezca la propiedad `isAutoInflateEnabled` en **True**, y `maximumThroughputUnits` en 10. Por ejemplo:

```json
"resources": [
        {
            "apiVersion": "2017-04-01",
            "name": "[parameters('namespaceName')]",
            "type": "Microsoft.EventHub/Namespaces",
            "location": "[variables('location')]",
            "sku": {
                "name": "Standard",
                "tier": "Standard"
            },
            "properties": {
                "isAutoInflateEnabled": true,
                "maximumThroughputUnits": 10
            },
            "resources": [
                {
                    "apiVersion": "2017-04-01",
                    "name": "[parameters('eventHubName')]",
                    "type": "EventHubs",
                    "dependsOn": [
                        "[concat('Microsoft.EventHub/namespaces/', parameters('namespaceName'))]"
                    ],
                    "properties": {},
                    "resources": [
                        {
                            "apiVersion": "2017-04-01",
                            "name": "[parameters('consumerGroupName')]",
                            "type": "ConsumerGroups",
                            "dependsOn": [
                                "[parameters('eventHubName')]"
                            ],
                            "properties": {}
                        }
                    ]
                }
            ]
        }
    ]
```

Para ver la plantilla completa, consulte la plantilla [Create Event Hubs namespace and enable inflate](https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.eventhub/eventhubs-create-namespace-and-enable-inflate) (Creación de un espacio de nombres de Event Hubs y habilitación del inflado) en GitHub.


## <a name="next-steps"></a>Pasos siguientes

Para más información acerca de Event Hubs, visite los vínculos siguientes:

* [Información general de Event Hubs](./event-hubs-about.md)
