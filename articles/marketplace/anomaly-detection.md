---
title: Administración de las anomalías de facturación de uso medido en el Centro de partners | Azure Marketplace
description: Obtenga información sobre cómo la detección automática de anomalías para la facturación de uso medido ayuda a asegurarse de que se facture correctamente a los clientes para el uso medido de las ofertas de marketplace comercial.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
ms.date: 5/03/2021
author: sayantanroy83
ms.author: sroy
ms.openlocfilehash: dd0f86af30c88a7c7e1d1eb9cec9a5f31505496a
ms.sourcegitcommit: 89c4843ec85d1baea248e81724781d55bed86417
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108795607"
---
# <a name="manage-metered-billing-anomalies-in-partner-center"></a>Administración de las anomalías de facturación de uso medido en el Centro de partners

La opción de facturación de uso medido personalizada está disponible actualmente para las ofertas de[software como servicio](plan-saas-offer.md) (SaaS) y las [aplicaciones de Azure](plan-azure-application-offer.md#types-of-plans) con un plan de aplicación administrada.

Si usa la opción de facturación de uso medido para crear ofertas en el programa de marketplace comercial que le permite cobrar el uso basado en unidades no estándar, debe saber cuándo el cliente ha usado un servicio más de lo esperado.

## <a name="use-the-anomaly-detection-feature"></a>Uso de la característica de detección de anomalías

Microsoft confía en usted, el asociado, para notificar el uso por encima del límite por parte de sus clientes de sus ofertas de aplicaciones administradas de Azure o SaaS antes de que Microsoft facture al cliente. Si se notifica el uso incorrecto, el cliente podría recibir una factura incorrecta, perjudicando la credibilidad tanto del asociado como de Microsoft.

Para asegurarse de que se factura a los clientes correctamente, use la característica de **detección de anomalías** tanto para las aplicaciones SaaS como para los planes de la aplicación administrada de la aplicación de Azure. Esta característica supervisa de forma proactiva la utilización de la facturación de uso medido y predice el valor esperado de uso dentro del intervalo esperado. Si el uso está fuera del intervalo esperado, se trata como inesperado (una anomalía), de modo que recibirá una notificación de alerta en la página Información general de la oferta en el programa de marketplace comercial del Centro de partners. Puede realizar un seguimiento diario del uso por parte de sus clientes de cada dimensión de medidor personalizada que haya definido.

## <a name="view-and-manage-metered-usage-anomalies"></a>Visualización y administración de anomalías de uso medido

1. Inicie sesión en el [Centro de partners](https://partner.microsoft.com/dashboard/home).
1. En el menú de navegación izquierdo, seleccione **Marketplace comercial** > **Analizar**.
1. Seleccione la pestaña **Anomalías de uso medido**.

    [![Muestra la pestaña Anomalías de uso medido en la página Uso.](./media/anomaly-detection/metered-usage-anomalies.png)](./media/anomaly-detection/metered-usage-anomalies.png#lightbox)<br>
    ***Figura 1: Pestaña Anomalías de uso medido***

1. En el caso de las anomalías de uso detectadas en la facturación de uso medido, como publicador, se le pedirá que investigue y confirme si la anomalía es verdadera o no. Seleccione **Marcar como anomalía** para confirmar el diagnóstico.

     [![Muestra el cuadro de diálogo Marcar como anomalía.](./media/anomaly-detection/mark-as-anomaly.png)](./media/anomaly-detection/mark-as-anomaly.png#lightbox)<br>
    ***Figura 2: Cuadro de diálogo Marcar como anomalía***

1. Si cree que la anomalía de uso por encima del límite que detectamos no es auténtica, puede proporcionar esos comentarios seleccionando **No es una anomalía** para la anomalía marcada del Centro de partners en el uso por encima del límite en particular.

    [![Muestra el cuadro de diálogo ¿Por qué no es una anomalía?](./media/anomaly-detection/why-is-it-not-an-anomaly.png)](./media/anomaly-detection/why-is-it-not-an-anomaly.png#lightbox)
    ***Figura 3: Cuadro de diálogo ¿Por qué no es una anomalía?***

1. Puede desplazarse hacia abajo en la página para ver una lista de inventario de anomalías no confirmadas. La lista proporciona un inventario de anomalías que no ha confirmado. Puede elegir marcar cualquiera de las anomalías marcadas del Centro de partners como auténtica o falsa.

   [![Muestra la lista de anomalías no confirmadas del Centro de partners en la página Uso.](./media/anomaly-detection/unacknowledged-anomalies.png)](./media/anomaly-detection/unacknowledged-anomalies.png#lightbox)<br>
    ***Ilustración 4: Lista de anomalías no confirmadas del Centro de partners***

    De forma predeterminada, las anomalías marcadas que tienen un impacto financiero estimado superior a 100 USD se muestran en el Centro de partners. Sin embargo, puede seleccionar la opción **Todos** de la lista **Impacto financiero estimado de anomalías** para ver todas las anomalías marcadas.

    :::image type="content" source="./media/anomaly-detection/all-anomalies.png" alt-text="Captura de pantalla de todas las anomalías de uso medido para la oferta seleccionada.":::

1. También vería un registro de acciones de anomalías que muestra las acciones que realizó en los usos por encima del límite. En el registro de acciones, podrá ver qué eventos de uso por encima del límite se marcaron como auténticos o falsos.

   [![Muestra el registro de acciones de anomalías en la página Uso.](./media/anomaly-detection/anomaly-action-log.png)](./media/anomaly-detection/anomaly-action-log.png#lightbox)<br>
   ***Figura 5: Registro de acciones de anomalías***

1. El análisis del Centro de partners no admitirá la redefinición eventos de uso por encima del límite en los informes de exportación. El Centro de partners le permite escribir el uso por encima del límite para una anomalía y los detalles se pasan a Microsoft Teams para su investigación. En función de la investigación, Microsoft emitirá reembolsos de crédito al cliente sobrecargado, según corresponda. Al seleccionar cualquiera de las anomalías marcadas, puede seleccionar **Marcar como anomalía** para marcar la anomalía de uso por encima del límite como genuina.

   [![Muestra el cuadro de diálogo Marcar como anomalía.](./media/anomaly-detection/new-reported-usage.png)](./media/anomaly-detection/new-reported-usage.png#lightbox)<br>
   ***Figura 6: Cuadro de diálogo Marcar como anomalía***

La primera vez que se marque un uso por encima del límite como irregular en el Centro de partners, obtendrá una ventana de 30 días de esa instancia para marcar la anomalía como auténtica o falsa. Después del período de 30 días, como publicador, no podría actuar sobre las anomalías.

> [!IMPORTANT]
> Las unidades de uso por encima del límite sin notificar no cumplen los requisitos para la redefinición y el ajuste financiero.

Puede ver todas las anomalías (confirmadas o de otro modo) durante el período de cálculo seleccionado. Los distintos períodos de cálculo son los últimos 30 días, los últimos 60 días y los últimos 90 días.

> [!IMPORTANT]
> Se le solicitará que actúe sobre las anomalías marcadas en un plazo de 30 días desde el momento en que las anomalías se notifiquen por primera vez en el Centro de partners.

El filtro modal en el widget permite seleccionar ofertas individuales y medidores personalizados individuales.

Después de marcar un uso por encima del límite como anomalía o reconocer un modelo que marcó una anomalía como genuina, no se puede cambiar la selección a "No es una anomalía".

> [!IMPORTANT]
> Puede volver a enviar los usos por encima del límite en caso de situaciones de sobrecarga.

## <a name="see-also"></a>Consulte también
- [Facturación según uso mediante el servicio de medición de marketplace comercial](./partner-center-portal/saas-metered-billing.md)
- [Facturación según uso de aplicaciones administradas](./partner-center-portal/azure-app-metered-billing.md)
- [Servicio de detección de anomalías para la facturación según uso](./partner-center-portal/anomaly-detection-service-for-metered-billing.md)
