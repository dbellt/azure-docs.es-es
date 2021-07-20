---
title: Configuración y creación de tokens de acceso de Teams
titleSuffix: An Azure Communication Services quickstart
description: Creación de un servicio que proporciona tokens de acceso de Teams
author: tomaschladek
manager: nmurav
services: azure-communication-services
ms.author: tchladek
ms.date: 06/30/2021
ms.topic: overview
ms.service: azure-communication-services
ms.openlocfilehash: e9e58659cfaa5b459a28278362aac002a1a87db5
ms.sourcegitcommit: f4e04fe2dfc869b2553f557709afaf057dcccb0b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/02/2021
ms.locfileid: "113223867"
---
# <a name="quickstart-set-up-and-manage-teams-access-tokens"></a>Inicio rápido: Configuración y administración de tokens de acceso de Teams

> [!IMPORTANT]
> Para habilitar o deshabilitar la experiencia del punto de conexión de Teams personalizado, rellene [este formulario](https://forms.office.com/r/B8p5KqCH19).

En este inicio rápido, crearemos una aplicación de consola de .NET para autenticar un token de usuario de AAD mediante la biblioteca MSAL. A continuación, intercambiaremos ese token por un token de acceso de Teams con el SDK de identidad de Azure Communication Services. Después, el SDK de llamadas de Azure Communication Services puede usar el token de acceso de Teams para crear un punto de conexión de Teams personalizado.

> [!NOTE]
> En entornos de producción, se recomienda implementar este mecanismo de intercambio en los servicios de back-end, ya que las solicitudes de intercambio se firman con un secreto.


## <a name="prerequisites"></a>Prerrequisitos
- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Un recurso activo de Communication Services y una cadena de conexión. [Cree un recurso de Communication Services](./create-communication-resource.md).
- Habilite la experiencia del punto de conexión de Teams personalizado mediante [este formulario](https://forms.office.com/r/B8p5KqCH19).
- Azure Active Directory con usuarios que tengan licencia de Teams

## <a name="introduction"></a>Introducción

Las identidades de Teams están enlazadas a inquilinos de Azure Active Directory. Los usuarios del mismo inquilino o de cualquier inquilino pueden usar la aplicación. En este inicio rápido, trabajaremos en un caso de uso multiinquilino con varios actores: usuarios, desarrolladores y administradores de las empresas ficticias Contoso y Fabrikam. En este caso de uso, Contoso es una empresa que crea una solución SaaS para Fabrikam. 

Las secciones siguientes le guiarán por los pasos para administradores, desarrolladores y usuarios. En los diagramas se muestra el caso de uso de varios inquilinos. Si trabaja con un solo inquilino, ejecute todos los pasos de Contoso y Fabrikam en un solo inquilino.

## <a name="admin-actions"></a>Acciones del administrador

El rol Administrador tiene permisos extendidos en AAD. Los miembros de este rol pueden aprovisionar recursos y pueden leer información de Azure Portal. En el diagrama siguiente, puede ver todas las acciones que deben ejecutar los administradores.

![Acciones del administrador para habilitar la experiencia del punto de conexión de Teams personalizado](./media/teams-identities/teams-identity-admin-overview.png)

1. El administrador de Contoso crea o selecciona la *aplicación* existente en Azure Active Directory. La propiedad *Tipos de cuenta admitidos* define si los usuarios de otro inquilino se pueden autenticar en la *aplicación*. La propiedad *Identificador URI de redirección* redirige las solicitudes de autenticación correctas al *servidor* de Contoso.
1. El administrador de Contoso amplía el manifiesto de *aplicación* con el permiso para VoIP de Azure Communication Services. 
1. El administrador de Contoso permite el flujo de cliente público para la *aplicación*.
1. El administrador de Contoso puede actualizar la opcionalidad.
1. El administrador de Contoso habilita la experiencia mediante [este formulario](https://forms.office.com/r/B8p5KqCH19).
1. El administrador de Contoso crea o selecciona los recursos de Communication Services existentes, que se usarán para la autenticación de las solicitudes de intercambio. Los tokens de usuario de AAD se intercambiarán por tokens de acceso de Teams. Para más información sobre la creación de nuevos recursos de Azure Communication Services, consulte [aquí](./create-communication-resource.md).
1. El administrador de Fabrikam aprovisiona una nueva entidad de servicio para Azure Communication Services en el inquilino de Fabrikam.
1. El administrador de Fabrikam concede el permiso para VoIP de Azure Communication Services a la *aplicación* de Contoso. Este paso solo es necesario si la *aplicación* de Contoso no está verificada.

### <a name="1-create-aad-application-registration-or-select-aad-application&quot;></a>1. Creación del registro de aplicación de AAD o selección de una aplicación de AAD 

Los usuarios se deben autenticar en aplicaciones de AAD con el permiso para `VoIP` de Azure Communication Services. Si no tiene una aplicación existente que le gustaría usar para este inicio rápido, puede crear un registro de aplicación. 

La siguiente configuración de la aplicación influye en la experiencia:
- La propiedad *Tipos de cuenta admitidos* define si la *aplicación* es de un solo inquilino (&quot;Solo cuentas de este directorio organizativo") o multiinquilino ("Cuentas de cualquier directorio organizativo"). Para este escenario, puede usar la opción de multiinquilino.
- El *identificador URI de redirección* define el identificador URI donde se redirige la solicitud de autenticación después de la autenticación. En este escenario, puede usar "Cliente público/nativo (móvil y escritorio)" y rellenar "http://localhost" como identificador URI.

[Aquí puede encontrar documentación detallada](/azure/active-directory/develop/quickstart-register-app#register-an-application). 

Cuando la *aplicación* está registrada, verá un identificador en la información general. Este identificador se usará en los pasos siguientes: **Id. de aplicación (cliente)** .

### <a name="2-allow-public-client-flows"></a>2. Permitir flujos de clientes públicos

En el panel *Autenticación* de la *aplicación*, puede ver la plataforma configurada para *Cliente público/nativo (móvil y escritorio)* con el identificador URI de redirección que apunta a *localhost*. En la parte inferior de la pantalla, puede encontrar el botón de alternancia *Permitir flujos de clientes públicos*, que para este inicio rápido se establecerá en **Sí**.

### <a name="3-update-publisher-domain-optional"></a>3. Actualización del dominio del publicador (opcional)
En el panel *Personalización de marca*, puede actualizar el dominio del publicador para la *aplicación*. Esto es útil para las aplicaciones multiinquilino, donde la aplicación se marcará como verificada por Azure. Puede encontrar detalles sobre cómo comprobar el publicador y cómo actualizar el dominio de la aplicación [aquí](/azure/active-directory/develop/howto-configure-publisher-domain).

### <a name="4-define-azure-communication-services-voip-permission-in-application"></a>4. Definición del permiso para VoIP de Azure Communication Services en la aplicación

Vaya a los detalles de la *aplicación* y seleccione el panel "Manifiesto". En el manifiesto, busque la propiedad llamada: *"requiredResourceAccess"* . Es una matriz de objetos que define los permisos de la *aplicación*. Extienda el manifiesto con los permisos para VoIP para la aplicación de primera entidad de Azure Communication Services. Agregue el siguiente objeto a la matriz.

> [!NOTE] 
> No cambie los GUID del fragmento de código, ya que identifican de forma única la aplicación y los permisos.

```json
{
   "resourceAppId": "1fd5118e-2576-4263-8130-9503064c837a",
   "resourceAccess": [
      {
         "id": "31f1efa3-6f54-4008-ac59-1bf1f0ff9958",
         "type": "Scope"
      }
   ]
}
```

A continuación, seleccione el botón *Guardar* para conservar los cambios. Ahora puede ver el permiso para VoIP de Azure Communication Services en el panel *Permisos de API*.

### <a name="5-enable-custom-teams-endpoint-experience-for-application"></a>5. Habilitación de la experiencia del punto de conexión de Teams personalizado para la *aplicación*

El administrador de AAD rellena el siguiente [formulario](https://forms.office.com/r/B8p5KqCH19) para habilitar la experiencia del punto de conexión de Teams personalizado para la *aplicación*.

### <a name="6-create-or-select-communication-services-resource"></a>6. Creación o selección del recurso de Communication Services

El recurso de Azure Communication Services se usará para autenticar todas las solicitudes de intercambio de tokens de usuario de AAD por tokens de acceso de Teams. Este intercambio se puede desencadenar mediante el SDK de identidad de Azure Communication Services, que se autentica con la clave de acceso o RBAC de Azure. Puede obtener la clave de acceso en Azure Portal o configurar RBAC de Azure mediante el panel *Control de acceso (IAM)* .

Si desea [crear un nuevo recurso de Communication Services, siga esta guía](./create-communication-resource.md).

### <a name="7-provision-azure-communication-services-service-principal"></a>7. Aprovisionamiento de la entidad de servicio de Azure Communication Services

Para habilitar la experiencia del punto de conexión de Teams personalizado en el inquilino de Fabrikam, el administrador de AAD de Fabrikam debe aprovisionar la entidad de servicio llamada Azure Communication Services con el identificador de aplicación: *1fd5118e-2576-4263-8130-9503064c837a*. Si no ve esta aplicación en el panel Aplicaciones empresariales de Azure Active Directory, se debe agregar manualmente.

El administrador de AAD de Fabrikam se conecta al inquilino de Azure mediante PowerShell. 

> [!NOTE]
> Reemplace [Tenant_ID] por el identificador del inquilino, que se puede encontrar en Azure Portal en la página de información general de AAD.

```azurepowershell
Connect-AzureAD -TenantId "[Tenant_ID]"
```

Si no se encuentra el comando, el módulo AzureAD no está instalado en PowerShell. Cierre PowerShell y ejecútelo con derechos de administración. A continuación, puede instalar el paquete de Azure-AD con el siguiente comando:

```azurepowershell
Install-Module AzureAD
```

Después de conectarse y autenticarse en Azure, ejecute el siguiente comando para aprovisionar la entidad de servicio de Communication Services. 

> [!NOTE]
> El parámetro AppId hace referencia a la aplicación de primera entidad de Azure Communication Services. No cambie este valor.

```azurepowershell
New-AzureADServicePrincipal -AppId "1fd5118e-2576-4263-8130-9503064c837a"
```

### <a name="8-provide-admin-consent"></a>8. Proporcionar consentimiento del administrador

Si la *aplicación* de Contoso no está verificada, el administrador de AAD debe conceder permiso a la *aplicación* de Contoso para VoIP de Azure Communication Services. El administrador de AAD de Fabrikam proporciona consentimiento mediante un vínculo único. Para crear el vínculo de consentimiento del administrador, siga las instrucciones:

1. Tome el siguiente vínculo: *https://login.microsoftonline.com/{Tenant_ID}/adminconsent?client_id={Application_ID}*
1. Reemplace {Tenant_ID} por el identificador de inquilino de Fabrikam.
1. Reemplace {Application_ID} por el identificador de aplicación de Contoso.
1. El administrador de AAD de Fabrikam va al vínculo en el explorador. 
1. El administrador de AAD de Fabrikam inicia sesión y concede permisos en nombre de la organización.

Se crea la entidad de servicio de la *aplicación* de Contoso en el inquilino de Fabrikam si se concede consentimiento. El administrador de Fabrikam puede revisar el consentimiento en AAD:

1. Inicie sesión en Azure Portal como administrador.
1. Vaya a Azure Active Directory.
1. Vaya al panel "Aplicaciones empresariales".
1. Establezca el filtro "Tipo de aplicación" en "Todas las aplicaciones".
1. En el campo para filtrar las aplicaciones, inserte el nombre de la aplicación de Contoso.
1. Seleccione "Aplicar" para filtrar los resultados.
1. Seleccione la entidad de servicio con el nombre necesario. 
1. Vaya al panel *Permisos*.

Puede ver que el permiso para VoIP de Azure Communication Services tiene ahora el estado *Concedido para {Nombre_del_directorio}* .

## <a name="developer-actions"></a>Acciones para el desarrollador

El desarrollador de Contoso debe configurar la *aplicación cliente* para la autenticación de usuarios. A continuación, el desarrollador debe crear un punto de conexión en el *servidor* back-end para procesar el token de usuario de AAD después del redireccionamiento. Cuando se recibe un token de usuario de AAD, se intercambia por un token de acceso de Teams y se devuelve a la *aplicación cliente*. Las acciones necesarias por parte de los desarrolladores se muestran en el diagrama siguiente:

![Acciones del desarrollador para habilitar la experiencia del punto de conexión de Teams personalizado](./media/teams-identities/teams-identity-developer-overview.png)

1. El desarrollador de Contoso configura la biblioteca MSAL para autenticar al usuario para la *aplicación* creada en pasos anteriores por el administrador para el permiso para VoIP de Azure Communication Services.
1. El desarrollador de Contoso inicializa el SDK de identidad de ACS e intercambia el token de usuario de AAD entrante por el token de acceso de Teams mediante el SDK. A continuación, se devuelve el token de acceso de Teams a la *aplicación cliente*.

La Biblioteca de autenticación de Microsoft (MSAL) permite que los desarrolladores adquieran tokens de usuario de AAD desde el punto de conexión de la Plataforma de identidad de Microsoft para autenticar a los usuarios y acceder a las API web protegidas. Se puede usar para proporcionar acceso seguro a Azure Communication Services. MSAL es compatible con muchas arquitecturas y plataformas de aplicación distintas, incluidas .NET, JavaScript, Java, Python, Android e iOS.

Puede encontrar más detalles sobre cómo configurar diferentes entornos en la documentación pública. [Introducción a la Biblioteca de autenticación de Microsoft (MSAL)](/azure/active-directory/develop/msal-overview).

> [!NOTE]
> En las secciones siguientes, se describe cómo intercambiar el token de acceso de AAD por el token de acceso de Teams para la aplicación de consola en .NET.

### <a name="create-new-application"></a>Creación de una nueva aplicación

En una ventana de consola (por ejemplo, cmd, PowerShell o Bash), use el comando dotnet new para crear una nueva aplicación de consola con el nombre *TeamsAccessTokensQuickstart*. Este comando crea un sencillo proyecto "Hola mundo" de C# con un solo archivo de origen: *Program.cs*.

```console
dotnet new console -o TeamsAccessTokensQuickstart
```

Cambie el directorio a la carpeta de la aplicación recién creada y use el comando dotnet build para compilar la aplicación.

```console
cd TeamsAccessTokensQuickstart
dotnet build
```
#### <a name="install-the-package"></a>Instalar el paquete
Mientras sigue en el directorio de la aplicación, instale el paquete de la biblioteca de identidad de Azure Communication Services para .NET con el comando dotnet add package.

```console
dotnet add package Azure.Communication.Identity
dotnet add package Microsoft.Identity.Client
```

#### <a name="set-up-the-app-framework"></a>Instalación del marco de la aplicación

Desde el directorio del proyecto:

- Abra el archivo Program.cs en un editor de texto.
- Agregue una directiva using para incluir los siguientes espacios de nombres: 
    - Azure.Communication
    - Azure.Communication.Identity
    - Microsoft.Identity.Client
- Actualice la declaración del método Main para admitir código asincrónico.

Use el código siguiente para empezar:

```csharp
using System;
using System.Text;
using Azure.Communication;
using Azure.Communication.Identity;
using Microsoft.Identity.Client;

namespace TeamsAccessTokensQuickstart
{
    class Program
    {
        static async System.Threading.Tasks.Task Main(string[] args)
        {
            Console.WriteLine("Azure Communication Services – Teams access tokens quickstart");

            // Quickstart code goes here
        }
    }
}
```

### <a name="1-receive-aad-user-token-via-msal-library"></a>1. Recepción del token de usuario de AAD mediante la biblioteca MSAL

Use la biblioteca MSAL para autenticar al usuario en AAD para la *aplicación* de Contoso con el permiso para VoIP de Azure Communication Services. Configure el cliente para la *aplicación* de Contoso (*parámetro applicationId*) en la nube pública (*parámetro authority*). El token de usuario de AAD se devolverá al identificador URI de redirección (*parámetro redirectUri*). Las credenciales se tomarán de una ventana emergente interactiva que se abrirá en el explorador predeterminado.

> [!NOTE] 
> El identificador URI de redirección debe coincidir con el valor definido en la *aplicación*. Consulte el primer paso de la guía del administrador para ver cómo configurar el identificador URI de redirección.

```csharp
const string applicationId = "Contoso's_Application_ID";
const string authority = "https://login.microsoftonline.com/common";
const string redirectUri = "http://localhost";

var client = PublicClientApplicationBuilder
                .Create(applicationId)
                .WithAuthority(authority)
                .WithRedirectUri(redirectUri)
                .Build();

const string scope = "https://auth.msft.communication.azure.com/VoIP";

var aadUserToken = await client.AcquireTokenInteractive(new[] { scope }).ExecuteAsync();

Console.WriteLine("\nAuthenticated user: " + aadUserToken.Account.Username);
Console.WriteLine("AAD user token expires on: " + aadUserToken.ExpiresOn);
```

La variable *aadUserToken* ahora lleva el token de usuario de Azure Active Directory válido, que se usará para el intercambio.

### <a name="2-exchange-aad-user-token-for-teams-access-token"></a>2. Intercambio del token de usuario de AAD por el token de acceso de Teams

Un token de usuario de AAD válido autentica al usuario en AAD para aplicaciones de terceros con el permiso para VoIP de Azure Communication Services. En el código siguiente, se usa el SDK de identidad de ACS para facilitar el intercambio del token de usuario de AAD por el token de acceso de Teams.

> [!NOTE]
> Reemplace el valor "&lt;Connection-String&gt;" por una cadena de conexión válida o use RBAC de Azure para la autenticación. Puede encontrar más información en [este inicio rápido](./access-tokens.md).

```csharp
var identityClient = new CommunicationIdentityClient("<Connection-String>");
var teamsAccessToken = identityClient.ExchangeTeamsToken(aadUserToken.AccessToken);

Console.WriteLine("\nTeams access token expires on: " + teamsAccessToken.Value.ExpiresOn);
```

Si se cumplen todas las condiciones definidas en los requisitos, se obtiene un token de acceso de Teams válido durante 24 horas.

#### <a name="run-the-code"></a>Ejecución del código
Ejecute la aplicación desde el directorio de aplicaciones con el comando dotnet run.

```console
dotnet run
```

La salida de la aplicación describe cada acción que se completa:

```console
Azure Communication Services - Teams access tokens quickstart

Authenticated user: john.smith@contoso.com
AAD user token expires on: 6/10/2021 10:13:17 AM +00:00

Teams access token expires on: 6/11/2021 9:13:18 AM +00:00
```

## <a name="user-actions"></a>Acciones del usuario

El usuario representa a los usuarios de Fabrikam de la *aplicación* de Contoso. La experiencia del usuario se muestra en el diagrama siguiente.

![Acciones del usuario para habilitar la experiencia del punto de conexión de Teams personalizado](./media/teams-identities/teams-identity-user-overview.png)

1. El usuario de Fabrikam usa la *aplicación cliente* de Contoso y se le pide que se autentique.
1. La *aplicación cliente* de Contoso usa la biblioteca MSAL para autenticar al usuario en el inquilino de Azure Active Directory de Fabrikam para la *aplicación* de Contoso con el permiso para VoIP de Azure Communication Services. 
1. La autenticación se redirige al *servidor*, tal como se define en la propiedad *Identificador URI de redirección* en MSAL y la *aplicación* de Contoso.
1. El *servidor* de Contoso intercambia el token de usuario de AAD por el token de acceso de Teams con el SDK de identidad de ACS y devuelve el token de acceso de Teams a la *aplicación cliente*.

Con el token de acceso de Teams válido en la *aplicación cliente*, el desarrollador puede integrar el SDK de llamadas de ACS y crear un punto de conexión de Teams personalizado.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a hacer lo siguiente:

> [!div class="checklist"]
> * Crear y configurar una aplicación en Azure Active Directory
> * Usar la biblioteca MSAL para emitir un token de usuario de Azure Active Directory
> * Usar el SDK de identidad de ACS para intercambiar el token de usuario de Azure Active Directory por el token de acceso de Teams

Puede que los siguientes documentos le resulten interesantes:

- Más información sobre el [punto de conexión de Teams personalizado](../concepts/teams-endpoint.md)
- Más información sobre la [interoperabilidad de Teams](../concepts/teams-interop.md)
