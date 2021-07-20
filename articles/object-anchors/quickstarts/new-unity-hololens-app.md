---
title: 'Inicio rápido: Creación de una nueva aplicación de HoloLens en Unity'
description: En este inicio rápido, aprenderá a crear una aplicación de HoloLens en Unity con Object Anchors.
author: RamonArguelles
manager: virivera
ms.author: rgarcia
ms.date: 06/23/2021
ms.topic: quickstart
ms.service: azure-object-anchors
ms.openlocfilehash: 174701b16d8ae9c8c8e5b1edf863aa9f1a65336c
ms.sourcegitcommit: 8b38eff08c8743a095635a1765c9c44358340aa8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/30/2021
ms.locfileid: "113095246"
---
# <a name="quickstart-step-by-step-instructions-to-create-a-new-hololens-unity-app-using-azure-object-anchors"></a>Inicio rápido: Instrucciones paso a paso para crear una nueva aplicación de HoloLens en Unity mediante Azure Object Anchors

En este inicio rápido, se muestra cómo crear una nueva aplicación de HoloLens en Unity con [Azure Object Anchors](../overview.md). Azure Object Anchors es un servicio en la nube administrado que convierte recursos 3D en modelos de inteligencia artificial (IA) que permiten experiencias de realidad mixta que reconocen objetos para HoloLens. Cuando haya terminado, tendrá una aplicación de HoloLens compilada con Unity, que puede detectar objetos en el mundo físico.

## <a name="prerequisites"></a>Prerrequisitos

Para completar esta guía de inicio rápido, asegúrese de que dispone de lo siguiente:

* Todos los requisitos previos de los inicios rápidos [Inicio rápido: Creación de una aplicación de HoloLens con Azure Object Anchors, en Unity](get-started-unity-hololens.md) e [Inicio rápido: Creación de una aplicación de HoloLens con Azure Object Anchors, en Unity con MRTK](get-started-unity-hololens-mrtk.md).
* <a href="https://unity3d.com/get-unity/download" target="_blank">Unity Hub con Unity 2020.3.8f1 o posterior</a>

## <a name="getting-started"></a>Introducción

Lo primero es configurar el proyecto y la escena de Unity:

1. Inicie Unity Hub.
1. Seleccione **New** (Nuevo) y elija Unity 2020.3.8f1 o posterior.
1. Asegúrese de que la opción **3D** está seleccionada.
1. Asigne un nombre al proyecto y escriba una **ubicación** donde guardarlo.
1. Seleccione **Crear**.
1. Una vez que se abra el **Editor Unity**, guarde la escena predeterminada vacía en un nuevo archivo mediante: **File** > **Save As** (Archivo > Guardar como).
1. Seleccione la carpeta **Scenes** (Escenas), asigne el nombre **Main** a la nueva escena y pulse el botón **Save** (Guardar).

## <a name="configure-as-uwp"></a>Configuración como UWP

1. Seleccione **File -> Building Settings** (Archivo > Configuración de compilación).
1. Seleccione **Universal Windows Platform** (Plataforma universal de Windows) y, a continuación, seleccione **Switch Platform** (Cambiar plataforma).
1. Si el Editor de Unity indica que primero hay que descargar varios componentes, descárguelos e instálelos.

## <a name="install-mixed-reality-feature-tool-feature-packages"></a>Instalación de los paquetes de características de la herramienta de características de Mixed Reality

1. Consulte la documentación de la <a a href="/windows/mixed-reality/develop/unity/welcome-to-mr-feature-tool" target="_blank">herramienta de características de Mixed Reality</a> para configurar la herramienta y aprender a usarla.
1. En la sección **Platform Support** (Compatibilidad con la plataforma), instale el paquete de características del **complemento OpenXR de Mixed Reality**, versión 1.0.0 o posterior, en la carpeta del proyecto de Unity.
1. En la sección **Azure Mixed Reality Services** (Servicios de Azure Mixed Reality), instale el paquete de características **Microsoft Azure Object Anchors** en la carpeta del proyecto de Unity.
1. Vuelva al **Editor Unity**. Pueden transcurrir unos minutos mientras se instalan los paquetes de características de la **herramienta de características de Mixed Reality**.
1. Verá un cuadro de diálogo que solicita confirmación para habilitar el nuevo sistema de entrada.
1. Haga clic en el botón **Sí**.
1. Una vez completado el proceso de instalación, Unity se reiniciará automáticamente.

## <a name="configure-openxr"></a>Configuración de OpenXR

