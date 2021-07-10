---
title: Identity Protection y acceso condicional en Azure AD B2C
description: Obtenga información sobre cómo Identity Protection le ofrece visibilidad sobre los inicios de sesión de riesgo y las detecciones de riesgos. Descubra cómo el acceso condicional permite aplicar directivas de la organización en función de los eventos de riesgo en los inquilinos de Azure AD B2C.
services: active-directory
ms.service: active-directory
ms.subservice: conditional-access
ms.topic: overview
ms.date: 05/28/2021
ms.author: mimart
author: msmimart
manager: celested
ms.collection: M365-identity-device-management
ms.openlocfilehash: 739c20cb39db5fda3a182bc4c5379d0bc9c4d06e
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110690335"
---
# <a name="identity-protection-and-conditional-access-for-azure-ad-b2c"></a>Identity Protection y acceso condicional para Azure AD B2C

Mejore la seguridad de Azure Active Directory B2C (Azure AD B2C) con Azure AD Identity Protection y el acceso condicional. Las características de detección de riesgos de Identity Protection, incluidos los usuarios de riesgo y los inicios de sesión de riesgo, se detectan y se muestran automáticamente en el inquilino de Azure AD B2C. Puede crear directivas de acceso condicional que utilicen estas detecciones de riesgos para determinar las acciones y aplicar las directivas de la organización. Juntas, estas funcionalidades proporcionan a los propietarios de aplicaciones de Azure AD B2C mayor control sobre las autenticaciones de riesgo y las directivas de acceso.
  
Si ya está familiarizado con [Identity Protection](../active-directory/identity-protection/overview-identity-protection.md) y el [acceso condicional](../active-directory/conditional-access/overview.md) en Azure AD, el uso de estas funcionalidades con Azure AD B2C será una experiencia familiar y las pequeñas diferencias se describen en este artículo.

![Acceso condicional en un inquilino de B2C](media/conditional-access-identity-protection-overview/conditional-access-b2c.png)

> [!NOTE]
> Se requiere Azure AD B2C **Premium P2** para crear directivas de inicio de sesión de riesgo. Los inquilinos **Premium P1** pueden crear una directiva basada en la ubicación, la aplicación, el usuario o directivas basadas en grupos. Para más información, consulte la sección [Cambio del plan de tarifa de Azure AD](billing.md#change-your-azure-ad-pricing-tier).

## <a name="benefits-of-identity-protection-and-conditional-access-for-azure-ad-b2c"></a>Ventajas de Identity Protection y el acceso condicional para Azure AD B2C  

Al emparejar las directivas de acceso condicional con la detección de riesgos de Identity Protection, puede responder a las autenticaciones de riesgo con la acción de directiva adecuada.

- **Consiga un nuevo nivel de visibilidad sobre los riesgos de autenticación para las aplicaciones y la base de clientes**. Con señales de miles de millones de autenticaciones mensuales de Azure AD y la cuenta de Microsoft, los algoritmos de detección de riesgos ahora marcarán las autenticaciones como de bajo, medio o alto riesgo para las autenticaciones de los consumidores o ciudadanos locales.
- **Solucione los riesgos automáticamente mediante la configuración de su propia autenticación adaptable**. En el caso de las aplicaciones especificadas, puede requerir que un conjunto específico de usuarios proporcione un segundo factor de autenticación, como en la autenticación multifactor (MFA). También puede bloquear el acceso en función del nivel de riesgo detectado. Al igual que con otras experiencias de Azure AD B2C, puede personalizar la experiencia del usuario final resultante con la voz, el estilo y la marca de su organización. También puede mostrar alternativas de mitigación si el usuario no puede obtener acceso.
- **Controle el acceso en función de la ubicación, los grupos y las aplicaciones**.  El acceso condicional también se puede usar para controlar situaciones que no se basan en el riesgo. Por ejemplo, puede requerir MFA para que los clientes accedan a una aplicación específica o bloquear el acceso desde las zonas geográficas especificadas.
- **Realice la integración con los flujos de usuario de Azure AD B2C y las directivas personalizadas de Identity Experience Framework**. Use las experiencias personalizadas existentes y agregue los controles que necesite para interactuar con el acceso condicional. También puede implementar escenarios avanzados para conceder acceso, como el acceso basado en conocimientos o su propio proveedor de MFA preferido.

## <a name="feature-differences-and-limitations"></a>Diferencias y limitaciones de las características

Identity Protection y el acceso condicional generalmente funcionan en Azure AD B2C de la misma manera que en Azure AD, con las siguientes excepciones:

- Security Center no está disponible en Azure AD B2C.

- Identity Protection y el acceso condicional no se admiten en flujos de servidor a servidor de ROPC en los inquilinos de Azure AD B2C.

- En los inquilinos de Azure AD B2C, las detecciones de riesgos de Identity Protection solo están disponibles para las cuentas de B2C locales y no para las identidades sociales como Google o Facebook.

- En los inquilinos de Azure AD B2C hay disponible un subconjunto de las detecciones de riesgo de Identity Protection. Consulte [Investigación del riesgo con Identity Protection](identity-protection-investigate-risk.md)y [Adición de acceso condicional a flujos de usuario](conditional-access-user-flow.md).

- La característica de cumplimiento de dispositivos del acceso condicional no está disponible en los inquilinos de Azure AD B2C.


## <a name="integrate-conditional-access-with-user-flows-and-custom-policies"></a>Integración del acceso condicional con los flujos de usuario y las directivas personalizadas

En Azure AD B2C, puede desencadenar condiciones del acceso condicional a partir de flujos de usuarios integrados. También puede incorporar el acceso condicional a las directivas personalizadas. Como con otros aspectos del flujo de usuario de B2C, la mensajería de la experiencia del usuario final se puede personalizar según las alternativas de voz, marca y mitigación de la organización. Consulte [Adición del acceso condicional a flujos de usuario](conditional-access-user-flow.md).

## <a name="microsoft-graph-api"></a>Microsoft Graph API

También puede administrar las directivas de acceso condicional de Azure AD B2C con Microsoft Graph API. Para más información, consulte la [documentación del acceso condicional](../active-directory/conditional-access/overview.md) y las [operaciones de Microsoft Graph](microsoft-graph-operations.md#conditional-access).

## <a name="next-steps"></a>Pasos siguientes

- [Adición del acceso condicional a flujos de usuario](conditional-access-user-flow.md)
- [Más información acerca de Identity Protection en Azure AD](../active-directory/identity-protection/overview-identity-protection.md)
- [Más información sobre el acceso condicional](../active-directory/conditional-access/overview.md)