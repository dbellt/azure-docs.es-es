---
title: Error de autenticación de Azure Digital Twins Explorer
description: Causas y resoluciones para el "Error de autenticación" en Azure Digital Twins Explorer.
ms.service: digital-twins
author: baanders
ms.author: baanders
ms.topic: troubleshooting
ms.date: 4/8/2021
ms.openlocfilehash: a91053f9a21aebf1a99c35a3a982fd3ad5514d04
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110475661"
---
# <a name="authentication-failed"></a>Error de autenticación

En este artículo se describen las causas y los pasos de resolución al recibir un "Error de autenticación" cuando se ejecuta el ejemplo de [Azure Digital Twins Explorer](/samples/azure-samples/digital-twins-explorer/digital-twins-explorer/) en el equipo local. 

## <a name="symptoms"></a>Síntomas

Al configurar y ejecutar la aplicación Azure Digital Twins Explorer, los intentos de autenticación con la aplicación reciben el siguiente mensaje de error:

:::image type="content" source="media/troubleshoot-error-azure-digital-twins-explorer-authentication/authentication-error.png" alt-text="Captura de pantalla de un mensaje de error de autenticación en Azure Digital Twins Explorer.":::

## <a name="causes"></a>Causas

### <a name="cause-1"></a>Causa 1

Este error también puede producirse si la cuenta de Azure no tiene configurados los permisos de control de acceso basado en rol de Azure (Azure RBAC) necesarios en la instancia de Azure Digital Twins. Para acceder a los datos de la instancia, debe tener el rol **Lector de datos de Azure Digital Twins** o **Propietario de datos de Azure Digital Twins** en la instancia que está intentando leer o administrar, respectivamente. 

Para obtener más información sobre la seguridad y los roles de Azure Digital Twins, consulte [Conceptos: Soluciones de seguridad para Azure Digital Twins](concepts-security.md).

## <a name="solutions"></a>Soluciones

### <a name="solution-1"></a>Solución 1

Compruebe que el usuario de Azure tiene el rol **Lector de datos de Azure Digital Twins** en la instancia de Azure Digital Twins si solo está intentando leer sus datos, o el rol **Propietario de datos de Azure Digital Twins** si está intentando administrar los datos.

Tenga en cuenta que este rol es diferente del...
* nombre anterior de este rol durante la versión preliminar, *Propietario de Azure Digital Twins (versión preliminar)* (el rol es el mismo, pero el nombre ha cambiado)
* rol de *propietario* de toda la suscripción de Azure *Propietario de datos de Azure Digital Twins* es un rol dentro de Azure Digital Twins y tiene el ámbito de esa instancia individual de Azure Digital Twins.
* rol de *propietario* en Azure Digital Twins. Son dos roles de administración distintos de Azure Digital Twins; *Propietario de datos de Azure Digital Twins* es el rol que debe usar para la administración.

 Si no tiene este rol, establézcalo para solucionar el problema.

#### <a name="check-current-setup"></a>Comprobación de la configuración actual

[!INCLUDE [digital-twins-setup-verify-role-assignment.md](../../includes/digital-twins-setup-verify-role-assignment.md)]

#### <a name="fix-issues"></a>Corrección de problemas 

Si no tiene esta asignación de roles, alguien con el rol de propietario en la **suscripción a Azure** debe ejecutar el siguiente comando para proporcionar al usuario de Azure el rol adecuado en la **instancia de Azure Digital Twins**. 

Si es propietario de la suscripción, puede ejecutar este comando por su cuenta. Si no, póngase en contacto con un propietario para que ejecute este comando en su nombre. El nombre del rol es **Propietario de datos de Azure Digital Twins** para el acceso de edición o **Lector de datos de Azure Digital Twins** para el acceso de lectura.

```azurecli-interactive
az dt role-assignment create --dt-name <your-Azure-Digital-Twins-instance> --assignee "<your-Azure-AD-email>" --role "<role-name>"
```

Para más información sobre este requisito de rol y el proceso de asignación, consulte la [sección de Configuración de los permisos de acceso del usuario en el artículo](how-to-set-up-instance-CLI.md#set-up-user-access-permissions) de *Procedimiento de configuración de una instancia y de la autenticación (mediante la CLI o el portal)* .

## <a name="next-steps"></a>Pasos siguientes

Lea los pasos de configuración para crear y autenticar una nueva instancia de Azure Digital Twins:
* [Procedimiento: Configuración de una instancia y de la autenticación (CLI)](how-to-set-up-instance-cli.md)

Obtenga más información sobre la seguridad y los permisos de Azure Digital Twins:
* [Conceptos: Seguridad para las soluciones de Azure Digital Twins](concepts-security.md)
