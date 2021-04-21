---
title: 'Inicio rápido: Creación de una aplicación de Unity para iOS'
description: En este inicio rápido, aprenderá a compilar una aplicación iOS con Unity mediante Spatial Anchors.
author: msftradford
manager: MehranAzimi-msft
services: azure-spatial-anchors
ms.author: parkerra
ms.date: 03/18/2021
ms.topic: quickstart
ms.service: azure-spatial-anchors
ms.openlocfilehash: b341237d0e703239233f7ac0e9664f5fb90bbb4b
ms.sourcegitcommit: 5f482220a6d994c33c7920f4e4d67d2a450f7f08
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/08/2021
ms.locfileid: "107105820"
---
# <a name="quickstart-create-a-unity-ios-app-with-azure-spatial-anchors"></a>Inicio rápido: Creación de una aplicación de Unity para iOS con Azure Spatial Anchors

En este inicio rápido se describe cómo crear una aplicación de Unity para iOS con [Azure Spatial Anchors](../overview.md). Azure Spatial Anchors es un servicio multiplataforma para desarrolladores que le permite crear experiencias de realidad mixta mediante objetos cuya ubicación persiste en todos los dispositivos a lo largo del tiempo. Cuando haya terminado, tendrá una aplicación iOS de ARKit compilada con Unity que puede guardar y recuperar un delimitador espacial.

Aprenderá a:

> [!div class="checklist"]
> * Crear una cuenta de Spatial Anchors
> * Preparar la configuración de compilación de Unity
> * Configurar la clave y el identificador de la cuenta de Spatial Anchors
> * Exportación del proyecto de Xcode
> * Implementarlo y ejecutarlo en un dispositivo iOS

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prerrequisitos

Para completar esta guía de inicio rápido, asegúrese de que dispone de lo siguiente:

- Una máquina macOS en que estén instalados <a href="https://geo.itunes.apple.com/us/app/xcode/id497799835?mt=12" target="_blank">Xcode</a> y <a href="https://unity3d.com/get-unity/download" target="_blank">Unity (LTS)</a>. Use **Unity 2020 LTS** con la versión 2.9 del SDK de ASA, o cualquier versión posterior (que usa el complemento [Unity XR Plug-in Framework](https://docs.unity3d.com/Manual/XRPluginArchitecture.html)), o **Unity 2019 LTS** con la versión 2.8 del SDK de ASA, o cualquier versión anterior.
- GIT instalado mediante HomeBrew. Escriba el siguiente comando en una sola línea de Terminal: `/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"`. A continuación, ejecute `brew install git` y `brew install git-lfs`.
- Un dispositivo iOS <a href="https://developer.apple.com/documentation/arkit/verifying_device_support_and_user_permission" target="_blank">compatible con ARKit</a> habilitado para el desarrollo.

[!INCLUDE [Create Spatial Anchors resource](../../../includes/spatial-anchors-get-started-create-resource.md)]

## <a name="download-and-open-the-unity-sample-project"></a>Descarga y apertura del proyecto de Unity de ejemplo

[!INCLUDE [Clone Sample Repo](../../../includes/spatial-anchors-clone-sample-repository.md)]

Siga las instrucciones que se proporcionan [aquí](../how-tos/setup-unity-project.md#download-asa-packages) para descargar e importar los paquetes del SDK de ASA necesarios para la plataforma iOS.

[!INCLUDE [Open Unity Project](../../../includes/spatial-anchors-open-unity-project.md)]

[!INCLUDE [iOS Unity Build Settings](../../../includes/spatial-anchors-unity-ios-build-settings.md)]

[!INCLUDE [Configure Unity Scene](../../../includes/spatial-anchors-unity-configure-scene.md)]

## <a name="export-the-xcode-project"></a>Exportación del proyecto de Xcode

[!INCLUDE [Export Unity Project](../../../includes/spatial-anchors-unity-export-project-snip.md)]

[!INCLUDE [Configure Xcode](../../../includes/spatial-anchors-unity-ios-xcode.md)]

En la aplicación, seleccione **BasicDemo** mediante las flechas y pulse el botón **Go!** para ejecutar la demostración. Siga las instrucciones para colocar y recuperar un delimitador.

![Captura de pantalla 1](./media/get-started-unity-ios/screenshot-1.jpg)
![Captura de pantalla 2](./media/get-started-unity-ios/screenshot-2.jpg)
![Captura de pantalla 3](./media/get-started-unity-ios/screenshot-3.jpg)

Cuando termine, presione **Stop** (Detener) en Xcode para detener la aplicación.

## <a name="troubleshooting"></a>Solución de problemas

### <a name="rendering-issues"></a>Problemas de representación

Cuando se ejecuta la aplicación, si no ve la cámara como fondo (en cambio ve, por ejemplo, una textura negra, azul o de otro tipo), es probable que deba volver a importar los recursos en Unity. Detenga la aplicación. En el menú superior de Unity, elija **Assets -> Re-import all** (Recursos -> Volver a importar todo). Luego, vuelva a ejecutar la aplicación.

[!INCLUDE [Clean-up section](../../../includes/clean-up-section-portal.md)]

[!INCLUDE [Next steps](../../../includes/spatial-anchors-quickstarts-nextsteps.md)]

> [!div class="nextstepaction"]
> [Tutorial: Uso compartido de Spatial Anchors entre dispositivos](../tutorials/tutorial-share-anchors-across-devices.md)

> [!div class="nextstepaction"]
> [Cómo: configurar Azure Spatial Anchors en un proyecto de Unity](../how-tos/setup-unity-project.md)
