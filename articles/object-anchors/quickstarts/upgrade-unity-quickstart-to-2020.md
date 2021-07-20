---
title: 'Inicio rápido: Actualización de la aplicación de inicio rápido a Unity 2020'
description: En este inicio rápido, aprenderá a actualizar la aplicación de inicio rápido a Unity 2020 para crear una aplicación de HoloLens en Unity con Object Anchors.
author: RamonArguelles
manager: virivera
ms.author: rgarcia
ms.date: 06/23/2021
ms.topic: quickstart
ms.service: azure-object-anchors
ms.openlocfilehash: 6ee047cd39eb092c55ed417995bf1e9df6134c5e
ms.sourcegitcommit: 8b38eff08c8743a095635a1765c9c44358340aa8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/30/2021
ms.locfileid: "113095242"
---
# <a name="quickstart-upgrade-quickstart-app-to-unity-2020"></a>Inicio rápido: Actualización de la aplicación de inicio rápido a Unity 2020

En este inicio rápido, actualizará una aplicación de HoloLens en Unity que usa [Azure Object Anchors](../overview.md) de Unity 2019 a Unity 2020. Azure Object Anchors es un servicio en la nube administrado que convierte recursos 3D en modelos de inteligencia artificial (IA) que permiten experiencias de realidad mixta que reconocen objetos para HoloLens. Cuando haya terminado, tendrá una aplicación de HoloLens compilada con Unity, que puede detectar objetos en el mundo físico.

Aprenderá a:

> [!div class="checklist"]
> * Actualizar la aplicación de Unity 2019 a Unity 2020.
> * Actualizar las dependencias de paquetes.
> * Actualizar la configuración de compilación de Unity.

## <a name="prerequisites"></a>Prerrequisitos

Para completar esta guía de inicio rápido, asegúrese de que dispone de lo siguiente:

* Todos los requisitos previos de los inicios rápidos [Inicio rápido: Creación de una aplicación de HoloLens con Azure Object Anchors, en Unity](get-started-unity-hololens.md) e [Inicio rápido: Creación de una aplicación de HoloLens con Azure Object Anchors, en Unity con MRTK](get-started-unity-hololens-mrtk.md).
* <a href="https://unity3d.com/get-unity/download" target="_blank">Unity Hub con Unity 2020.3.8f1 o posterior</a>

## <a name="open-and-upgrade-the-sample-project"></a>Apertura y actualización del proyecto de ejemplo

