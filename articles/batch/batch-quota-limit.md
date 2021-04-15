---
title: Límites y cuotas del servicio
description: Obtenga información sobre las restricciones, los límites y las cuotas de Azure Batch predeterminados y cómo solicitar un aumento de la cuota.
ms.topic: conceptual
ms.date: 04/06/2021
ms.custom: seodec18
ms.openlocfilehash: 6e17a90cc573205bcb964a0428e0b7320323b8a6
ms.sourcegitcommit: b0557848d0ad9b74bf293217862525d08fe0fc1d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "106553556"
---
# <a name="batch-service-quotas-and-limits"></a>Límites y cuotas del servicio Batch

Al igual que en otros servicios de Azure, existen límites en determinados recursos asociados con el servicio Batch. Muchos de ellos son cuotas predeterminadas que Azure aplica en el nivel de cuenta o suscripción.

Tenga en cuenta estas cuotas al diseñar y escalar verticalmente sus cargas de trabajo de Batch. Por ejemplo, si su grupo no alcanza el número objetivo de nodos de proceso especificado, es posible que se haya alcanzado el límite de cuota de núcleos de la cuenta de Batch.

Se pueden ejecutar varias cargas de trabajo de Batch en una sola cuenta de Batch, o bien distribuir las cargas de trabajo entre cuentas de Batch que se encuentren en la misma suscripción, pero en diferentes regiones de Azure.

