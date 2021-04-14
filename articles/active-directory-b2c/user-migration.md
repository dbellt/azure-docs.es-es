---
title: Enfoques de migración de usuarios
titleSuffix: Azure AD B2C
description: Migre las cuentas de usuario de otro proveedor de identidades a Azure AD B2C con los métodos de migración previa o de migración de conexión directa.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 03/11/2021
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 3e60b80a4ebeaef7d31d4c0c1d9d4bfc41ec3a56
ms.sourcegitcommit: 20f8bf22d621a34df5374ddf0cd324d3a762d46d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/09/2021
ms.locfileid: "107256215"
---
# <a name="migrate-users-to-azure-ad-b2c"></a>Migrar usuarios a Azure AD B2C

La migración desde otro proveedor de identidades a Azure Active Directory B2C (Azure AD B2C) puede requerir también la migración de las cuentas de usuario existentes. Aquí se describen dos métodos de migración, *migración previa* y *migración de conexión directa*. Con cualquiera de estos enfoques, es necesario escribir una aplicación o un script que use [Microsoft Graph API](microsoft-graph-operations.md) para crear cuentas de usuario en Azure AD B2C.

## <a name="pre-migration"></a>Migración previa

En el flujo de migración previa, la aplicación de migración realiza estos pasos para cada cuenta de usuario:

1. Lea la cuenta de usuario del proveedor de identidades anterior, incluidas sus credenciales actuales (nombre de usuario y contraseña).
1. Cree una cuenta correspondiente en el directorio de Azure AD B2C con las credenciales actuales.

Use el flujo de migración previa en cualquiera de estas dos situaciones:

- Tiene acceso a las credenciales de texto no cifrado de un usuario (su nombre de usuario y contraseña).
- Las credenciales están cifradas, pero puede descifrarlas.

Para información sobre la creación de cuentas de usuario mediante programación, consulte [Administrar cuentas de usuario de Azure AD B2C con Microsoft Graph](microsoft-graph-operations.md).

## <a name="seamless-migration"></a>Migración de conexión directa

Use el flujo de migración de conexión directa si no se puede tener acceso a las contraseñas de texto no cifrado del proveedor de identidades anterior. Por ejemplo, cuando:

- La contraseña se almacena en formato cifrado unidireccional, como con una función hash.
- El proveedor de identidades heredado ha almacenado la contraseña de modo que no puede tener acceso. Por ejemplo, cuando el proveedor de identidades valida las credenciales mediante una llamada a un servicio web.

El flujo de migración de conexión directa sigue requiriendo la migración previa de las cuentas de usuario, pero después usa una [directiva personalizada](user-flow-overview.md) para consultar una [API de REST](custom-policy-rest-api-intro.md) (que usted crea) a fin de establecer la contraseña de cada usuario al iniciar sesión por primera vez.

Por lo tanto, el flujo de migración de conexión directa tiene dos fases: *migración previa* y *establecimiento de credenciales*.

### <a name="phase-1-pre-migration"></a>Fase 1: Migración previa

1. La aplicación de migración lee las cuentas de usuario del proveedor de identidades anterior.
1. La aplicación de migración crea las cuentas de usuario correspondientes en el directorio de Azure AD B2C, pero *establece contraseñas aleatorias* que usted genera.

### <a name="phase-2-set-credentials"></a>Fase 2: Establecer credenciales

Una vez completada la migración previa de las cuentas, la directiva personalizada y la API de REST realizan lo siguiente cuando un usuario inicia sesión:

1. Leen la cuenta de usuario de Azure AD B2C correspondiente a la dirección de correo electrónico especificada.
1. Comprueban si la cuenta está marcada para la migración mediante la evaluación de un atributo de extensión booleano.
    - Si el atributo de extensión devuelve `True`, llame a la API de REST para validar la contraseña con el proveedor de identidades heredado.
      - Si la API de REST determina que la contraseña es incorrecta, devuelve un error descriptivo al usuario.
      - Si la API de REST determina que la contraseña es correcta, escriba la contraseña para la cuenta de Azure AD B2C y cambie el atributo de extensión booleano a `False`.
    - Si el atributo de extensión booleano devuelve `False`, continúe el proceso de inicio de sesión de la forma habitual.

