---
title: Uso de máquinas virtuales de acceso puntual de Azure
description: Aprenda a usar máquinas virtuales de acceso puntual de Azure para ahorrar costos.
author: JagVeerappan
ms.author: jagaveer
ms.service: virtual-machines
ms.subservice: spot
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 10/05/2020
ms.reviewer: cynthn
ms.openlocfilehash: b85855238beb9887a35369011ff8daaaf58bef9e
ms.sourcegitcommit: 32ee8da1440a2d81c49ff25c5922f786e85109b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109787246"
---
# <a name="use-azure-spot-virtual-machines"></a>Uso de máquinas virtuales de acceso puntual de Azure 

El uso de máquinas virtuales de acceso puntual de Azure permite aprovechar las ventajas de nuestra capacidad no utilizada con un importante ahorro en los costos. Siempre que Azure necesite recuperar la capacidad, la infraestructura de esta plataforma expulsará las máquinas virtuales de acceso puntual de Azure. Por lo tanto, estas máquinas son excelentes para cargas de trabajo que puedan soportar interrupciones, como los trabajos de procesamiento por lotes, los entornos de desarrollo y pruebas, las cargas de trabajo de proceso de gran tamaño, etc.

La cantidad de capacidad sin usar disponible varía, por ejemplo, en función del tamaño, la región o la hora del día. Al implementar máquinas virtuales de acceso puntual de Azure, Azure las asigna si hay capacidad disponible, pero no hay un Acuerdo de Nivel de Servicio para ellas. Una máquina virtual de acceso puntual de Azure no ofrece ninguna garantía de alta disponibilidad. Siempre que Azure necesite recuperar la capacidad, la infraestructura de esta plataforma expulsará las máquinas virtuales de acceso puntual de Azure con un aviso de 30 segundos. 


## <a name="eviction-policy"></a>Directiva de expulsión

Las máquinas virtuales se pueden expulsar en función de la capacidad o del precio máximo establecido. Al crear una máquina virtual de acceso puntual de Azure, puede establecer la directiva de expulsión en *Desasignar* (valor predeterminado) o *Eliminar*. 

La directiva *Deallocate* (Desasignar) mueve las VM al estado stopped-deallocated, lo que le permite volver a implementarlas más tarde. Sin embargo, no hay ninguna garantía de que la asignación se realizará correctamente. Las VM desasignadas se siguen teniendo en cuenta en la cuota y se cobrarán los costos de almacenamiento de los discos subyacentes. 

Si quiere que la VM se elimine al expulsarse, puede establecer la directiva de expulsión en *delete* (eliminar). Las VM expulsadas se eliminan junto con sus discos subyacentes y, por tanto, no se le cobrará el almacenamiento. 

Puede optar por recibir notificaciones en las VM mediante [Azure Scheduled Events](./linux/scheduled-events.md). De este modo se le notificará que se van a expulsar las máquinas virtuales y tendrá 30 segundos para terminar los trabajos y cerrar las tareas antes de que esto ocurra. 


| Opción | Resultado |
|--------|---------|
| El precio máximo se establece en un valor igual o mayor que el precio actual. | La máquina virtual se implementa si hay capacidad y cuota disponibles. |
| El precio máximo se establece en un valor menor que el precio actual. | La máquina virtual no se implementa. Obtendrá un mensaje de error que indicará que el precio máximo debe ser igual o mayor que el precio actual. |
| Reinicio de una máquina virtual detenida o desasignada si el precio máximo es igual o mayor que el precio actual | Si hay capacidad y cuota, la máquina virtual se implementa. |
| Reinicio de una máquina virtual detenida o desasignada si el precio máximo es menor que el precio actual | Obtendrá un mensaje de error que indicará que el precio máximo debe ser igual o mayor que el precio actual. | 
| El precio de la máquina virtual ha aumentado y ahora es mayor que el precio máximo. | La máquina virtual se expulsa. Recibirá una notificación 30 segundos antes de la expulsión. | 
| Después, el precio de la máquina virtual volverá a ser menor que el precio máximo. | La máquina virtual no se reiniciará automáticamente. Puede reiniciar la máquina virtual por su cuenta y se le cobrará el precio actual. |
| Si el precio máximo está establecido en `-1` | La máquina virtual no se expulsará por motivos de precio. El precio máximo será el precio actual, hasta como máximo el precio de las máquinas virtuales estándar. No se le cobrará por encima del precio estándar.| 
| Cambio del precio máximo | Debe desasignar la máquina virtual para cambiar el precio máximo. Desasigne la máquina virtual, establezca un nuevo precio máximo y actualícela. |


## <a name="limitations"></a>Limitaciones

Los siguientes tamaños de máquina virtual no se admiten en las máquinas virtuales de acceso puntual de Azure:
 - Serie B
 - Versiones de promoción de cualquier tamaño (como los tamaños de promoción Dv2, NV, NC, H)

