---
title: 'Inicio rápido: Creación de una aplicación HoloLens con Unity'
description: En este inicio rápido, aprenderá a compilar una aplicación HoloLens en Unity con Object Anchors.
author: craigktreasure
manager: virivera
ms.author: crtreasu
ms.date: 03/02/2021
ms.topic: quickstart
ms.service: azure-object-anchors
ms.openlocfilehash: 2c96f1392228c4fcce65e05711b70a6ce689592a
ms.sourcegitcommit: 190658142b592db528c631a672fdde4692872fd8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112006905"
---
# <a name="quickstart-create-a-hololens-app-with-azure-object-anchors-in-unity"></a>Inicio rápido. Creación de una aplicación de HoloLens con Azure Object Anchors, en Unity

En este artículo de inicio rápido se creará una aplicación de HoloLens en Unity con [Azure Object Anchors](../overview.md). Azure Object Anchors es un servicio en la nube administrado que convierte recursos 3D en modelos de inteligencia artificial (IA) que permiten experiencias de realidad mixta que reconocen objetos para HoloLens. Cuando haya terminado, tendrá una aplicación de HoloLens compilada con Unity, que puede detectar objetos en el mundo físico.

Aprenderá a:

> [!div class="checklist"]
> * Preparar la configuración de compilación de Unity.
> * Exportar el proyecto HoloLens de Visual Studio.
> * Implementar la aplicación y ejecutarla en un dispositivo HoloLens 2.

[!INCLUDE [Unity quickstart prerequisites](../../../includes/object-anchors-quickstart-unity-prerequisites.md)]

[!INCLUDE [Unity device setup](../../../includes/object-anchors-quickstart-unity-device-setup.md)]

## <a name="open-the-sample-project"></a>Apertura del proyecto de ejemplo

[!INCLUDE [Clone Sample Repo](../../../includes/object-anchors-clone-sample-repository.md)]

[!INCLUDE [Download Unity Package](../../../includes/object-anchors-quickstart-unity-download-package.md)]

En Unity, abra el proyecto `quickstarts/apps/unity/basic`.

[!INCLUDE [Import Unity Package](../../../includes/object-anchors-quickstart-unity-import-package.md)]

[!INCLUDE [Unity build sample scene 1](../../../includes/object-anchors-quickstart-unity-build-sample-scene-1.md)]

[!INCLUDE [Unity build sample scene 2](../../../includes/object-anchors-quickstart-unity-build-sample-scene-2.md)]

[!INCLUDE [Unity build and deploy](../../../includes/object-anchors-quickstart-unity-build-deploy.md)]

### <a name="run-the-sample-app"></a>Ejecutar la aplicación de ejemplo

Encienda el dispositivo, seleccione **All Apps** (Todas las aplicaciones) y, a continuación, busque e inicie la aplicación. Después de la pantalla de presentación de Unity, verá un mensaje que indica que se inicializó el observador de objeto. Sin embargo, deberá agregar el modelo a la aplicación.

[!INCLUDE [Unity setup Windows Device Portal](../../../includes/object-anchors-quickstart-unity-setup-device-portal.md)]

[!INCLUDE [Unity upload your model](../../../includes/object-anchors-quickstart-unity-upload-model.md)]

La aplicación busca objetos en el campo visual actual y, a continuación, realiza un seguimiento de ellos una vez detectados. Una instancia se quitará cuando esté a 6 metros de la ubicación del usuario. El texto de depuración muestra detalles sobre una instancia, como el identificador, la marca de tiempo actualizada y la relación de cobertura de superficie.

[!INCLUDE [Unity troubleshooting](../../../includes/object-anchors-quickstart-unity-troubleshooting.md)]

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [HoloLens para Unity con MRTK](get-started-unity-hololens-mrtk.md)

> [!div class="nextstepaction"]
> [Conceptos: Información general del SDK](../concepts/sdk-overview.md)

> [!div class="nextstepaction"]
> [P+F](../faq.md)

> [!div class="nextstepaction"]
> [SDK de conversión](/dotnet/api/overview/azure/mixedreality.objectanchors.conversion-readme-pre)
