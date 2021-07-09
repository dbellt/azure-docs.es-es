---
title: 'Inicio rápido: Creación de palabras clave en el servicio Speech'
titleSuffix: Azure Cognitive Services
description: El dispositivo siempre espera una palabra clave (o frase). Cuando el usuario dice la palabra clave, el dispositivo envía todo el audio subsiguiente a la nube, hasta que el usuario deja de hablar. La personalización de la palabra clave es una forma eficaz de diferenciar el dispositivo y de reforzar la marca.
services: cognitive-services
author: trevorbye
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 11/03/2020
ms.author: trbye
ms.custom: devx-track-csharp
zone_pivot_groups: keyword-quickstart
ms.openlocfilehash: a1ec49ca9ee9d0adcbae97eae2e4093274a524c6
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110099647"
---
# <a name="get-started-with-custom-keyword"></a>Introducción a Palabra clave personalizada

En este inicio rápido, aprenderá los conceptos básicos de trabajar con palabras clave personalizadas mediante Speech Studio y el SDK de voz. Una palabra clave es una palabra o frase corta que permite activar por voz el producto. Se crean modelos de palabras clave en Speech Studio y, a continuación, se exporta un archivo de modelo que se usará con el SDK de Voz en sus aplicaciones.

## <a name="prerequisites"></a>Requisitos previos

Los pasos descritos en este artículo requieren una suscripción a Voz y el SDK de Voz. Si no dispone ya de una suscripción, [pruebe el servicio de voz de forma gratuita](overview.md#try-the-speech-service-for-free). Para obtener el SDK, consulte la [guía de instalación](quickstarts/setup-platform.md) para su plataforma.

## <a name="create-a-keyword-in-speech-studio"></a>Creación de una palabra clave en Speech Studio

Antes de que pueda usar una palabra clave personalizada, deberá crear una palabra clave con la página [Custom Keyword](https://aka.ms/sdsdk-wakewordportal) (Palabra clave personalizada) en [Speech Studio](https://aka.ms/sdsdk-speechportal). Después de proporcionar una palabra clave, se genera un archivo `.table` que puede usar con el SDK de Voz.

> [!IMPORTANT]
> Los modelos de palabra clave personalizada y los archivos `.table` resultantes **solo** pueden crearse en Speech Studio.
> No se pueden crear palabras clave personalizadas a partir del SDK o mediante llamadas REST.

1. Vaya a [Speech Studio](https://aka.ms/sdsdk-speechportal) e **inicie sesión** o, si todavía no tiene una suscripción a Voz, elija [**Crear una suscripción**](https://go.microsoft.com/fwlink/?linkid=2086754).

1. En la página [Custom Keyword](https://aka.ms/sdsdk-wakewordportal) (Palabra clave personalizada), cree un **Nuevo proyecto**. 

1. Escriba un **Nombre**, una **Descripción** opcional y seleccione el idioma. Necesita un proyecto por idioma. La compatibilidad está limitada actualmente a inglés (Estados Unidos) y chino (mandarín, simplificado).

    ![Descripción del proyecto de palabra clave](media/custom-keyword/custom-kws-portal-new-project.png)

1. Seleccione el proyecto de la lista. 

    ![Selección del proyecto de palabra clave](media/custom-keyword/custom-kws-portal-project-list.png)

1. Para crear un nuevo modelo de palabra clave, haga clic en **Entrenar modelo**.

1. Escriba un **Nombre** para el modelo, una **Descripción** opcional y la **Palabra clave** de su elección y después haga clic en **Siguiente**. Consulte las [directrices](keyword-recognition-guidelines.md#choosing-an-effective-keyword) sobre cómo elegir una palabra clave eficaz.

    ![Escriba su palabra clave](media/custom-keyword/custom-kws-portal-new-model.png)

1. El portal creará pronunciaciones candidatas para la palabra clave. Para escuchar a cada candidato, haga clic en los botones de reproducción y desactive las pronunciaciones que sean incorrectas. Una vez que solo las pronunciaciones correctas estén marcadas como activas, haga clic en **Entrenar** para empezar a generar el modelo de palabra clave. 

    ![Captura de pantalla que muestra dónde se eligen las pronunciaciones correctas.](media/custom-keyword/custom-kws-portal-choose-prons.png)

1. El modelo puede tardar hasta treinta minutos en generarse. La lista de palabras clave cambiará de **Procesando** a **Correcto** cuando el modelo esté completo. Luego puede descargar el archivo.

    ![Revisión de la palabra clave](media/custom-keyword/custom-kws-portal-download-model.png)

1. El archivo descargado es un archivo `.zip`. Extraiga el archivo y verá un archivo con la extensión `.table`. Este es el archivo que usará con el SDK en la sección siguiente, por lo que debe asegurarse de anotar la ruta de acceso. El nombre del archivo refleja el nombre de la palabra clave; por ejemplo, la palabra clave **Activate device** (activa el dispositivo) tiene el nombre de archivo `Activate_device.table`.

## <a name="use-a-keyword-model-with-the-speech-sdk"></a>Uso de un modelo de palabras clave con el SDK de Voz

::: zone pivot="programming-language-csharp"
[!INCLUDE [C# Basics include](includes/how-to/keyword-recognition/keyword-basics-csharp.md)]
::: zone-end

::: zone pivot="programming-language-python"
[!INCLUDE [Python Basics include](includes/how-to/keyword-recognition/keyword-basics-python.md)]
::: zone-end

::: zone pivot="programming-languages-objectivec-swift"
[!INCLUDE [ObjectiveC/Swift Basics include](includes/how-to/keyword-recognition/keyword-basics-objc.md)]
::: zone-end

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Obtención del SDK de voz](speech-sdk.md)
