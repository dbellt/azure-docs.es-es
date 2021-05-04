---
title: Reactivación de una suscripción deshabilitada de Azure
description: Describe cuándo podría tener una suscripción a Azure deshabilitada y cómo volver a activarla.
keywords: suscripción a azure deshabilitada
author: bandersmsft
ms.reviewer: amberb
tags: billing
ms.service: cost-management-billing
ms.subservice: billing
ms.topic: how-to
ms.date: 03/30/2021
ms.author: banders
ms.openlocfilehash: 37093900bf49e8a7613e2e3f4311548675791ceb
ms.sourcegitcommit: 3f684a803cd0ccd6f0fb1b87744644a45ace750d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/02/2021
ms.locfileid: "106220890"
---
# <a name="reactivate-a-disabled-azure-subscription"></a>Reactivación de una suscripción deshabilitada de Azure

Es posible que la suscripción de Azure se deshabilite porque expiró su crédito, alcanzó el límite de gasto, tiene una factura vencida, alcanzó su límite de tarjeta de crédito, o bien porque el administrador de la cuenta canceló la suscripción. Vea qué problema le afecta y siga los pasos de este artículo para reactivar su suscripción.

## <a name="your-credit-is-expired"></a>El crédito caducó

Al registrarse para obtener una cuenta gratuita de Azure, obtiene una suscripción de evaluación gratuita, que le proporciona 200 dólares en crédito de Azure en la moneda de facturación durante 30 días y 12 meses de servicios gratuitos. Al terminar los 30 días, Azure deshabilita la suscripción. La suscripción se deshabilita para protegerle de cargos accidentales por el uso que supere los servicios con crédito y gratuitos que se incluyen con esta. Para seguir usando los servicios de Azure, debe [actualizar la suscripción](upgrade-azure-subscription.md). Después de actualizar, la suscripción tendrá acceso a servicios gratuitos durante 12 meses. Solo se le cobrará el uso que supere los límites de cantidad del servicio gratuito.

## <a name="you-reached-your-spending-limit"></a>Alcanzó el límite de gasto

Las suscripciones de Azure con crédito como la evaluación gratuita y Visual Studio Enterprise tienen límites de gasto. Solo puede usar los servicios hasta el crédito incluido. Cuando el uso alcanza el límite de gasto, Azure deshabilita la suscripción para el resto de ese período de facturación. La suscripción se deshabilita para protegerle de cargos accidentales por el uso que supere el crédito que se incluye con esta. Para quitar el límite de gasto, consulte [Eliminación del límite de gasto en Azure Portal](spending-limit.md#remove).

> [!NOTE]
> Si tiene una suscripción de evaluación gratuita y quita el límite de gasto, la suscripción se convierte en una suscripción individual con tarifas de pago por uso al final de dicha evaluación. El crédito restante se conserva durante los 30 días posteriores a la creación de la suscripción. También tendrá acceso a servicios gratuitos durante 12 meses.

Para supervisar y administrar la actividad de facturación de Azure, consulte [Planificación para administrar costos de Azure](../understand/plan-manage-costs.md).

## <a name="your-bill-is-past-due"></a>Su factura está vencida

Para resolver un saldo vencido, consulte uno de los siguientes artículos:

- En el caso de las suscripciones del Programa de suscripción en línea de Microsoft, incluido el pago por uso, consulte [Resolución del saldo de la suscripción de Azure vencido tras recibir un correo electrónico de Azure](resolve-past-due-balance.md).
- En el caso de las suscripciones de los Contratos de clientes de Microsoft, consulte [Cómo pagar la factura de Microsoft Azure](../understand/pay-bill.md).

## <a name="the-bill-exceeds-your-credit-card-limit"></a>La factura supera el límite de la tarjeta de crédito

Para resolver este problema,[cambie a otra tarjeta de crédito](change-credit-card.md). O bien, si representa a una empresa, puede [cambiar a pago mediante factura](pay-by-invoice.md).

## <a name="the-subscription-was-accidentally-canceled"></a>La suscripción se canceló accidentalmente

Si es el administrador de la cuenta y accidentalmente cancela una suscripción de pago por uso, puede reactivarla en Azure Portal.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Vaya a Suscripciones y, después, seleccione la suscripción cancelada.
1. Seleccione **Reactivar**.
1. Confirme la reactivación mediante **Aceptar**.  
    :::image type="content" source="./media/subscription-disabled/reactivate-sub.png" alt-text="Captura de pantalla que muestra la opción Confirmar reactivación" :::

Para otros tipos de suscripción, [póngase en contacto con soporte técnico](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) para que reactive su suscripción.

## <a name="after-reactivation"></a>Después de la reactivación

Una vez reactivada la suscripción, puede haber un retraso en la creación o la administración de recursos. Si el retraso supera los 30 minutos, póngase en contacto con el [soporte técnico de facturación de Azure](https://go.microsoft.com/fwlink/?linkid=2083458) para obtener ayuda. La mayoría de los recursos de Azure se reanudan automáticamente y no requieren ninguna acción. Sin embargo, es aconsejable que compruebe los recursos de sus servicios de Azure y que reinicie los que no se reanuden automáticamente.

## <a name="need-help-contact-us"></a>¿Necesita ayuda? Póngase en contacto con nosotros.

Si tiene alguna pregunta o necesita ayuda, [cree una solicitud de soporte técnico](https://go.microsoft.com/fwlink/?linkid=2083458).

## <a name="next-steps"></a>Pasos siguientes
- Información sobre la [Planificación para administrar costos de Azure](../understand/plan-manage-costs.md).
