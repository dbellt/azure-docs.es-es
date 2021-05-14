---
title: Movimiento de una suscripción de Azure VMware Solution a otra
description: En este artículo se describe cómo mover una suscripción de Azure VMware Solution a otra. Pueden ser varios los motivos por los que quiera mover los recursos, por ejemplo, de cara a la facturación.
ms.custom: subject-moving-resources
ms.topic: how-to
ms.date: 04/26/2021
ms.openlocfilehash: 0cd06eb72f8ed93cc5a491070baded76f9dc9f6f
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108145572"
---
# <a name="move-azure-vmware-solution-subscription-to-another-subscription"></a>Movimiento de una suscripción de Azure VMware Solution a otra

En este artículo se describe cómo mover una suscripción de Azure VMware Solution a otra. Pueden ser varios los motivos por los que quiera mover la suscripción, por ejemplo, de cara a la facturación.

## <a name="prerequisites"></a>Prerrequisitos
Debe tener al menos derechos de colaborador en las suscripciones de **origen** y **destino**. 

>[!IMPORTANT]
>La red virtual y la puerta de enlace de red virtual no se pueden trasladar de una suscripción a otra. Además, el traslado de las suscripciones no tiene ningún impacto en la administración y las cargas de trabajo, como las máquinas virtuales de vCenter, NSX y carga de trabajo.

## <a name="prepare-and-move"></a>Preparación y traslado 

1. Inicie sesión en Azure Portal y seleccione la nube privada que quiere mover.

   :::image type="content" source="media/move-subscriptions/source-subscription-id.png" alt-text="Captura de pantalla que muestra los detalles de información general de la nube privada seleccionada.":::

1. Desde un símbolo del sistema, haga ping a los componentes y a las cargas de trabajo para comprobar que hacen ping desde la misma suscripción.  

   :::image type="content" source="media/move-subscriptions/verify-components-workloads.png" alt-text="Captura de pantalla que muestra el comando ping y los resultados de hacer ping.":::

1. Seleccione el vínculo **Suscripción (cambiar)** .

   :::image type="content" source="media/move-subscriptions/private-cloud-overview-subscription-id.png" alt-text="Captura de pantalla que muestra los detalles de la nube privada."::: 

1. Proporcione los detalles de la suscripción para **Destino** y seleccione **Siguiente**.

   :::image type="content" source="media/move-subscriptions/move-resources-subscription-target.png" alt-text="Captura de pantalla del recurso de destino.":::

1. Confirme la validación de los recursos que ha seleccionado para mover.  Esta acción valida todos los recursos seleccionados para mover. Durante la validación de los recursos seleccionados, verá el estado **Validación pendiente**. 

   :::image type="content" source="media/move-subscriptions/pending-move-resources-subscription-target.png" alt-text="Captura de pantalla que muestra el recurso que se trasladará.":::

1. Una vez que la validación se haya realizado correctamente, seleccione **Siguiente** para iniciar la migración de la nube privada.

   :::image type="content" source="media/move-subscriptions/move-resources-succeeded.png" alt-text=" Captura de pantalla que muestra el estado de validación Correcto.":::

1. Active la casilla para indicar que entiende que las herramientas y los scripts asociados no funcionarán hasta que los actualice para usar los nuevos identificadores de recurso. Luego, seleccione **Mover**.

   :::image type="content" source="media/move-subscriptions/review-move-resources-subscription-target.png" alt-text="Captura de pantalla que muestra el resumen del recurso seleccionado que se va a mover.":::

## <a name="verify-the-move"></a>Comprobación del traslado

Una vez que se completa el traslado de recursos, aparece una notificación. 

:::image type="content" source="media/move-subscriptions/notification-move-resources-subscription-target.png" alt-text="Captura de pantalla de la notificación después de terminar de mover los recursos.":::

La nueva suscripción aparece en la Información general de la nube privada.

:::image type="content" source="media/move-subscriptions/moved-subscription-target.png" alt-text="Captura de pantalla que muestra una nueva suscripción.":::

## <a name="next-steps"></a>Pasos siguientes
Más información sobre:

- [Directrices para el traslado de recursos de red](/azure/azure-resource-manager/management/move-limitations/networking-move-limitations)
- [Guía del traslado de máquinas virtuales](/azure/azure-resource-manager/management/move-limitations/virtual-machines-move-limitations)
- [Orientaciones para el traslado de recursos de App Service](/azure/azure-resource-manager/management/move-limitations/app-service-move-limitations)



