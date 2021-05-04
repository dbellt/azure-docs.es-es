---
title: Visualización del uso de las reservas de Azure
description: Obtenga información sobre cómo obtener el uso y los detalles de la reserva.
author: yashesvi
ms.reviewer: yashar
ms.service: cost-management-billing
ms.subservice: reservations
ms.topic: how-to
ms.date: 04/15/2021
ms.author: banders
ms.openlocfilehash: 28b61781f0b22e79d10d79c1a46e757737a401cf
ms.sourcegitcommit: 590f14d35e831a2dbb803fc12ebbd3ed2046abff
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107568758"
---
# <a name="view-reservation-utilization-after-purchase"></a>Visualización del uso de las reservas después de la compra

Puede ver el porcentaje de uso de las reservas y los recursos que utilizaron la reserva en Azure Portal y en la aplicación Cost Management de Power BI.

## <a name="view-utilization-in-the-azure-portal-with-azure-rbac-access"></a>Visualización del uso en Azure Portal con el control de acceso basado en roles de Azure

Para ver el uso de la reserva, debe tener control de acceso basado en roles de Azure a la reserva o tener privilegios de acceso elevado para administrar todas las suscripciones y grupos de administración de Azure.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Vaya a [Reservas](https://portal.azure.com/#blade/Microsoft_Azure_Reservations/ReservationsBrowseBlade).
1. La lista muestra todas las reservas donde tenga el rol de propietario o lector. Cada reserva muestra el último porcentaje de utilización conocido.
1. Seleccione el porcentaje de uso para ver el historial y los detalles de uso. En el siguiente vídeo se muestra un ejemplo.
   > [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE4sYwk] 

## <a name="view-utilization-as-billing-administrator"></a>Visualización del uso como administrador de facturación

El administrador de un Contrato Enterprise (EA) o el administrador de facturación de un Contrato de cliente de Microsoft (MCA) pueden ver el uso en **Cost Management + Billing**.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Vaya a **Cost Management + Billing** > **Reservas**.
1. Seleccione el porcentaje de uso para ver el historial y los detalles de uso.

## <a name="get-reservations-and-utilization-using-apis-powershell-and-cli"></a>Obtención de reservas y uso mediante API, PowerShell y la CLI

Puede obtener el [uso de la reserva](/rest/api/billing/enterprise/billing-enterprise-api-reserved-instance-usage) mediante la API de uso de la instancia reservada.

## <a name="see-reservations-and-utilization-in-power-bi"></a>Visualización de reservas y uso en Power BI

Los usuarios de Power BI tienen dos opciones:

- Conector de Azure Cost Management para Power BI Desktop: los datos de uso y la fecha de compra de las reservas están disponibles en el [conector de Azure Cost Management para Power BI Desktop](/power-bi/desktop-connect-azure-cost-management). Cree los informes que desee mediante el conector.
- Aplicación Azure Cost Management para Power BI: use [esta aplicación](https://appsource.microsoft.com/product/power-bi/costmanagement.azurecostmanagementapp) para obtener informes creados previamente que puede personalizar.

## <a name="next-steps"></a>Pasos siguientes

- [Administración de reservas de Azure](manage-reserved-vm-instance.md)
- [Información sobre el uso de reservas para suscripciones de pago por uso](understand-reserved-instance-usage.md).
- [Información sobre el uso de reservas para la inscripción Enterprise](understand-reserved-instance-usage-ea.md).
- [Información sobre el uso de reservas para suscripciones de CSP](/partner-center/azure-reservations).
