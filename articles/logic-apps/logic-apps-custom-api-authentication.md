---
title: Agregar autenticación para proteger las llamadas a las API personalizadas
description: Configuración de la autenticación para mejorar la seguridad de las llamadas a las API personalizadas desde Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: klam, logicappspm
ms.topic: article
ms.date: 09/22/2017
ms.openlocfilehash: ab3b051a47a8ef2150848c73e612d366575560ce
ms.sourcegitcommit: 1b19b8d303b3abe4d4d08bfde0fee441159771e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2021
ms.locfileid: "109753640"
---
# <a name="increase-security-for-calls-to-custom-apis-from-azure-logic-apps"></a>Mejora de la seguridad de las llamadas a las API personalizadas desde Azure Logic Apps

Para mejorar la seguridad de las llamadas a sus API, puede configurar la autenticación de Azure Active Directory (Azure AD) en Azure Portal para no tener que actualizar el código.
También puede requerir y aplicar la autenticación a través del código de la API.

## <a name="authentication-options-for-your-api"></a>Opciones de autenticación para la API

Puede mejorar la seguridad de las llamadas a la API personalizada de las siguientes maneras:

* [Ningún cambio de código](#no-code): proteja su API con [Azure Active Directory (Azure AD)](../active-directory/fundamentals/active-directory-whatis.md) por medio de Azure Portal, de forma que no tenga que actualizar el código ni volver a implementar la API.

  > [!NOTE]
  > De forma predeterminada, la autenticación de Azure AD que se activa en Azure Portal no lleva a cabo una autorización específica.
  > Por ejemplo, esta autenticación bloquea la API a un solo inquilino concreto, no a una aplicación o un usuario específicos.

* [Actualice el código de la API](#update-code): proteja su API aplicando la [autenticación de certificado](#certificate), la [autenticación básica](#basic) o la [autenticación de Azure AD](#azure-ad-code) mediante código.

<a name="no-code"></a>

### <a name="authenticate-calls-to-your-api-without-changing-code"></a>Autenticación de las llamadas a la API sin cambiar el código

Estos son los pasos generales para este método:

1. Cree dos identidades de aplicación de Azure Active Directory (Azure AD): una para la aplicación lógica y otra para la aplicación web (o aplicación de API).

2. Para autenticar llamadas a la API, use las credenciales (identificador y secreto de cliente) de la entidad de servicio asociada con la identidad de aplicación de Azure AD para su aplicación lógica.

3. Incluya los identificadores de aplicación en la definición de la aplicación lógica.

#### <a name="part-1-create-an-azure-ad-application-identity-for-your-logic-app"></a>Parte 1: Creación de una identidad de aplicación de Azure AD para la aplicación lógica

La aplicación lógica usa esta identidad de aplicación de Azure AD para autenticarse en Azure AD.
Solo tiene que establecer esta identidad una vez para el directorio.
Por ejemplo, puede usar la misma identidad para todas las aplicaciones lógicas, aunque puede crear identidades únicas para cada una.
Puede configurar estas identidades en Azure Portal o usar [PowerShell](#powershell).

**Creación de la identidad de aplicación para la aplicación lógica en Azure Portal**

1. En [Azure Portal](https://portal.azure.com "https://portal.azure.com"), elija **Azure Active Directory**.

2. Confirme que se encuentra en el mismo directorio que la aplicación web o de API.

   > [!TIP]
   > Para cambiar de directorio, seleccione el perfil y seleccione otro.
   > O bien, elija **Información general** > **Cambiar directorio**.

3. En el menú de directorio, en **Administrar**, elija **Registros de aplicaciones** > **Nuevo registro de aplicaciones**.

   > [!TIP]
   > De forma predeterminada, la lista de registros de aplicaciones muestra todos los presentes en el directorio.
   > Para ver solo sus registros de aplicaciones, junto al cuadro de búsqueda, seleccione **Mis aplicaciones**.

   ![Crear registro de aplicaciones](./media/logic-apps-custom-api-authentication/new-app-registration-azure-portal.png)

4. Asigne un nombre a la identidad de aplicación, deje **Tipo de aplicación** establecido en **Aplicación web o API**, proporcione una cadena única con formato de dominio para **Dirección URL de inicio de sesión** y elija **Crear**.

   ![Proporcionar un nombre y una dirección URL de inicio de sesión para la identidad de aplicación](./media/logic-apps-custom-api-authentication/logic-app-identity-azure-portal.png)

   La identidad de aplicación que ha creado para la aplicación lógica aparece ahora en la lista de registros de aplicaciones.

   ![Identidad de aplicación para la aplicación lógica](./media/logic-apps-custom-api-authentication/logic-app-identity-created.png)

5. En la lista de registros de aplicaciones, seleccione la nueva identidad de aplicación.
Copie y guarde el valor de **Id. de la aplicación** para usarlo como "identificador de cliente" de la aplicación lógica en la parte 3.

   ![Copiar y guardar el identificador de aplicación para la aplicación lógica](./media/logic-apps-custom-api-authentication/logic-app-application-id.png)

6. Si la configuración de la identidad de aplicación no está visible, elija **Configuración** o **Toda la configuración**.

7. En **Acceso de API**, elija **Claves**. En **Descripción**, proporcione un nombre para la clave. En **Expira**, seleccione una duración para la clave.

   La clave que va a crear actúa como "secreto" o contraseña de la identidad de aplicación para la aplicación lógica.

   ![Crear clave para la identidad de aplicación lógica](./media/logic-apps-custom-api-authentication/create-logic-app-identity-key-secret-password.png)

8. En la barra de herramientas, elija **Guardar**. En **Valor**, ahora aparece su clave.
**Asegúrese de copiar y guardar la clave** para usarla más adelante, ya que está oculta una vez que se sale de la hoja **Claves**.

   Cuando configure la aplicación lógica en la parte 3, especifique esta clave como "secreto" o contraseña.

   ![Copiar y guardar la clave para más tarde](./media/logic-apps-custom-api-authentication/logic-app-copy-key-secret-password.png)

<a name="powershell"></a>

**Creación de la identidad de aplicación para la aplicación lógica en PowerShell**

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Puede realizar esta tarea mediante Azure Resource Manager con PowerShell.
En PowerShell, ejecute estos comandos:

1. `Add-AzAccount`

1. `$SecurePassword = Read-Host -AsSecureString`

1. Escriba una contraseña y presione ENTRAR.

1. `New-AzADApplication -DisplayName "MyLogicAppID" -HomePage "http://mydomain.tld" -IdentifierUris "http://mydomain.tld" -Password $SecurePassword`

1. Asegúrese de copiar el valor de **Identificador de inquilino** (GUID para el inquilino de Azure AD), el de **Id. de la aplicación** y la contraseña que usó.

Para más información, consulte cómo [crear una entidad de servicio con PowerShell para acceder a recursos](../active-directory/develop/howto-authenticate-service-principal-powershell.md).

#### <a name="part-2-create-an-azure-ad-application-identity-for-your-web-app-or-api-app"></a>Parte 2: Creación de una identidad de aplicación de Azure AD para la aplicación web o de API

Si ya se ha implementado la aplicación web o de API, puede activar la autenticación y crear la identidad de aplicación en Azure Portal. En caso contrario, puede [activar la autenticación cuando implemente con una plantilla de Azure Resource Manager](#authen-deploy).

**Creación de la identidad de aplicación y activación de la autenticación en Azure Portal para aplicaciones implementadas**

1. En [Azure Portal](https://portal.azure.com "https://portal.azure.com"), busque y seleccione la aplicación web o de API.

2. Haga clic en **Configuración** y elija **Autenticación/autorización**.
En **Autenticación de App Service**, elija **Activar** la autenticación.
En **Proveedores de autenticación**, elija **Azure Active Directory**.

   ![Activar la autenticación](./media/logic-apps-custom-api-authentication/custom-web-api-app-authentication.png)

3. Ahora cree una identidad de aplicación para la aplicación web o de API como se muestra aquí.
En la página **Configuración de Azure Active Directory**, establezca **Modo de administración** en **Rápido**. Elija **Crear nueva aplicación de AD**.
Asigne un nombre a la identidad de aplicación y elija **Aceptar**.

   ![Crear una identidad de aplicación para la aplicación web o de API](./media/logic-apps-custom-api-authentication/custom-api-application-identity.png)

4. En la página **Autenticación/autorización**, elija **Guardar**.

Ahora debe buscar el identificador de cliente y el de inquilino para la identidad de aplicación que esté asociada con la aplicación web o de API. Ha usado estos identificadores en la parte 3.
Por tanto, continúe con estos pasos para Azure Portal.

**Búsqueda de los identificadores de cliente y de inquilino de la identidad de aplicación para su aplicación web o de API en Azure Portal**

1. En **Proveedores de autenticación**, elija **Azure Active Directory**.

   ![Elegir "Azure Active Directory"](./media/logic-apps-custom-api-authentication/custom-api-app-identity-client-id-tenant-id.png)

2. En la página **Configuración de Azure Active Directory**, establezca **Modo de administración** en **Avanzado**.

3. Copie el valor de **Id. de cliente** y guarde ese GUID para usarlo en la parte 3.

   > [!TIP]
   > Si **Id. de cliente** y **URL del emisor** no aparecen, pruebe a actualizar Azure Portal y repita el paso 1.

4. En **URL del emisor**, copie y guarde solo el GUID para la parte 3.
También puede usar este GUID en la plantilla de implementación de su aplicación web o de API, si es necesario.

   Este GUID es para su inquilino específico ("Identificador de inquilino") y debería aparecer en esta dirección URL: `https://sts.windows.net/{GUID}`

5. Sin guardar los cambios, cierre la página **Configuración de Azure Active Directory**.

<a name="authen-deploy"></a>

**Activación de la autenticación cuando implemente con una plantilla de Azure Resource Manager**

Todavía debe crear para su aplicación web o de API una identidad de aplicación de Azure AD que difiera de la identidad de aplicación para la aplicación lógica. Para crear la identidad de aplicación, siga los pasos anteriores en la parte 2 para Azure Portal.

También puede seguir los pasos en la parte 1, pero asegúrese de usar `https://{URL}` para la aplicación web o de API en **Dirección URL de inicio de sesión** y **URI de id. de aplicación**.
En estos pasos, tiene que guardar el identificador de cliente y el de inquilino para usarlos en la plantilla de implementación de la aplicación y también en la parte 3.

> [!NOTE]
> Cuando cree la identidad de aplicación de Azure AD para su aplicación web o de API, debe usar Azure Portal, no PowerShell.
> El cmdlet de PowerShell no configura los permisos necesarios para iniciar la sesión de los usuarios en un sitio web.

Una vez que obtenga los identificadores de cliente y de inquilino, inclúyalos como un subrecurso de la aplicación web o de API en la plantilla de implementación:

``` json
"resources": [
   {
      "apiVersion": "2015-08-01",
      "name": "web",
      "type": "config",
      "dependsOn": ["[concat('Microsoft.Web/sites/','parameters('webAppName'))]"],
      "properties": {
         "siteAuthEnabled": true,
         "siteAuthSettings": {
            "clientId": "<client-ID>",
            "issuer": "https://sts.windows.net/<tenant-ID>/"
         }
      }
   }
]
```

Para implementar automáticamente una aplicación web en blanco y una aplicación lógica juntas con la autenticación de Azure Active Directory, [vea la plantilla completa aquí](https://azure.microsoft.com/resources/templates/quickstarts/microsoft.logic/logic-app-custom-api/) o haga clic en **Implementar en Azure** aquí:

[![Implementación en Azure](media/logic-apps-custom-api-authentication/deploybutton.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fquickstarts%2Fmicrosoft.logic%2Flogic-app-custom-api%2Fazuredeploy.json)

#### <a name="part-3-populate-the-authorization-section-in-your-logic-app"></a>Parte 3: Relleno de la sección de autorización en la aplicación lógica

La plantilla anterior ya tiene esta sección de autorización configurada pero, si va a crear la aplicación lógica directamente, debe incluir la sección de autorización completa.

Abra la definición de aplicación lógica en la vista de código, vaya a la definición de la acción **HTTP**, busque la sección **Autorización** e incluya estas propiedades:

```json
{
   "tenant": "<tenant-ID>",
   "audience": "<client-ID-from-Part-2-web-app-or-API app>",
   "clientId": "<client-ID-from-Part-1-logic-app>",
   "secret": "<key-from-Part-1-logic-app>",
   "type": "ActiveDirectoryOAuth"
}
```

| Propiedad | Obligatorio | Descripción |
| -------- | -------- | ----------- |
| tenant | Sí | El GUID para el inquilino de Azure AD |
| audience | Sí | El GUID para el recurso de destino al que desea acceder, que es el identificador de cliente de la identidad de aplicación para su aplicación web o de API |
| clientId | Sí | El GUID para el cliente que solicita acceso, que es el identificador de cliente de la identidad de aplicación para la aplicación lógica |
| secret | Sí | La clave o contraseña de la identidad de aplicación para el cliente que solicita el token de acceso |
| type | Sí | El tipo de autenticación. En autenticación ActiveDirectoryOAuth, el valor es `ActiveDirectoryOAuth`. |
||||

Por ejemplo:

``` json
{
   "actions": {
      "HTTP": {
         "inputs": {
            "method": "POST",
            "uri": "https://your-api-azurewebsites.net/api/your-method",
            "authentication": {
               "tenant": "tenant-ID",
               "audience": "client-ID-from-azure-ad-app-for-web-app-or-api-app",
               "clientId": "client-ID-from-azure-ad-app-for-logic-app",
               "secret": "key-from-azure-ad-app-for-logic-app",
               "type": "ActiveDirectoryOAuth"
            }
         }
      }
   }
}
```

<a name="update-code"></a>

### <a name="secure-api-calls-through-code"></a>Protección de las llamadas API mediante código

<a name="certificate"></a>

#### <a name="certificate-authentication"></a>Autenticación de certificado

Para validar las solicitudes entrantes desde la aplicación lógica hacia la aplicación web o de API, puede usar certificados de cliente. Para configurar su código, aprenda a [configurar la autenticación mutua de TLS](../app-service/app-service-web-configure-tls-mutual-auth.md).

En la sección **Autorización**, incluya estas propiedades:

```json
{
   "type": "ClientCertificate",
   "password": "<password>",
   "pfx": "<long-pfx-key>"
}
```

| Propiedad | Obligatorio | Descripción |
| -------- | -------- | ----------- |
| `type` | Sí | El tipo de autenticación. Para los certificados de cliente TLS/SSL, el valor debe ser `ClientCertificate`. |
| `password` | No | La contraseña para acceder al certificado de cliente (archivo PFX) |
| `pfx` | Sí | Contenido con codificación base64 del certificado del cliente (archivo PFX) |
||||

<a name="basic"></a>

#### <a name="basic-authentication"></a>Autenticación básica

Para validar solicitudes entrantes desde la aplicación lógica hacia la aplicación web o de API, puede usar la autenticación básica, por ejemplo, un nombre de usuario y una contraseña.
La autenticación básica es un patrón habitual, y puede usarla en cualquier lenguaje utilizado para compilar la aplicación web o de API.

En la sección **Autorización**, incluya estas propiedades:

```json
{
   "type": "Basic",
   "username": "<username>",
   "password": "<password>"
}
```

| Propiedad | Obligatorio | Descripción |
| -------- | -------- | ----------- |
| type | Sí | El tipo de autenticación que desea utilizar. Para la autenticación básica, el valor debe ser `Basic`. |
| username | Sí | El nombre de usuario que quiere usar en la autenticación |
| password | Sí | La contraseña que quiere usar en la autenticación |
||||

<a name="azure-ad-code"></a>

#### <a name="azure-active-directory-authentication-through-code"></a>Autenticación de Azure Active Directory mediante código

De forma predeterminada, la autenticación de Azure AD que se activa en Azure Portal no lleva a cabo una autorización específica.
Por ejemplo, esta autenticación bloquea la API a un solo inquilino concreto, no a una aplicación o un usuario específicos.

Para restringir el acceso de la API a la aplicación lógica por medio de código, extraiga el encabezado que tenga JSON Web Token (JWT).
Compruebe la identidad del llamador y rechace las solicitudes que no coinciden.

<!-- Going further, to implement this authentication entirely in your own code,
and not use the Azure portal, learn how to
[authenticate with on-premises Active Directory in your Azure app](../app-service/overview-authentication-authorization.md).

To create an application identity for your logic app and use that identity to call your API,
you must follow the previous steps. -->

## <a name="next-steps"></a>Pasos siguientes

* [Implementación y llamada de API personalizadas desde flujos de trabajo de aplicación lógica](../logic-apps/logic-apps-custom-api-host-deploy-call.md)
