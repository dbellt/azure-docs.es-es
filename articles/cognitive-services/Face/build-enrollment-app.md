---
title: Creación de una aplicación de React para agregar usuarios a un servicio de Face
titleSuffix: Azure Cognitive Services
description: Obtenga información sobre cómo configurar el entorno de desarrollo e implementar una aplicación de Face para obtener el consentimiento de los clientes.
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: face-api
ms.topic: conceptual
ms.date: 11/17/2020
ms.author: pafarley
ms.openlocfilehash: 2bc2b12127af175d051f433f15ad4a9ffdee0467
ms.sourcegitcommit: c1b0d0b61ef7635d008954a0d247a2c94c1a876f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2021
ms.locfileid: "109627787"
---
# <a name="build-a-react-app-to-add-users-to-a-face-service"></a>Creación de una aplicación de React para agregar usuarios a un servicio de Face

En esta guía se muestra cómo empezar a trabajar con la aplicación de inscripción de Face de ejemplo. La aplicación muestra procedimientos recomendados para obtener un consentimiento significativo para agregar usuarios en un servicio de reconocimiento facial y adquirir datos faciales de gran precisión. Un sistema integrado podría usar una aplicación como esta para proporcionar control de acceso sin contacto, comprobación de identidad, seguimiento de la asistencia o una pantalla completa de personalización, en función de los datos faciales.

Cuando se inicia, la aplicación muestra a los usuarios una pantalla de consentimiento detallada. Si el usuario da su consentimiento, la aplicación solicita un nombre de usuario y una contraseña y, después, captura una imagen de alta calidad de la cara mediante la cámara del dispositivo.

La aplicación de ejemplo está escrita en JavaScript y el marco React Native. Actualmente se puede implementar en dispositivos Android y iOS. En el futuro, habrá más opciones de implementación.

## <a name="prerequisites"></a>Prerrequisitos 

