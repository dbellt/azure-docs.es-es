---
title: 'Personalización de un modelo de lenguaje en Azure Video Analyzer for Media (anteriormente, Video Indexer): Azure'
titleSuffix: Azure Media Services
description: En este artículo se proporciona información general sobre qué es un modelo de lenguaje en Azure Video Analyzer for Media (anteriormente, Video Indexer) y cómo personalizarlo.
services: media-services
author: anikaz
manager: johndeu
ms.topic: article
ms.date: 05/15/2019
ms.author: kumud
ms.openlocfilehash: afeda546743db40f3b1a966342be5cd7d43a27ba
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110388464"
---
# <a name="customize-a-language-model-with-video-analyzer-for-media"></a>Personalización de un modelo de lenguaje con Video Analyzer for Media

Azure Video Analyzer for Media (anteriormente, Video Indexer) es compatible con el reconocimiento de voz automático mediante la integración en el [servicio Habla personalizada](https://azure.microsoft.com/services/cognitive-services/custom-speech-service/) de Microsoft. Puede personalizar el modelo de lenguaje mediante la carga de texto de adaptación, es decir, texto del dominio a cuyo vocabulario desea que se adapte el motor. Una vez entrenado el modelo, se reconocerán las nuevas palabras que aparezcan en el texto de adaptación, adoptando la pronunciación predeterminada, y el modelo de lenguaje aprenderá nuevas secuencias probables de palabras. Los modelos de lenguaje personalizados admitidos son inglés, español, francés, alemán, italiano, chino (simplificado), japonés, ruso, portugués, hindi y coreano. 

Usemos como ejemplo una palabra muy específica, como "Kubernetes" (en el contexto de Azure Kubernetes Service). Dado que la palabra es nueva en Video Analyzer for Media, se reconoce como "comunidades". Es necesario entrenar al modelo para que la reconozca como "Kubernetes". En otros casos, las palabras existen, pero el modelo de lenguaje no espera que aparezcan en un contexto determinado. Por ejemplo, "servicio de contenedor" no es una secuencia de dos palabras que un modelo de lenguaje no especializado reconocería como un conjunto de palabras específico.

Tiene la opción de cargar palabras sin contexto en una lista en un archivo de texto. Se considera una adaptación parcial. También puede cargar archivos de texto de documentación u oraciones relacionadas con el contenido para una mejor adaptación.

Puede usar el sitio web o las API de Video Analyzer for Media para crear y editar modelos de lenguaje personalizados, como se describe en los temas de la sección [Pasos siguientes](#next-steps) de este tema.

## <a name="best-practices-for-custom-language-models"></a>Procedimientos recomendados para modelos de lenguaje personalizados

Video Analyzer for Media aprende según las probabilidades de combinaciones de palabras, por lo que para que aprenda de la mejor forma posible:

* Ofrézcale suficientes ejemplos de oraciones reales del modo en que se pronunciarían.
* Coloque solo una oración por línea, no más. De lo contrario, el sistema aprenderá probabilidades con las oraciones.
* No pasa nada por poner una palabra como oración para favorecerla frente a otras, pero el sistema aprende mejor de oraciones completas.
* Al introducir palabras o acrónimos nuevos, si es posible, proporcione tantos ejemplos de uso como pueda en una oración completa para ofrecer al sistema tanto contexto como sea posible.
* Intente colocar varias opciones de adaptación y vea cómo funcionan.
* Evite la repetición de la misma oración exacta varias veces. Puede crear un sesgo en el resto de la entrada.
* Evite incluir símbolos raros (~, # @ % &), ya que se descartarán. Las oraciones en que las que aparezcan también se descartarán.
* Evite colocar entradas demasiado grandes, como cientos de miles de oraciones, porque, al hacerlo, reducirá el efecto de impulso.

## <a name="next-steps"></a>Pasos siguientes

[Personalización del modelo de lenguaje mediante las API](customize-language-model-with-api.md)

[Customize Language model using website](customize-language-model-with-website.md) (Personalización del modelo de lenguaje mediante el sitio web)
