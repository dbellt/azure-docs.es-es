---
title: 'Inicio rápido: Creación de una aplicación HoloLens con Unity'
description: En este inicio rápido, aprenderá a crear una aplicación HoloLens en Unity con Spatial Anchors.
author: msftradford
manager: MehranAzimi-msft
services: azure-spatial-anchors
ms.author: parkerra
ms.date: 03/18/2021
ms.topic: quickstart
ms.service: azure-spatial-anchors
ms.openlocfilehash: c0cce7717dda73a381ec977c3bf520bf8db5bbb3
ms.sourcegitcommit: 5f482220a6d994c33c7920f4e4d67d2a450f7f08
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/08/2021
ms.locfileid: "107105874"
---
# <a name="quickstart-create-a-unity-hololens-app-that-uses-azure-spatial-anchors"></a>Inicio rápido: Creación de una aplicación HoloLens en Unity que use Azure Spatial Anchors

En este inicio rápido se creará una aplicación HoloLens en Unity con [Azure Spatial Anchors](../overview.md). Spatial Anchors es un servicio multiplataforma para desarrolladores que permite crear experiencias de realidad mixta con objetos cuya ubicación persiste en todos los dispositivos a lo largo del tiempo. Cuando haya terminado, tendrá una aplicación HoloLens compilada con Unity que puede guardar y recuperar un delimitador espacial.

Aprenderá a:

- Crear una cuenta de Spatial Anchors.
- Preparar la configuración de compilación de Unity.
- Configurar la clave y el identificador de la cuenta de Spatial Anchors.
- Exportar el proyecto HoloLens de Visual Studio.
- Implementar la aplicación y ejecutarla en un dispositivo HoloLens.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prerrequisitos

Para completar esta guía de inicio rápido:

