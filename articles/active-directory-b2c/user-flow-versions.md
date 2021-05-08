---
title: Versiones de flujos de usuario de Azure Active Directory B2C | Microsoft Docs
description: Obtenga información sobre las versiones de los flujos de usuario disponibles en Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 04/22/2021
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 4b47a311513a1216555583b64095d4bc46e4b48e
ms.sourcegitcommit: 19dcad80aa7df4d288d40dc28cb0a5157b401ac4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107895908"
---
# <a name="user-flow-versions-in-azure-active-directory-b2c"></a>Versiones de flujos de usuario de Azure Active Directory B2C

Los flujos de usuario de Azure Active Directory B2C (Azure AD B2C) le permiten configurar [directivas](user-flow-overview.md) que describan de forma exhaustiva la experiencia de los clientes con las identidades. Estas experiencias incluyen las operaciones de registro, inicio de sesión, restablecimiento de contraseña y edición de perfiles. En las tablas siguientes se describen los flujos de usuario que están disponibles en Azure AD B2C.

> [!IMPORTANT]
> Hemos cambiado el modo en que llamamos a las versiones de flujos de usuario. Anteriormente, se ofrecían las versiones V1 (para entornos de producción) y V1.1 y V2 (preliminares). Ahora, hemos consolidado los flujos de usuario en dos versiones:
>
>- Los flujos de usuario **recomendados** son los flujos de usuario de próxima generación disponibles con carácter general con las características más recientes. Combinan todas las características de las versiones **V1**, **V1.1** y **V2** heredadas. En el futuro, seguirán realizándose tareas de mantenimiento y habrá actualizaciones para los **flujos de usuario** recomendados. Una vez que se traslade a estos nuevos flujos de usuario recomendados, tendrá acceso a las nuevas características a medida que se vayan lanzando.
>- Los flujos de usuario **estándar (heredado)** , anteriormente conocidos como **V1**, son flujos de usuario heredados. A menos que tenga una necesidad empresarial específica, no se recomienda usar estas versiones de flujos de usuario porque no se mantendrán ni actualizarán.
>
>Todas las versiones preliminares de flujos de usuario heredadas (V1.1 y V2) estarán en proceso de desuso el **1 de agosto de 2021**. Siempre que sea posible, recomendamos encarecidamente que [cambie a las versiones **recomendadas**](#how-to-switch-to-a-recommended-user-flow) lo antes posible para que pueda aprovechar siempre las últimas características y actualizaciones. *Estos cambios solo se aplican a la nube pública de Azure. Otros entornos seguirán usando el [control de versiones de flujos de usuario heredado](user-flow-versions-legacy.md).*

## <a name="recommended-user-flows"></a>Flujos de usuario recomendados

Los flujos de usuario recomendados son los flujos de usuario de próxima generación disponibles con carácter general con las características más recientes. En el futuro, seguirán realizándose tareas de mantenimiento y habrá actualizaciones para los nuevos flujos de usuario recomendados.

| Flujo de usuario | Descripción |
| --------- | ----------- |
| Restablecimiento de contraseña | Permite a los usuarios elegir una contraseña nueva después de verificar el correo electrónico. Con este flujo de usuario, puede configurar: <ul><li>[Autenticación multifactor](multi-factor-authentication.md)</li><li>Configuración de compatibilidad de tokens</li><li>[Restricción de acceso por edad](age-gating.md)</li><li>[Requisitos de complejidad de la contraseña](password-complexity.md)</li></ul> |
| Edición de perfiles | Permite al usuario configurar los atributos de usuario. Con este flujo de usuario, puede configurar: <ul><li>[Vigencia de tokens](tokens-overview.md)</li><li>Configuración de compatibilidad de tokens</li><li>Comportamiento de la sesión</li></ul> |
| Iniciar sesión | Permite a los usuarios iniciar sesión en sus cuentas. Con este flujo de usuario, puede configurar: <ul><li>[Autenticación multifactor](multi-factor-authentication.md)</li><li>[Vigencia de tokens](tokens-overview.md)</li><li>Configuración de compatibilidad de tokens</li><li>Comportamiento de la sesión</li><li>[Restricción de acceso por edad](age-gating.md)</li><li>Personalización de la página de inicio de sesión</li></ul> |
| Suscripción | Permite a los usuarios crear una cuenta. Con este flujo de usuario, puede configurar: <ul><li>[Autenticación multifactor](multi-factor-authentication.md)</li><li>[Vigencia de tokens](tokens-overview.md)</li><li>Configuración de compatibilidad de tokens</li><li>Comportamiento de la sesión</li><li>[Restricción de acceso por edad](age-gating.md)</li><li>[Requisitos de complejidad de la contraseña](password-complexity.md)</li></ul> |
| Registrarse e iniciar sesión | Permite a los usuarios crear una cuenta o iniciar sesión en sus cuentas. Con este flujo de usuario, puede configurar: <ul><li>[Autenticación multifactor](multi-factor-authentication.md)</li><li>[Restricción de acceso por edad](age-gating.md)</li><li>[Requisitos de complejidad de la contraseña](password-complexity.md)</li></ul> |

## <a name="standard-user-flows"></a>Flujos de usuario estándar

Los flujos de usuario estándar (anteriormente denominados V1), antes conocidos como **V1,** son flujos de usuario heredados. A menos que tenga una necesidad empresarial específica, no se recomienda usar estas versiones de flujos de usuario porque no se actualizarán en el futuro.

| Flujo de usuario | Descripción |
| --------- | ----------- |
| Restablecimiento de contraseña | Permite a los usuarios elegir una contraseña nueva después de verificar el correo electrónico. Con este flujo de usuario, puede configurar: <ul><li>[Autenticación multifactor](multi-factor-authentication.md)</li><li>Configuración de compatibilidad de tokens</li><li>[Requisitos de complejidad de la contraseña](password-complexity.md)</li></ul> |
| Edición de perfiles | Permite al usuario configurar los atributos de usuario. Con este flujo de usuario, puede configurar: <ul><li>[Vigencia de tokens](tokens-overview.md)</li><li>Configuración de compatibilidad de tokens</li><li>Comportamiento de la sesión</li></ul> |
| Iniciar sesión | Permite a los usuarios iniciar sesión en sus cuentas. Con este flujo de usuario, puede configurar: <ul><li>[Autenticación multifactor](multi-factor-authentication.md)</li><li>[Vigencia de tokens](tokens-overview.md)</li><li>Configuración de compatibilidad de tokens</li><li>Comportamiento de la sesión</li><li>Bloqueo de inicios de sesión</li><li>Forzar el restablecimiento de contraseñas</li><li>Mantener la sesión iniciada (KMSI)</ul><br>No se puede personalizar la interfaz de usuario con este flujo de usuario. |
| Suscripción | Permite a los usuarios crear una cuenta. Con este flujo de usuario, puede configurar: <ul><li>[Autenticación multifactor](multi-factor-authentication.md)</li><li>[Vigencia de tokens](tokens-overview.md)</li><li>Configuración de compatibilidad de tokens</li><li>Comportamiento de la sesión</li><li>[Requisitos de complejidad de la contraseña](password-complexity.md)</li></ul> |
| Registrarse e iniciar sesión | Permite a los usuarios crear una cuenta o iniciar sesión en sus cuentas. Con este flujo de usuario, puede configurar: <ul><li>[Autenticación multifactor](multi-factor-authentication.md)</li><li>[Vigencia de tokens](tokens-overview.md)</li><li>Configuración de compatibilidad de tokens</li><li>Comportamiento de la sesión</li><li>[Requisitos de complejidad de la contraseña](password-complexity.md)</li></ul>|


## <a name="how-to-switch-to-a-recommended-user-flow"></a>Cambio a un flujo de usuario recomendado

Para cambiar de una versión heredada de un flujo de usuario a la versión **recomendada**, siga estos pasos:

1. Cree una nueva directiva de flujos de usuario siguiendo los pasos que se indican en [Tutorial: Creación de flujos de usuario en Azure Active Directory](tutorial-create-user-flows.md). Cuando cree el flujo de usuario, seleccione la versión **recomendada**.

3. Configure el nuevo flujo de usuario con la misma configuración que la directiva heredada.

4. Actualice la dirección URL de inicio de sesión de la aplicación por la de la directiva que acaba de crear.

5. Después de probar el flujo de usuario y confirmar que funciona, elimine el flujo de usuario heredado siguiendo estos pasos:
   1. En el menú de información general del inquilino de Azure AD B2C, seleccione **Flujos de usuario**.
   2. Busque el usuario que desee eliminar.
   3. En la última columna, seleccione el menú contextual ( **...** ) y **Eliminar**.

## <a name="frequently-asked-questions"></a>Preguntas más frecuentes

### <a name="can-i-still-create-legacy-v2-and-v11-user-flows"></a>¿Puedo seguir creando flujos de usuario con las versiones V2 y V1.1 heredadas?

No puede crear nuevos flujos de usuario basados en las versiones V2 y V1.1 heredadas, pero puede seguir leyendo, actualizando y eliminando los flujos de usuario de las versiones V2 y V1.1 heredadas que esté usando actualmente.

### <a name="is-there-any-reason-to-continue-using-legacy-v2-and-v11-user-flows"></a>¿Existe alguna razón para seguir usando flujos de usuario de las versiones V2 y V1.1 heredadas?

No. Las versiones **recomendadas** tienen la misma funcionalidad que las versiones V2 y V1.1 heredadas. No se ha quitado nada; es más, ahora contienen algunas características adicionales.

### <a name="if-i-dont-switch-from-legacy-v2-and-v11-policies-how-will-it-impact-my-application"></a>Si no cambio las directivas de las versiones V2 y V1.1 heredadas, ¿cómo afectará a la aplicación?

Si usa un flujo de usuario de las versiones V2 o V1.1 heredadas, la aplicación no se verá afectada por este cambio de versiones. Sin embargo, para poder acceder a las nuevas características o a los cambios de directivas en el futuro, deberá cambiar a las nuevas versiones **recomendadas**.

### <a name="will-microsoft-still-support-my-legacy-v2-or-v11-user-flow-policy"></a>¿Microsoft seguirá admitiendo mi directiva de flujo de usuario de la versión V2 o V1.1 heredada?

En la nube pública, todos los flujos de usuario de las versiones preliminares heredadas (V1.1 y V2) se encuentran en un proceso de desuso que finalizará el 1 de agosto de 2021. Siempre que sea posible, recomendamos encarecidamente que [cambie a las versiones **recomendadas**](#how-to-switch-to-a-recommended-user-flow) lo antes posible para que pueda aprovechar siempre las últimas características y actualizaciones. *Estos cambios solo se aplican a la nube pública de Azure. Otros entornos seguirán usando el [control de versiones de flujos de usuario heredado](user-flow-versions-legacy.md).*