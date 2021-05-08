---
title: 'Acceso condicional: bloqueo del acceso (Azure Active Directory)'
description: Cree una directiva de acceso condicional personalizada para
services: active-directory
ms.service: active-directory
ms.subservice: conditional-access
ms.topic: how-to
ms.date: 05/26/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: calebb,
ms.collection: M365-identity-device-management
ms.openlocfilehash: c792888b65f331332c320deecbcd2b5f8d7e2ee3
ms.sourcegitcommit: 516eb79d62b8dbb2c324dff2048d01ea50715aa1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108175773"
---
# <a name="conditional-access-block-access"></a>Acceso condicional: Bloquear acceso

En el caso de las organizaciones con un enfoque conservador de migración a la nube, el bloqueo de todas las directivas es una opción que se puede usar. 

> [!CAUTION]
> La configuración incorrecta de una directiva de bloqueo puede dar lugar a que las organizaciones no puedan acceder a Azure Portal.

Este tipo de directivas pueden tener efectos secundarios imprevistos. Las pruebas y la validación adecuadas son vitales antes de habilitarlas. Los administradores deben usar herramientas del acceso condicional como el [modo de solo informe](concept-conditional-access-report-only.md) y la [herramienta What If](what-if-tool.md) al realizar cambios.

## <a name="user-exclusions"></a>Exclusiones de usuarios

Las directivas de acceso condicional son herramientas eficaces, por lo que se recomienda excluir las siguientes cuentas de la directiva:

* Cuentas de **acceso de emergencia** para **evitar** el bloqueo de cuentas en todo el inquilino. En el caso improbable de que todos los administradores estén bloqueados en el inquilino, la cuenta administrativa de acceso de emergencia se puede usar para iniciar sesión en el inquilino y realizar los pasos para recuperar el acceso.
   * Se puede encontrar más información en el artículo [Administración de cuentas de acceso de emergencia en Azure AD](../roles/security-emergency-access.md).
* **Cuentas de servicio** y **entidades de servicio**, como la cuenta de sincronización de Azure AD Connect. Las cuentas de servicio son cuentas no interactivas que no están asociadas a ningún usuario en particular. Los servicios back-end las usan normalmente para permitir el acceso mediante programación a las aplicaciones, pero también se utilizan para iniciar sesión en los sistemas con fines administrativos. Las cuentas de servicio como estas se deben excluir porque MFA no se puede completar mediante programación. El acceso condicional no bloquea las llamadas realizadas por las entidades de servicio.
   * Si su organización usa estas cuentas en scripts o código, piense en la posibilidad de reemplazarlas por [identidades administradas](../managed-identities-azure-resources/overview.md). Como solución temporal, puede excluir estas cuentas específicas de la directiva de línea de base.

## <a name="create-a-conditional-access-policy"></a>Creación de una directiva de acceso condicional

Los pasos siguientes le ayudarán a crear directivas de acceso condicional para bloquear el acceso a todas las aplicaciones excepto [Office 365](concept-conditional-access-cloud-apps.md#office-365) si los usuarios no están en una red de confianza. Estas directivas se colocan inicialmente en [modo de solo informe](howto-conditional-access-insights-reporting.md), para que los administradores puedan determinar el impacto que tendrán en los usuarios existentes. Cuando los administradores consideren que las directivas se aplican de la forma prevista, pueden cambiarlas a **Activadas**.

La primera directiva bloquea el acceso a todas las aplicaciones (menos las de Microsoft 365) si no se encuentran en una ubicación de confianza.

1. Inicie sesión en **Azure Portal** como administrador global, administrador de seguridad o administrador de acceso condicional.
1. Vaya a **Azure Active Directory** > **Seguridad** > **Acceso condicional**.
1. Seleccione **Nueva directiva**.
1. Asigne un nombre a la directiva. Se recomienda que las organizaciones creen un estándar significativo para los nombres de sus directivas.
1. En **Asignaciones**, seleccione **Usuarios y grupos**.
   1. En **Incluir**, seleccione **Todos los usuarios**.
   1. En **Excluir**, seleccione **Usuarios y grupos** y, luego, elija las cuentas de acceso de emergencia de la organización. 
   1. Seleccione **Listo**.
1. En **Aplicaciones o acciones en la nube** seleccione las opciones siguientes:
   1. En **Incluir**, seleccione **Todas las aplicaciones en la nube**.
   1. En **Excluir**, seleccione **Office 365**, **Seleccionar** y, luego, **Listo**.
1. En **Condiciones**:
   1. En **Condiciones** > **Ubicación**:
      1. Establezca **Configurar** en **Sí**.
      1. En **Incluir**, seleccione **Cualquier ubicación**.
      1. En **Excluir**, seleccione **Todas las ubicaciones de confianza**.
      1. Seleccione **Listo**.
   1. En **Aplicaciones cliente (versión preliminar)** , establezca **Configurar** en **Sí**, seleccione **Listo** y, luego, **Listo**.
1. En **Controles de acceso** > **Conceder**, seleccione **Bloquear acceso** y, después, **Seleccionar**.
1. Confirme la configuración y establezca **Habilitar directiva** en **Solo informe**.
1. Seleccione **Crear** para crear la directiva.

A continuación se crea una segunda directiva para requerir la autenticación multifactor o un dispositivo compatible para los usuarios de Microsoft 365.

1. Seleccione **Nueva directiva**.
1. Asigne un nombre a la directiva. Se recomienda que las organizaciones creen un estándar significativo para los nombres de sus directivas.
1. En **Asignaciones**, seleccione **Usuarios y grupos**.
   1. En **Incluir**, seleccione **Todos los usuarios**.
   1. En **Excluir**, seleccione **Usuarios y grupos** y, luego, elija las cuentas de acceso de emergencia de la organización. 
   1. Seleccione **Listo**.
1. En **Aplicaciones en la nube o acciones** > **Incluir**, elija sucesivamente **Seleccionar aplicaciones**, **Office 365**, **Seleccionar** y, luego, **Listo**.
1. En **Controles de acceso** > **Conceder**, seleccione **Conceder acceso**.
   1. Seleccione **Requerir autenticación multifactor**, **Requerir que el dispositivo esté marcado como compatible** y, luego, **Seleccionar**.
   1. Asegúrese de que la opción **Requerir todos los controles seleccionados** esté seleccionada.
   1. Elija **Seleccionar**.
1. Confirme la configuración y establezca **Habilitar directiva** en **Solo informe**.
1. Seleccione **Crear** para crear la directiva.

> [!NOTE]
> Las directivas de acceso condicional se aplican una vez que se completa la autenticación en una fase. El acceso condicional no pretende ser una primera línea de defensa de una organización en escenarios como los ataques por denegación de servicio (DoS), pero puede usar señales de estos eventos para determinar el acceso.

## <a name="next-steps"></a>Pasos siguientes

[Directivas de acceso condicional habituales](concept-conditional-access-policy-common.md)

[Determinación del impacto mediante el modo de solo informe de acceso condicional](howto-conditional-access-insights-reporting.md)

[Simulación del comportamiento de inicio de sesión mediante la herramienta What If de acceso condicional](troubleshoot-conditional-access-what-if.md)
