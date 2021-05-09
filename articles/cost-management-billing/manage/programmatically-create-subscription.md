---
title: Creación de suscripciones de Azure mediante programación
description: Este artículo le ayudará a comprender las opciones disponibles para crear suscripciones de Azure mediante programación.
author: bandersmsft
ms.service: cost-management-billing
ms.subservice: billing
ms.topic: how-to
ms.date: 03/11/2021
ms.reviewer: andalmia
ms.author: banders
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: ce08ebf473b11eecae327c7de050c791f5bc1b1a
ms.sourcegitcommit: 2654d8d7490720a05e5304bc9a7c2b41eb4ae007
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107379016"
---
# <a name="create-azure-subscriptions-programmatically"></a>Creación de suscripciones de Azure mediante programación

Este artículo le ayudará a comprender las opciones disponibles para crear suscripciones de Azure mediante programación.

Mediante el uso de varias API REST, puede crear una suscripción para los siguientes tipos de contrato de Azure:

- Contrato Enterprise (EA)
- Contrato de cliente de Microsoft (MCA).
- Microsoft Partner Agreement (MPA)

No es posible crear suscripciones adicionales mediante programación para otros tipos de contratos con las API REST.

Los requisitos y los detalles para crear suscripciones varían según el tipo de contrato y la versión de API. Consulte los siguientes artículos que hacen referencia a su situación:

API más recientes:

- [Creación de suscripciones de EA](programmatically-create-subscription-enterprise-agreement.md)
- [Creación de suscripciones de MCA](programmatically-create-subscription-microsoft-customer-agreement.md)
- [Creación de suscripciones de MPA](programmatically-create-subscription-microsoft-partner-agreement.md)

En estos artículos también se muestra cómo crear suscripciones con una plantilla de Azure Resource Manager. Las plantillas de ARM permiten automatizar el proceso de creación de suscripciones.

Si aún usa las [versiones preliminares de las API](programmatically-create-subscription-preview.md), puede seguir creando suscripciones con ellas. 

## <a name="next-steps"></a>Pasos siguientes

* Después de crear una suscripción, puede otorgar esa capacidad a otros usuarios y entidades de servicio. Para más información, vea [Concesión de acceso para crear suscripciones de EA (versión preliminar)](grant-access-to-create-subscription.md).
* Para más información sobre la administración de grandes cantidades de suscripciones mediante grupos de administración, consulte [Organización de los recursos con grupos de administración de Azure](../../governance/management-groups/overview.md).
