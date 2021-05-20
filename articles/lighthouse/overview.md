---
title: ¿Qué es Azure Lighthouse?
description: Azure Lighthouse permite a los proveedores de servicios ofrecer servicios administrados para sus clientes con mayor automatización y eficacia a escala.
ms.date: 05/11/2021
ms.topic: overview
ms.openlocfilehash: 3f8b09584a5f025e22543a65eefd50dbfa7bb894
ms.sourcegitcommit: 32ee8da1440a2d81c49ff25c5922f786e85109b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109785914"
---
# <a name="what-is-azure-lighthouse"></a>¿Qué es Azure Lighthouse?

Azure Lighthouse permite la administración de varios inquilinos, lo que proporciona no solo unas mayores automatización y escalabilidad, sino también una mejor gobernanza de recursos e inquilinos.

Con Azure Lighthouse, los proveedores de servicios pueden ofrecer servicios administrados mediante herramientas de administración completas y sólidas integradas en la plataforma Azure. Los clientes mantienen el control sobre quién puede acceder a su inquilino, a qué recursos se puede acceder y qué acciones se pueden realizar. Esta oferta también puede ayudar a las [organizaciones de TI empresariales](concepts/enterprise.md) a administrar recursos en varios inquilinos.

La [experiencia de administración entre inquilinos](concepts/cross-tenant-management-experience.md) le permite trabajar de forma más eficaz con los servicios de administración de Azure, como Azure Policy, Azure Security Center, etc. En el registro de actividad se realiza un seguimiento de toda la actividad del proveedor de servicios, que se almacena en el inquilino del cliente (y se puede ver por usuarios en el inquilino administrador). Los usuarios pueden ver qué cambios se han realizado y por quién.

![Diagrama de información general de Azure Lighthouse](media/azure-lighthouse-overview.jpg)

## <a name="benefits"></a>Ventajas

Azure Lighthouse ayuda a los proveedores de servicios a compilar y ofrecer servicios administrados de forma eficaz. Dicha integración aporta las siguientes ventajas:

- **Administración a escala**: la involucración del cliente y las operaciones de ciclo de vida para administrar los recursos del cliente son más fáciles y escalables. Las API, las herramientas de administración y los flujos de trabajo existentes se pueden usar con recursos delegados, incluidas las máquinas hospedadas fuera de Azure, con independencia de las regiones en las que se encuentren.
- **Mayor visibilidad y control para los clientes**: los clientes tienen un control preciso sobre los ámbitos en los que delegan la administración y sobre los permisos admitidos. Pueden auditar las acciones del proveedor de servicios y retirar el acceso por completo si así lo desean.
- **Herramientas de plataforma completas y unificadas**: nuestra experiencia de herramientas aborda los escenarios clave del proveedor de servicios, incluidos varios modelos de licencia como EA, CSP y pago por uso. Azure Lighthouse funciona con herramientas y API existentes, modelos de licencias, [aplicaciones administradas de Azure](concepts/managed-applications.md) y programas de asociados, como el [programa del proveedor de soluciones en la nube (CSP)](/partner-center/csp-overview). Azure Lighthouse se puede integrar en los flujos de trabajo y aplicaciones existentes, y puede realizar un seguimiento de su efecto sobre las interacciones de los clientes [vinculando su identificador de asociado](./how-to/partner-earned-credit.md).

## <a name="capabilities"></a>Capacidades

Azure Lighthouse incluye varias formas de ayudar a simplificar la administración y la involucración:

- **Administración de recursos delegados de Azure**: [administre los recursos de Azure de sus clientes de forma segura desde su propio inquilino](concepts/architecture.md) sin tener que cambiar los planos de contexto y control. Las suscripciones y los grupos de recursos de los clientes se pueden delegar a usuarios y roles especificados en el inquilino de administración, con la posibilidad de quitar el acceso según sea necesario.
- **Nuevas experiencias de Azure Portal**: Vea la información de distintos inquilinos en la página [**Mis clientes** de](how-to/view-manage-customers.md) Azure Portal. Una página de [**proveedores de servicios** correspondiente](how-to/view-manage-service-providers.md) permite a los clientes ver y administrar el acceso de los proveedores de servicios.
- **Plantillas de Azure Resource Manager**: use plantillas de Resource Manager para [incorporar recursos de clientes delegados](how-to/onboard-customer.md) y [realizar tareas de administración entre inquilinos](samples/index.md).
- **Ofertas de servicio administrado en Azure Marketplace**: [ofrezca sus servicios a los clientes](concepts/managed-services-offers.md) mediante ofertas públicas o privadas e incorpórelos automáticamente a Azure Lighthouse.

> [!TIP]
> Una oferta similar, [Microsoft 365 Lighthouse](https://techcommunity.microsoft.com/t5/small-and-medium-business-blog/announcing-microsoft-365-lighthouse-for-managed-service/ba-p/1698181), ayudará a los asociados de TI a incorporar, supervisar y administrar sus clientes de Microsoft 365 a escala. Microsoft 365 Lighthouse se encuentra actualmente en versión preliminar privada.

## <a name="pricing-and-availability"></a>Precios y disponibilidad

No hay costos adicionales asociados con el uso de Azure Lighthouse para administrar los recursos de Azure. Cualquier cliente o asociado de Azure puede usar Azure Lighthouse.

## <a name="cross-region-and-cloud-considerations"></a>Consideraciones sobre el entorno entre regiones y la nube

Azure Lighthouse no es un servicio regional. Puede administrar recursos delegados que estén ubicados en diferentes [regiones](../availability-zones/az-overview.md#regions). Sin embargo, no se admite la delegación de suscripciones entre una [nube nacional](../active-directory/develop/authentication-national-cloud.md) y la nube pública de Azure o entre dos nubes nacionales independientes.

## <a name="support-for-azure-lighthouse"></a>Compatibilidad con Azure Lighthouse

Para obtener ayuda con el uso de Azure Lighthouse, abra una solicitud de soporte técnico en Azure Portal. En **Tipo de problema**, elija **Técnico**. Seleccione una suscripción y luego **Lighthouse** (en **Supervisión y administración**).

## <a name="next-steps"></a>Pasos siguientes

- Más información [sobre cómo funciona Azure Lighthouse en un nivel técnico](concepts/architecture.md).
- Explore las [experiencias de administración entre inquilinos](concepts/cross-tenant-management-experience.md).
- Vea cómo [usar Azure Lighthouse en una empresa](concepts/enterprise.md).
- Vea los detalles de [disponibilidad](https://azure.microsoft.com/global-infrastructure/services/?products=azure-lighthouse&regions=all) y del [ámbito de auditoría de FedRAMP y DoD CC SRG](../azure-government/compliance/azure-services-in-fedramp-auditscope.md) para Azure Lighthouse.