Las máquinas virtuales de acceso puntual de Azure se pueden implementar en cualquier región, excepto en 21Vianet de Microsoft Azure China.

<a name="channel"></a>

Actualmente se admiten los siguientes [tipos de ofertas](https://azure.microsoft.com/support/legal/offer-details/):

-   Contrato Enterprise 
-   Código de oferta de pago por uso (003P)
-   Patrocinado (0036P y 0136P)
- Para obtener un proveedor de servicios en la nube (CSP), póngase en contacto con su asociado.


## <a name="pricing"></a>Precios

Los precios de las máquinas virtuales de acceso puntual de Azure son variables, según la región y la SKU. Para más información, consulte precios de las máquinas virtuales para [Linux](https://azure.microsoft.com/pricing/details/virtual-machines/linux/) y [Windows](https://azure.microsoft.com/pricing/details/virtual-machines/windows/). 

También puede consultar la información sobre con la [API de precios de venta directa de Azure](/rest/api/cost-management/retail-prices/azure-retail-prices) para consultar información sobre los precios de Spot. Tanto `meterName` como `skuName` contendrán `Spot`.

La variabilidad en los precios permite establecer un precio máximo, en dólares estadounidenses (USD), con un máximo de 5 decimales. Por ejemplo, el valor `0.98765` correspondería a un precio máximo de 0,98765 USD por hora. Si establece el precio máximo en `-1`, la máquina virtual no se expulsará por precio. El precio de la máquina virtual será el actual de Spot o el de una máquina virtual estándar, el menor de los dos, siempre que haya capacidad y cuota disponibles.

## <a name="pricing-and-eviction-history"></a>Historial de precios y expulsiones

Puede ver los precios históricos y las tasas de expulsión por tamaño en una región en el portal. Seleccione **View pricing history and compare prices in nearby regions** (Ver el historial de precios y comparar precios en regiones cercanas) para ver una tabla o gráfico de precios para un tamaño específico.  Los precios y las tasas de expulsión en las siguientes imágenes son solo ejemplos. 

**Gráfico**:

:::image type="content" source="./media/spot-chart.png" alt-text="Captura de pantalla de las opciones de región con la diferencia de precios y tasas de expulsión en forma de gráfico.":::

**Tabla**:

:::image type="content" source="./media/spot-table.png" alt-text="Captura de pantalla de las opciones de región con la diferencia de precios y tasas de expulsión en forma de tabla.":::



##  <a name="frequently-asked-questions"></a>Preguntas más frecuentes

**P:** Una vez que se crea, ¿una máquina virtual de acceso puntual de Azure es igual que una máquina virtual estándar normal?

**R:** Sí, salvo que no hay ningún Acuerdo de Nivel de Servicio para las máquinas virtuales de acceso puntual y se pueden expulsar en cualquier momento.


**P:** ¿Qué se debe hacer si se produce la expulsión pero aún se necesita capacidad?

**R:** Si necesita capacidad de inmediato, se recomienda usar máquinas virtuales estándar en lugar de máquinas virtuales de acceso puntual.


**P:** ¿Cómo se administra la cuota de las máquinas virtuales de acceso puntual de Azure?

**R:** Las máquinas virtuales de acceso puntual de Azure tendrán un grupo de cuotas distinto. La cuota de Spot se compartirá entre las máquinas virtuales y los conjuntos de escalado. Para más información, consulte [Límites, cuotas y restricciones de suscripción y servicios de Microsoft Azure](../azure-resource-manager/management/azure-subscription-service-limits.md).


**P:** ¿Puedo solicitar cuota adicional para las máquinas virtuales de acceso puntual de Azure?

**R:** Sí, podrá enviar la solicitud para aumentar su cuota en las máquinas virtuales de acceso puntual de Azure mediante el [proceso de solicitud de cuota estándar](../azure-portal/supportability/per-vm-quota-requests.md).


**P:** ¿Dónde puedo publicar preguntas?

**R:** Puede publicar y etiquetar la pregunta con `azure-spot` en [Q&A](/answers/topics/azure-spot.html) (Preguntas y respuestas). 


**P:** ¿Cómo puedo cambiar el precio máximo de una máquina virtual de acceso puntual?

**R:** Para cambiar el precio máximo, debe desasignar la máquina virtual. Luego puede cambiar el precio máximo en el portal, en la sección **Configuración** de la máquina virtual. 

## <a name="next-steps"></a>Pasos siguientes
Para implementar máquinas virtuales de acceso puntual, use la [CLI](./linux/spot-cli.md), el [portal](spot-portal.md), una [plantilla de Resource Manager](./linux/spot-template.md) o [PowerShell](./windows/spot-powershell.md).

También puede implementar un [conjunto de escalado con instancias de máquina virtual de acceso puntual de Azure](../virtual-machine-scale-sets/use-spot.md).

Si se produce un error, consulte [Códigos de error](./error-codes-spot.md).
