---
title: Aumento del límite de redes
description: Aumento del límite de redes
author: anavinahar
ms.author: anavin
ms.date: 01/23/2020
ms.topic: how-to
ms.assetid: ce37c848-ddd9-46ab-978e-6a1445728a3b
ms.openlocfilehash: 801f0424e7ec15fbde58f35975f4c7eca4c9a5de
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107775572"
---
# <a name="networking-limit-increase"></a>Aumento del límite de redes

Use [Azure Portal](https://portal.azure.com) para aumentar la cuota de redes.

Para ver el uso de redes y la cuota actuales en Azure Portal, abra la suscripción y, a continuación, seleccione **Usos y cuotas**. También puede usar las siguientes opciones para ver los límites y el uso de la red.

* [CLI de uso](/cli/azure/network#az_network_list_usages)
* [PowerShell](/powershell/module/azurerm.network/get-azurermnetworkusage)
* [API de uso de red](/rest/api/virtualnetwork/virtualnetworks/listusage)

Puede solicitar un aumento mediante **Ayuda y soporte técnico** o bien en **Usos y cuotas** en el portal.

> [!Note]
> Para cambiar el tamaño predeterminado de **Prefijos de IP pública**, seleccione **Longitud mínima de prefijo de red IP pública** en la lista desplegable.

## <a name="request-networking-quota-increase-at-subscription-level-using-help--support"></a>Solicitud de un aumento de cuota de redes en el nivel de suscripción mediante Ayuda y soporte técnico

Siga las instrucciones siguientes para crear una solicitud de soporte técnico mediante **Ayuda y soporte técnico** en Azure Portal.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) y, a continuación, seleccione **Ayuda y soporte técnico** en el menú de Azure Portal o busque y seleccione **Ayuda y soporte técnico**.

    ![Ayuda y soporte técnico](./media/networking-quota-request/help-plus-support.png)

1. Seleccione **Nueva solicitud de soporte técnico**.

    ![Nueva solicitud de soporte](./media/networking-quota-request/new-support-request.png)

1. En **Tipo de problema**, elija **Límites de servicio y suscripción (cuotas)** .

    ![Selección de los límites de suscripción en la lista desplegable de tipo de problema](./media/networking-quota-request/select-quota-issue-type.png)

1. Seleccione la suscripción que necesita una cuota mayor.

    ![Selección de la suscripción en la nueva solicitud de soporte técnico](./media/networking-quota-request/select-subscription-support-request.png)

1. En **Tipo de cuota**, seleccione **Redes**. Seleccione **Siguiente: Soluciones**.

    ![Selección del tipo de cuota](./media/networking-quota-request/select-quota-type-network.png)

1. En **Detalles del problema**, haga clic en **Proporcionar detalles** para aportar información adicional que ayude a procesar la solicitud.

    ![Proporcionar detalles](./media/networking-quota-request/provide-details-link.png)

1. En el panel **Detalles de la cuota**, seleccione un modelo de implementación, una ubicación y los recursos que se van a incluir en la solicitud.

    ![Modelo de implementación en Detalles de la cuota](./media/networking-quota-request/quota-details-network.png)

1. Escriba los nuevos límites que quiere en la suscripción. Para eliminar una línea, anule la selección del recurso en el menú **Recursos** o seleccione el icono de descarte "x". Después de escribir la cuota para cada recurso, seleccione **Guardar y continuar** para continuar con la creación de la solicitud de soporte técnico.

    ![Nuevos límites](./media/networking-quota-request/network-new-limits.png)

## <a name="request-networking-quota-increase-at-subscription-level-using-usages--quotas"></a>Solicitud de un aumento de la cuota de redes en el nivel de suscripción mediante Usos y cuotas

Siga estas instrucciones para crear una solicitud de soporte técnico mediante **Uso y cuota** en Azure Portal.

1. En https://portal.azure.com, busque y seleccione **Suscripciones**.

    ![Suscripciones](./media/networking-quota-request/search-for-suscriptions.png)

1. Seleccione la suscripción que necesita una cuota mayor.

    ![Seleccionar suscripción](./media/networking-quota-request/select-subscription-change-quota.png)

1. Seleccione **Uso y cuotas**.

    ![Selección de uso y cuotas](./media/networking-quota-request/select-usage-plus-quotas.png)

1. En la esquina superior derecha, seleccione **Solicitar aumento**.

    ![Solicitud de aumento](./media/networking-quota-request/request-increase-from-subscription.png)

1. Siga los pasos a partir del paso 3 en [Solicitud de aumento de la cuota de redes en el nivel de suscripción](#request-networking-quota-increase-at-subscription-level-using-help--support).

## <a name="about-networking-limits"></a>Acerca de los límites de red

Para más información sobre los límites de redes, consulte la [sección de redes](../../azure-resource-manager/management/azure-subscription-service-limits.md#networking-limits) de la página de límites o las Preguntas más frecuentes sobre los límites de red.
