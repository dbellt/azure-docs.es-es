---
title: archivo de inclusión
description: archivo de inclusión
services: azure-communication-services
author: mikben
manager: mikben
ms.service: azure-communication-services
ms.subservice: azure-communication-services
ms.date: 03/10/2021
ms.topic: include
ms.custom: include file
ms.author: mikben
ms.openlocfilehash: a015ba06be6a10985a4771e842a6d910dd4cd9ab
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108157666"
---
En el **ejemplo de elementos principales de llamada grupal** de Azure Communication Services se muestra cómo se puede usar el SDK web de llamadas de Communication Services para crear una experiencia de llamada grupal.

En esta guía de inicio rápido de ejemplo, veremos cómo funciona el ejemplo antes de ejecutarlo en la máquina local. A continuación, implementaremos el ejemplo en Azure con sus propios recursos de Azure Communication Services.

## <a name="download-code"></a>Descarga de código

Busque el proyecto de este ejemplo en [GitHub](https://github.com/Azure-Samples/communication-services-web-calling-hero). Se puede encontrar una versión del ejemplo con la [interoperabilidad de Teams](../../concepts/teams-interop.md) en una [rama](https://github.com/Azure-Samples/communication-services-web-calling-hero/tree/teams-interop) independiente.

## <a name="overview"></a>Información general

El ejemplo tiene una aplicación del lado cliente y una aplicación del lado servidor. La **aplicación del lado cliente** es una aplicación web React/Redux que usa el marco de interfaz de usuario Fluent de Microsoft. Esta aplicación envía solicitudes a una **aplicación del lado servidor** de ASP.NET Core que ayuda a la aplicación del lado cliente a conectarse a Azure.

El ejemplo tendrá una apariencia similar a la siguiente:

:::image type="content" source="../media/calling/landing-page.png" alt-text="Captura de pantalla que muestra la página de aterrizaje de la aplicación de ejemplo.":::

Cuando se presiona el botón "Start a call", la aplicación web captura un token de acceso de usuario de la aplicación del lado servidor. A continuación, este token se usa para conectar la aplicación cliente con Azure Communication Services. Una vez recuperado el token, se le pedirá que especifique la cámara y el micrófono que quiere usar. Podrá deshabilitar o habilitar los dispositivos con los controles de alternancia:

:::image type="content" source="../media/calling/pre-call.png" alt-text="Captura de pantalla que muestra la pantalla anterior a la llamada en la aplicación de ejemplo.":::

Una vez que configure el nombre para mostrar y los dispositivos, puede unirse a la sesión de llamada. Verá el panel de llamada principal en el que reside la experiencia de llamada.

:::image type="content" source="../media/calling/main-app.png" alt-text="Captura de pantalla que muestra la pantalla principal de la aplicación de ejemplo.":::

Componentes de la pantalla principal de llamada:

- **Galería multimedia**: la fase principal en la que se muestran los participantes. Si un participante tiene habilitada la cámara, aquí se muestra su fuente de vídeo. Cada participante tiene un icono individual que muestra su nombre para mostrar y la transmisión de vídeo (si hay alguna).
- **Encabezado**: aquí es donde se encuentran los controles de llamada principales para ajustar la configuración y la barra lateral de participantes, activar o desactivar el vídeo y mezclas, compartir la pantalla y abandonar la llamada.
- **Barra lateral**: aquí es donde se muestran los participantes y la información de configuración cuando se ajustan con los controles del encabezado. El componente se puede descartar con la "X" de la esquina superior derecha. En la barra lateral de participantes se mostrará una lista de participantes y un vínculo para invitar a más usuarios a conversar. La barra lateral de configuración permite configurar las opciones del micrófono y la cámara.

> [!NOTE]
> En función de las limitaciones del SDK de llamadas web, solo se representa una secuencia de vídeo remota. Para más información, consulte [Compatibilidad con secuencias del SDK de llamadas](../../concepts/voice-video-calling/calling-sdk-features.md#calling-sdk-streaming-support).

A continuación encontrará más información sobre los requisitos previos y los pasos para configurar el ejemplo.

## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa. Para más información, consulte [Creación de una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)
- [Node.js (12.18.4 y posterior)](https://nodejs.org/en/download/)
- [Visual Studio (2019 y posterior)](https://visualstudio.microsoft.com/vs/)
- [.NET Core 3.1](https://dotnet.microsoft.com/download/dotnet-core/3.1) (asegúrese de instalar la versión que se corresponda con su instancia de Visual Studio, 32 o 64 bits)
- Un recurso de Azure Communication Services. Para obtener más información, consulte [Creación de un recurso de Azure Communication Services](../../quickstarts/create-communication-resource.md). Deberá registrar la **cadena de conexión** del recurso para esta guía de inicio rápido.

## <a name="locally-deploy-the-service--client-applications"></a>Implementación local del servicio y las aplicaciones cliente

El ejemplo de llamada grupal consta esencialmente dos aplicaciones: ClientApp y Service.NET.

Cuando queremos realizar una implementación localmente, es necesario iniciar ambas aplicaciones. Cuando se visita la aplicación de servidor desde el explorador, se usa la instancia de ClientApp implementada localmente para la experiencia del usuario.

Para probar el ejemplo localmente, puede abrir varias sesiones del explorador con la dirección URL de su llamada para simular una llamada multiusuario.

### <a name="before-running-the-sample-for-the-first-time"></a>Antes de ejecutar el ejemplo por primera vez

1. Abra una instancia de PowerShell, Terminal Windows, símbolo del sistema o equivalente y navegue hasta el directorio donde le gustaría clonar el ejemplo.
2. `git clone https://github.com/Azure-Samples/communication-services-web-calling-hero.git`
3. Obtenga `Connection String` de Azure Portal. Para obtener más información sobre las cadenas de conexión, consulte [Creación de recursos de Azure Communication Services](../../quickstarts/create-communication-resource.md).
4. Una vez que obtenga `Connection String`, agregue la cadena de conexión al archivo **Calling/appsetting.json** que se encuentra en la carpeta de Service .NET. Escriba la cadena de conexión en la variable: `ResourceConnectionString`.

### <a name="local-run"></a>Ejecución local

1. Vaya a la carpeta de llamadas y abra la solución `Calling.csproj` en Visual Studio.
2. Ejecute el proyecto `Calling`. El explorador se abrirá en `localhost:5001`.

## <a name="publish-the-sample-to-azure"></a>Publicación del ejemplo en Azure

1. Haga clic con el botón derecho en el proyecto `Calling` y seleccione Publicar.
2. Cree un nuevo perfil de publicación y seleccione la suscripción a Azure.
3. Antes de publicar, agregue la cadena de conexión con `Edit App Service Settings` y rellene `ResourceConnectionString` como clave y proporcione la cadena de conexión (copiada de appsettings.json) como valor.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si quiere limpiar y quitar una suscripción a Communication Services, puede eliminar el recurso o grupo de recursos. Al eliminar el grupo de recursos, también se elimina cualquier otro recurso que esté asociado a él. Obtenga más información sobre la [limpieza de recursos](../../quickstarts/create-communication-resource.md#clean-up-resources).

## <a name="next-steps"></a>Pasos siguientes

>[!div class="nextstepaction"]
>[Descargue el ejemplo de GitHub](https://github.com/Azure-Samples/communication-services-web-calling-hero).

Para más información, consulte los siguientes artículos.

- Familiarícese con el [uso del SDK de llamadas](../../quickstarts/voice-video-calling/calling-client-samples.md).
- Más información sobre [cómo funciona la llamada](../../concepts/voice-video-calling/about-call-types.md)

### <a name="additional-reading"></a>Lecturas adicionales

- [Ejemplos](./../overview.md): encuentre más ejemplos en nuestra página de información general de ejemplos.
- [Redux](https://redux.js.org/): Administración de estado del lado cliente
- [FluentUI](https://aka.ms/fluent-ui): Biblioteca de interfaz de usuario con tecnología de Microsoft
- [React](https://reactjs.org/): Biblioteca para compilar interfaces de usuario
- [ASP.NET Core](/aspnet/core/introduction-to-aspnet-core?preserve-view=true&view=aspnetcore-3.1): Marco para compilar aplicaciones web