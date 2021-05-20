---
title: Modelo de facturación de MAU para Azure AD for External Identities
description: Obtenga información acerca del modelo de facturación de usuarios activos mensuales (MAU) de Azure AD for External Identities para la colaboración de usuarios invitados (B2B) en Azure AD. Obtenga información sobre cómo vincular el inquilino de Azure AD a una suscripción de Azure.
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: conceptual
ms.date: 05/05/2021
ms.author: mimart
author: msmimart
manager: celestedg
ms.workload: identity
ms.collection: M365-identity-device-management
ms.openlocfilehash: 2083df29754a8791848fd125136b04d1d7a06fb3
ms.sourcegitcommit: 5da0bf89a039290326033f2aff26249bcac1fe17
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109713223"
---
# <a name="billing-model-for-azure-ad-external-identities"></a>Modelo de facturación para Azure AD for External Identities

Los precios de Azure Active Directory (Azure AD) for External Identities se basan en los usuarios activos mensuales (MAU), que es el recuento de usuarios únicos con actividad de autenticación dentro de un mes natural. Este modelo de facturación se aplica tanto a la colaboración de usuarios invitados (B2B) de Azure AD como a los [inquilinos de Azure AD B2C](../../active-directory-b2c/billing.md). La facturación de MAU le ayuda a reducir los costos al ofrecer un nivel gratis y precios flexibles y predecibles. En este artículo obtendrá información sobre la facturación de MAU y la vinculación de los inquilinos de Azure AD a una suscripción.

> [!IMPORTANT]
> Este artículo no contiene detalles de precios. Para conocer la información más reciente sobre la facturación por uso y los precios, consulte [Precios de Azure Active Directory](https://azure.microsoft.com/pricing/details/active-directory/).

## <a name="what-do-i-need-to-do"></a>¿Qué tengo que hacer?

Para aprovechar las ventajas de la facturación de MAU, el inquilino de Azure AD debe estar vinculado a una suscripción de Azure.

|Si el inquilino es:  |Necesita:  |
|---------|---------|
| Un inquilino de Azure AD ya vinculado a una suscripción     | No haga nada. Al usar las características de identidades externas para colaborar con los usuarios invitados, se le facturará automáticamente mediante el modelo de MAU.        |
| Un inquilino de Azure AD no vinculado aún a una suscripción     | [Vincule el inquilino de Azure AD a una suscripción](#link-your-azure-ad-tenant-to-a-subscription) para activar la facturación de MAU.        |
|  |  |

## <a name="about-monthly-active-users-mau-billing"></a>Acerca de la facturación de usuarios activos mensuales (MAU)

En el inquilino de Azure AD, el uso de la colaboración de usuarios invitados se factura en función del número de usuarios invitados únicos con actividad de autenticación dentro de un mes natural. Este modelo reemplaza el modelo de facturación con una relación 1:5, que permitía hasta cinco usuarios invitados para cada licencia Premium de Azure AD del inquilino. Cuando el inquilino está vinculado a una suscripción y usa las características de identidades externas para colaborar con los usuarios invitados, se le facturará automáticamente mediante el modelo de facturación basado en MAU.

El plan de tarifa que se aplica a los usuarios invitados se basa en el plan de tarifa más alto asignado al inquilino de Azure AD. Para más información, consulte [Precios de Azure Active Directory for External Identities](https://azure.microsoft.com/en-us/pricing/details/active-directory/external-identities/).

## <a name="link-your-azure-ad-tenant-to-a-subscription"></a>Vinculación del inquilino de Azure AD a una suscripción

Se debe vincular un inquilino de Azure AD a una suscripción de Azure para una facturación adecuada y el acceso a las características. Si el directorio todavía no tiene una suscripción a la que se pueda vincular, tendrá la oportunidad de agregar una durante este proceso.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) con una cuenta de Azure a la que se haya asignado al menos el rol [Colaborador](../../role-based-access-control/built-in-roles.md) dentro de la suscripción o un grupo de recursos dentro de la suscripción.

2. Seleccione el directorio que desee vincular: En la barra de herramientas de Azure Portal, seleccione el icono de **Directorio + suscripción** y, luego, seleccione el directorio.

    ![Selección del icono de directorio y suscripción](media/external-identities-pricing/portal-mau-pick-directory.png)

3. En **Servicios de Azure**, seleccione **Azure Active Directory**.

4. En el menú de la izquierda, seleccione **External Identities**.

5. En **Suscripciones**, seleccione **Suscripciones vinculadas**.

6. En la lista de inquilinos, active la casilla situada junto al inquilino y, a continuación, seleccione **Vinculación de la suscripción**.

    ![Selección del inquilino y vinculación de una suscripción](media/external-identities-pricing/linked-subscriptions.png)

7. En el panel Vinculación de una suscripción, seleccione una **suscripción** y un **grupo de recursos**. Luego, seleccione **Aplicar**.

   > [!NOTE]
   > Si no aparecen suscripciones, puede [asociar una suscripción al inquilino](../fundamentals/active-directory-how-subscriptions-associated-directory.md). También puede agregar una nueva suscripción seleccionando el vínculo **.Si aún no tiene una suscripción, puede crear una aquí**.

    ![Selección de una suscripción y un grupo de recursos](media/external-identities-pricing/link-subscription-resource.png)

Después de completar estos pasos, la suscripción de Azure se factura según los detalles del Contrato Enterprise o de Azure Direct, si procede.

## <a name="next-steps"></a>Pasos siguientes

Para obtener la información de precios más reciente, consulte [Precios de Azure Active Directory](https://azure.microsoft.com/pricing/details/active-directory/).