1. Todavía debería estar dentro de la ventana **Build Settings** (Configuración de compilación).
1. Seleccione el botón **Player settings...** (Configuración del Reproductor).
1. Se abrirá la ventana **Project Settings** (Configuración del proyecto).
1. Seleccione la entrada **XR Plug-in Management** (Administración del complemento XR).
1. Siga la documentación <a href="/windows/mixed-reality/develop/unity/xr-project-setup#configuring-xr-plugin-management-for-openxr" target="_blank">Configuración de la administración del complemento XR para OpenXR</a> para configurar **OpenXR** con el **conjunto de características de Microsoft HoloLens** en la lista **Plug-in Providers** (Proveedores de complementos).

## <a name="set-capabilities"></a>Establecimiento de funcionalidades

1. Todavía debería estar dentro de la ventana **Project Settings** (Configuración del proyecto).
1. Seleccione la entrada **Player** (Reproductor).
1. En el **panel Inspector** de **Player Settings** (Configuración del Reproductor), asegúrese de que el icono **Universal Windows Platform settings** (Configuración de Plataforma universal de Windows) esté seleccionado.
1. En la sección de funcionalidades **Publishing Settings** (Configuración de publicación), asegúrese de que los elementos **InternetClientServer**, **WebCam** y **SpatialPerception** estén seleccionados.

## <a name="set-up-the-project-settings"></a>Configuración del proyecto

1. Todavía debería estar dentro de la ventana **Project Settings** (Configuración del proyecto).
1. Seleccione la entrada **Quality** (Calidad).
1. En la columna situada bajo el logotipo de la **Plataforma universal de Windows**, seleccione en la flecha de la fila **Default**(Predeterminado) y seleccione **Very Low** (Muy baja). Sabrá que la configuración se ha aplicado correctamente cuando el cuadro de la columna **Universal Windows Platform** (Plataforma universal de Windows) y la fila **Very Low** (Muy baja) estén en color verde.
1. Cierre las ventanas **Project Settings** (Configuración del proyecto) y **Build Settings** (Configuración de compilación).
1. Siga la documentación <a href="/windows/mixed-reality/develop/unity/xr-project-setup#optimization" target="_blank">Optimización</a> para aplicar la configuración recomendada del proyecto para HoloLens 2.

## <a name="set-up-the-main-virtual-camera"></a>Configuración de la cámara virtual principal

1. En el panel **Hierarchy** (Jerarquía), seleccione **Main Camera** (Cámara principal).
1. En **Inspector**, establezca su posición de transformación en **0,0,0**.
1. Busque la propiedad **Clear Flags** (Borrar marcas) y cambie la lista desplegable de **Skybox** a **Solid Color** (Color sólido).
1. Seleccione el campo **Background** (Fondo) para abrir el selector de colores.
1. Establezca **R, G, B, and A** (R, G, B y A) en **0**.
1. Cambie la propiedad **Clipping Planes Near** (Recorte de planos cercanos) a 0,1.
1. Seleccione **Add Component** (Agregar componente) y busque y agregue **Tracked Pose Driver** (Controlador de posición con seguimiento).
1. Seleccione **File** > **Save** (Archivo > Guardar).

## <a name="trying-it-out"></a>Prueba

Para probar que todo funciona, compile la aplicación en **Unity** e impleméntela desde **Visual Studio**. Para ello, siga el capítulo 6 del curso <a href="/windows/mixed-reality/holograms-100#chapter-6---build-and-deploy-to-device-from-visual-studio" target="_blank">**MR Basics 100: Introducción a Unity**</a>. Verá la pantalla de inicio de Unity y, luego, una opción para borrar la pantalla.

## <a name="create-your-script"></a>Creación del script

1. En el panel **Project** (Proyecto), cree una carpeta, **Scripts**, en la carpeta **Assets** (Activos).
1. Haga clic con el botón derecho en la carpeta y seleccione **Create (Crear) >** , **C# Script** (Script de C#). Asígnele el nombre **ObjectSearch**.
1. Vaya a **GameObject** -> **Create Empty** (Crear vacío).
1. Selecciónelo y, en **Inspector**, cambie su nombre de **GameObject** a **Object Observer** (Observador de objetos).
1. Seleccione **Add Component** (Agregar componente) y busque y agregue el script **ObjectSearch**.
1. Seleccione **File** > **Save** (Archivo > Guardar).

## <a name="start-implementing-your-app"></a>Inicio de la implementación de la aplicación

Está listo para empezar a agregar su propio código al script **ObjectSearch** mediante el SDK del entorno de ejecución de Object Anchors. Puede consultar [Introducción al SDK del entorno de ejecución](../concepts/sdk-overview.md) como punto de partida para conocer los conceptos básicos y usar parte del código de ejemplo para probarlo.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Conceptos: Información general del SDK](../concepts/sdk-overview.md)

> [!div class="nextstepaction"]
> [P+F](../faq.md)

> [!div class="nextstepaction"]
> [Biblioteca cliente de Azure Object Anchors para .NET: versión 0.3.0-beta.1](/dotnet/api/overview/azure/mixedreality.objectanchors.conversion-readme-pre)
