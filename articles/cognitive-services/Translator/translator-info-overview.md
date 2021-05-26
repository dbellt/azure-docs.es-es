---
title: Servicio Microsoft Translator
titlesuffix: Azure Cognitive Services
description: Integre Translator en aplicaciones, sitios web, herramientas u otras soluciones para proporcionar experiencias de usuario en varios idiomas.
services: cognitive-services
author: laujan
manager: nitinme
ms.service: cognitive-services
ms.topic: overview
ms.subservice: translator-text
ms.date: 03/15/2021
ms.author: lajanuar
ms.custom: cog-serv-seo-aug-2020
keywords: translator, text translation, machine translation, translation service
ms.openlocfilehash: 8ece17a0f1452c7ea7f90fc5e14758c03ac36651
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110076517"
---
# <a name="what-is-the-translator-service"></a>¿Qué es el servicio Translator?

Translator es un servicio de traducción automática basado en la nube que forma parte de la familia de API de [Azure Cognitive Services](../../index.yml?panel=ai&pivot=products) utilizadas para crear aplicaciones inteligentes. Translator se integra con facilidad en cualquier aplicación, sitio web, herramienta y solución. Le permite agregar experiencias de usuario en varios idiomas en [90 idiomas y dialectos](./language-support.md) y se puede usar para la traducción de texto con cualquier sistema operativo.

Esta documentación contiene los siguientes tipos de artículos:

* Los [**inicios rápidos**](quickstart-translator.md) son instrucciones de inicio que le guiarán a la hora de hacer solicitudes al servicio.
* Las [**guías de procedimientos**](translator-how-to-signup.md) contienen instrucciones para usar el servicio de una manera más específica o personalizada.
* Los [**tutoriales**](tutorial-wpf-translation-csharp.md) son guías más largas que muestran cómo usar el servicio como un componente en soluciones empresariales más amplias.

## <a name="about-microsoft-translator"></a>Acerca de Microsoft Translator

Translator da servicio a muchos productos y servicios de Microsoft, y lo utilizan miles de empresas en todo el mundo en sus aplicaciones y flujos de trabajo.

La traducción de voz, con la tecnología de Translator, también está disponible mediante el [servicio de Voz de Azure](../speech-service/index.yml). Combina la funcionalidad de Translator Speech API y Custom Speech Service en un servicio totalmente personalizable y unificado.

## <a name="language-support"></a>Compatibilidad con idiomas

Translator proporciona compatibilidad con varios idiomas para traducción de texto, transliteración, detección de idioma y diccionarios. Consulte [Compatibilidad con idiomas](language-support.md) para obtener una lista completa o puede acceder a la lista mediante programación con la [API REST](./reference/v3-0-languages.md).

## <a name="microsoft-translator-neural-machine-translation"></a>Traducción automática neuronal de Microsoft Translator

La traducción automática neuronal (NMT) es el nuevo estándar para las traducciones automáticas de alta calidad basadas en inteligencia artificial. Este estándar reemplaza la traducción automática estadística (SMT) que alcanzó un nivel estable a mediados de 2010.

NMT proporciona mejores traducciones que SMT, no solo a la hora de puntuar la calidad de la traducción bruta, sino también porque suenan más fluidas y humanas. El motivo principal de esta fluidez es que NMT usa el contexto completo de una frase para traducir las palabras. SMT solo tomaba el contexto inmediato de unas cuantas palabras antes y después de cada palabra.

Los modelos NMT se sitúan en el centro de la API y no son visibles para los usuarios finales. La única diferencia perceptible es una mejor calidad de la traducción, en especial en los idiomas chino, japonés y árabe.

Más información en [¿Qué es la traducción automática?](https://www.microsoft.com/translator/mt.aspx#nnt).

## <a name="improve-translations-with-custom-translator"></a>Mejora de las traducciones con Traductor personalizado

 [Traductor personalizado](customization.md) es una extensión del servicio Translator, se puede usar para personalizar el sistema de traducción neuronal y mejorar la traducción de su terminología y estilo específicos.

Con Traductor personalizado puede crear sistemas de traducción que administran la terminología usada en su propio negocio o sector. Luego, los sistemas de traducción personalizados se pueden integrar fácilmente en las aplicaciones, flujos de trabajo, sitios web y dispositivos ya existentes, mediante el componente normal Translator, por medio del parámetro de categoría.

## <a name="next-steps"></a>Pasos siguientes

* [Cree una instancia del servicio Translator](./translator-how-to-signup.md) para obtener las claves de acceso y el punto de conexión.
* Pruebe nuestra [Guía de inicio rápido](quickstart-translator.md) para aprender a llamar rápidamente al servicio Translator.
* [Referencia de API](./reference/v3-0-reference.md) proporciona la documentación técnica de las API.
* [Detalles de precios](https://azure.microsoft.com/pricing/details/cognitive-services/translator-text-api/)