Para ver un ejemplo de una directiva personalizada y una API de REST, consulte el [ejemplo de migración de usuarios de conexión directa](https://aka.ms/b2c-account-seamless-migration) en GitHub.

![Diagrama de flujo del enfoque de migración de conexión directa de la migración de usuarios](./media/user-migration/diagram-01-seamless-migration.png)<br />*Diagrama: flujo de migración de conexión directa*

## <a name="best-practices"></a>Procedimientos recomendados

### <a name="security"></a>Seguridad

El enfoque de migración de conexión directa usa su propia API de REST personalizada para validar las credenciales de un usuario con el proveedor de identidades heredado.

**Debe proteger la API de REST contra ataques por fuerza bruta.** Un atacante puede enviar varias contraseñas con la esperanza de adivinar las credenciales de un usuario. Para ayudar a frustrar estos ataques, deje de servir solicitudes a la API de REST cuando el número de intentos de inicio de sesión supere un determinado umbral. Además, proteja la comunicación entre Azure AD B2C y la API REST. Para aprender a proteger las API RESTful para la producción, consulte [Proteger la API RESTful](secure-rest-api.md).

### <a name="user-attributes"></a>Atributos de usuario

No toda la información del proveedor de identidades heredado debe migrarse al directorio de Azure AD B2C. Identifique el conjunto de atributos de usuario adecuado que se debe almacenar en Azure AD B2C antes de la migración.

- **ALMACENE** en Azure AD B2C
  - El nombre de usuario, la contraseña, las direcciones de correo electrónico, los números de teléfono, los números de pertenencia o identificadores.
  - Marcadores de consentimiento para la directiva de privacidad y los contratos de licencia para el usuario final.
- **NO ALMACENE** en Azure AD B2C
  - Datos confidenciales, como números de tarjetas de crédito, números de seguridad social (SSN), registros médicos u otros datos regulados por el gobierno u organismos de cumplimiento del sector.
  - Preferencias de marketing o de comunicación, comportamientos de los usuarios y conocimientos.

### <a name="directory-clean-up"></a>Limpieza de directorios

Antes de comenzar el proceso de migración, aproveche la oportunidad para limpiar el directorio.

- Identifique el conjunto de atributos de usuario que se van a almacenar en Azure AD B2C y migre solo lo que necesite. Si es necesario, puede crear [atributos personalizados](user-flow-custom-attributes.md) para almacenar más datos sobre un usuario.
- Si va a migrar desde un entorno con varios orígenes de autenticación (por ejemplo, cada aplicación tiene su propio directorio de usuario), migre a una cuenta unificada en Azure AD B2C.
- Si varias aplicaciones tienen distintos nombres de usuario, puede almacenarlas todas en una cuenta de usuario de Azure AD B2C mediante la colección de identidades. Con respecto a la contraseña, permita que el usuario elija una y establézcala en el directorio. Por ejemplo, con la migración de conexión directa, solo la contraseña elegida debe almacenarse en la cuenta de Azure AD B2C.
- Quite las cuentas de usuario no utilizadas antes de la migración o no migre cuentas obsoletas.

### <a name="password-policy"></a>Directiva de contraseñas

Si las cuentas que se van a migrar tienen una seguridad de contraseña inferior a la [seguridad de contraseña segura](../active-directory/authentication/concept-sspr-policy.md) que exige Azure AD B2C, puede deshabilitar el requisito de contraseña segura. Para obtener más información, vea [Propiedad de directiva de contraseñas](user-profile-attributes.md#password-policy-attribute).

## <a name="next-steps"></a>Pasos siguientes

El repositorio [azure-ad-b2c/user-migration](https://github.com/azure-ad-b2c/user-migration) de GitHub contiene un ejemplo de directiva personalizada de migración de conexión directa y un ejemplo de código de API de REST:

[Directiva personalizada de migración de usuarios de conexión directa y ejemplo de código de API de REST](https://aka.ms/b2c-account-seamless-migration)
