---
title: Introducción a Speech Studio - Servicio Voz
titleSuffix: Azure Cognitive Services
description: Speech Studio es un conjunto de herramientas basadas en la interfaz de usuario para compilar e integrar características del servicio Voz de Azure en las aplicaciones.
services: cognitive-services
author: trevorbye
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 05/07/2021
ms.author: trbye
ms.openlocfilehash: ddf7b85f5775ee25a260f19ae81e43afff95facb
ms.sourcegitcommit: f9e368733d7fca2877d9013ae73a8a63911cb88f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111902345"
---
# <a name="what-is-speech-studio"></a>¿Qué es el Speech Studio?

[Speech Studio](https://speech.microsoft.com) es un conjunto de herramientas basadas en interfaz de usuario para compilar e integrar características del servicio Voz de Azure en las aplicaciones. Crea proyectos en Speech Studio mediante un enfoque sin código y, a continuación, hace referencia a los recursos que crea en las aplicaciones mediante el [SDK de Voz](speech-sdk.md), la [CLI de Voz](spx-overview.md) o diversas API REST.

## <a name="set-up-your-azure-account"></a>Configuración de la cuenta de Azure

Para poder usar [Speech Studio](https://speech.microsoft.com) necesita tener una cuenta de Azure y una suscripción al servicio Voz. Si no dispone de una cuenta y una suscripción, [pruebe el servicio de voz de forma gratuita](overview.md#try-the-speech-service-for-free).

> [!NOTE]
> Asegúrese de crear una suscripción estándar (S0). No se admiten las suscripciones Gratis (F0).

Tras crear una cuenta de Azure y una suscripción al servicio Voz:

1. Inicie sesión en [Speech Studio](https://speech.microsoft.com).
1. Seleccione la suscripción que necesita para trabajar y crear un proyecto de voz.
1. Si desea modificarla, seleccione el botón del engranaje en el menú superior.

## <a name="speech-studio-features"></a>Características de Speech Studio

Las características siguientes del servicio Voz están disponibles como tipos de proyecto en Speech Studio.

* **Voz en texto en tiempo real**: arrastre y coloque archivos de audio para probar rápidamente la conversión de voz en texto sin usar ningún código. Esta es una herramienta de demostración que permite ver cómo funciona la conversión de voz en texto en sus muestras de audio; consulte la [introducción](speech-to-text.md) a la conversión de voz en texto para explorar toda la funcionalidad disponible.
* **Habla personalizada**: esta característica permite crear modelos de reconocimiento de voz adaptados a conjuntos de vocabulario y estilos de habla específicos. A diferencia del uso de un modelo base de reconocimiento de voz, los modelos de Habla personalizada constituyen parte de su ventaja competitiva única, porque no son accesibles públicamente. Consulte el [inicio rápido](how-to-custom-speech-test-and-train.md) para empezar a usar la carga de muestras de audio para crear un modelo de Habla personalizada.
* **Evaluación de la pronunciación**: esta característica evalúa la pronunciación y ofrece información a los hablantes sobre la precisión y la fluidez del audio hablado. Speech Studio proporciona un espacio aislado para probar esta característica rápidamente sin código, pero puede consultar el artículo de [procedimientos](how-to-pronunciation-assessment.md) para usar la característica con el SDK de voz en las aplicaciones.
* **Galería de voces**: cree aplicaciones y servicios que hablen de forma natural. Elija entre más de 170 voces en más de 70 idiomas y variantes. Dé vida a los escenarios con voces neuronales idénticas a las humanas y altamente expresivas.
* **Voz personalizada**: esta característica permite crear voces personalizadas únicas para la conversión de texto a voz. Puede proporcionar archivos de audio, crear las transcripciones correspondientes en Speech Studio y, a continuación, usar las voces personalizadas en las aplicaciones. Consulte el artículo de [procedimientos](how-to-custom-voice-create-voice.md) sobre la creación y el uso de voces personalizadas mediante puntos de conexión. 
* **Creación de contenido de audio**: [Creación de contenido de audio](how-to-audio-content-creation.md) es una herramienta fácil de usar que permite crear contenido de audio muy natural para diversos escenarios como audiolibros, retransmisión de noticias, narraciones en vídeo y bots de chat. Speech Studio permite exportar los archivos de audio creados para usarlos en las aplicaciones.
* **Palabra clave personalizada**: una palabra clave personalizada es una palabra o frase corta que permite activar el producto por voz. Cree una palabra clave personalizada en Speech Studio y, a continuación, genere un archivo binario para [usarlo con el SDK de Voz](custom-keyword-basics.md) en las aplicaciones.
* **Comandos personalizados**: esta característica facilita la creación de aplicaciones de comandos de voz enriquecidas y optimizadas para las opciones de interacción que priorizan la voz. Proporciona una experiencia de creación sin código en Speech Studio, un modelo de hospedaje automático y una complejidad relativamente menor, lo que ayuda a centrarse en la creación de la mejor solución para sus escenarios de comandos de voz. Consulte la guía [paso a paso](how-to-develop-custom-commands-application.md) para crear aplicaciones de Comandos personalizados y también la guía para [integrar la aplicación de Comandos personalizados con el SDK de Voz.](how-to-custom-commands-setup-speech-sdk.md)

## <a name="next-steps"></a>Pasos siguientes

[Explore Speech Studio](https://speech.microsoft.com) y cree un proyecto.