* Una suscripción a Azure ([cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services/)).  
* Una vez que tenga la suscripción de Azure, [cree un recurso de Face](https://portal.azure.com/#create/Microsoft.CognitiveServicesFace) en Azure Portal para obtener la clave y el punto de conexión. Tras su implementación, seleccione **Ir al recurso**.  
  * Necesitará la clave y el punto de conexión del recurso creado para conectar la aplicación a Face API.  
  * Solo para desarrollo y pruebas locales, la clave de API y el punto de conexión son variables de entorno. Para la implementación final, almacene la clave de API en una ubicación segura y nunca en el código o las variables de entorno.  

### <a name="important-security-considerations"></a>Consideraciones de seguridad importantes
* Para el desarrollo local y las pruebas limitadas iniciales, es aceptable (aunque no es un procedimiento recomendado) usar variables de entorno para contener la clave de API y el punto de conexión. En el caso de las implementaciones piloto y finales, la clave de API debe almacenarse de forma segura, lo que probablemente implica el uso de un servicio intermedio para validar un token de usuario generado durante el inicio de sesión. 
* No almacene nunca la clave de API ni el punto de conexión en el código ni los confíe a un sistema de control de versiones (por ejemplo, Git). Si esto ocurre por error, debe generar inmediatamente una nueva clave o punto de conexión de API y revocar los anteriores.
* Como procedimiento recomendado, considere la posibilidad de tener claves de API independientes para desarrollo y producción.

## <a name="set-up-the-development-environment"></a>Configuración del entorno de desarrollo

#### <a name="android"></a>[Android](#tab/android)
 
1. Clone el repositorio de Git para la [aplicación de ejemplo](https://github.com/azure-samples/cognitive-services-FaceAPIEnrollmentSample).
1. Para configurar el entorno de desarrollo, siga la <a href="https://reactnative.dev/docs/environment-setup"  title="documentación de React Native"  target="_blank">React Native documentation <span class="docon docon-navigate-external x-hidden-focus"></span></a>. Seleccione **React Native CLI Quickstart** (Inicio rápido de la CLI de React Native). Seleccione el sistema operativo de desarrollo y **Android** como sistema operativo de destino. Complete las secciones **Installing dependencies** (Instalación de dependencias) y **Android development environment** (Entorno de desarrollo de Android).
1. Descargue el editor de texto que prefiera, como [Visual Studio Code](https://code.visualstudio.com/).
1. Recupere el punto de conexión de FaceAPI y la clave en Azure Portal en la pestaña **Información general** del recurso. No inserte su clave de Face API en el repositorio remoto.
1. Ejecute la aplicación mediante el emulador de dispositivo virtual Android desde Android Studio o con su propio dispositivo Android. Para probar la aplicación en un dispositivo físico, siga la <a href="https://reactnative.dev/docs/running-on-device"  title="documentación de React Native"  target="_blank">React Native documentation <span class="docon docon-navigate-external x-hidden-focus"></span></a> pertinente.

#### <a name="ios"></a>[iOS](#tab/ios)

1. Clone el repositorio de Git para la [aplicación de ejemplo](https://github.com/azure-samples/cognitive-services-FaceAPIEnrollmentSample).
1. Para configurar el entorno de desarrollo, siga la <a href="https://reactnative.dev/docs/environment-setup"  title="documentación de React Native"  target="_blank">React Native documentation <span class="docon docon-navigate-external x-hidden-focus"></span></a>. Seleccione **React Native CLI Quickstart** (Inicio rápido de la CLI de React Native). Seleccione **macOS** como SO de desarriollo y **iOS** como SO de destino. Complete la sección **Installing dependencies** (Instalación de dependencias).
1. Descargue el editor de texto que prefiera, como [Visual Studio Code](https://code.visualstudio.com/). También tendrá que descargar Xcode. 
1. Recupere el punto de conexión de FaceAPI y la clave en Azure Portal en la pestaña **Información general** del recurso. No inserte su clave de Face API en el repositorio remoto.
1. Ejecute la aplicación mediante un dispositivo simulado de Xcode o su propio dispositivo iOS. Para probar la aplicación en un dispositivo físico, siga la <a href="https://reactnative.dev/docs/running-on-device"  title="documentación de React Native"  target="_blank">React Native documentation <span class="docon docon-navigate-external x-hidden-focus"></span></a> pertinente.

---

## <a name="create-a-user-add-experience"></a>Creación de una experiencia de adición de usuario  

Ahora que ha configurado la aplicación de ejemplo, puede adaptarla a sus propias necesidades.

Por ejemplo, es posible que desee agregar información específica de la situación en la página de consentimiento:

> [!div class="mx-imgBorder"]
> ![página de consentimiento de la aplicación](./media/enrollment-app/1-consent-1.jpg)

El servicio proporciona comprobaciones de calidad de la imagen para ayudarle a elegir si la imagen tiene una calidad suficiente para agregar al cliente o intentar el reconocimiento facial. Esta aplicación muestra cómo acceder a los fotogramas de la cámara del dispositivo, seleccionar los fotogramas de mayor calidad y agregar la cara detectada en el servicio Face API. 

Muchos de los problemas de reconocimiento facial se deben a la baja calidad de las imágenes de referencia. Los siguientes son algunos de los factores que pueden degradar el rendimiento del modelo:
* Tamaño de la cara (caras alejadas de la cámara)
* Orientación de la cara (caras giradas o inclinadas con respecto a la cámara)
* Condiciones de iluminación deficientes (poca luz o contraluz) que hacen que la imagen pueda quedar mal expuesta o tener demasiado ruido
* Oclusión (caras parcialmente ocultas u obstruidas), incluidos accesorios como sombreros o gafas gruesas
* Desenfoque (por ejemplo, por un rápido movimiento de la cara al tomar la fotografía). 

> [!div class="mx-imgBorder"]
> ![página de instrucciones de captura de imagen de la aplicación](./media/enrollment-app/4-instruction.jpg)

Tenga en cuenta que la aplicación también ofrece funcionalidad para eliminar la información del usuario y la opción de volver a agregarlo.

> [!div class="mx-imgBorder"]
> ![página de administración del perfil](./media/enrollment-app/10-manage-2.jpg)

Para ampliar la funcionalidad de la aplicación para abarcar la experiencia completa, consulte la [información general](enrollment-overview.md) para familiarizarse con características de implementación adicionales y procedimientos recomendados.

## <a name="deploy-the-app"></a>Implementar la aplicación

#### <a name="android"></a>[Android](#tab/android)

En primer lugar, asegúrese de que la aplicación está lista para la implementación de producción: quite cualquier clave o secreto del código de la aplicación y asegúrese de que ha seguido los [procedimientos recomendados de seguridad](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-security?tabs=command-line%2Ccsharp).

Cuando todo esté listo para publicar la aplicación para producción, generará un archivo APK listo para la publicación, que es el formato de archivo de paquete para aplicaciones Android. Este archivo APK debe estar firmado con una clave privada. Con esta versión de lanzamiento, puede empezar a distribuir la aplicación directamente a sus dispositivos. 

Siga la documentación de <a href="https://developer.android.com/studio/publish/preparing#publishing-build"  title="preparación para la publicación"  target="_blank">Prepare for release <span class="docon docon-navigate-external x-hidden-focus"></span></a> para obtener información sobre cómo generar una clave privada, firmar la aplicación y generar un archivo APK de publicación.  

Una vez creado un archivo APK firmado, consulte la documentación de <a href="https://developer.android.com/studio/publish"  title="publicación de la aplicación"  target="_blank">Publish your app <span class="docon docon-navigate-external x-hidden-focus"></span></a> para obtener más información sobre cómo publicar la aplicación.

#### <a name="ios"></a>[iOS](#tab/ios)

En primer lugar, asegúrese de que la aplicación está lista para la implementación de producción: quite cualquier clave o secreto del código de la aplicación y asegúrese de que ha seguido los [procedimientos recomendados de seguridad](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-security?tabs=command-line%2Ccsharp). Para prepararse para la distribución, deberá crear un icono de aplicación y una pantalla de inicio y configurar la información de implementación. Siga la [documentación de Xcode](https://developer.apple.com/documentation/Xcode/preparing_your_app_for_distribution) para preparar la aplicación para su distribución. 

Cuando esté listo para lanzar la aplicación para producción, compilará un archivo de la aplicación. Siga la [documentación de Xcode](https://developer.apple.com/documentation/Xcode/distributing_your_app_for_beta_testing_and_releases) sobre cómo crear una compilación de archivo y las opciones para distribuir la aplicación.  

---

## <a name="next-steps"></a>Pasos siguientes  

En esta guía, ha aprendido a configurar el entorno de desarrollo y a empezar a trabajar con la aplicación de ejemplo. Si no está familiarizado con React Native, puede leer su [documentación de introducción](https://reactnative.dev/docs/getting-started) para obtener más información general. También puede ser útil familiarizarse con [Face API](Overview.md). Consulte las demás secciones sobre la adición de usuarios antes de comenzar con el desarrollo.
