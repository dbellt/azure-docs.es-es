---
title: 'Autenticación y autorización de la API: Azure Time Series Insights | Microsoft Docs'
description: En este artículo se describe cómo configurar la autenticación y la autorización para una aplicación personalizada que llama a la API de Azure Time Series Insights.
ms.service: time-series-insights
author: deepakpalled
ms.author: shresha
manager: dpalled
ms.reviewer: v-mamcge, jasonh, kfile
ms.devlang: csharp
ms.workload: big-data
ms.topic: conceptual
ms.date: 02/23/2021
ms.custom: seodec18, has-adal-ref, devx-track-azurecli
ms.openlocfilehash: 225e72bc00ce0a80ff655a76562b5c6b70b7fa79
ms.sourcegitcommit: afb79a35e687a91270973990ff111ef90634f142
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107479498"
---
# <a name="authentication-and-authorization-for-azure-time-series-insights-api"></a>Autenticación y autorización para la API de Azure Time Series Insights

En función de las necesidades empresariales, es posible que la solución incluya una o varias aplicaciones cliente que se usan para interactuar con las [API](/rest/api/time-series-insights/reference-data-access-overview)del entorno de Azure Time Series Insights. Azure Time Series Insights realiza la autenticación mediante [tokens de seguridad de Azure AD basados en OAUTH 2.0](../active-directory/develop/security-tokens.md#json-web-tokens-and-claims). Para autenticar el cliente, tendrá que obtener un token de portador con los permisos correctos y pasarlo junto con las llamadas API. En este documento se describen varios métodos para obtener credenciales que puede usar para obtener un token de portador y autenticarse, incluido el uso de la identidad administrada y el registro de la aplicación de Azure Active Directory.

## <a name="managed-identities"></a>Identidades administradas

En las secciones siguientes se describe cómo usar una identidad administrada de Azure Active Directory (Azure AD) para acceder a la API de Azure Time Series Insights. En Azure, las identidades administradas eliminan la necesidad de que los desarrolladores administren credenciales, al proporcionar una identidad para el recurso de Azure en Azure AD y usarla para obtener tokens de Azure Active Directory (Azure AD). Estas son algunas de las ventajas de usar las identidades administradas:

- No es necesario administrar credenciales. Las credenciales ni siquiera están accesible.
- Puede usar las identidades administradas para autenticarse en cualquier servicio de Azure que admita la autenticación de Azure AD, como Azure Key Vault.
- Las identidades administradas se pueden usar sin ningún costo adicional.

Para obtener más información sobre los dos tipos de identidades administradas, lea [¿Qué son las identidades administradas de recursos de Azure?](../active-directory/managed-identities-azure-resources/overview.md)

Puede usar identidades administradas desde:

- Máquinas virtuales de Azure
- Azure App Services
- Azure Functions
- Azure Container Instances
- Y mucho más...

Vea [Servicios de Azure que admiten identidades administradas para recursos de Azure](../active-directory/managed-identities-azure-resources/services-support-managed-identities.md#azure-services-that-support-managed-identities-for-azure-resources) para obtener la lista completa.

## <a name="azure-active-directory-app-registration"></a>Registro de la aplicación de Azure Active Directory

Se recomienda usar identidades administradas siempre que sea posible para que no sea necesario administrar las credenciales. Si la aplicación cliente no está hospedada en un servicio de Azure que admite identidades administradas, puede registrarla con un inquilino de Azure AD. Cuando registra una aplicación con Azure AD, crea una configuración de identidad para la aplicación, lo que permite integrarla con Azure AD. Cuando registra una aplicación en [Azure Portal](https://portal.azure.com/), elige si es de un solo inquilino (solo accesible en el inquilino) o multiinquilino (accesible en otros inquilinos) y, opcionalmente, puede establecer un identificador URI de redirección (al que se envía el token de acceso).

Una vez completado el registro de la aplicación, tiene una instancia globalmente única de la aplicación (el objeto de aplicación) que reside en su directorio o inquilino principal. También tiene un identificador único global para la aplicación (el identificador de la aplicación o del cliente). En el portal, puede entonces agregar secretos o certificados y ámbitos para que la aplicación funcione, personalizar la marca de la aplicación en el cuadro de diálogo de inicio de sesión y mucho más.

Al registrar una aplicación en Azure Portal, se crean automáticamente un objeto de aplicación y un objeto de entidad de servicio en su inquilino principal. Si registra o crea una aplicación mediante las API de Microsoft Graph, la creación del objeto de entidad de servicio se hace en un paso independiente. Para solicitar tokens se necesita un objeto de entidad de servicio.

Asegúrese de revisar la lista de comprobación de [seguridad](../active-directory/develop/identity-platform-integration-checklist.md#security) de la aplicación. Como procedimiento recomendado, debe usar [credenciales de certificado](../active-directory/develop/active-directory-certificate-credentials.md), no de contraseña (secretos de cliente).

Para obtener más información, vea [Objetos de aplicación y de entidad de servicio de Azure Active Directory](../active-directory/develop/app-objects-and-service-principals.md).

## <a name="step-1-create-your-managed-identity-or-app-registration"></a>Paso 1: Creación de la identidad administrada o el registro de aplicación

Una vez que haya identificado si va a usar una identidad administrada o un registro de aplicación, el siguiente paso consiste en aprovisionarlo.

### <a name="managed-identity"></a>Identidad administrada

Los pasos que usará para crear una identidad administrada variarán en función de dónde se encuentre el código y de si se va a crear o no una identidad asignada por el usuario o por el sistema. Lea [Tipos de identidad administrada](../active-directory/managed-identities-azure-resources/overview.md#managed-identity-types) para comprender la diferencia. Una vez que haya seleccionado el tipo de identidad, busque y siga el tutorial correcto en la [documentación](../active-directory/managed-identities-azure-resources/index.yml) sobre identidades administradas por Azure AD. Allí encontrará instrucciones sobre cómo configurar identidades administradas para:

- [Máquinas virtuales de Azure](../active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm.md#enable-system-assigned-managed-identity-during-creation-of-a-vm)
- [App Service y Azure Functions](../app-service/overview-managed-identity.md)
- [Azure Container Instances](../container-instances/container-instances-managed-identity.md)
- Y mucho más...

### <a name="application-registration"></a>Registro de la aplicación

Siga los pasos enumerados en [Registro de una aplicación](../active-directory/develop/quickstart-register-app.md#register-an-application).

[!INCLUDE [Azure Active Directory app registration](../../includes/time-series-insights-aad-registration.md)]

## <a name="step-2-grant-access"></a>Paso 2: Concesión de acceso

Cuando el entorno de Azure Time Series Insights recibe una solicitud, primero se valida el token de portador del autor de la llamada. Si la validación es correcta, el autor de la llamada se ha autenticado y, después, se realiza otra comprobación para asegurarse de que está autorizado para llevar a cabo la acción solicitada. Para autorizar a cualquier usuario o entidad de servicio, primero debe concederles acceso al entorno asignándoles el rol de lector o colaborador.

- Para conceder acceso mediante la interfaz de usuario de [Azure Portal](https://portal.azure.com/), siga las instrucciones indicadas en el artículo [Concesión de acceso a datos en un entorno](concepts-access-policies.md). Al seleccionar el usuario, puede buscar la identidad administrada o el registro de aplicación por su nombre o por identificador.

- Para conceder acceso mediante la CLI de Azure, ejecute el comando siguiente. Revise [esta](/cli/azure/ext/timeseriesinsights/tsi/access-policy) documentación a fin de obtener la lista completa de los comandos disponibles para administrar el acceso.

   ```azurecli-interactive
   az tsi access-policy create --name "ap1" --environment-name "env1" --description "some description" --principal-object-id "aGuid" --roles Reader Contributor --resource-group "rg1"
   ```

> [!Note]
> La extensión timeseriesinsights para la CLI de Azure necesita la versión 2.11.0 o posterior. La extensión se instalará de forma automática la primera vez que ejecute un comando az tsi access-policy. [Obtenga más información](/cli/azure/azure-cli-extensions-overview) sobre las extensiones.

## <a name="step-3-requesting-tokens"></a>Paso 3: Solicitud de tokens

Una vez que el registro de aplicación o la identidad administrada se haya aprovisionado y se le haya asignado un rol, está listo para empezar a usarlo a fin de solicitar tokens de portador de OAuth 2.0. El método que use para obtener un token variará en función de dónde se hospede el código y del lenguaje que elija. Al especificar el recurso (también conocido como "audiencia" del token), puede identificar Azure Time Series Insights por su dirección URL o GUID:

* `https://api.timeseries.azure.com/`
* `120d688d-1518-4cf7-bd38-182f158850b6`

> [!IMPORTANT]
> Si usa la dirección URL como el identificador de recurso, el token se debe emitir exactamente como `https://api.timeseries.azure.com/`. Se requiere la barra diagonal.

> * Por tanto, si usa [Postman](https://www.getpostman.com/), **AuthURL** será: `https://login.microsoftonline.com/microsoft.onmicrosoft.com/oauth2/authorize?scope=https://api.timeseries.azure.com//.default`.
> * `https://api.timeseries.azure.com/` es válido, pero `https://api.timeseries.azure.com` no.

### <a name="managed-identities"></a>Identidades administradas

Al acceder desde Azure App Service o Functions, siga las instrucciones de [Obtención de tokens para recursos de Azure](../app-service/overview-managed-identity.md).

Para las aplicaciones y funciones de .NET, la manera más sencilla de trabajar con una identidad administrada es mediante la [biblioteca cliente Azure Identity](/dotnet/api/overview/azure/identity-readme) para .NET. Esta biblioteca cliente es popular debido a su simplicidad y beneficios de seguridad. Los desarrolladores pueden escribir código una vez y dejar que la biblioteca cliente determine cómo realizar la autenticación en función del entorno de la aplicación, ya sea en una estación de trabajo de desarrollador mediante una cuenta de desarrollador, o bien implementada en Azure con una identidad de servicio administrada. Para obtener instrucciones sobre la migración de la biblioteca AppAuthentication anterior, lea [Instrucciones de migración de AppAuthentication a Azure.Identity](/dotnet/api/overview/azure/app-auth-migration).

Solicite un token para Azure Time Series Insights mediante C# y la biblioteca cliente Azure Identity para .NET:

   ```csharp
   using Azure.Identity;
   // ...
   var credential = new DefaultAzureCredential();
   var token = credential.GetToken(
   new Azure.Core.TokenRequestContext(
       new[] { "https://api.timeseries.azure.com/" }));
   var accessToken = token.Token;
   ```

### <a name="app-registration"></a>Registro de la aplicación

* Los desarrolladores pueden usar la [Biblioteca de autenticación de Microsoft](../active-directory/develop/msal-overview.md) (MSAL) a fin de obtener tokens para los registros de aplicaciones.

MSAL se puede utilizar en muchos escenarios de aplicación, incluidos los siguientes (entre otros):

* [Single page applications (JavaScript)](../active-directory/develop/scenario-spa-overview.md) (Aplicaciones de una sola página [JavaScript])
* [Web application signing in a user and calling a web API on behalf of the user](../active-directory/develop/scenario-web-app-call-api-overview.md) (Aplicación web que inicia sesión de un usuario y llama a una API web en nombre del usuario)
* [Web API calling another downstream web API on behalf of the signed-in user](../active-directory/develop/scenario-web-api-call-api-overview.md) (API web que llama a otra API web de nivel inferior en nombre del usuario que inició sesión)
* [Desktop application calling a web API on behalf of the signed-in user](../active-directory/develop/scenario-desktop-overview.md) (Aplicación de escritorio que llama a una API web en nombre del usuario que inició sesión)
* [Aplicación móvil que llama a una API web en nombre del usuario que ha iniciado sesión de manera interactiva](../active-directory/develop/scenario-mobile-overview.md).
* [Desktop/service daemon application calling web API on behalf of itself](../active-directory/develop/scenario-daemon-overview.md) (Aplicación de demonio de escritorio/servicio que llama a una API web en su propio nombre)

Para obtener un ejemplo de código de C# en el que se muestra cómo adquirir un token como un registro de aplicación y consultar datos de un entorno Gen2, vea la aplicación de ejemplo en [GitHub](https://github.com/Azure-Samples/Azure-Time-Series-Insights/blob/master/gen2-sample/csharp-tsi-gen2-sample/DataPlaneClientSampleApp/Program.cs).

> [!IMPORTANT]
> Si usa la [Biblioteca de autenticación de Azure Active Directory (ADAL)](../active-directory/azuread-dev/active-directory-authentication-libraries.md), lea acerca de la [migración a MSAL](../active-directory/develop/msal-net-migration.md).

## <a name="common-headers-and-parameters"></a>Parámetros y encabezados comunes

En esta sección, se describen los encabezados y parámetros comunes de la solicitud HTTP que se usan para realizar consultas frente a las API de Azure Time Series Insights Gen1 y Gen2. Los requisitos específicos de las API se describen con mayor detalle en la [documentación de referencia de la API de REST de Azure Time Series Insights](/rest/api/time-series-insights/).

> [!TIP]
> Lea la [Referencia de la API de REST de Azure](/rest/api/azure/) para obtener más información sobre cómo usar las API de REST, realizar solicitudes HTTP y manipular las respuestas HTTP.

### <a name="http-headers"></a>Encabezados HTTP

A continuación se describen los encabezados de solicitud obligatorios.

| Encabezado de solicitud obligatorio | Descripción |
| --- | --- |
| Authorization | Para realizar la autenticación en Azure Time Series Insights, es necesario pasar un token de portador de OAuth 2.0 válido en el [encabezado de autorización](/rest/api/apimanagement/2019-12-01/authorizationserver/createorupdate). |

> [!TIP]
> Lea la [visualización de muestra del SDK de cliente](https://tsiclientsample.azurewebsites.net/) hospedado de Azure Time Series Insights para aprender a autenticarse con las API de Azure Time Series Insights mediante programación, usando el [SDK de cliente de JavaScript](https://github.com/microsoft/tsiclient/blob/master/docs/API.md) junto con gráficos.

A continuación se describen los encabezados de solicitud opcionales.

| Encabezado de solicitud opcional | Descripción |
| --- | --- |
| Content-type | Solo se admite `application/json`. |
| x-ms-client-request-id | Un id. de solicitud de cliente. El servicio registra este valor. Permite que el servicio realice el seguimiento de la operación en todos los servicios. |
| x-ms-client-session-id | Un id. de sesión de cliente. El servicio registra este valor. Permite que el servicio realice el seguimiento de un grupo de operaciones relacionadas en todos los servicios. |
| x-ms-client-application-name | Nombre de la aplicación que generó esta solicitud. El servicio registra este valor. |

A continuación se describen los encabezados de respuesta opcionales, pero recomendados.

| Encabezado de respuesta | Descripción |
| --- | --- |
| Content-type | Solo se admite `application/json`. |
| x-ms-request-id | Id. de solicitud que creó el servidor. Se puede usar para ponerse en contacto con Microsoft para investigar una solicitud. |
| x-ms-property-not-found-behavior | Encabezado de respuesta opcional de la API GA. Los valores posibles son `ThrowError` (valor predeterminado) o `UseNull`. |

### <a name="http-parameters"></a>Parámetros de HTTP

> [!TIP]
> Encontrará más detalles sobre la información de consulta necesaria y opcional en la [documentación de referencia](/rest/api/time-series-insights/).

Los parámetros de la cadena de consulta de la URL obligatorios dependen de la versión de la API.

| Release | Valores de versión de API |
| --- |  --- |
| Gen1 | `api-version=2016-12-12`|
| Gen2 | `api-version=2020-07-31`|

Los parámetros de cadena de consulta de dirección URL opcionales incluyen establecer un tiempo de espera para los tiempos de ejecución de la solicitud HTTP.

| Parámetro de consulta opcional | Descripción | Versión |
| --- |  --- | --- |
| `timeout=<timeout>` | Tiempo de espera del lado servidor para la ejecución de la solicitud. Esto es aplicable solo a las API [Get Environment Events](/rest/api/time-series-insights/dataaccess(preview)/query/getavailability) y [Get Environment Aggregates](/rest/api/time-series-insights/gen1-query-api#get-environment-aggregates-api). El valor del tiempo de espera debe estar en un formato de duración de tipo ISO 8601, por ejemplo `"PT20S"`, y debe estar en el intervalo `1-30 s`. El valor predeterminado es `30 s`. | Gen1 |
| `storeType=<storeType>` | En el caso de los entornos de Gen2 con almacenamiento intermedio habilitado, la consulta se puede ejecutar en `WarmStore` o `ColdStore`. Este parámetro de la consulta define en qué almacén debe ejecutarse la consulta. Si no se define, la consulta se ejecutará en el almacén frío. Para consultar la tienda en caliente, **storeType** debe establecerse en `WarmStore`. Si no se define, la consulta se ejecutará en el almacén frío. | Gen2 |

## <a name="next-steps"></a>Pasos siguientes

* Para obtener código de ejemplo que llama a la API de Azure Time Series Insights Gen1, lea [Consulta de datos de Gen1 mediante C#](./time-series-insights-query-data-csharp.md).

* Para obtener código de ejemplo que llama a la API de Azure Time Series Insights Gen2, lea [Consulta de datos de Gen2 mediante C#](./time-series-insights-update-query-data-csharp.md).

* Para información de referencia de la API, lea la documentación de [referencia sobre la API de consulta](/rest/api/time-series-insights/reference-query-apis).
