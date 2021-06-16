---
title: Guía de publicación de ofertas de aplicaciones administradas de Azure - Azure Marketplace
description: En este artículo se describen los requisitos para publicar una aplicación administrada en Azure Marketplace.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
author: msjogarrig
ms.author: jogarrig
ms.date: 09/04/2020
ms.openlocfilehash: 73c8daf49ccdfeee1903a3c0ad823cdd3e52c141
ms.sourcegitcommit: bd65925eb409d0c516c48494c5b97960949aee05
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/06/2021
ms.locfileid: "111537863"
---
# <a name="publishing-guide-for-azure-managed-applications"></a>Guía de publicación de aplicaciones administradas de Azure

Una oferta de *aplicación administrada* de Azure es una manera de publicar una aplicación de Azure en Azure Marketplace. Las aplicaciones administradas son ofertas de transacciones que se implementan y facturan mediante Azure Marketplace. La opción de publicación que el usuario ve es *Obtener ahora*.

En este artículo se explican los requisitos para el tipo de oferta de aplicación administrada.

Use el tipo de oferta de aplicación administrada con las siguientes condiciones:

- Está implementando una solución basada en suscripción para su cliente mediante una máquina virtual (VM) o una solución completa basada en la infraestructura como servicio (IaaS).
- Usted o su cliente requieren que un asociado administre la solución.

>[!NOTE]
>Por ejemplo, un asociado puede ser un integrador de sistemas o un proveedor de servicios administrados (MSP).  

## <a name="managed-application-offer-requirements"></a>Requisitos de las ofertas de aplicaciones administradas

|Requisitos |Detalles  |
|---------|---------|
|Una suscripción de Azure | Las aplicaciones administradas se deben implementar en la suscripción del cliente, pero las puede administrar un tercero. |
|Facturación y medición    |  Los recursos se proporcionan en la suscripción de Azure de un cliente. Los recursos de Azure que usan el modelo de pago por uso se realizan con el cliente mediante Microsoft y se facturan con la suscripción a Azure del cliente. <br><br> En el caso de los recursos de Azure de traiga su propia licencia, Microsoft factura los costos de infraestructura derivados de la suscripción del cliente, mientras que usted realizará la transacción de las tarifas de licencia de software directamente con el cliente.        |
|Un paquete de aplicación administrado por Azure    |   La plantilla de Azure Resource Manager configurada y la definición de la interfaz de usuario que se usarán para implementar la aplicación en la suscripción del cliente.<br><br>Para obtener más información sobre cómo crear una aplicación administrada, consulte [Información general sobre las aplicaciones administradas](../azure-resource-manager/managed-applications/publish-service-catalog-app.md).|

---

> [!NOTE]
> Las aplicaciones administradas se deben poder implementar desde Azure Marketplace. Si la comunicación con el cliente es una preocupación, póngase en contacto con los clientes interesados después de habilitar el uso compartido de clientes potenciales.  

> [!Note]
> La participación en el canal de asociados de Proveedores de soluciones en la nube (CSP) ya está disponible. Para más información sobre cómo comercializar su oferta a través de los canales de asociados de CSP de Microsoft, consulte [Proveedores de soluciones en la nube](./cloud-solution-providers.md).

## <a name="next-steps"></a>Pasos siguientes

Si aún no lo ha hecho, aprenda a [Ampliar su negocio en la nube con Azure Marketplace](https://azuremarketplace.microsoft.com/sell).

Para registrarse y empezar a trabajar en el Centro de partners, haga lo siguiente:

- [Inicie sesión en el Centro de partners](https://partner.microsoft.com/dashboard/account/v3/enrollment/introduction/partnership) para crear o completar la oferta.
- Para obtener más información, consulte [Creación de una oferta de aplicaciones de Azure](azure-app-offer-setup.md).
