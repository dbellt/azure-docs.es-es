---
title: 'Inicio rápido: Creación de una aplicación HoloLens con DirectX'
description: En este inicio rápido, aprenderá a compilar una aplicación HoloLens con Object Anchors.
author: craigktreasure
manager: virivera
ms.author: crtreasu
ms.date: 02/02/2021
ms.topic: quickstart
ms.service: azure-object-anchors
ms.openlocfilehash: 05c131abf23ac060db24b19b6affbeb5e632468b
ms.sourcegitcommit: 190658142b592db528c631a672fdde4692872fd8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112004770"
---
# <a name="quickstart-create-a-hololens-app-with-azure-object-anchors-in-cwinrt-and-directx"></a>Inicio rápido: Creación de una aplicación HoloLens con Azure Object Anchors en C++/WinRT y DirectX

En este inicio rápido, se describe cómo crear una aplicación HoloLens con [Azure Object Anchors](../overview.md) en C++/WinRT y DirectX. Azure Object Anchors es un servicio en la nube administrado que convierte recursos 3D en modelos de inteligencia artificial (IA) que permiten experiencias de realidad mixta que reconocen objetos para HoloLens. Cuando haya terminado, tendrá una aplicación de HoloLens que puede detectar un objeto y su posición en una aplicación de Holographic DirectX 11 (Windows universal).

Aprenderá a:

> [!div class="checklist"]
> * Crear y cargar una aplicación de HoloLens.
> * Detectar un objeto y visualizar su modelo.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prerrequisitos

Para completar esta guía de inicio rápido, asegúrese de que dispone de lo siguiente:

* Un objeto físico de su entorno y su modelo 3D (ya sea CAD o digitalizado).
* Una máquina Windows que tenga instalado lo siguiente:
  * <a href="https://git-scm.com" target="_blank">Git para Windows</a>
  * <a href="https://www.visualstudio.com/downloads/" target="_blank">Visual Studio 2019</a> con la carga de trabajo de **desarrollo de Plataforma universal de Windows** y el componente **Windows 10 SDK (10.0.18362.0 o versiones posteriores)**
