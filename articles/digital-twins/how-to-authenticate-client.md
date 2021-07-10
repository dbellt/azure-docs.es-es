---
title: Escritura de código de autenticación de aplicación
titleSuffix: Azure Digital Twins
description: Vea cómo escribir código de autenticación en una aplicación cliente
author: baanders
ms.author: baanders
ms.date: 10/7/2020
ms.topic: how-to
ms.service: digital-twins
ms.openlocfilehash: f37699e9710acc9e4098fd38f0d9da0f50e65a58
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111953982"
---
# <a name="write-client-app-authentication-code"></a>Escritura de código de autenticación de aplicación cliente

Después de [configurar una instancia y la autenticación de Azure Digital Twins](how-to-set-up-instance-portal.md), puede crear una aplicación cliente para usarla para interactuar con la instancia. Una vez configurado un proyecto cliente de inicio, necesitará **escribir código en esa aplicación cliente para autenticarla** en la instancia de Azure Digital Twins.

Azure Digital Twins realiza la autenticación mediante [tokens de seguridad de Azure AD basados en OAUTH 2.0](../active-directory/develop/security-tokens.md#json-web-tokens-and-claims). Para autenticar el SDK, deberá obtener un token de portador con los permisos correctos para Azure Digital Twins y pasarlo junto con las llamadas de API. 

En este artículo se describe cómo obtener credenciales mediante la biblioteca de cliente `Azure.Identity`. Aunque en este artículo se muestran ejemplos de código en C#, como lo que escribiría para el [SDK de .NET (C#)](/dotnet/api/overview/azure/digitaltwins/client?view=azure-dotnet&preserve-view=true), puede usar una versión de `Azure.Identity` independientemente del SDK que use (para obtener más información sobre los SDK disponibles para Azure Digital Twins, consulte [Conceptos: uso de las API y los SDK de Azure Digital Twins](concepts-apis-sdks.md)).

## <a name="prerequisites"></a>Requisitos previos

En primer lugar, realice los pasos de configuración de [Configuración de una instancia y autenticación](how-to-set-up-instance-portal.md). Esto garantiza que tiene una instancia de Azure Digital Twins y que el usuario tiene permisos de acceso. Después de toda esta configuración, está listo para escribir código de aplicación cliente.

Para continuar, necesita un proyecto de aplicación cliente en el que escribir el código. Si aún no tiene un proyecto de aplicación cliente configurado, cree un proyecto básico en el lenguaje que prefiera para usarlo con este tutorial.

## <a name="common-authentication-methods-with-azureidentity"></a>Métodos de autenticación comunes con Azure.Identity

`Azure.Identity` es una biblioteca de cliente que proporciona varios métodos para obtener credenciales que puede usar para obtener un token de portador y autenticarse con el SDK. Aunque en este artículo se proporcionan ejemplos en C# , puede ver `Azure.Identity` para varios idiomas, incluido...

* [.NET (C#)](/dotnet/api/azure.identity?view=azure-dotnet&preserve-view=true)
* [Java](/java/api/overview/azure/identity-readme?view=azure-java-stable&preserve-view=true)
* [JavaScript](/javascript/api/overview/azure/identity-readme?view=azure-node-latest&preserve-view=true)
* [Python](/python/api/overview/azure/identity-readme?view=azure-python&preserve-view=true)

Tres métodos comunes de obtención de credenciales en `Azure.Identity` son:

* [DefaultAzureCredential](/dotnet/api/azure.identity.defaultazurecredential?view=azure-dotnet&preserve-view=true) proporciona un flujo de autenticación `TokenCredential` predeterminado para las aplicaciones que se implementarán en Azure y es **la opción recomendada para el desarrollo local**. También se puede habilitar para probar los otros dos métodos recomendados en este artículo: contiene `ManagedIdentityCredential` y puede tener acceso a `InteractiveBrowserCredential` con una variable de configuración.
* [ManagedIdentityCredential](/dotnet/api/azure.identity.managedidentitycredential?view=azure-dotnet&preserve-view=true) funciona bien en los casos en los que necesita [identidades administradas (MSI)](../active-directory/managed-identities-azure-resources/overview.md) y es un buen candidato para trabajar con Azure Functions y realizar implementaciones en servicios de Azure.
* [InteractiveBrowserCredential](/dotnet/api/azure.identity.interactivebrowsercredential?view=azure-dotnet&preserve-view=true) está diseñado para aplicaciones interactivas y se puede usar para crear un cliente de SDK autenticado.

En los ejemplos siguientes se muestra cómo usar cada uno de estos con el SDK de .NET (C#).

## <a name="authentication-examples-net-c-sdk"></a>Ejemplos de autenticación: SDK de .NET (C#)

En esta sección se muestra un ejemplo en C# sobre el uso del SDK para .NET proporcionado para escribir el código de autenticación.

En primer lugar, incluya el paquete del SDK `Azure.DigitalTwins.Core` y el paquete `Azure.Identity` en el proyecto. En función de las herramientas que elija, puede incluir los paquetes con el administrador de paquetes de Visual Studio o con la herramienta de línea de comandos `dotnet`. 

También deberá agregar las siguientes instrucciones using al código del proyecto:

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/authentication.cs" id="Azure_Digital_Twins_dependencies":::

A continuación, agregue código para obtener credenciales mediante uno de los métodos de `Azure.Identity`.

### <a name="defaultazurecredential-method"></a>Método DefaultAzureCredential

[DefaultAzureCredential](/dotnet/api/azure.identity.defaultazurecredential?view=azure-dotnet&preserve-view=true) proporciona un flujo de autenticación `TokenCredential` predeterminado para las aplicaciones que se implementarán en Azure y es **la opción recomendada para el desarrollo local**.

Para usar las credenciales predeterminadas de Azure, necesitará la dirección URL de la instancia de Azure Digital Twins ([instrucciones para encontrarla](how-to-set-up-instance-portal.md#verify-success-and-collect-important-values)).

A continuación se muestra un ejemplo de código para agregar `DefaultAzureCredential` al proyecto:

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/authentication.cs" id="DefaultAzureCredential_full":::

#### <a name="set-up-local-azure-credentials"></a>Configuración de credenciales locales de Azure

[!INCLUDE [Azure Digital Twins: local credentials prereq (inner)](../../includes/digital-twins-local-credentials-inner.md)]

### <a name="managedidentitycredential-method"></a>Método ManagedIdentityCredential

El método [ManagedIdentityCredential](/dotnet/api/azure.identity.managedidentitycredential?view=azure-dotnet&preserve-view=true) funciona muy bien en los casos en los que necesita [identidades administradas (MSI)](../active-directory/managed-identities-azure-resources/overview.md), por ejemplo, al trabajar con Azure Functions.

Esto significa que puede usar `ManagedIdentityCredential` en el mismo proyecto que `DefaultAzureCredential` o `InteractiveBrowserCredential` para autenticar otra parte del proyecto.

Para usar las credenciales predeterminadas de Azure, necesitará la dirección URL de la instancia de Azure Digital Twins ([instrucciones para encontrarla](how-to-set-up-instance-portal.md#verify-success-and-collect-important-values)).

En una función de Azure, puede usar las credenciales de identidad administrada de la siguiente manera:

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/authentication.cs" id="ManagedIdentityCredential":::

### <a name="interactivebrowsercredential-method"></a>Método InteractiveBrowserCredential

El método [InteractiveBrowserCredential](/dotnet/api/azure.identity.interactivebrowsercredential?view=azure-dotnet&preserve-view=true) está pensado para aplicaciones interactivas y abrirá un explorador web para la autenticación. Puede utilizarlo en lugar de `DefaultAzureCredential` en los casos en los que necesite una autenticación interactiva.

Para usar las credenciales interactivas del explorador, necesitará un **registro de la aplicación** que tenga permisos para las API de Azure Digital Twins. Para conocer los pasos para configurar el registro de esta aplicación, consulte [Guía paso a paso: creación de un registro de aplicación](./how-to-create-app-registration-portal.md). Una vez configurado el registro de la aplicación, necesitará...
* el [id. de la aplicación (cliente) del registro de la aplicación;](./how-to-create-app-registration-portal.md#collect-client-id-and-tenant-id)
* el [id. del directorio (inquilino) del registro de la aplicación;](./how-to-create-app-registration-portal.md#collect-client-id-and-tenant-id)
* [la dirección URL de la instancia de Azure Digital Twins](how-to-set-up-instance-portal.md#verify-success-and-collect-important-values)

Este es un ejemplo del código para crear un cliente de SDK autenticado mediante `InteractiveBrowserCredential`.

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/authentication.cs" id="InteractiveBrowserCredential":::

>[!NOTE]
> Aunque puede especificar el identificador de cliente, el identificador de inquilino y la dirección URL de la instancia directamente en el código, como se muestra arriba, se recomienda que, en su lugar, el código obtenga estos valores de un archivo de configuración o una variable de entorno.

#### <a name="other-notes-about-authenticating-azure-functions"></a>Otras notas sobre la autenticación de Azure Functions

Consulte [ Configure una función de Azure para procesar los datos](how-to-create-azure-function.md), a fin de obtener un ejemplo más completo en el que se explican algunas de las opciones de configuración importantes en el contexto de las funciones.

Además, para usar la autenticación en una función, recuerde:
* [Habilitar una entidad administrada](../app-service/overview-managed-identity.md?tabs=dotnet).
* Usar [variables de entorno](/sandbox/functions-recipes/environment-variables?tabs=csharp) según sea necesario
* Asignar permisos a la aplicación de funciones que le permiten acceder a las API de Digital Twins. Para obtener más información sobre los procesos de Azure Functions, vea  [Configuración de una función de Azure para procesar datos](how-to-create-azure-function.md).

## <a name="authenticate-across-tenants"></a>Autenticación entre inquilinos

Azure Digital Twins es un servicio que solo admite un [inquilino de Azure Active Directory (Azure AD)](../active-directory/develop/quickstart-create-new-tenant.md): el inquilino principal de la suscripción donde se encuentra la instancia de Azure Digital Twins.

[!INCLUDE [digital-twins-tenant-limitation](../../includes/digital-twins-tenant-limitation.md)]

Si necesita acceder a la instancia de Azure Digital Twins mediante una entidad de servicio o una cuenta de usuario que pertenezca a un inquilino distinto de la instancia, puede hacer que cada identidad federada de otro inquilino solicite un **token** del inquilino "principal" de la instancia de Azure Digital Twins. 

[!INCLUDE [digital-twins-tenant-solution-1](../../includes/digital-twins-tenant-solution-1.md)]

También puede especificar el inquilino principal en las opciones de credenciales en el código. 

[!INCLUDE [digital-twins-tenant-solution-2](../../includes/digital-twins-tenant-solution-2.md)]

## <a name="other-credential-methods"></a>Otros métodos de credenciales

Si los escenarios de autenticación resaltados anteriores no cubren las necesidades de la aplicación, puede explorar otros tipos de autenticación que se ofrecen en la [plataforma de Microsoft Identity](../active-directory/develop/v2-overview.md#getting-started). La documentación de esta plataforma abarca escenarios de autenticación adicionales organizados por tipo de aplicación.

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre cómo funciona la seguridad en Azure Digital Twins:
* [Conceptos: Seguridad para las soluciones de Azure Digital Twins](concepts-security.md)

O bien, ahora que la autenticación está configurada, continúe con la creación y administración de modelos en la instancia:
* [Procedimiento: Administración de modelos DTDL](how-to-manage-model.md)