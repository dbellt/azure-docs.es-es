---
title: 'Acceso condicional basado en riesgos de inicio de sesión: Azure Active Directory'
description: Crear directivas de acceso condicional con el riesgo de inicio de sesión de Identity Protection
services: active-directory
ms.service: active-directory
ms.subservice: conditional-access
ms.topic: how-to
ms.date: 05/04/2021
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: calebb, rogoya
ms.collection: M365-identity-device-management
ms.openlocfilehash: 99d8fbf65cdfd4a56f4e7bec197131a1274b0beb
ms.sourcegitcommit: 6323442dbe8effb3cbfc76ffdd6db417eab0cef7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110612807"
---
# <a name="conditional-access-sign-in-risk-based-conditional-access"></a>Acceso condicional: Acceso condicional basado en el riesgo del inicio de sesión

La mayoría de los usuarios tienen un comportamiento normal que puede seguirse, cuando se salen de esta norma, podría ser peligroso permitirles que simplemente inicien sesión. Es posible que sea conveniente bloquear a ese usuario o quizás simplemente puede pedirle que lleve a cabo la autenticación multifactor para demostrar que realmente es quien dice ser. 

Un riesgo de inicio de sesión representa la probabilidad de que el propietario de la identidad no haya autorizado una solicitud de autenticación determinada. Las organizaciones con licencias de Azure AD Premium P2 pueden crear directivas de acceso condicional que incorporen [detecciones de riesgo de inicio de sesión de Azure AD Identity Protection](../identity-protection/concept-identity-protection-risks.md#sign-in-risk).

Hay dos ubicaciones donde se puede configurar esta directiva: Acceso condicional e Identity Protection. La configuración mediante una directiva de acceso condicional es el método preferido que proporciona más contexto, incluidos datos de diagnóstico mejorados, integración con el modo de solo informe, compatibilidad con Graph API y la posibilidad de utilizar otros atributos de acceso condicional en la directiva.

## <a name="enable-with-conditional-access-policy"></a>Habilitar la directiva de acceso condicional

1. Inicie sesión en **Azure Portal** como administrador global, administrador de seguridad o administrador de acceso condicional.
1. Vaya a **Azure Active Directory** > **Seguridad** > **Acceso condicional**.
1. Seleccione **Nueva directiva**.
1. Asigne un nombre a la directiva. Se recomienda que las organizaciones creen un estándar significativo para los nombres de sus directivas.
1. En **Asignaciones**, seleccione **Usuarios y grupos**.
   1. En **Incluir**, seleccione **Todos los usuarios**.
   1. En **Excluir**, seleccione **Usuarios y grupos** y, luego, elija las cuentas de acceso de emergencia de la organización. 
   1. Seleccione **Listo**.
1. En **Aplicaciones en la nube o acciones** > **Incluir**, seleccione **Todas las aplicaciones en la nube**.
1. En **Condiciones** > **Riesgo de inicio de sesión**, establezca **Configurar** en **Sí**. En **Seleccionar el nivel de riesgo de inicio de sesión, esta directiva se aplicará a** 
   1. Seleccione **Alto** y **Medio**.
   1. Seleccione **Listo**.
1. En **Controles de acceso** > **Conceder**, seleccione **Conceder acceso**, **Requerir autenticación multifactor** y **Seleccionar**.
1. Confirme la configuración y establezca **Habilitar directiva** en **Activado**.
1. Seleccione **Crear** para crear la directiva.

## <a name="enable-through-identity-protection"></a>Habilitar a través de Identity Protection

1. Inicie sesión en **Azure Portal**.
1. Seleccione **Todos los servicios** y, después, vaya a **Azure AD Identity Protection**.
1. Seleccione **Directiva de riesgo de inicio de sesión**.
1. En **Asignaciones**, seleccione **Usuarios**.
   1. En **Incluir**, seleccione **Todos los usuarios**.
   1. En **Excluir**, elija **Seleccionar usuarios excluidos**, elija las cuentas de acceso de emergencia de la organización y elija **Seleccionar**.
   1. Seleccione **Listo**.
1. En **Condiciones**, seleccione **Riesgo de inicio de sesión** y, luego, elija **Medio y superior**.
   1. Seleccione **Seleccionar** y, después, **Listo**.
1. En **Controles** > **Acceso**, elija **Permitir acceso** y, luego, seleccione **Requerir autenticación multifactor**.
   1. Elija **Seleccionar**.
1. Establezca **Aplicar directiva** en **Activado**.
1. Seleccione **Guardar**.

## <a name="next-steps"></a>Pasos siguientes

[Directivas de acceso condicional habituales](concept-conditional-access-policy-common.md)

[Acceso condicional basado en riesgos de usuario](howto-conditional-access-policy-risk-user.md)

[Determinación del impacto mediante el modo de solo informe de acceso condicional](howto-conditional-access-insights-reporting.md)

[Simulación del comportamiento de inicio de sesión mediante la herramienta What If de acceso condicional](troubleshoot-conditional-access-what-if.md)

[¿Qué es Azure Active Directory Identity Protection?](../identity-protection/overview-identity-protection.md)
