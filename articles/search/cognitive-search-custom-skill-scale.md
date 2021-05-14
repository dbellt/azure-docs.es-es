---
title: Escala y administración de aptitudes personalizadas
titleSuffix: Azure Cognitive Search
description: Obtenga información sobre las herramientas y técnicas para escalar horizontalmente una aptitud personalizada de forma eficaz a fin de obtener el máximo rendimiento. Las aptitudes personalizadas invocan modelos o lógicas de IA que puede agregar a una canalización de indexación enriquecida con IA en Azure Cognitive Search.
manager: luisca
author: vkurpad
ms.author: vikurpad
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 01/28/2021
ms.openlocfilehash: f80e78da5c7d226c704c8b13fc496e416f5d0f0d
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108125596"
---
# <a name="efficiently-scale-out-a-custom-skill"></a>Escalado horizontal de una aptitud personalizada de forma eficaz

Las aptitudes personalizadas son las API web que implementan una interfaz específica. Una aptitud personalizada se puede implementar en cualquier recurso direccionable de forma pública. Las implementaciones más comunes para las aptitudes personalizadas son las siguientes:
* Azure Functions para aptitudes de lógica personalizadas
* Azure Web Apps para aptitudes de IA sencilla en contenedores
* Azure Kubernetes Service para aptitudes más complejas o más grandes

## <a name="prerequisites"></a>Requisitos previos

+ Repase el artículo sobre la [interfaz de una aptitud personalizada](cognitive-search-custom-skill-interface.md) para ver una introducción sobre la interfaz de entrada/salida que debe implementar una aptitud personalizada.

+ Configure el entorno. Podría empezar por [este tutorial de principio a fin](../azure-functions/create-first-function-vs-code-python.md) para configurar una instancia de Azure Function sin servidor con las extensiones de Visual Studio Code y Python.

## <a name="skillset-configuration"></a>Configuración del conjunto de aptitudes

La configuración de una aptitud personalizada para maximizar el rendimiento del proceso de indexación requiere la comprensión de la aptitud, las configuraciones del indexador y saber cómo se relacionan las aptitudes con cada documento. Por ejemplo, el número de veces que se invoca una aptitud por documento y la duración esperada por cada invocación.

### <a name="skill-settings"></a>Configuración de aptitudes

En la [aptitud personalizada](cognitive-search-custom-skill-web-api.md), establezca los parámetros siguientes.

1. Establezca el elemento `batchSize` de la aptitud personalizada para configurar el número de registros enviados a la aptitud en una única invocación de esta.

2. Establezca el elemento `degreeOfParallelism` para calibrar el número de solicitudes simultáneas que el indexador realizará en la aptitud.

3. Establezca el elemento `timeout` en un valor suficiente para que la aptitud responda con una respuesta válida.

