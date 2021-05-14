---
title: 'Emparejamiento y alineación de oraciones: Custom Translator'
titleSuffix: Azure Cognitive Services
description: 'Durante la ejecución de un entrenamiento, las oraciones disponibles en los documentos paralelos se emparejan o se alinean. Custom Translator aprende las traducciones una a la vez: primero, lee una oración y, después, lee la traducción de dicha oración. A continuación, alinea entre sí las palabras y frases presentes en ambas oraciones.'
author: laujan
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.date: 04/19/2021
ms.author: lajanuar
ms.topic: conceptual
ms.openlocfilehash: f286e2bcded99ad59b706bd7e6d8b2b29f4b4c0a
ms.sourcegitcommit: fc9fd6e72297de6e87c9cf0d58edd632a8fb2552
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108292369"
---
# <a name="sentence-pairing-and-alignment-in-parallel-documents"></a>Emparejamiento y alineación de oraciones en documentos paralelos

Una vez cargados los documentos, las oraciones presentes en los documentos paralelos se emparejan o se alinean. Custom Translator notifica el número de oraciones que pudo emparejar con el nombre de Oraciones alineadas en cada uno de los conjuntos de datos.

## <a name="pairing-and-alignment-process"></a>Proceso de emparejamiento y alineación

Custom Translator aprende la traducción de las oraciones una a la vez. Lee una oración del texto de origen y, a continuación, la traducción de esta oración del texto de destino. A continuación, alinea entre sí las palabras y frases presentes en ambas oraciones. Este proceso le permite asignar las palabras y frases en una oración a las palabras y frases equivalentes en la traducción de la oración. La alineación intenta asegurarse de que el sistema se entrena con oraciones que se corresponden entre sí como traducciones.

## <a name="pre-aligned-documents"></a>Documentos alineados previamente

Si sabe que tiene documentos paralelos, puede invalidar la alineación de oraciones al proporcionar los archivos de texto alineados previamente. Puede extraer todas las oraciones de ambos documentos en un archivo de texto, organizar una oración por línea y cargarlas con la extensión `.align`. La extensión `.align` indica a Custom Translator que debe omitir la alineación de oraciones.

Para obtener mejores resultados, intente asegurarse de que hay una oración por línea en sus archivos.  No incluya caracteres de nueva línea en una oración, ya que esto ocasionará alineaciones deficientes.

## <a name="suggested-minimum-number-of-sentences"></a>Número mínimo de frases sugerido

Para que un entrenamiento se realice correctamente, en la tabla siguiente se muestra el número mínimo de frases necesarias en cada tipo de documento. Esta limitación es una red de seguridad para asegurarse de que las frases paralelas contienen suficiente vocabulario único para entrenar correctamente un modelo de traducción. La directriz general consiste en tener más frases paralelas en el dominio con calidad de traducción humana y generar modelos de mayor calidad.

| Tipo de documento   | Número mínimo sugerido de frases | Número máximo de frases |
|------------|--------------------------------------------|--------------------------------|
| Cursos   | 10 000                                     | No hay límite superior                 |
| Ajuste     | 500                                      | 2,500       |
| Prueba    | 500                                      | 2,500  |
| Diccionario | 0                                          | 250 000                 |

> [!NOTE]
>
> - El entrenamiento no se iniciará y se producirá un error si no se cumple el número mínimo de 10 000 frases para el entrenamiento.
> - La optimización y las pruebas son opcionales. Si no las proporciona, el sistema quitará un porcentaje adecuado del entrenamiento que se usará para la validación y las pruebas.
> - Puede entrenar un modelo usando solo los datos de un diccionario. Consulte [Qué es el diccionario](./what-is-dictionary.md).
> - Si el diccionario contiene más de 250 000 oraciones, es probable que la **[traducción de documentos](../document-translation/overview.md)** sea una mejor opción.

## <a name="next-steps"></a>Pasos siguientes

- Aprenda a usar un [diccionario](what-is-dictionary.md) en Custom Translator.