- Necesita un equipo Windows que tenga instalados <a href="https://unity3d.com/get-unity/download" target="_blank">Unity (LTS)</a> y <a href="https://www.visualstudio.com/downloads/" target="_blank">Visual Studio 2019</a>, o cualquier versión posterior. Use **Unity 2020 LTS** con la versión 2.9 del SDK de ASA, o cualquier versión posterior (que usa el complemento [Unity XR Plug-in Framework](https://docs.unity3d.com/Manual/XRPluginArchitecture.html)), o **Unity 2019 LTS** con la versión 2.8 del SDK de ASA, o cualquier versión anterior. La instalación de Visual Studio debe incluir la carga de trabajo de **desarrollo de la Plataforma universal de Windows** y el componente **SDK de Windows 10 (10.0.18362.0 o posterior)** . También se debe instalar <a href="https://git-scm.com/download/win" target="_blank">Git para Windows</a> y <a href="https://git-lfs.github.com/">Git LFS</a>.
- Se necesita un dispositivo HoloLens en el que esté habilitado el [modo de desarrollador](/windows/mixed-reality/using-visual-studio). La [actualización de Windows 10 de mayo de 2020](/windows/mixed-reality/whats-new/release-notes-may-2020) se debe instalar en el dispositivo. Para actualizar a la versión más reciente en HoloLens, abra la aplicación **Settings** (Configuración), vaya a **Update & Security** (Actualización y seguridad) y seleccione **Check for updates** (Buscar actualizaciones).
- En la aplicación, debe habilitar la funcionalidad **SpatialPerception**. Este valor está en **Build Settings** > **Player Settings** > **Publishing Settings** > **Capabilities** (Configuración de compilación > Configuración del reproductor > Configuración de publicación > Funcionalidades).
- En la aplicación, es preciso habilitar **Virtual Reality Supported** (Admite realidad virtual) con el **SDK de Windows Mixed Reality**. Este valor está en **Build Settings** > **Player Settings** > **XR Settings** (Configuración de compilación > Configuración del reproductor > Configuración de XR).

[!INCLUDE [Create Spatial Anchors resource](../../../includes/spatial-anchors-get-started-create-resource.md)]

## <a name="download-and-open-the-unity-sample-project"></a>Descarga y apertura del proyecto de Unity de ejemplo

[!INCLUDE [Clone Sample Repo](../../../includes/spatial-anchors-clone-sample-repository.md)]

Siga las instrucciones que se proporcionan [aquí](../how-tos/setup-unity-project.md#download-asa-packages) para descargar e importar los paquetes del SDK de ASA necesarios para la plataforma HoloLens.

[!INCLUDE [Open Unity Project](../../../includes/spatial-anchors-open-unity-project.md)]

Seleccione **File** > **Build Settings** (Archivo > Configuración de compilación) para abrir **Build Settings** (Configuración de compilación).

En la sección **Platform** (Plataforma), seleccione **Universal Windows Platform** (Plataforma universal de Windows). Cambie el valor de **Target Device** (Dispositivo de destino) a **HoloLens**.

Seleccione **Switch Platform** (Cambiar plataforma) para cambiar la plataforma a **Universal Windows Platform** (Plataforma universal de Windows). Unity puede solicitarle que instale los componentes de compatibilidad con Plataforma universal de Windows si no están presentes.

![Ventana Build Settings (Configuración de compilación) de Unity](./media/get-started-unity-hololens/unity-build-settings.png)

Cierre la ventana **Build Settings** (Configuración de compilación).

[!INCLUDE [Configure Unity Scene](../../../includes/spatial-anchors-unity-configure-scene.md)]

## <a name="export-the-hololens-visual-studio-project"></a>Exportar el proyecto de HoloLens de Visual Studio

[!INCLUDE [Export Unity Project](../../../includes/spatial-anchors-unity-export-project-snip.md)]

Seleccione **Build** (Compilar). En el cuadro de diálogo, seleccione la carpeta en la que se va a exportar el proyecto HoloLens de Visual Studio.

Cuando se complete la exportación, aparecerá una carpeta que contiene el proyecto HoloLens exportado.

## <a name="deploy-the-hololens-application"></a>Implementación de la aplicación HoloLens

En la carpeta, haga doble clic en **HelloAR U3D.sln** para abrir el proyecto en Visual Studio.

Cambie el valor de **Configuración de solución** a **Versión**, cambie el de **Plataforma de solución** a **x86** y seleccione **Dispositivo** en las opciones de destino de implementación.

Si usa HoloLens 2, utilice **ARM64** como **Plataforma de soluciones**, en lugar de **x86**.

   ![Configuración de Visual Studio](./media/get-started-unity-hololens/visual-studio-configuration.png)

Encienda el dispositivo HoloLens, inicie sesión y conéctelo al equipo mediante un cable USB.

Seleccione **Depurar** > **Iniciar depuración** para implementar la aplicación y comenzar la depuración.

En la aplicación, seleccione **BasicDemo** mediante las flechas y pulse el botón **Go!** para ejecutar la demostración. Siga las instrucciones para colocar y recuperar un delimitador.

![Captura de pantalla 1](./media/get-started-unity-hololens/screenshot-1.jpg)
![Captura de pantalla 2](./media/get-started-unity-hololens/screenshot-2.jpg)
![Captura de pantalla 3](./media/get-started-unity-hololens/screenshot-3.jpg)
![Captura de pantalla 4](./media/get-started-unity-hololens/screenshot-4.jpg)

En Visual Studio, seleccione **Detener depuración** o presione Mayús + F5 para detener la depuración.

[!INCLUDE [Clean-up section](../../../includes/clean-up-section-portal.md)]

[!INCLUDE [Next steps](../../../includes/spatial-anchors-quickstarts-nextsteps.md)]

> [!div class="nextstepaction"]
> [Tutorial: Uso compartido de Spatial Anchors entre dispositivos](../tutorials/tutorial-share-anchors-across-devices.md)

> [!div class="nextstepaction"]
> [Cómo: configurar Azure Spatial Anchors en un proyecto de Unity](../how-tos/setup-unity-project.md)