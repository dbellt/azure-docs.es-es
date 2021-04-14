---
title: Comprobación de la instancia del contenedor para la extracción de frases clave
titleSuffix: Azure Cognitive Services
description: Aprenda a comprobar la instancia del contenedor para la extracción de frases clave.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: include
ms.date: 04/01/2020
ms.author: aahi
ms.openlocfilehash: 5dca4828a5c1127133461ddf9fc06099fc176b68
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "96009947"
---
### <a name="verify-the-key-phrase-extraction-container-instance"></a>Comprobación de la instancia del contenedor para la extracción de frases clave

1. Seleccione la pestaña **Información general** y copie la dirección IP.
1. Abra una nueva pestaña del explorador y escriba la dirección IP. Por ejemplo, escriba `http://<IP-address>:5000 (http://55.55.55.55:5000`). Se mostrará la página principal del contenedor, que le permitirá saber que el contenedor se está ejecutando.

    ![Visualización de la página principal del contenedor para comprobar que se está ejecutando](../media/how-tos/container-instance/swagger-docs-on-container.png)

1. Seleccione el vínculo de **Descripción de API de servicio** para ir a la página Swagger del contenedor.

1. Elija cualquiera de las API **POST** y seleccione **Probar**. Se muestran los parámetros, que incluyen la entrada de este ejemplo:

    ```json
    {
      "documents": [
        {
          "id": "1",
          "text": "Hello world"
        },
        {
          "id": "2",
          "text": "Bonjour tout le monde"
        },
        {
          "id": "3",
          "text": "La carretera estaba atascada. Había mucho tráfico el día de ayer."
        },
        {
          "id": "4",
          "text": ":) :( :D"
        }
      ]
    }
    ```

1. Reemplace la entrada con el contenido JSON siguiente:

    ```json
    {
      "documents": [
        {
          "language": "en",
          "id": "7",
          "text": "I was fortunate to attend the KubeCon Conference in Barcelona, it is one of the best conferences I have ever attended. Great people, great sessions and I thoroughly enjoyed it!"
        }
      ]
    }
    ```

1. Establezca **showStats** en `true`.

1. Seleccione **Execute** (Ejecutar) para determinar la opinión del texto.

    El modelo empaquetado en el contenedor genera una puntuación comprendida entre 0 y 1, donde 0 es negativa y 1 es positiva.

    La respuesta JSON devuelta incluye las opiniones de la entrada de texto actualizado:

    ```json
    {
      "documents": [
        {
          "id": "7",
          "keyPhrases": [
            "Great people",
            "great sessions",
            "KubeCon Conference",
            "Barcelona",
            "best conferences"
          ],
          "statistics": {
            "charactersCount": 176,
            "transactionsCount": 1
          }
        }
      ],
      "errors": [],
      "statistics": {
        "documentsCount": 1,
        "validDocumentsCount": 1,
        "erroneousDocumentsCount": 0,
        "transactionsCount": 1
      }
    }
    ```

Ahora podemos correlacionar el documento `id` de los datos JSON de la carga de respuesta con el documento `id`de carga de solicitud original. El documento resultante tiene una matriz `keyPhrases` que contiene la lista de frases clave que se han extraído del documento de entrada correspondiente. Además, hay varias estadísticas, como `characterCount` y `transactionCount`, para cada documento resultante.