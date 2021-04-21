---
title: 'Inicio rápido: Creación de una aplicación para Unity Android'
description: En este tutorial, aprenderá a compilar una aplicación Android con Unity mediante Spatial Anchors.
author: msftradford
manager: MehranAzimi-msft
services: azure-spatial-anchors
ms.author: parkerra
ms.date: 03/18/2021
ms.topic: quickstart
ms.service: azure-spatial-anchors
ms.openlocfilehash: 4f4bfdb5d09da6a9967b53ca56bb2491976592a1
ms.sourcegitcommit: 5f482220a6d994c33c7920f4e4d67d2a450f7f08
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/08/2021
ms.locfileid: "107105891"
---
# <a name="quickstart-create-a-unity-android-app-with-azure-spatial-anchors"></a>Inicio rápido: Creación de una aplicación para Unity Android con Azure Spatial Anchors

En este inicio rápido se describe cómo crear una aplicación para Unity Android con [Azure Spatial Anchors](../overview.md). Azure Spatial Anchors es un servicio multiplataforma para desarrolladores que le permite crear experiencias de realidad mixta mediante objetos cuya ubicación persiste en todos los dispositivos a lo largo del tiempo. Cuando haya terminado, tendrá una aplicación Android de ARCore compilada con Unity que puede guardar y recuperar un delimitador espacial.

Aprenderá a:

> [!div class="checklist"]
> * Crear una cuenta de Spatial Anchors
> * Preparar la configuración de compilación de Unity
> * Configurar la clave y el identificador de la cuenta de Spatial Anchors
> * Exportación del proyecto de Android Studio
> * Realizar la implementación y ejecución en un dispositivo Android

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Requisitos previos

Para completar esta guía de inicio rápido, asegúrese de que dispone de lo siguiente:

- Una máquina con Windows o macOS con <a href="https://unity3d.com/get-unity/download" target="_blank">Unity (LTS)</a>, incluidos los módulos **Android Build Support** con **Android SDK & NDK Tools** y **OpenJDK**. Use **Unity 2020 LTS** con la versión 2.9 del SDK de ASA, o cualquier versión posterior (que usa el complemento [Unity XR Plug-in Framework](https://docs.unity3d.com/Manual/XRPluginArchitecture.html)), o **Unity 2019 LTS** con la versión 2.8 del SDK de ASA, o cualquier versión anterior.
  - Si se ejecuta en Windows, también necesitará <a href="https://git-scm.com/download/win" target="_blank">Git para Windows</a> y <a href="https://git-lfs.github.com/">Git LFS</a>.
  - Si se ejecuta en macOS, instale Git a través de HomeBrew. Escriba el siguiente comando en una sola línea de Terminal: `/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"`. A continuación, ejecute `brew install git` y `brew install git-lfs`.
- Un dispositivo Android <a href="https://developer.android.com/studio/debug/dev-options" target="_blank">habilitado para el desarrollo</a> y <a href="https://developers.google.com/ar/discover/supported-devices" target="_blank">preparado para ARCore</a>.
  - Puede que se necesiten controladores de dispositivo adicionales para que el equipo se comunique con el dispositivo Android. [Aquí](https://developer.android.com/studio/run/device.html) encontrará más información e instrucciones.

[!INCLUDE [Create Spatial Anchors resource](../../../includes/spatial-anchors-get-started-create-resource.md)]

## <a name="download-and-open-the-unity-sample-project"></a>Descarga y apertura del proyecto de Unity de ejemplo

[!INCLUDE [Clone Sample Repo](../../../includes/spatial-anchors-clone-sample-repository.md)]

Siga las instrucciones que se proporcionan [aquí](../how-tos/setup-unity-project.md#download-asa-packages) para descargar e importar los paquetes del SDK de ASA necesarios para la plataforma Android.

[!INCLUDE [Open Unity Project](../../../includes/spatial-anchors-open-unity-project.md)]

[!INCLUDE [Android Unity Build Settings](../../../includes/spatial-anchors-unity-android-build-settings.md)]

[!INCLUDE [Configure Unity Scene](../../../includes/spatial-anchors-unity-configure-scene.md)]

## <a name="export-the-android-studio-project"></a>Exportación del proyecto de Android Studio

[!INCLUDE [Export Unity Project](../../../includes/spatial-anchors-unity-export-project-snip.md)]

Seleccione el dispositivo en **Run Device** (Dispositivo de ejecución) y, después, seleccione **Build And Run** (Compilar y ejecutar). Se le pedirá que guarde un archivo `.apk` para el que puede elegir cualquier nombre.

En la aplicación, seleccione **BasicDemo** mediante las flechas y pulse el botón **Go!** para ejecutar la demostración. Siga las instrucciones para colocar y recuperar un delimitador.

![Captura de pantalla 1](./media/get-started-unity-android/screenshot-1.jpg)
![Captura de pantalla 2](./media/get-started-unity-android/screenshot-2.jpg)
![Captura de pantalla 3](./media/get-started-unity-android/screenshot-3.jpg)

Siga las instrucciones de la aplicación para colocar y recuperar un delimitador.

## <a name="troubleshooting"></a>Solución de problemas

### <a name="rendering-issues"></a>Problemas de representación

Al ejecutar la aplicación, si no ve la cámara como fondo (en cambio ve, por ejemplo, una textura blanca, azul o de otro tipo), es probable que deba volver a importar los recursos en Unity. Detenga la aplicación. En el menú superior de Unity, elija **Assets -> Reimport all** (Recursos -> Volver a importar todo). Luego, vuelva a ejecutar la aplicación.

[!INCLUDE [Clean-up section](../../../includes/clean-up-section-portal.md)]

[!INCLUDE [Next steps](../../../includes/spatial-anchors-quickstarts-nextsteps.md)]

> [!div class="nextstepaction"]
> [Tutorial: Uso compartido de Spatial Anchors entre dispositivos](../tutorials/tutorial-share-anchors-across-devices.md)

> [!div class="nextstepaction"]
> [Cómo: configurar Azure Spatial Anchors en un proyecto de Unity](../how-tos/setup-unity-project.md)
