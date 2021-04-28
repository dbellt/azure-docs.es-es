---
title: 'Portal para desarrolladores: Preguntas más frecuentes'
titleSuffix: Azure API Management
description: Preguntas frecuentes sobre el portal para desarrolladores en API Management. El portal para desarrolladores es un sitio web personalizable, donde los consumidores de API pueden explorarlas.
services: api-management
documentationcenter: API Management
author: mikebudzynski
ms.service: api-management
ms.topic: article
ms.date: 04/15/2021
ms.author: apimpm
ms.openlocfilehash: b4413bc53cf5c86c311d049046db790582737ca4
ms.sourcegitcommit: 425420fe14cf5265d3e7ff31d596be62542837fb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107741320"
---
# <a name="api-management-developer-portal---frequently-asked-questions"></a>Portal para desarrolladores de API Management: Preguntas más frecuentes

## <a name="what-if-i-need-functionality-that-isnt-supported-in-the-portal"></a>¿Qué ocurre si necesito una funcionalidad que no se admite en el portal?

Puede abrir una solicitud de característica en el [repositorio de GitHub](https://github.com/Azure/api-management-developer-portal) o [implementar por sí mismo la funcionalidad que falta](developer-portal-implement-widgets.md). Obtenga más información sobre la [extensibilidad](api-management-howto-developer-portal.md#managed-vs-self-hosted) del el portal para desarrolladores.


## <a name="can-i-have-multiple-developer-portals-in-one-api-management-service"></a>¿Puedo tener varios portales para desarrolladores en un servicio API Management?

Puede tener un portal administrado y varios portales autohospedados. El contenido de todos los portales se almacena en el mismo servicio API Management, por lo que serán idénticos. Si quiere diferenciar la apariencia y la funcionalidad de los portales, puede autohospedarlos con sus propios widgets personalizados, que personalizan dinámicamente las páginas en runtime, por ejemplo, en función de la dirección URL.

## <a name="does-the-portal-support-azure-resource-manager-templates-andor-is-it-compatible-with-api-management-devops-resource-kit"></a>¿Admite el portal plantillas de Azure Resource Manager o es compatible con el kit de recursos de DevOps de API Management?

No.

## <a name="is-the-portals-content-saved-with-the-backuprestore-functionality-in-api-management"></a>¿Se guarda el contenido del portal con la funcionalidad de copia de seguridad y restauración en API Management?

No.

## <a name="do-i-need-to-enable-additional-vnet-connectivity-for-the-managed-portal-dependencies"></a>¿Debo habilitar una conectividad de red virtual adicional para las dependencias del portal administrado?

En la mayoría de los casos, no.

Si el servicio API Management se encuentra en una red virtual interna, solo se puede acceder al portal para desarrolladores desde dentro de la red. El nombre de host del punto de conexión de administración debe resolverse en la dirección VIP interna del servicio desde la máquina que se usa para acceder a la interfaz administrativa del portal. Asegúrese de que el punto de conexión de administración está registrado en el DNS. En caso de que haya una configuración incorrecta, verá un error: `Unable to start the portal. See if settings are specified correctly in the configuration (...)`.

Si el servicio API Management se encuentra en una red virtual interna y accede a él a través de Application Gateway desde Internet, no olvide habilitar la conectividad con el portal para desarrolladores y los puntos de conexión de administración de API Management. Es posible que tenga que deshabilitar las reglas de Web Application Firewall. Consulte [este artículo de la documentación](api-management-howto-integrate-internal-vnet-appgateway.md) para obtener más detalles.

## <a name="i-assigned-a-custom-api-management-domain-and-the-published-portal-doesnt-work"></a>He asignado un dominio de API Management personalizado y el portal publicado no funciona

Después de actualizar el dominio, debe [volver a publicar el portal](api-management-howto-developer-portal-customize.md#publish) para que los cambios surtan efecto.

## <a name="i-added-an-identity-provider-and-i-cant-see-it-in-the-portal"></a>He agregado un proveedor de identidades y no lo veo en el portal

Después de configurar un proveedor de identidades (por ejemplo, Azure AD o Azure AD B2C), debe [volver a publicar el portal](api-management-howto-developer-portal-customize.md#publish) para que los cambios surtan efecto. Asegúrese de que las páginas del portal para desarrolladores incluyen el widget con los botones de OAuth.

## <a name="i-set-up-delegation-and-the-portal-doesnt-use-it"></a>He configurado la delegación y el portal no la usa

Después de configurar la delegación, debe [volver a publicar el portal](api-management-howto-developer-portal-customize.md#publish) para que los cambios surtan efecto.

## <a name="my-other-api-management-configuration-changes-havent-been-propagated-in-the-developer-portal"></a>Los demás cambios de configuración de API Management no se han propagado en el portal para desarrolladores

La mayoría de los cambios de configuración (por ejemplo, la red virtual, el inicio de sesión y los términos del producto) requieren [volver a publicar el portal](api-management-howto-developer-portal-customize.md#publish).

## <a name="im-getting-a-cors-error-when-using-the-interactive-console"></a><a name="cors"></a> Aparece un error de CORS cuando uso la consola interactiva

La consola interactiva realiza una solicitud de API del lado cliente desde el explorador. Resuelva el problema de CORS incorporando [una directiva de CORS](api-management-cross-domain-policies.md#CORS) en las API.

Puede comprobar el estado de la directiva de CORS en la sección **Portal overview** (Información general del portal) del servicio API Management en Azure Portal. Un cuadro de advertencia indica que falta una directiva o que está mal configurada.

> [!NOTE]
> 
> Solo se ejecuta una directiva de CORS. Si especificó varias directivas de CORS (por ejemplo, en el nivel de API y en el nivel de todas las API), es posible que la consola interactiva no funcione según lo esperado.

![Captura de pantalla que muestra dónde puede comprobar el estado de la directiva de CORS.](media/developer-portal-faq/cors-azure-portal.png)

Aplique automáticamente la directiva de CORS haciendo clic en el botón **Habilitar CORS**.

También puede habilitar CORS manualmente.

1. Haga clic en el vínculo **Manually apply it on the global level** (Aplicarlo manualmente en el nivel global) para ver el código generado de la directiva.
2. Vaya a **Todas las API** en la sección **API** del servicio API Management en Azure Portal.
3. Haga clic en el icono **</>** en la sección **Procesamiento de entrada**.
4. Inserte la directiva en la sección **<inbound>** del archivo XML. Asegúrese de que el valor de **<origin>** coincide con el dominio del portal para desarrolladores.

> [!NOTE]
> 
> Si aplica la directiva de CORS en el ámbito del producto, en lugar del ámbito de las API, y la API usa la autenticación de clave de suscripción a través de un encabezado, la consola no funcionará.
>
> El explorador emite automáticamente una solicitud HTTP `OPTIONS`, que no contiene un encabezado con la clave de suscripción. Como falta la clave de suscripción, API Management no puede asociar la llamada a `OPTIONS` con ningún producto, por lo que no puede aplicar la directiva de CORS.
>
> Como solución alternativa, puede pasar la clave de suscripción en un parámetro de consulta.

## <a name="what-is-the-cors-proxy-feature-and-when-should-i-use-it"></a>¿Qué es la característica de proxy de CORS y cuándo debo usarla?

Seleccione la opción **Use CORS proxy** (Usar proxy de CORS) en la configuración del widget de detalles de la operación de API para enrutar las llamadas API de la consola interactiva a través del back-end del portal en el servicio API Management. En esta configuración, ya no es necesario aplicar una directiva de CORS para las API, y no se requiere conectividad con el punto de conexión de puerta de enlace desde la máquina local. Si las API se exponen a través de una puerta de enlace autohospedada o si el servicio está en una red virtual, se requiere la conectividad desde el servicio back-end de API Management a la puerta de enlace. Si usa el portal autohospedado, especifique el punto de conexión de back-end del portal mediante la opción `backendUrl` en los archivos de configuración. De lo contrario, el portal autohospedado no estará al tanto de la ubicación del servicio back-end.

## <a name="what-permissions-do-i-need-to-edit-the-developer-portal"></a>¿Qué permisos necesito para editar el portal para desarrolladores?

Si ve el error `Oops. Something went wrong. Please try again later.` al abrir el portal en el modo administrativo, es posible que no tenga los permisos necesarios (RBAC de Azure).

Los portales heredados requieren el permiso `Microsoft.ApiManagement/service/getssotoken/action` en el ámbito de servicio (`/subscriptions/<subscription-id>/resourceGroups/<resource-group>/providers/Microsoft.ApiManagement/service/<apim-service-name>`) para permitir que el administrador de usuarios tenga acceso a los portales. El nuevo portal requiere el permiso `Microsoft.ApiManagement/service/users/token/action` en el ámbito `/subscriptions/<subscription-id>/resourceGroups/<resource-group>/providers/Microsoft.ApiManagement/service/<apim-service-name>/users/1`.

Puede usar el siguiente script de PowerShell para crear un rol con el permiso necesario. Recuerde cambiar el parámetro `<subscription-id>`. 

```powershell
#New Portals Admin Role 
Import-Module Az 
Connect-AzAccount 
$contributorRole = Get-AzRoleDefinition "API Management Service Contributor" 
$customRole = $contributorRole 
$customRole.Id = $null
$customRole.Name = "APIM New Portal Admin" 
$customRole.Description = "This role gives the user ability to log in to the new Developer portal as administrator" 
$customRole.Actions = "Microsoft.ApiManagement/service/users/token/action" 
$customRole.IsCustom = $true 
$customRole.AssignableScopes.Clear() 
$customRole.AssignableScopes.Add('/subscriptions/<subscription-id>') 
New-AzRoleDefinition -Role $customRole 
```
 
Una vez creado el rol, se puede conceder a cualquier usuario de la sección **Control de acceso (IAM)** de Azure Portal. Al asignar este rol a un usuario, se asignará el permiso en el ámbito del servicio. El usuario podrá generar tokens de SAS en nombre de *cualquier* usuario en el servicio. Como mínimo, este rol debe asignarse al administrador del servicio. El siguiente comando de PowerShell muestra cómo asignar el rol a un usuario `user1` en el ámbito más bajo para evitar la concesión de permisos innecesarios al usuario: 

```powershell
New-AzRoleAssignment -SignInName "user1@contoso.com" -RoleDefinitionName "APIM New Portal Admin" -Scope "/subscriptions/<subscription-id>/resourceGroups/<resource-group>/providers/Microsoft.ApiManagement/service/<apim-service-name>/users/1" 
```

Una vez concedidos los permisos a un usuario, este debe cerrar sesión e iniciar sesión de nuevo en Azure Portal para que los nuevos permisos surtan efecto.

## <a name="im-seeing-the-unable-to-start-the-portal-see-if-settings-are-specified-correctly--error"></a>Veo el error `Unable to start the portal. See if settings are specified correctly (...)`

Este error se muestra cuando se produce un error en una llamada `GET` a `https://<management-endpoint-hostname>/subscriptions/xxx/resourceGroups/xxx/providers/Microsoft.ApiManagement/service/xxx/contentTypes/document/contentItems/configuration?api-version=2018-06-01-preview`. La interfaz administrativa del portal emite la llamada desde el explorador.

Si el servicio API Management se encuentra en una red virtual, consulte la [pregunta de conectividad de la red virtual](#do-i-need-to-enable-additional-vnet-connectivity-for-the-managed-portal-dependencies).

El error de llamada también puede deberse a un certificado TLS/SSL, que se asigna a un dominio personalizado y no es de confianza para el explorador. Como medida de mitigación, puede quitar el dominio personalizado del punto de conexión de administración: API Management revertirá al punto de conexión predeterminado con un certificado de confianza.

## <a name="whats-the-browser-support-for-the-portal"></a>¿Cuál es la compatibilidad del explorador con el portal?

| Browser                     | Compatible       |
|-----------------------------|-----------------|
| Apple Safari                | Sí<sup>1</sup> |
| Google Chrome               | Sí<sup>1</sup> |
| Microsoft Edge              | Sí<sup>1</sup> |
| Microsoft Internet Explorer | No              |
| Mozilla Firefox             | Sí<sup>1</sup> |

 <small><sup>1</sup> Se admite en las dos últimas versiones de producción.</small>

## <a name="local-development-of-my-self-hosted-portal-is-no-longer-working"></a>El desarrollo local de mi portal autohospedado ya no funciona

Si la versión local del portal para desarrolladores no puede guardar ni recuperar información de la cuenta de almacenamiento ni de la instancia de API Management, es posible que los tokens de SAS hayan expirado. Para corregirlo, genere nuevos tokens. Para obtener instrucciones, consulte el tutorial para [autohospedar el portal para desarrolladores](developer-portal-self-host.md#step-2-configure-json-files-static-website-and-cors-settings).

## <a name="how-can-i-remove-the-developer-portal-content-provisioned-to-my-api-management-service"></a>¿Cómo puedo quitar el contenido del portal para desarrolladores aprovisionado en mi servicio API Management?

Proporcione los parámetros necesarios en el script `scripts.v3/cleanup.bat` en el [repositorio de GitHub](https://github.com/Azure/api-management-developer-portal) del portal para desarrolladores y ejecute el script.

```sh
cd scripts.v3
.\cleanup.bat
cd ..
```

## <a name="how-do-i-enable-single-sign-on-sso-authentication-to-self-hosted-developer-portal"></a>¿Cómo habilito la autenticación de inicio de sesión único (SSO) en el portal para desarrolladores autohospedado?

Entre otros métodos de autenticación, el portal para desarrolladores admite el inicio de sesión único (SSO). Para autenticarse con este método, tiene que realizar una llamada a `/signin-sso` con el token en el parámetro de consulta:

```html
https://contoso.com/signin-sso?token=[user-specific token]
```
### <a name="generate-user-tokens"></a>Generación de tokens de usuario
Puede generar *tokens específicos del usuario* (incluidos tokens de administrador) mediante la operación [Obtención del token de acceso compartido](/rest/api/apimanagement/2019-12-01/user/getsharedaccesstoken) de la [API REST de API Management](/rest/api/apimanagement/apimanagementrest/api-management-rest).

> [!NOTE]
> El token debe tener una codificación URL.

## <a name="next-steps"></a>Pasos siguientes

Más información acerca del nuevo portal para desarrolladores:

- [Acceso a la versión administrada del portal para desarrolladores y su personalización](api-management-howto-developer-portal-customize.md)
- [Configuración de la versión autohospedada del portal](developer-portal-self-host.md)
- [Implementación de su propio widget](developer-portal-implement-widgets.md)

Examine otros recursos:

- [Repositorio de GitHub con el código fuente](https://github.com/Azure/api-management-developer-portal)

