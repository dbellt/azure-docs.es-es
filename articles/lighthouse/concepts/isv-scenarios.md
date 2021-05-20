---
title: Azure Lighthouse en escenarios de ISV
description: Los ISV pueden usar las capacidades de Azure Lighthouse a fin obtener más flexibilidad con las ofertas de los clientes.
ms.date: 05/11/2021
ms.topic: conceptual
ms.openlocfilehash: 9fc8c261f49e8a12189708ada1d0da31965e5159
ms.sourcegitcommit: 32ee8da1440a2d81c49ff25c5922f786e85109b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109788686"
---
# <a name="azure-lighthouse-in-isv-scenarios"></a>Azure Lighthouse en escenarios de ISV

Un escenario muy común de [Azure Lighthouse](../overview.md) es un proveedor de servicios que administra recursos en los inquilinos de Azure Active Directory (Azure AD) de sus clientes. Los fabricantes independientes de software (ISV) también pueden usar las capacidades de Azure Lighthouse mediante ofertas basadas en SaaS con sus clientes. Azure Lighthouse puede ser especialmente útil para los ISV que ofrecen servicios administrados o soporte técnico que requieren acceso al ámbito de la suscripción.

## <a name="managed-service-offers-in-azure-marketplace"></a>Ofertas de servicios administrados en Azure Marketplace

Como ISV, es posible que ya haya publicado soluciones en Azure Marketplace. Si ofrece servicios administrados a sus clientes, puede hacerlo mediante la publicación de una oferta de servicio administrado. Estas ofertas simplifican el proceso de incorporación y hacen que los servicios sean más escalables a tantos clientes como sea necesario. Azure Lighthouse admite una amplia gama de [escenarios y tareas de administración](cross-tenant-management-experience.md#enhanced-services-and-scenarios) que se pueden usar para proporcionar valor a sus clientes.

Para obtener más información, vea [Publicación de una oferta de servicio administrado en Azure Marketplace](../how-to/publish-managed-services-offers.md).

## <a name="using-azure-lighthouse-with-azure-managed-applications"></a>Uso de Azure Lighthouse y Azure Managed Applications

[Azure Managed Applications](../../azure-resource-manager/managed-applications/overview.md) es otra forma en que los ISV pueden ofrecer servicios a sus clientes. Puede usar Azure Lighthouse junto con Azure Managed Applications para habilitar escenarios mejorados.

Para obtener más información, vea [Uso de Azure Lighthouse y Azure Managed Applications](managed-applications.md).

## <a name="saas-based-multi-tenant-offerings"></a>Ofertas de multiinquilino basadas en SaaS

Un escenario adicional es aquel en el que el ISV hospeda los recursos de una suscripción en su propio inquilino y luego usa Azure Lighthouse para permitir que los clientes accedan a estos recursos. El cliente puede entonces iniciar sesión en su propio inquilino y acceder a estos recursos según sea necesario. Los ISV mantienen su IP en su propio inquilino y pueden usar sus propios planes de soporte técnico para generar vales relacionados con la solución hospedada en su inquilino, en lugar de usar el plan del cliente. Dado que los recursos se encuentran en el inquilino del ISV, este puede realizar directamente todas las acciones, como iniciar sesión en las máquinas virtuales, instalar aplicaciones y realizar tareas de mantenimiento.

En este escenario, a los usuarios del inquilino del cliente se les concede acceso esencialmente como "inquilino de administración", aunque el cliente no administra los recursos del ISV. Dado que acceden directamente al inquilino del ISV, es importante conceder solo los permisos mínimos necesarios para que los clientes no puedan realizar cambios accidentales en la solución o en otros recursos de ISV.

Para habilitar esta arquitectura, el ISV debe obtener el identificador de objeto de un grupo de usuarios en el inquilino de Azure AD del cliente, junto con su identificador de inquilino. A continuación, el ISV crea una plantilla de ARM que concede a este grupo de usuarios los permisos adecuados y [la implementa en la suscripción del ISV](../how-to/onboard-customer.md) que contiene los recursos a los que el cliente tendrá acceso.

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre las [experiencias de administración entre inquilinos](cross-tenant-management-experience.md).
- Obtenga más información sobre la [arquitectura de Azure Lighthouse](architecture.md).
