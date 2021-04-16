---
title: 'Términos clave: Traductor personalizado'
titleSuffix: Azure Cognitive Services
description: Lista de los términos clave usados en los artículos del Traductor personalizado.
author: laujan
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.date: 04/02/2021
ms.author: lajanuar
ms.topic: reference
ms.openlocfilehash: 4a0308f2b961ce9b1321fab3e202e038eb199674
ms.sourcegitcommit: b8995b7dafe6ee4b8c3c2b0c759b874dff74d96f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/03/2021
ms.locfileid: "106286426"
---
# <a name="custom-translator-key-terms"></a>Términos clave del Traductor personalizado

En la tabla siguiente se presenta una lista de términos clave que puede encontrar al trabajar con el [Traductor personalizado](https://portal.customtranslator.azure.ai).

| Palabra o expresión|Definición|
|------------------|-----------|
| Idioma de origen | El idioma de origen es el idioma de partida que desea traducir a otro idioma ("idioma de destino").|
| Idioma de destino| El idioma de destino es el idioma que desea que la traducción automática proporcione después de recibir el idioma de origen. |
| Archivo monolingüe | Un archivo monolingüe solo tiene un idioma y no está emparejado con otro archivo en un idioma diferente. |
| Archivos paralelos | Un archivo paralelo es una combinación de dos archivos con texto que se corresponde. Uno de ellos tiene el idioma de origen. El otro está en el idioma de destino.|
| Alineación de frases| El conjunto de datos paralelo debe tener las frases alineadas para aquellas frases que representan el mismo texto en ambos idiomas. Por ejemplo, en un archivo paralelo de origen la primera frase debería, en teoría, asignarse a la primera frase del archivo paralelo de destino.|
| Texto alineado | Uno de los pasos más importantes de la validación de archivos consiste en alinear las frases de los documentos paralelos. Las cosas se expresan de forma diferente en distintos idiomas. Además, los idiomas tienen un orden de las palabras diferente. Este paso permite realizar el trabajo de alineación de aquellas frases que tienen el mismo contenido para que se puedan usar en el aprendizaje. Un grado bajo de alineación de frases indica que hay algún error en uno de los archivos o en ambos. |
| División de palabras o anulación de la división | La división de palabras es la función encargada de marcar los límites entre las palabras. Muchos sistemas de escritura usan un espacio para denotar el límite entre las palabras. La anulación de la división hace referencia a la eliminación de cualquier marcador visible que se haya insertado entre las palabras en un paso anterior. |
| Delimitadores   | Los delimitadores son las formas en que una frase se divide en segmentos o se delimita el margen entre las frases. Por ejemplo, en inglés los espacios delimitan las palabras, los dos puntos y el punto y coma delimitan oraciones, y el punto delimita frases. |
| Archivos de aprendizaje | Un archivo de aprendizaje se usa para enseñar al sistema de traducción automática como realizar asignaciones de un idioma (idioma de origen) a otro (idioma de destino). Cuantos más datos proporcione, mejor funcionará el sistema. |
| Archivos de optimización | Estos archivos normalmente se derivan del conjunto de entrenamiento (si no selecciona ningún conjunto de optimización). Las frases seleccionadas automáticamente se usan para optimizar el sistema y garantizar que funciona correctamente. Si desea crear un modelo de traducción de uso general y sus propios archivos de optimización, asegúrese de que son un conjunto aleatorio de frases elegidas entre dominios. |
| Archivos de prueba| Estos archivos son normalmente archivos derivados, seleccionados aleatoriamente del conjunto de aprendizaje (si no ha seleccionado ningún conjunto de prueba). El propósito de estas frases es evaluar la precisión del modelo de traducción. Dado que estas son las oraciones que quiere comprobar que el sistema traduce con precisión, puede crear un conjunto de pruebas y cargarlo en el traductor. De este modo, se asegurará de que estas oraciones se usan en la evaluación del sistema (generación de una puntuación BLEU).   |
| Archivo combinado   | Un tipo de archivo en el que se encuentran frases del idioma de origen y frases traducidas contenidas en el mismo archivo. Formatos de archivo admitidos (.tmx, .xliff, .xlf, .ici, .xlsx). |
| Archivo de almacenamiento | Un archivo que contiene los demás archivos. Formatos de archivo admitidos: zip, gz, tgz.  |
| Puntuación BLEU   | [BLEU](what-is-bleu-score.md) es el método estándar del sector para evaluar la precisión del modelo de traducción. Aunque existen otros métodos de evaluación, Microsoft Translator se basa en el método BLEU para notificar la precisión a los propietarios de los proyectos.
