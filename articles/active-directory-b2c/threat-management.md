---
title: Administración de amenazas en los recursos y los datos
titleSuffix: Azure AD B2C
description: Obtenga información sobre las técnicas de detección y mitigación de los ataques por denegación de servicio y los ataques a contraseñas en Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 09/26/2019
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 55c77354e50ce96ae80ce922798fbb8c2a49f41f
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108126892"
---
# <a name="manage-threats-to-resources-and-data-in-azure-active-directory-b2c"></a>Administración de amenazas en los recursos y los datos en Azure Active Directory B2C

Azure Active Directory B2C (Azure AD B2C) tiene características integradas que pueden ayudarle a protegerse frente a amenazas en los recursos y los datos. Estas amenazas incluyen los ataques de denegación de servicio y contraseña. Los ataques por denegación de servicio pueden hacer que los recursos no estén disponibles para los usuarios previstos. Los ataques a contraseñas pueden dar lugar a un acceso no autorizado a los recursos.

## <a name="denial-of-service-attacks"></a>Ataques por denegación de servicio

Azure AD B2C protege frente a ataques de inundación SYN mediante una cookie SYN. Azure AD B2C también protege frente a ataques de denegación de servicio mediante límites en las tasas y conexiones.

## <a name="password-attacks"></a>Ataques a contraseñas

Las contraseñas establecidas por los usuarios deben tener una complejidad razonable. Azure AD B2C también dispone de técnicas de mitigación para los ataques a contraseñas. La mitigación incluye la detección de los ataques a contraseñas por fuerza bruta y ataques de diccionario. Mediante el uso de diversas señales, Azure AD B2C analiza la integridad de las solicitudes. Azure AD B2C está diseñado para diferenciar de forma inteligente los usuarios previstos frente a los hackers y las redes de robots (botnets).

Azure AD B2C usa una sofisticada estrategia para bloquear las cuentas. Las cuentas se bloquean en función de la dirección IP de la solicitud y las contraseñas escritas. El tiempo que dura el bloqueo también aumenta en función de la probabilidad de que sea un ataque. Una vez que se ha intentado una contraseña sin éxito diez veces (el umbral de intento predeterminado), se produce un bloqueo de un minuto. La próxima vez que un inicio de sesión no tenga éxito después de que la cuenta se desbloquee (es decir, después de que el servicio haya desbloqueado automáticamente la cuenta una vez que expire el período de bloqueo), se producirá otro bloqueo de un minuto y continuará para cada inicio de sesión incorrecto. Escribir repetidamente la misma contraseña no cuenta como varios inicios de sesión incorrectos.

Los diez primeros períodos de bloqueo son de un minuto. Los diez períodos de bloqueo siguientes son ligeramente más largos y la duración aumenta después de cada diez períodos de bloqueo. El contador de bloqueos se restablece en cero tras un inicio de sesión correcto, cuando la cuenta no está bloqueada. Los períodos de bloqueo pueden durar hasta cinco horas.

## <a name="manage-password-protection-settings"></a>Administración de la configuración de protección con contraseña

Para administrar la configuración de protección con contraseña, incluido el umbral de bloqueo:

1. Inicie sesión en el [Portal de Azure](https://portal.azure.com)
1. Use el filtro **Directorio y suscripción** del menú superior para seleccionar el directorio que contiene el inquilino de Azure AD B2C.
1. En el menú de la izquierda, seleccione **Azure AD B2C**. O bien, seleccione **Todos los servicios** y busque y seleccione **Azure AD B2C**.
1. En **Seguridad**, seleccione **Métodos de autenticación (Vista preliminar)** y, a continuación, seleccione **Protección con contraseña**.
1. Especifique la configuración de protección con contraseña deseada y después seleccione **Guardar**.

    ![Página Protección con contraseña de Azure Portal en la configuración de Azure AD](./media/threat-management/portal-02-password-protection.png)
    <br />*Establezca el umbral de bloqueo en 5 en la configuración de **Protección con contraseña***.

## <a name="view-locked-out-accounts"></a>Visualización de las cuentas bloqueadas

Para más información sobre las cuentas bloqueadas, puede comprobar el [informe de actividad de inicio de sesión](../active-directory/reports-monitoring/concept-sign-ins.md) de Active Directory. En **Estado**, seleccione **Error**. Los intentos de inicio de sesión con un **código de error de inicio de sesión** de `50053` indican una cuenta bloqueada:

![Sección del informe de inicio de sesión de Azure AD que muestra la cuenta bloqueada](./media/threat-management/portal-01-locked-account.png)

Para más información sobre cómo ver el informe de actividad de inicio de sesión en Azure Active Directory, consulte [Códigos de error de los informes de actividad de inicio de sesión](../active-directory/reports-monitoring/concept-sign-ins.md).