Si planea ejecutar cargas de trabajo de producción en Batch, es posible que tenga que aumentar el valor predeterminado de una o varias de las cuotas. Para generar una cuota, puede [solicitar un aumento de la cuota](#increase-a-quota) sin cargo alguno.

## <a name="resource-quotas"></a>Cuotas de recursos

Una cuota es un límite, no una garantía de capacidad. Si tiene necesidades de capacidad a gran escala, póngase en contacto con el soporte técnico de Azure.

Tenga en cuenta también que las cuotas no son valores garantizados. Las cuotas pueden variar a partir de los cambios del servicio Batch o una solicitud de usuario para cambiar un valor de cuota.

[!INCLUDE [azure-batch-limits](../../includes/azure-batch-limits.md)]

## <a name="core-quotas"></a>Cuotas de núcleos

### <a name="cores-quotas-in-batch-service-mode"></a>Cuotas de núcleos en el modo de servicio de Batch

Existen cuotas de núcleos para cada serie de máquinas virtuales admitidas por Batch, que se muestran en la página **Cuotas** en Azure Portal. Los límites de cuota de las series de máquinas virtuales se pueden actualizar mediante una solicitud de soporte técnico, tal como se detalla a continuación. En el caso de los nodos dedicados, Batch impone un límite de cuota principal para cada serie de VM, así como un límite de cuota de núcleo total para toda la cuenta de Batch. En el caso de los nodos de prioridad baja, Batch solo aplica una cuota de núcleos total para la cuenta de Batch sin distinción entre distintas series de VM.

### <a name="cores-quotas-in-user-subscription-mode"></a>Cuotas de núcleos en el modo de suscripción de usuario

Si ha creado una [cuenta de Batch](accounts.md) con el modo de asignación de grupo establecido en **suscripción de usuario**, las máquinas virtuales y otros recursos de Batch se crean directamente en su suscripción cuando se crea un grupo. No se aplicarán las cuotas de núcleos de Azure Batch y sí se usarán y aplicarán las cuotas de la suscripción para los núcleos de proceso regionales, los núcleos de proceso por serie y otros recursos.

Para más información sobre estas cuotas, consulte [Límites, cuotas y restricciones de suscripción y servicios de Microsoft Azure](../azure-resource-manager/management/azure-subscription-service-limits.md).

## <a name="pool-size-limits"></a>Límites de tamaño de grupo

El servicio Batch establece los límites de tamaño del grupo. A diferencia de las [cuotas de recursos](#resource-quotas), estos valores no se pueden cambiar. Solo los grupos con la comunicación entre nodos e imágenes personalizadas tienen restricciones diferentes de la cuota estándar.

| **Recurso** | **Límite máximo** |
| --- | --- |
| **Nodos de proceso en el [grupo con la comunicación entre nodos habilitada](batch-mpi.md)**  ||
| Modo de asignación de grupo de servicio Batch | 100 |
| Modo de asignación de grupo de suscripción Batch | 80 |
| **Nodos de proceso en el [grupo creado con un recurso de imagen administrada](batch-custom-images.md)** <sup>1</sup> ||
| Nodos dedicados | 2000 |
| Nodos de prioridad baja | 1000 |

<sup>1</sup> Para los grupos que no tienen habilitada la comunicación entre nodos.

## <a name="other-limits"></a>Otros límites

El servicio Batch establece estos límites adicionales. A diferencia de [las cuotas de recursos](#resource-quotas), estos valores no se pueden cambiar.

| **Recurso** | **Límite máximo** |
| --- | --- |
| [Tareas simultáneas](batch-parallel-node-tasks.md) por nodo de proceso | 4 × número de núcleos de nodo |
| [Aplicaciones](batch-application-packages.md) por cuenta de Batch | 20 |
| Paquetes de aplicación por aplicación | 40 |
| Paquetes de aplicación por grupo | 10 |
| Duración máxima de la tarea | 180 días<sup>1</sup> |
| [Montajes](virtual-file-mount.md) por nodo de ejecución | 10 |
| Certificados por grupo | 12 |

<sup>1</sup> La duración máxima de una tarea, desde el momento en que se agrega al trabajo hasta que se completa, es de 180 días. Las tareas completadas se mantienen durante siete días; los datos de las tareas no completadas dentro de la duración máxima no están accesibles.

## <a name="view-batch-quotas"></a>Visualización de las cuotas de Batch

Para ver las cuotas de la cuenta de Batch en [Azure Portal](https://portal.azure.com):

1. Seleccione **Cuentas de Batch** y, después, seleccione la cuenta de Batch que le interesa.
1. Seleccione **Cuotas** en el menú de la cuenta de Batch.
1. Vea las cuotas aplicadas actualmente a la cuenta de Batch.

:::image type="content" source="./media/batch-quota-limit/account-quota-portal.png" alt-text="Captura de pantalla que muestra las cuotas de la cuenta de Batch en Azure Portal.":::

## <a name="increase-a-quota"></a>Aumento de la cuota

Puede solicitar un aumento de la cuota para la cuenta de Batch o la suscripción mediante [Azure Portal](https://portal.azure.com) o mediante la [API REST de cuotas de Azure](#azure-quota-rest-api).

El tipo de aumento de cuota depende del modo de asignación de grupo de su cuenta de Batch. Para solicitar un aumento de cuota, debe incluir la serie de VM para la que quiere aumentar la cuota. Cuando se aplica el aumento de cuota, se aplica a todas las series de VM.

Una vez que haya enviado la solicitud de soporte técnico, el servicio de soporte técnico de Azure se comunicará con usted. Las solicitudes de cuota se pueden completar en unos pocos minutos o en hasta dos días laborables.

### <a name="azure-portal"></a>Azure portal

1. En la página **Cuotas**, seleccione **Solicitar aumento de cuota**. Como alternativa, puede seleccionar el icono **Ayuda y soporte técnico** en el panel del portal [o en el signo de interrogación ( **?** ) situado en la esquina superior derecha del portal] y, a continuación, seleccionar **Nueva solicitud de soporte técnico.**

1. En **Aspectos básicos**:

    1. En **Tipo de problema**, seleccione **Límites de servicio y suscripción (cuotas)** .
    1. Seleccione su suscripción.
    1. En **Tipo de cuota**, seleccione **Batch**.
    1. Seleccione **Next** (Siguiente) para continuar.

1. En **Details** (Detalles):

    1. En la sección **Proporcionar detalles**, especifique la ubicación, el tipo de cuota y la cuenta de Batch (si procede) y, a continuación, seleccione las cuotas que desea aumentar.

       :::image type="content" source="media/batch-quota-limit/quota-increase.png" alt-text="Captura de pantalla de la pantalla Detalles de la cuota al solicitar un aumento de la cuota.":::

       Entre los tipos de cuota, se incluyen los siguientes:

       - **Por cuenta de Batch**  
         Use esta opción para solicitar aumentos de cuota específicos de una sola cuenta de Batch, incluidos los núcleos dedicados y de baja prioridad, y el número de trabajos y grupos.

         Si selecciona esta opción, especifique la cuenta de Batch a la que se debe aplicar esta solicitud y, a continuación, seleccione las cuotas que desea actualizar. Proporcione el nuevo límite que solicita para cada recurso.

         La cuota de prioridad baja es un valor único en todas las series de VM. Si necesita SKU restringidas, debe seleccionar **Núcleos de baja prioridad** e incluir las familias de VM que se solicitarán.

       - **Todas las cuentas de esta región**  
         Use esta opción para solicitar los aumentos de cuota que se aplican a todas las cuentas de Batch en una región, como el número de cuentas de Batch por región y suscripción.

    1. En **Método de soporte técnico**, seleccione una de las opciones en **Gravedad** según su [impacto en el negocio](https://aka.ms/supportseverity) y el método de contacto y el idioma de soporte que prefiera.

    1. En **Información de contacto**, compruebe y escriba los detalles de contacto necesarios.

1. Seleccione **Revisar y crear** y, a continuación, **Crear** para enviar la solicitud de soporte técnico.

### <a name="azure-quota-rest-api"></a>API REST de cuotas de Azure

Puede usar la API REST de cuotas de Azure para solicitar un aumento de la cuota en el nivel de suscripción o en el nivel de la cuenta de Batch.

Para obtener más información y ejemplos, consulte [Solicitud de un aumento de la cuota mediante la API REST de Soporte técnico de Azure](/rest/api/support/quota-payload#azure-batch).

## <a name="related-quotas-for-vm-pools"></a>Cuotas relacionadas para grupos de máquinas virtuales

[Los grupos de Batch en la configuración de máquina virtual implementada en una red virtual de Azure](batch-virtual-network.md) asignan automáticamente recursos de red de Azure adicionales. Estos recursos se crean en la suscripción que contiene la red virtual proporcionada al crear el grupo de Batch.

Se crean los siguientes recursos por cada 100 nodos de grupo en una red virtual:

- Un [grupo de seguridad de red](../virtual-network/network-security-groups-overview.md#network-security-groups)
- Una [dirección IP pública](../virtual-network/public-ip-addresses.md)
- Un [equilibrador de carga](../load-balancer/load-balancer-overview.md)

Estos recursos están limitados por las [cuotas de recursos](../azure-resource-manager/management/azure-subscription-service-limits.md) de la suscripción. Si planea implementaciones de grupos grandes en una red virtual, puede que necesite solicitar un aumento de la cuota para uno o varios de estos recursos.

## <a name="next-steps"></a>Pasos siguientes

- Conozca el [flujo de trabajo y los recursos principales del servicio Batch](batch-service-workflow-features.md), como grupos, nodos, trabajos y tareas.
- Conozca los [límites, las cuotas y las restricciones de suscripción y servicios de Azure](../azure-resource-manager/management/azure-subscription-service-limits.md).