4. En la definición `indexer`, establezca [`batchSize`](/rest/api/searchservice/create-indexer#indexer-parameters) en el número de documentos que deben leerse del origen de datos y que se han enriquecido de forma simultánea.

### <a name="considerations"></a>Consideraciones

La configuración de estas variables para optimizar el rendimiento de los indexadores requiere determinar si la aptitud funciona mejor con muchas solicitudes pequeñas simultáneas o con menos solicitudes de gran tamaño. Hay que tener en cuenta algunas preguntas, como las siguientes:

* ¿Cuál es la cardinalidad de invocación de la aptitud? ¿La aptitud se ejecuta una vez para cada documento (por ejemplo, una aptitud de clasificación de documentos) o podría ejecutarse varias veces por documento (por ejemplo, una aptitud de clasificación de párrafos)?

* En promedio, ¿cuántos documentos se leen desde el origen de datos para rellenar una solicitud de aptitud en función del tamaño del lote de aptitudes? Idealmente, debe ser menor que el tamaño del lote del indexador. Con tamaños de lote mayores que 1, su aptitud puede recibir registros de varios documentos de origen. Por ejemplo, si el recuento de lotes del indexador es 5, el de las aptitudes es de 50 y cada documento genera solo cinco registros, el indexador deberá rellenar una solicitud de aptitud personalizada en varios de sus lotes.

* El número promedio de solicitudes que un lote del indexador puede generar debe proporcionar una configuración óptima para los grados de paralelismo. Si la infraestructura que hospeda la aptitud no admite ese nivel de simultaneidad, considere la posibilidad de marcar los grados de paralelismo. Como procedimiento recomendado, pruebe la configuración con unos cuantos documentos para validar las opciones de los parámetros.

* Realice pruebas con una muestra más pequeña de documentos y evalúe el tiempo de ejecución de su aptitud en el tiempo total necesario para procesar el subconjunto de documentos. ¿Dedica su indexador más tiempo a compilar un lote o a esperar una respuesta de su aptitud? 

* Tenga en cuenta las implicaciones ascendentes del paralelismo. Si la entrada a una aptitud personalizada es una salida de una aptitud anterior, ¿todas las aptitudes del conjunto de aptitudes se escalan horizontalmente de forma eficaz para minimizar la latencia?

## <a name="error-handling-in-the-custom-skill"></a>Control de errores en la aptitud personalizada

Las aptitudes personalizadas deben devolver un código de estado correcto HTTP 200 cuando la aptitud se complete correctamente. Si uno o varios registros de un lote producen errores, considere la posibilidad de devolver el código de varios estados 207. La lista de errores o advertencias para el registro debe contener el mensaje adecuado.

Cualquier elemento de un lote en el que se produzcan errores producirá, asimismo, un error en el documento correspondiente. Si necesita que el documento se ejecute correctamente, devuelva una advertencia.

Cualquier código de estado superior a 299 se evalúa como un error y todos los enriquecimientos son erróneos, lo que da como resultado un documento con errores. 

### <a name="common-error-messages"></a>Mensajes comunes de error

* `Could not execute skill because it did not execute within the time limit '00:00:30'. This is likely transient. Please try again later. For custom skills, consider increasing the 'timeout' parameter on your skill in the skillset.` Establezca el parámetro timeout en la aptitud para permitir una duración de ejecución más larga.

* `Could not execute skill because Web Api skill response is invalid.` Indicativo de que la aptitud no devuelve un mensaje en el formato de respuesta de aptitudes personalizado. Esto podría ser el resultado de una excepción no detectada en la aptitud.

* `Could not execute skill because the Web Api request failed.` Probablemente debido a errores de autorización o excepciones.

* `Could not execute skill.` Normalmente, el resultado de la respuesta de aptitudes se asigna a una propiedad existente en la jerarquía de documentos.

## <a name="testing-custom-skills"></a>Prueba de aptitudes personalizadas

Empiece por probar su aptitud personalizada con un cliente de la API REST para validar lo siguiente:

* La aptitud implementa la interfaz personalizada de aptitudes para las solicitudes y las respuestas.

* La aptitud devuelve un archivo JSON válido con el tipo MIME `application/JSON`.

* Devuelve un código de estado HTTP válido.

Cree una [sesión de depuración](cognitive-search-debug-session.md) para agregar su aptitud al conjunto de aptitudes y asegúrese de que produce un enriquecimiento válido. Aunque una sesión de depuración no permite optimizar el rendimiento de la aptitud, le permite asegurarse de que esta aptitud se configura con valores válidos y devuelve los objetos enriquecidos esperados.

## <a name="best-practices"></a>Procedimientos recomendados

* Aunque las aptitudes pueden aceptar y devolver cargas de mayor tamaño, considere la posibilidad de limitar la respuesta a 150 MB o menos al devolver JSON.

* Considere la posibilidad de establecer el tamaño del lote en el indexador y la aptitud para asegurarse de que cada lote de origen de datos genera una carga completa para su aptitud.

* En el caso de las tareas de larga duración, establezca el tiempo de espera en un valor suficientemente alto para asegurarse de que el indexador no provoca un error al procesar documentos simultáneamente.

* Optimice el tamaño del lote del indexador, el tamaño del lote de aptitudes y los grados de paralelismo de la aptitud para generar el patrón de carga que espera su aptitud, menos solicitudes de gran tamaño o muchas solicitudes pequeñas.

* Supervise las aptitudes personalizadas con registros detallados de los errores, ya que puede tener escenarios en los que se produzcan errores de forma coherente en las solicitudes específicas como resultado de la variabilidad en los datos.


## <a name="next-steps"></a>Pasos siguientes
Felicidades. Su aptitud personalizada ahora se ha escalado de forma adecuada para maximizar el rendimiento del indexador. 

+ [Aptitudes avanzadas: un repositorio de aptitudes personalizadas](https://github.com/Azure-Samples/azure-search-power-skills)
+ [Incorporación de una aptitud personalizada a una canalización de enriquecimiento de inteligencia artificial](cognitive-search-custom-skill-interface.md)
+ [Incorporación de una aptitud de Azure Machine Learning](./cognitive-search-aml-skill.md)
+ [Uso de sesiones de depuración para probar los cambios](./cognitive-search-debug-session.md)