Siga los pasos de los inicios rápidos [Inicio rápido: Creación de una aplicación de HoloLens con Azure Object Anchors, en Unity](get-started-unity-hololens.md) o [Inicio rápido: Creación de una aplicación de HoloLens con Azure Object Anchors, en Unity con MRTK](get-started-unity-hololens-mrtk.md) para clonar el [repositorio de ejemplos](https://github.com/Azure/azure-object-anchors) y descargar el paquete de Azure Object Anchors para Unity.

Abra Unity Hub. Seleccione el botón **Add** (Agregar) y elija el proyecto `quickstarts/apps/unity/basic` o el proyecto `quickstarts/apps/unity/mrtk`. A continuación, en la columna **Unity Version** (Versión de Unity), seleccione en la lista desplegable la versión Unity 2020 que ha instalado en la máquina. En la columna **Target Platform** (Plataforma de destino), seleccione **Universal Windows Platform** (Plataforma universal de Windows). Por último, seleccione la columna **Project Name** (Nombre de proyecto) y abra el ejemplo en Unity.

:::image type="content" source="./media/upgrade-unity-2020.png" alt-text="Actualización a Unity 2020":::

Verá un cuadro de diálogo que le solicita confirmación para actualizar el proyecto. Seleccione el botón **Confirm** (Confirmar).

:::image type="content" source="./media/confirm-unity-upgrade.png" alt-text="Confirmar actualización de Unity":::

## <a name="upgrade-package-dependencies"></a>Actualización de las dependencias de paquetes

Una vez completado el proceso de actualización, se abrirá el **Editor Unity**.

Consulte la documentación de la <a a href="/windows/mixed-reality/develop/unity/welcome-to-mr-feature-tool" target="_blank">herramienta de características de Mixed Reality</a> para configurar la herramienta y aprender a usarla.

En la sección **Platform Support** (Compatibilidad con la plataforma), instale el paquete de características del **complemento OpenXR de Mixed Reality**, versión 1.0.0 o posterior, en la carpeta del proyecto de Unity. Si está trabajando con el proyecto `quickstarts/apps/unity/mrtk`, abra también la sección **Mixed Reality Toolkit** (Kit de herramientas de Mixed Reality), busque los paquetes de características **Mixed Reality Toolkit Foundation** y **Mixed Reality Toolkit Tools** y actualícelos a la versión 2.7.0 o posterior.

Vuelva al **Editor Unity**. Pueden transcurrir unos minutos mientras se instalan los paquetes de características de la **herramienta de características de Mixed Reality**.

Verá un cuadro de diálogo que solicita confirmación para habilitar el nuevo sistema de entrada. Haga clic en el botón **Sí**.

:::image type="content" source="./media/new-input-system.png" alt-text="Nuevo sistema de entrada":::

 Si aparece un cuadro de diálogo que le solicita que sobrescriba los sombreadores de MRTK, seleccione **Yes** (Sí).

:::image type="content" source="./media/mrtk-shaders.png" alt-text="Sombreadores de MRTK":::

Una vez completado el proceso de instalación, Unity se reiniciará automáticamente.

## <a name="update-configuration-settings"></a>Actualizar valores de configuración

De nuevo en el **Editor Unity**, siga la documentación <a href="/windows/mixed-reality/develop/unity/xr-project-setup#configuring-xr-plugin-management-for-openxr" target="_blank">Configuración de administración de complementos XR para OpenXR</a> para configurar **XR Plugin Management** (Administración de complementos XR) en **Project Settings** (Configuración del proyecto). A continuación, siga la documentación <a href="/windows/mixed-reality/develop/unity/xr-project-setup#optimization" target="_blank">Optimización</a> para aplicar la configuración recomendada del proyecto para HoloLens 2.

## <a name="update-mrtk-settings"></a>Actualización de la configuración de MRTK

Si está trabajando con el proyecto `quickstarts/apps/unity/mrtk`, MRTK también necesitará algunos ajustes. En ese caso, siga los pasos que se indican a continuación. De lo contrario, vaya a la sección **Compilación, implementación y ejecución de la aplicación**.

En el **Editor Unity**, vaya a `Assets/MixedReality.AzureObjectAnchors/Scenes` y abra **AOASampleScene**. En el panel **Hierarchy** (Jerarquía), seleccione el objeto **MixedRealityToolkit**.

:::image type="content" source="./media/open-sample-scene.png" alt-text="Abrir escena de ejemplo":::

En el panel **Inspector**, seleccione el botón **Camera** (Cámara) y cambie el perfil de **ObsoleteXRSDKCameraProfile** a **DefaultMixedRealityCameraProfile**.

:::image type="content" source="./media/update-camera-profile.png" alt-text="Actualizar el perfil de la cámara":::

Todavía en el panel **Inspector**, seleccione el botón **Input** (Entrada) y expanda el elemento desplegable **Input Data Providers** (Proveedores de datos de entrada). A continuación, siga la documentación <a href="/windows/mixed-reality/mrtk-unity/configuration/getting-started-with-mrtk-and-xrsdk#configuring-mrtk-for-the-xr-sdk-pipeline" target="_blank">Configuración de MRTK para la canalización del SDK de XR</a> para configurar los proveedores de datos de entrada adecuados (**OpenXRDeviceManager** y **WindowsMixedRealityDeviceManager**).

:::image type="content" source="./media/update-input-profile.png" alt-text="Actualizar perfil de entrada":::

## <a name="build-deploy-and-run-the-app"></a>Compilación, implementación y ejecución de la aplicación

El proyecto ahora está totalmente actualizado a Unity 2020. Siga las instrucciones de los inicios rápidos [Inicio rápido: Creación de una aplicación de HoloLens con Azure Object Anchors, en Unity](get-started-unity-hololens.md) o [Inicio rápido: Creación de una aplicación de HoloLens con Azure Object Anchors, en Unity con MRTK](get-started-unity-hololens-mrtk.md) para compilar, implementar y ejecutar la aplicación.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Conceptos: Información general del SDK](../concepts/sdk-overview.md)

> [!div class="nextstepaction"]
> [P+F](../faq.md)

> [!div class="nextstepaction"]
> [Biblioteca cliente de Azure Object Anchors para .NET: versión 0.3.0-beta.1](/dotnet/api/overview/azure/mixedreality.objectanchors.conversion-readme-pre)
