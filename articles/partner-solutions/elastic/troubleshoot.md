---
title: 'Solución de problemas de Elastic: soluciones de asociados de Azure'
description: En este artículo se proporciona información sobre la solución de problemas de integración de Elastic con Azure.
ms.service: partner-services
ms.topic: conceptual
ms.date: 05/20/2021
author: tfitzmac
ms.author: tomfitz
ms.openlocfilehash: f4bc4bc9d9bb5890ed95bbb446f570d25e43d813
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111952575"
---
# <a name="troubleshooting-elastic-integration-with-azure"></a>Solución de problemas de integración de Elastico con Azure

Este documento contiene información sobre cómo solucionar los problemas de las soluciones que usan Elastic.

## <a name="unable-to-create-an-elastic-resource"></a>No se puede crear un recurso de Elastic

La integración de Elastic con Azure solo la pueden configurar los usuarios que tengan acceso de *propietario* en la suscripción de Azure. [Confirme que tiene el acceso adecuado](../../role-based-access-control/check-access.md).

## <a name="logs-not-being-emitted-to-elastic"></a>Registros que no se emiten en Elastic

Los recursos enumerados en las [categorías de registro de recursos de Azure Monitor](../../azure-monitor/essentials/resource-logs-categories.md) son los únicos que emiten registros a Elastic. Para comprobar si el recurso emitiendo registros a Elastic, vaya a la [configuración de diagnóstico de Azure](../../azure-monitor/essentials/diagnostic-settings.md) del recurso. Compruebe si hay alguna opción de configuración de diagnóstico disponible.

:::image type="content" source="media/troubleshoot/check-diagnostic-setting.png" alt-text="Comprobación de la configuración de diagnóstico":::

## <a name="purchase-errors"></a>Errores de compra

* Los errores de compra se producen porque una tarjeta de crédito válida no está conectada a la suscripción de Azure o un método de pago no está asociado a la suscripción.

  Use una suscripción de Azure diferente. O bien, agregue o actualice la tarjeta de crédito o el método de pago de la suscripción. Para obtener más información, consulte cómo [actualizar el método de pago y la tarjeta de crédito](../../cost-management-billing/manage/change-credit-card.md).

* Una suscripción de Contrato Enterprise (EA) no permite realizar compras en Marketplace.

  Use otra suscripción. O bien, compruebe si su suscripción de EA está habilitada para la compra en Marketplace. Para obtener más información, consulte la sección [Habilitación de compras en Azure Marketplace](../../cost-management-billing/manage/ea-azure-marketplace.md#enabling-azure-marketplace-purchases).

## <a name="get-support"></a>Obtención de soporte técnico

Para ponerse en contacto con el servicio de soporte técnico en relación con la integración de Elastic con Azure, seleccione **Nueva solicitud de soporte técnico** en el panel de la izquierda. Seleccione **Abrir una incidencia de soporte técnico de Elastic**.

:::image type="content" source="media/troubleshoot/open-ticket.png" alt-text="Abrir una incidencia de soporte técnico":::

En el sitio de Elastic, abra una solicitud de soporte técnico.

:::image type="content" source="media/troubleshoot/elastic-support.png" alt-text="Abrir solicitud de soporte técnico de Elastic":::

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre [cómo administrar una instancia](manage.md) de Elastic.