* Un dispositivo HoloLens 2 actualizado y con el [modo de desarrollador](/windows/mixed-reality/using-visual-studio#enabling-developer-mode) habilitado.
  * Para actualizar a la versión más reciente en HoloLens, abra la aplicación **Settings** (Configuración), vaya a **Update & Security** (Actualización y seguridad) y seleccione **Check for updates** (Buscar actualizaciones).

## <a name="open-the-sample-project"></a>Apertura del proyecto de ejemplo

[!INCLUDE [Clone Sample Repo](../../../includes/object-anchors-clone-sample-repository.md)]

Abra `quickstarts/apps/directx/DirectXAoaSampleApp.sln` en Visual Studio.

Cambie **Configuración de la solución** a **Versión**, cambie **Plataforma de la solución** a **ARM64** y seleccione **Dispositivo** en las opciones de destino de implementación. A continuación, compile el proyecto **AoaSampleApp**. Para ello, haga clic con el botón derecho en el proyecto y seleccione **Compilar**.

:::image type="content" source="./media/vs-deploy-to-device.png" alt-text="Configuración del proyecto de Visual Studio que se va a implementar":::

## <a name="deploy-the-app-to-hololens"></a>Implementación de la aplicación en HoloLens

Después de compilar el proyecto de ejemplo correctamente, puede implementar la aplicación en HoloLens.

Encienda el dispositivo HoloLens, inicie sesión y conéctelo al equipo mediante un cable USB. Asegúrese de que la opción de destino de implementación seleccionada sea **Dispositivo** (consulte más arriba).

Haga clic con el botón derecho en el proyecto **AoaSampleApp** y, luego, haga clic en **Implementar** en el menú emergente para instalar la aplicación. Si no se muestra ningún error en la **Ventana de salida** de Visual Studio, la aplicación se instalará en HoloLens.

:::image type="content" source="./media/vs-deploy-app.png" alt-text="Implementación de la aplicación en HoloLens":::

Antes de iniciar la aplicación, debe cargar un modelo de objetos. Siga las instrucciones que aparecen en la sección **Ingesta de un modelo de objetos y detección de su instancia** a continuación.

Para iniciar y depurar la aplicación, seleccione **Depurar > Iniciar la depuración**. Para detener la aplicación, seleccione **Detener la depuración** o presione **Mayús + F5**.

## <a name="ingest-object-model-and-detect-its-instance"></a>Ingesta de un modelo de objetos y detección de su instancia

Deberá crear un modelo de objetos para ejecutar la aplicación de ejemplo. Supongamos que ya tiene un modelo de malla 3D, ya sea CAD o digitalizado, de un objeto en su espacio. Consulte el [Inicio rápido: Ingesta de un modelo 3D](./get-started-model-conversion.md) para instrucciones sobre cómo crear un modelo.

Descargue ese modelo, **chair.ou** en este caso, a su equipo. Luego, en el portal de dispositivos HoloLens, seleccione **Sistema > Explorador de archivos > LocalAppData > AoaSampleApp > LocalState** y seleccione **Examinar…** . Luego, seleccione su archivo de modelo, por ejemplo, **chair.ou**, y seleccione **Cargar**. Debería ver el archivo de modelo en la caché local.

:::image type="content" source="../../../includes/media/object-anchors-quickstarts/portal-upload-model.png" alt-text="Modelo de carga en el portal":::

En el dispositivo HoloLens, inicie la aplicación **AoaSampleApp**. Si ya está abierta, ciérrela y vuelva a abrirla. Camine cerca, a unos 2 m de distancia, del objeto de destino (silla) y mírelo desde varias perspectivas para digitalizarlo. Debería ver un rectángulo delimitador de color rosa alrededor del objeto con algunos puntos amarillos representados cerca de la superficie del objeto, lo que indica que se detectó.

:::image type="content" source="./media/chair-detection.png" alt-text="Detección de la silla":::

Figura: una silla detectada representada con su rectángulo delimitador (rosa), su nube de puntos (amarillos) y un área de búsqueda (rectángulo grande amarillo).

Para definir un espacio de búsqueda para el objeto en la aplicación, haga clic en el aire con un dedo de la mano derecha o de la mano izquierda. El espacio de búsqueda cambiará entre una esfera con un radio de 2 metros, un rectángulo delimitador de 4 m^3 y un tronco de vista. En el caso de objetos más grandes, como automóviles, la mejor opción será usar la selección del tronco de la vista mientras se observa de pie una esquina del objeto a unos 2 metros de distancia.
Cada vez que el área de búsqueda cambia, la aplicación quitará las instancias de las que se realiza el seguimiento actualmente y, a continuación, intentará volver a encontrarlas en el área de búsqueda nueva.

Esta aplicación puede realizar un seguimiento de varios objetos al mismo tiempo. Para ello, cargue varios modelos en la carpeta **LocalState** y establezca un área de búsqueda que abarque todos los objetos de destino. Puede tardar más tiempo en detectar y realizar el seguimiento de varios objetos.

La aplicación alinea estrechamente un modelo 3D con su homólogo físico. Para activar el modo de seguimiento de alta precisión, que calcula una posición más exacta, el usuario puede hacer un movimiento de clic con la mano izquierda en el aire. Esta sigue siendo una característica experimental, que consumo más recursos del sistema y puede generar una mayor inestabilidad en la posición estimada. Vuelva a hacer un movimiento de clic con la mano izquierda en el aire para volver al modo de seguimiento habitual.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Inicio rápido: Ingesta de un modelo 3D](./get-started-model-conversion.md)

> [!div class="nextstepaction"]
> [Conceptos: Información general del SDK](../concepts/sdk-overview.md)

> [!div class="nextstepaction"]
> [P+F](../faq.md)

> [!div class="nextstepaction"]
> [SDK de conversión](/dotnet/api/overview/azure/mixedreality.objectanchors.conversion-readme-pre)
