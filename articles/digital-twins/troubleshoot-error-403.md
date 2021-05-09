---
title: 'Error en la solicitud de Azure Digital Twins con el estado: 403 (Prohibido)'
description: 'Causas y soluciones de "Service request failed. Estado: 403 (Forbidden) (Error en la solicitud del servicio. Estado: 403 [prohibido]) en Azure Digital Twins.'
ms.service: digital-twins
author: baanders
ms.author: baanders
ms.topic: troubleshooting
ms.date: 7/20/2020
ms.openlocfilehash: b49ded912037c8935b67e64d4e4bb09402576bc7
ms.sourcegitcommit: a5dd9799fa93c175b4644c9fe1509e9f97506cc6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108203638"
---
# <a name="service-request-failed-status-403-forbidden"></a>Error en la solicitud del servicio. Estado: 403 (Prohibido)

En este artículo se describen las causas y los pasos de resolución al recibir un error 403 de las solicitudes de servicio a Azure Digital Twins. 

## <a name="symptoms"></a>Síntomas

Este error puede producirse en muchos tipos de solicitudes de servicio que requieren autenticación. El efecto es que se produce un error en la solicitud de API y se devuelve un estado de error `403 (Forbidden)`.

## <a name="causes"></a>Causas

### <a name="cause-1"></a>Causa 1

A menudo, este error indica que los permisos de control de acceso basado en rol (RBAC de Azure) para el servicio no están configurados correctamente. Muchas acciones para una instancia de Azure Digital Twins requieren que tenga el rol *Propietario de datos de Azure Digital Twins* **en la instancia que intenta administrar**. 

### <a name="cause-2"></a>Causa 2

Si usa una aplicación cliente para comunicarse con Azure Digital Twins que se autentique con el [registro de aplicación](how-to-create-app-registration.md), este error puede producirse porque el registro de aplicación no tiene permisos configurados para el servicio Azure Digital Twins.

Este registro de aplicación debe tener configurados los permisos de acceso a las API de Azure Digital Twins. Después, cuando la aplicación cliente se autentique durante el registro de la aplicación, se le concederán los permisos que se hayan configurado en este.

## <a name="solutions"></a>Soluciones

### <a name="solution-1"></a>Solución 1

La primera solución es comprobar que el usuario de Azure tenga el rol _**Propietario de datos de Azure Digital Twins**_ en la instancia que intenta administrar. Si no tiene este rol, establézcalo.

Tenga en cuenta que este rol es diferente del...
* nombre anterior de este rol durante la versión preliminar, *Propietario de Azure Digital Twins (versión preliminar)* (el rol es el mismo, pero el nombre ha cambiado)
* rol de *propietario* de toda la suscripción de Azure *Propietario de datos de Azure Digital Twins* es un rol dentro de Azure Digital Twins y tiene el ámbito de esa instancia individual de Azure Digital Twins.
* rol de *propietario* en Azure Digital Twins. Son dos roles de administración distintos de Azure Digital Twins; *Propietario de datos de Azure Digital Twins* es el rol que debe usar para la administración.

#### <a name="check-current-setup"></a>Comprobación de la configuración actual

[!INCLUDE [digital-twins-setup-verify-role-assignment.md](../../includes/digital-twins-setup-verify-role-assignment.md)]

#### <a name="fix-issues"></a>Corrección de problemas 

Si no tiene esta asignación de roles, alguien con el rol de propietario en la **suscripción de Azure** debe ejecutar el siguiente comando para proporcionar al usuario de Azure el rol *Propietario de datos de Azure Digital Twins* en la **instancia de Azure Digital Twins**. 

Si es propietario de la suscripción, puede ejecutar este comando por su cuenta. Si no, póngase en contacto con un propietario para que ejecute este comando en su nombre.

```azurecli-interactive
az dt role-assignment create --dt-name <your-Azure-Digital-Twins-instance> --assignee "<your-Azure-AD-email>" --role "Azure Digital Twins Data Owner"
```

Para más información sobre este requisito de rol y el proceso de asignación, consulte la [sección de Configuración de los permisos de acceso del usuario en el artículo](how-to-set-up-instance-CLI.md#set-up-user-access-permissions) de *Procedimiento de configuración de una instancia y de la autenticación (mediante la CLI o el portal)* .

Si ya tiene esta asignación de roles *y* usa un registro de aplicación de Azure AD para autenticar una aplicación cliente, puede continuar con la solución siguiente si esta no resolvió el problema 403.

### <a name="solution-2"></a>Solución 2

Si usa un registro de aplicación de Azure AD para autenticar una aplicación cliente, la segunda solución posible es comprobar que el registro de aplicación tenga permisos configurados para el servicio Azure Digital Twins. Si no están configurados, hágalo.

#### <a name="check-current-setup"></a>Comprobación de la configuración actual

Para comprobar si los permisos se han configurado correctamente, vaya a la [página de información general del registro de aplicaciones de Azure AD](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/RegisteredApps) en Azure Portal. Para acceder a esta página, escriba *Registros de aplicaciones* en la barra de búsqueda del portal.

Cambie a la pestaña *Todas las aplicaciones* para ver todos los registros de aplicaciones que se han creado en su suscripción.

Verá que aparece en la lista el registro de aplicaciones que acaba de crear. Selecciónelo para abrir sus detalles.

:::image type="content" source="media/troubleshoot-error-403/app-registrations.png" alt-text="Página Registros de aplicaciones en Azure Portal":::

En primer lugar, compruebe que la configuración de permisos de Azure Digital Twins se estableció correctamente en el registro. Para ello, seleccione *Manifiesto* en la barra de menús para ver el código del manifiesto del registro de la aplicación. Desplácese hasta la parte inferior de la ventana de código y busque estos campos en `requiredResourceAccess`. Los valores deben coincidir con los de la siguiente captura de pantalla:

:::image type="content" source="media/troubleshoot-error-403/verify-manifest.png" alt-text="Vista del portal del manifiesto para el registro de aplicaciones de Azure AD":::

A continuación, seleccione *Permisos de API* en la barra de menús para comprobar que este registro de aplicaciones contiene permisos de lectura y escritura para Azure Digital Twins. Debería ver una entrada como la siguiente:

:::image type="content" source="media/troubleshoot-error-403/verify-api-permissions.png" alt-text="Vista del portal de los permisos de API para el registro de la aplicación de Azure AD que muestra &quot;Acceso de lectura y escritura&quot; para Azure Digital Twins":::

#### <a name="fix-issues"></a>Corrección de problemas

Si alguno de estos problemas aparece de forma diferente a lo descrito, siga las instrucciones sobre cómo configurar el registro de una aplicación en [Procedimiento: Creación de un registro de aplicación](how-to-create-app-registration.md).

## <a name="next-steps"></a>Pasos siguientes

Lea los pasos de configuración para crear y autenticar una nueva instancia de Azure Digital Twins:
* [Procedimiento: Configuración de una instancia y de la autenticación (CLI)](how-to-set-up-instance-cli.md)

Obtenga más información sobre la seguridad y los permisos de Azure Digital Twins:
* [Conceptos: Seguridad para las soluciones de Azure Digital Twins](concepts-security.md)
