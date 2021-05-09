---
title: ¿Qué es Custom Translator?
titleSuffix: Azure Cognitive Services
description: Custom Translator ofrece unas funcionalidades parecidas a las que presenta Microsoft Translator Hub para la traducción automática estadística (SMT), pero exclusivamente para sistemas de traducción automática neuronales (NMT).
author: laujan
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.date: 12/09/2019
ms.author: lajanuar
ms.topic: overview
ms.openlocfilehash: 07bc3cd9fdda98e6eeb0d7fe4ebefd562aafc47b
ms.sourcegitcommit: 18cd3c1c8cc47258c6a1a04e0e03d6248c52ef24
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/25/2021
ms.locfileid: "107992311"
---
# <a name="what-is-custom-translator"></a>¿Qué es Custom Translator?

[Custom Translator](https://portal.customtranslator.azure.ai) es una característica del servicio Microsoft Translator que permite a las empresas de traducción, los desarrolladores de aplicaciones y los proveedores de servicios de lenguaje compilar sistemas de traducción automática neuronales (NMT) personalizados. Los sistemas de traducción personalizados se integran fácilmente en las aplicaciones, flujos de trabajo y sitios web existentes.

Los sistemas de traducción creados con [Traductor personalizado](https://portal.customtranslator.azure.ai) están disponibles en la misma Microsoft Translator [Text API V3](../reference/v3-0-translate.md?tabs=curl), que se basa en la nube, es segura, de alto rendimiento y altamente escalable, que realiza miles de millones de traducciones todos los días.

Custom Translator admite más de 36 idiomas y los asigna directamente a los idiomas disponibles para NMT. Para obtener una lista completa, consulte [Idiomas de Microsoft Translator](../language-support.md#customization).

Esta documentación contiene los siguientes tipos de artículos:

* Los [**inicios rápidos**](quickstart-build-deploy-custom-model.md) son instrucciones de inicio que le guiarán a la hora de hacer solicitudes al servicio.  
* Las [**guías de procedimientos**](how-to-create-project.md) contienen instrucciones para usar la característica de una manera más específica o personalizada.  
* Los [**conceptos**](workspace-and-project.md) proporcionan explicaciones detalladas sobre la funcionalidad de la característica.  


## <a name="features"></a>Características

Traductor personalizado proporciona diferentes características para crear un sistema de traducción personalizado y posteriormente acceder a él.

|Característica  |Descripción  |
|---------|---------|
|[Aplicación de la tecnología de traducción automática neuronal](https://www.microsoft.com/translator/blog/2016/11/15/microsoft-translator-launching-neural-network-based-translations-for-all-its-speech-languages/)     |  Mejore la traducción mediante la aplicación de la traducción automática neuronal (NMT) que proporciona Traductor personalizado.       |
|[Creación de sistemas que conocen la terminología del negocio](what-are-parallel-documents.md)     |  Personalice y compile sistemas de traducción mediante documentos paralelos que comprendan la terminología usada en su propia empresa y sector.       |
|[Uso de un diccionario para compilar los modelos](what-is-dictionary.md)     |   Si no tiene un conjunto de datos de aprendizaje, puede entrenar un modelo solo con datos de diccionario.       |
|[Colaboración con otros usuarios](how-to-manage-settings.md#share-your-workspace)     |   Colabore con su equipo compartiendo su trabajo con diferentes personas.     |
|[Acceso al modelo de traducción personalizado](../reference/v3-0-translate.md?tabs=curl)     |  Se puede acceder al modelo de traducción personalizado en cualquier momento mediante las aplicaciones y programas existentes a través de Microsoft Translator Text API V3.       |

## <a name="get-better-translations"></a>Obtención de mejores traducciones

Microsoft Translator lanzó la [traducción automática neuronal (NMT)](https://www.microsoft.com/translator/blog/2016/11/15/microsoft-translator-launching-neural-network-based-translations-for-all-its-speech-languages/) en 2016. La NMT proporciona avances importantes en lo que se refiere a la calidad de la traducción con respecto a la tecnología estándar de [traducción automática estadística (SMT)](https://en.wikipedia.org/wiki/Statistical_machine_translation). Como la NMT captura mejor el contexto de oraciones completas antes de traducirlas, proporciona unas traducciones de mayor calidad, un tono más natural y más fluidas. [Custom Translator](https://portal.customtranslator.azure.ai) proporciona NMT para sus modelos personalizados lo cual da como resultado una traducción de mejor calidad.

Puede usar los documentos traducidos anteriormente para crear un sistema de traducción. Estos documentos incluyen la terminología y el estilo específicos de un dominio de mejor forma que lo hace un sistema de traducción estándar. Los usuarios pueden cargar documentos ALIGN, PDF, LCL, HTML, HTM, XLF, TMX, XLIFF, TXT, DOCX y XLSX.

Custom Translator acepta también datos paralelos en el nivel de documento para que la recopilación de datos y su preparación sea más eficaz. Si los usuarios tienen acceso a versiones del mismo contenido en varios idiomas, pero en documentos independientes Custom Translator podrá hace concordar automáticamente las frases de los distintos documentos.

Si se proporciona el tipo y la cantidad adecuados de datos de aprendizaje, no es raro observar una mejora de la [puntuación BLEU](what-is-bleu-score.md) de entre 5 y 10 puntos mediante el uso de Custom Translator.

## <a name="be-productive-and-cost-effective"></a>Productivo y rentable

Con [Custom Translator](https://portal.customtranslator.azure.ai), el aprendizaje e implementación de un sistema personalizado no requiere ningún conocimiento de programación.

Mediante el portal seguro de [Custom Translator](https://portal.customtranslator.azure.ai), los usuarios pueden cargar datos de aprendizaje y entrenar sistemas, probarlos e implementarlos en un entorno de producción mediante una interfaz de usuario intuitiva. El sistema estará disponible para su uso a escala en unas pocas horas (el tiempo real depende del tamaño de los datos de aprendizaje).

También se puede acceder a [Custom Translator](https://portal.customtranslator.azure.ai) mediante programación a través de una [API dedicada](https://custom-api.cognitive.microsofttranslator.com/swagger/) (actualmente en versión preliminar). La API permite a los usuarios administrar la creación o actualización del entrenamiento mediante su propia aplicación o servicio web.

El costo de usar un modelo personalizado para traducir contenido se basa en el plan de tarifa de Translator Text API del usuario. Consulte la [página web de precios de Translator Text API](https://azure.microsoft.com/pricing/details/cognitive-services/translator-text-api/) para Cognitive Services para más información.

## <a name="securely-translate-anytime-anywhere-on-all-your-apps-and-services"></a>Traduzca de forma segura siempre y en cualquier lugar en todas sus aplicaciones y servicios.

Se puede acceder a los sistemas personalizados fácilmente e integrarlos en cualquier producto o flujo de trabajo empresarial, o en cualquier dispositivo, a través de Microsoft Translator Text API mediante tecnología REST estándar.

## <a name="next-steps"></a>Pasos siguientes

- Lea acerca de los [detalles de precios](https://azure.microsoft.com/pricing/details/cognitive-services/translator-text-api/).

- Con la [guía de inicio rápido](quickstart-build-deploy-custom-model.md) aprenderá a compilar un modelo de traducción en Custom Translator.
