---
title: 'Modelos personalizados: Form Recognizer'
titleSuffix: Azure Cognitive Services
description: 'Conozca los conceptos relacionados con los modelos personalizados de la API de Form Recognizer: uso y límites.'
services: cognitive-services
author: laujan
manager: nitinme
ms.service: cognitive-services
ms.subservice: forms-recognizer
ms.topic: conceptual
ms.date: 03/25/2021
ms.author: lajanuar
ms.openlocfilehash: 9d339d02310fddc3e5fd463f7d37e253614dba17
ms.sourcegitcommit: 73fb48074c4c91c3511d5bcdffd6e40854fb46e5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/31/2021
ms.locfileid: "106067595"
---
# <a name="form-recognizer-custom-models"></a>Modelos personalizados de Form Recognizer

Form Recognizer utiliza tecnología avanzada de aprendizaje automático para analizar y extraer datos de sus formularios y documentos. Un modelo de Form Recognizer es una representación de los datos extraídos que se utiliza como referencia para analizar el contenido específico. Hay dos tipos de modelos de Form Recognizer:

* **Modelos personalizados.** Los modelos personalizados de Form Recognizer representan datos extraídos de _formularios_ específicos de su empresa. Los modelos personalizados deben entrenarse para analizar los distintos datos de formulario.

* **Modelos precompilados**. Form Recognizer admite actualmente modelos precompilados para _recibos, tarjetas de presentación, tarjetas de identificación_ y _facturas_. Los modelos precompilados detectan y extraen información de las imágenes del documento y devuelven los datos extraídos en una salida JSON estructurada.

## <a name="what-does-a-custom-model-do"></a>¿Qué hace un modelo personalizado?

Con Form Recognizer, puede entrenar un modelo que extraerá información de los formularios que son pertinentes para su caso de uso. Solo necesita cinco ejemplos del mismo tipo de formulario para empezar. El modelo personalizado se puede entrenar con o sin conjuntos de valores etiquetados.

## <a name="create-use-and-manage-your-custom-model"></a>Creación, uso y administración de su modelo personalizado

En un nivel alto, los pasos para crear, entrenar y usar el modelo personalizado son los siguientes:

> [!div class="nextstepaction"]
>[&#120783;. Ensamble del conjunto de conocimientos de entrenamiento](build-training-data-set.md#custom-model-input-requirements)

La construcción de un modelo personalizado comienza con el establecimiento de su conjunto de datos de entrenamiento. Necesitará un mínimo de cinco formularios completados del mismo tipo para su conjunto de datos de ejemplo. Pueden ser de distintos tipos de archivo y contener texto y escritura a mano. Los formularios deben ser del mismo tipo de documento y seguir los [requisitos de entrada](build-training-data-set.md#custom-model-input-requirements) para Form Recognizer.  

> [!div class="nextstepaction"]
> [&#120784;. Carga del conjunto de datos de entrenamiento](build-training-data-set.md#upload-your-training-data)

Tendrá que cargar los datos de entrenamiento en un contenedor de Azure Blob Storage. Si no sabe cómo crear una cuenta de almacenamiento de Azure con un contenedor, *vea el* [inicio rápido de Azure Storage para Azure Portal](../../storage/blobs/storage-quickstart-blobs-portal.md). Use el plan de tarifa gratuito (F0) para probar el servicio y actualícelo más adelante a un nivel de pago para producción.  

> [!div class="nextstepaction"]
>[&#120785;. Entrenamiento del modelo personalizado](quickstarts/client-library.md#train-a-custom-model)

Puede entrenar el modelo [sin](quickstarts/client-library.md#train-a-model-without-labels) o [con](quickstarts/client-library.md#train-a-model-with-labels) conjuntos de datos con etiqueta. Los conjuntos de datos no etiquetados se basan únicamente en la API de diseño para detectar e identificar la información clave sin necesidad de añadir información humana. Los conjuntos de datos etiquetados también se basan en la API de diseño, pero se incluye información humana complementaria, como sus etiquetas específicas y las ubicaciones de los campos. Para utilizar datos etiquetados y no etiquetados, comience con al menos cinco formularios completados del mismo tipo para los datos de entrenamiento etiquetados y luego añada datos no etiquetados al conjunto de datos requerido.  

>[!div class="nextstepaction"]
>[&#120786;. Análisis de documentos con el modelo personalizado](quickstarts/client-library.md#analyze-forms-with-a-custom-model)

Pruebe el modelo recién entrenado con un formulario que no formase parte del conjunto de datos de entrenamiento. Puede seguir entrenando para mejorar el rendimiento de su modelo personalizado.  

> [!div class="nextstepaction"]
>[&#120787;. Administración de modelos personalizados](quickstarts/client-library.md#manage-custom-models)

En cualquier momento, puede ver una lista de todos los modelos personalizados de su suscripción, recuperar información acerca de un modelo personalizado específico o eliminar un modelo personalizado de su cuenta.

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la biblioteca cliente de Form Recognizer, consulte nuestra documentación de referencia de la API.

> [!div class="nextstepaction"]
> [Referencia de la API Form Recognizer](https://westcentralus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1-preview-3/operations/AnalyzeWithCustomForm)
>
