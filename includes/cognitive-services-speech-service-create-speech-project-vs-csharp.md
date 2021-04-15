---
author: wolfma61
ms.service: cognitive-services
ms.topic: include
ms.date: 08/30/2019
ms.author: wolfma
ms.openlocfilehash: c7332177795a45331749225e0f3dc3856e28a6a2
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "96001118"
---
Para crear un proyecto de Visual Studio para el desarrollo de Windows, debe crear el proyecto, configurar Visual Studio para el desarrollo de escritorio de .NET, instalar el SDK de Voz y elegir la arquitectura de destino.

### <a name="create-the-project-and-add-the-workload"></a>Creación del proyecto e incorporación de la carga de trabajo

Para empezar, cree el proyecto en Visual Studio y asegúrese de que Visual Studio está configurado para el desarrollo de escritorio de .NET:

1. Abra Visual Studio 2019.

1. En la ventana Inicio, seleccione **Crear un proyecto**. 

1. En la ventana **Crear un proyecto**, elija **Aplicación de consola (.NET Framework)** y seleccione **Siguiente**.

1. En la ventana **Configure su nuevo proyecto**, escriba *helloworld* en **Nombre del proyecto**, elija o cree la ruta de acceso del directorio en **Ubicación** y seleccione **Crear**.

1. En la barra de menús de Visual Studio, seleccione **Herramientas** > **Obtener herramientas y características**, que abre el Instalador de Visual Studio y muestra el cuadro de diálogo **Modificando**.

1. Compruebe si la carga de trabajo **Desarrollo de escritorio de .NET** está disponible. Si la carga de trabajo aún no se ha instalado, active la casilla que hay al lado y seleccione **Modificar** para iniciar la instalación. La descarga e instalación pueden tardar unos minutos.

   Si la casilla que está junto a **Desarrollo de escritorio de .NET** ya está seleccionada, seleccione **Cerrar** para salir del cuadro de diálogo.

   ![Habilitación del desarrollo de escritorio .NET](../articles/cognitive-services/speech-service/media/sdk/vs-enable-net-desktop-workload.png)

1. Cierre el Instalador de Visual Studio.

### <a name="install-the-speech-sdk"></a>Instalación de Speech SDK

El siguiente paso consiste en instalar el [paquete NuGet del SDK de Voz](https://aka.ms/csspeech/nuget) para que pueda hacer referencia a él en el código.

1. En el Explorador de soluciones, haga clic con el botón derecho en el proyecto **helloworld** y seleccione **Administrar paquetes NuGet** para mostrar el Administrador de paquetes NuGet.

   ![Administrador de paquetes de NuGet](../articles/cognitive-services/speech-service/media/sdk/vs-nuget-package-manager.png)

1. En la esquina superior derecha, busque el cuadro desplegable **Origen del paquete** y asegúrese de que **nuget.org** está seleccionado.

1. En la esquina superior izquierda, seleccione **Examinar**.

1. En el cuadro de búsqueda, escriba *Microsoft.CognitiveServices.Speech* y seleccione **Entrar**.

1. En los resultados de la búsqueda, seleccione el paquete **Microsoft.CognitiveServices.Speech** y, después, seleccione **Instalar** para instalar la versión estable más reciente.

   ![Instalación del paquete NuGet Microsoft.CognitiveServices.Speech](../articles/cognitive-services/speech-service/media/sdk/qs-csharp-dotnet-windows-03-nuget-install-1.0.0.png)

1. Acepte todos los contratos y licencias para iniciar la instalación.

   Después de instalar el paquete aparecerá una confirmación en la ventana **Consola del administrador de paquetes**.

### <a name="choose-the-target-architecture"></a>Elección de la arquitectura de destino

Ahora, para compilar y ejecutar la aplicación de consola, cree una configuración de plataforma que coincida con la arquitectura del equipo.

1. En la barra de menús, seleccione **Compilar** > **Administrador de configuración**. Aparecerá el cuadro de diálogo **Administrador de configuración**.

   ![Cuadro de diálogo Administrador de configuración](../articles/cognitive-services/speech-service/media/sdk/vs-configuration-manager-dialog-box.png)

1. En el cuadro desplegable **Active solution platform** (Plataforma de soluciones activas), seleccione **Nuevo**. Aparecerá el cuadro de diálogo **Nueva plataforma de solución**.

1. En el cuadro desplegable **escriba o seleccione la nueva plataforma**:
   - Si está ejecutando Windows de 64 bits, seleccione **x64**.
   - Si está ejecutando Windows de 32 bits, seleccione **x86**.

1. Seleccione **Aceptar** y, después, **Cerrar**.
