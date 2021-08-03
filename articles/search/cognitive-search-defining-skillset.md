---
title: Creación de un conjunto de aptitudes
titleSuffix: Azure Cognitive Search
description: Defina la extracción de datos, el procesamiento de lenguaje natural o los pasos de análisis de imágenes para enriquecer y extraer información estructurada de los datos para su uso en Azure Cognitive Search.
manager: nitinme
author: luiscabrer
ms.author: luisca
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 11/04/2019
ms.openlocfilehash: 2ec7f9a874bff6eaa0e23f5fb926bf031f2b059d
ms.sourcegitcommit: 832e92d3b81435c0aeb3d4edbe8f2c1f0aa8a46d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/07/2021
ms.locfileid: "111555978"
---
# <a name="how-to-create-a-skillset-in-an-ai-enrichment-pipeline-in-azure-cognitive-search"></a>Creación de un conjunto de aptitudes en una canalización de enriquecimiento de inteligencia artificial en Azure Cognitive Search 

![Fases del indexador](media/cognitive-search-defining-skillset/indexer-stages-skillset.png "fases del indexador")

Un conjunto de aptitudes define las operaciones que extraen y enriquecen los datos para que se puedan buscar. Un conjunto de aptitudes se ejecuta después de extraer el contenido del texto y de la imagen de los documentos de origen, y después de que los campos del documento de origen se asignen (opcionalmente) a los campos de destino de un índice o almacén de conocimiento.

Un conjunto de aptitudes contiene una o varias *aptitudes cognitivas* que representan una operación de enriquecimiento específica, como la traducción de texto, la extracción de frases clave o la realización de reconocimiento óptico de caracteres de un archivo de imagen. Para crear un conjunto de aptitudes, puede usar [conocimientos integrados](cognitive-search-predefined-skills.md) de Microsoft o conocimientos personalizados que contengan modelos o lógica de procesamiento que proporcione (vea [Ejemplo: Creación de una aptitud personalizada en una canalización de enriquecimiento de inteligencia artificial](cognitive-search-create-custom-skill-example.md) para más información).

En este artículo aprenderá a crear una canalización de enriquecimiento para las aptitudes que desee utilizar. Hay un conjunto de aptitudes asociado a un [indexador](search-indexer-overview.md) de Azure Cognitive Search. Una parte del diseño de canalización, que se trata en este artículo, es la construcción del propio conjunto de aptitudes. 

> [!NOTE]
> Otra parte del diseño de canalización es la especificación de un indexador, que se trata en el [siguiente paso](#next-step). Una definición de indexador incluye una referencia al conjunto de aptitudes, además de las asignaciones de campos que se usan para conectar las entradas a las salidas en el índice de destino.

Puntos clave para recordar:

+ Solo puede tener un conjunto de aptitudes por indexador.
+ Un conjunto de aptitudes debe tener al menos una aptitud.
+ Puede crear varias aptitudes del mismo tipo (por ejemplo, variantes de una aptitud de análisis de imagen).

## <a name="begin-with-the-end-in-mind"></a>Inicio pensando en el final

Un paso inicial recomendado consiste en decidir qué datos se extraen de los datos sin procesar y cómo desea usar esos datos en una solución de búsqueda. La creación de una ilustración de la canalización de enriquecimiento completa puede ayudarle a identificar los pasos necesarios.

Supongamos que está interesado en el procesamiento de un conjunto de comentarios de los analistas financieros. Por cada archivo, desea extraer los nombres de las compañías y la opinión general de los comentarios. También le podría interesar escribir un enriquecedor personalizado que utilice el servicio Entity Search de Bing para buscar información adicional acerca de la compañía, como a qué tipo de negocio se dedica dicha compañía. Básicamente, desea extraer información similar a la siguiente, indexada para cada documento:

| texto de registro | compañías | opinión | descripciones de las compañías |
|--------|-----|-----|-----|
|registro de ejemplo| ["Microsoft", "LinkedIn"] | 0,99 | ["Microsoft Corporation es una compañía de tecnología multinacional estadounidense...", "LinkedIn es una red social orientada a las empresas y al empleo..."]

El siguiente diagrama muestra una canalización de enriquecimiento hipotética:

![Una canalización de enriquecimiento hipotética](media/cognitive-search-defining-skillset/sample-skillset.png "Una canalización de enriquecimiento hipotética")


Una vez que tenga una idea clara de lo que desea en la canalización, puede expresar el conjunto de aptitudes que proporciona estos pasos. Funcionalmente, el conjunto de aptitudes se expresa al cargar la definición de indexador en Azure Cognitive Search. Para más información sobre cómo cargar el indexador, consulte la [documentación del indexador](/rest/api/searchservice/create-indexer).


En el diagrama, el paso de *descifrado de documentos* se realiza automáticamente. Básicamente, Azure Cognitive Search sabe cómo abrir archivos conocidos y crea un campo *content* que contiene el texto extraído de cada documento. Los cuadros blancos son enriquecedores integrados y el cuadro "Entity Search de Bing" con puntos representa un enriquecedor personalizado que está creando. Como se muestra, el conjunto de aptitudes contiene tres aptitudes.

## <a name="skillset-definition-in-rest"></a>Definición del conjunto de aptitudes en REST

Un conjunto de aptitudes se define como una matriz de aptitudes. Cada aptitud define el origen de sus entradas y el nombre de las salidas generadas. Mediante la [API REST de creación de conjuntos de aptitudes](/rest/api/searchservice/create-skillset), puede definir un conjunto de aptitudes que se corresponde con el diagrama anterior: 

```http
PUT https://[servicename].search.windows.net/skillsets/[skillset name]?api-version=2020-06-30
api-key: [admin key]
Content-Type: application/json
```

```json
{
  "description": 
  "Extract sentiment from financial records, extract company names, and then find additional information about each company mentioned.",
  "skills":
  [
    {
      "@odata.type": "#Microsoft.Skills.Text.EntityRecognitionSkill",
      "context": "/document",
      "categories": [ "Organization" ],
      "defaultLanguageCode": "en",
      "inputs": [
        {
          "name": "text",
          "source": "/document/content"
        }
      ],
      "outputs": [
        {
          "name": "organizations",
          "targetName": "orgs"
        }
      ]
    },
    {
      "@odata.type": "#Microsoft.Skills.Text.SentimentSkill",
      "inputs": [
        {
          "name": "text",
          "source": "/document/content"
        }
      ],
      "outputs": [
        {
          "name": "score",
          "targetName": "mySentiment"
        }
      ]
    },
    {
      "@odata.type": "#Microsoft.Skills.Custom.WebApiSkill",
     "description": "Calls an Azure function, which in turn calls Bing Entity Search",
      "uri": "https://indexer-e2e-webskill.azurewebsites.net/api/InvokeTextAnalyticsV3?code=foo",
      "httpHeaders": {
          "Ocp-Apim-Subscription-Key": "foobar"
      },
      "context": "/document/orgs/*",
      "inputs": [
        {
          "name": "query",
          "source": "/document/orgs/*"
        }
      ],
      "outputs": [
        {
          "name": "description",
          "targetName": "companyDescription"
        }
      ]
    }
  ]
}
```

## <a name="create-a-skillset"></a>Creación de un conjunto de aptitudes

Al crear un conjunto de aptitudes, puede proporcionar una descripción para que el conjunto de aptitudes se autodocumente. Una descripción es opcional, pero resulta útil para realizar el seguimiento de lo que hace un conjunto de aptitudes. Puesto que el conjunto de aptitudes es un documento JSON, que no se permiten comentarios; debe usar un elemento `description` para esto.

```json
{
  "description": 
  "This is our first skill set, it extracts sentiment from financial records, extract company names, and then finds additional information about each company mentioned.",
  ...
}
```

La siguiente parte del conjunto de aptitudes es una matriz de aptitudes. Cada aptitud se puede considerar como un tipo primitivo de enriquecimiento. Cada aptitud realiza una tarea pequeña en esta canalización de enriquecimiento. Cada una de ellas toma una entrada (o un conjunto de entradas) y devuelve algunas salidas. Las siguientes secciones se centran en cómo especificar aptitudes integradas y personalizadas al encadenar aptitudes mediante referencias de entrada y salida. Las entradas pueden proceder de datos de origen o de otra aptitud. Las salidas se pueden asignar a un campo en un índice de búsqueda o utilizarse como entrada para una aptitud de bajada.

## <a name="add-built-in-skills"></a>Agregar aptitudes integradas

Echemos un vistazo a la primera aptitud, que es la [aptitud de reconocimiento de entidades](cognitive-search-skill-entity-recognition-v3.md) integrada:

```json
    {
      "@odata.type": "#Microsoft.Skills.Text.V3.EntityRecognitionSkill",
      "context": "/document",
      "categories": [ "Organization" ],
      "defaultLanguageCode": "en",
      "inputs": [
        {
          "name": "text",
          "source": "/document/content"
        }
      ],
      "outputs": [
        {
          "name": "organizations",
          "targetName": "orgs"
        }
      ]
    }
```

* Cada aptitud integrada tiene las propiedades `odata.type`, `input` y `output`. Las propiedades específicas de la aptitud proporcionan información adicional aplicable a dicha aptitud. Para el reconocimiento de entidades, `categories` es una entidad entre un conjunto fijo de tipos de entidad que el modelo previamente entrenado puede reconocer.

* Cada aptitud debe tener un ```"context"```. El contexto representa el nivel en el que tienen lugar las operaciones. En la aptitud anterior, el contexto es todo el documento, lo que significa que a la aptitud de reconocimiento de entidades se la llama una vez por documento. Las salidas también se producen en ese nivel. La aptitud devuelve una propiedad denominada ```organizations``` que se captura como ```orgs```. Más concretamente, ```"orgs"``` ahora se agregó como miembro de ```"/document"```. En las aptitudes de bajada, puede hacer referencia a este enriquecimiento recién creado como ```"/document/orgs"```.  Si el campo ```"context"``` no se establece explícitamente, el contexto predeterminado es el documento.

* Las salidas de una aptitud pueden estar en conflicto con las salidas de otra aptitud. Si tiene varias aptitudes que devuelven una propiedad ```result```, puede usar la propiedad ```targetName``` de las salidas de aptitud para capturar una salida JSON con nombre de una aptitud hacia una propiedad diferente.

* La aptitud tiene una entrada denominada "text", con una entrada de origen establecida en ```"/document/content"```. La aptitud (reconocimiento de entidades) funciona en el campo *content* de cada documento, que es un campo estándar que crea el indexador de Azure Blob. 

* La aptitud tiene una salida denominada ```"organizations"``` que se captura en una propiedad ```orgs```. Salidas existen únicamente durante el procesamiento. Para encadenar esta salida a la entrada de una aptitud de bajada, haga referencia a la salida como ```"/document/orgs"```.

* Para un documento determinado, el valor de ```"/document/orgs"``` es una matriz de las organizaciones extraídas del texto. Por ejemplo:

  ```json
  ["Microsoft", "LinkedIn"]
  ```

Algunas situaciones requieren que se haga referencia a cada elemento de una matriz por separado. Por ejemplo, imagine que desea pasar cada elemento de ```"/document/orgs"``` por separado a otra aptitud (por ejemplo, el enriquecedor de Entity Search de Bing personalizado). Puede hacer referencia a cada elemento de la matriz agregando un asterisco a la ruta de acceso: ```"/document/orgs/*"``` 

La segunda aptitud para la extracción de opiniones sigue el mismo patrón que el primer enriquecedor. Toma ```"/document/content"``` como entrada y devuelve una puntuación de opinión para cada instancia del contenido. Puesto que no estableció el campo ```"context"``` explícitamente, la salida (mySentiment) ahora es un elemento secundario de ```"/document"```.

```json
    {
      "@odata.type": "#Microsoft.Skills.Text.SentimentSkill",
      "inputs": [
        {
          "name": "text",
          "source": "/document/content"
        }
      ],
      "outputs": [
        {
          "name": "score",
          "targetName": "mySentiment"
        }
      ]
    },
```

## <a name="add-a-custom-skill"></a>Incorporación de una aptitud personalizada

Recupere la estructura del enriquecedor de Entity Search de Bing personalizado:

```json
    {
      "@odata.type": "#Microsoft.Skills.Custom.WebApiSkill",
     "description": "This skill calls an Azure function, which in turn calls Bing Entity Search",
      "uri": "https://indexer-e2e-webskill.azurewebsites.net/api/InvokeTextAnalyticsV3?code=foo",
      "httpHeaders": {
          "Ocp-Apim-Subscription-Key": "foobar"
      },
      "context": "/document/orgs/*",
      "inputs": [
        {
          "name": "query",
          "source": "/document/orgs/*"
        }
      ],
      "outputs": [
        {
          "name": "description",
          "targetName": "companyDescription"
        }
      ]
    }
```

Esta definición es una [aptitud personalizada](cognitive-search-custom-skill-web-api.md) que llama a una API web como parte del proceso de enriquecimiento. En cada organización identificada por el reconocimiento de entidades, esta aptitud llama a una API web para encontrar la descripción de esa organización. La orquestación de cuándo llamar a la API web y cómo pasar la información recibida se controla internamente mediante el motor de enriquecimiento. Sin embargo, se debe proporcionar la inicialización necesaria para llamar a esta API personalizada en JSON (por ejemplo, el identificador uri, httpHeaders y las entradas esperadas). Para obtener instrucciones para crear una API web personalizada para la canalización de enriquecimiento, consulte [How to define a custom interface](cognitive-search-custom-skill-interface.md) (Definición de una interfaz personalizada).

Tenga en cuenta que el campo "context" está establecido en ```"/document/orgs/*"``` con un asterisco, lo que significa que se llama al paso de enriquecimiento *para cada* organización bajo ```"/document/orgs"```. 

La salida, en este caso una descripción de la compañía, se genera para cada organización identificada. Cuando se hace referencia a la descripción en un paso de bajada (por ejemplo, en la extracción de frases clave), se usaría la ruta de acceso ```"/document/orgs/*/description"``` para hacerlo. 

## <a name="add-structure"></a>Agregar estructura

El conjunto de aptitudes genera información estructurada a partir de datos no estructurados. Considere el ejemplo siguiente:

*"En el cuarto trimestre, Microsoft registró unos ingresos por valor de 1100 millones de dólares de LinkedIn, la compañía de red social que compró el año pasado. La adquisición permite a Microsoft combinar las funcionalidades de LinkedIn con su CRM y funcionalidades de Office. Los accionistas están satisfechos con el progreso hasta ahora.".*

Una salida probable sería una estructura generada similar a la siguiente ilustración:

![Ejemplo de estructura de salida](media/cognitive-search-defining-skillset/enriched-doc.png "Ejemplo de estructura de salida")

Hasta ahora, esta estructura ha sido solo de uso interno y solo de memoria, además de emplearse exclusivamente en índices de Azure Cognitive Search. La adición de un almacén de conocimiento proporciona una manera de guardar enriquecimientos con forma para su uso más allá de la búsqueda.

## <a name="add-a-knowledge-store"></a>Agregar un almacén de conocimiento

El [almacén de conocimiento](knowledge-store-concept-intro.md) es una característica de Azure Cognitive Search para guardar el documento enriquecido. Un almacén de conocimiento que se crea, respaldado por una cuenta de Azure Storage, es el repositorio donde se colocan los datos enriquecidos. 

Una definición de almacén de conocimiento se agrega a un conjunto de aptitudes. Para obtener un tutorial de todo el proceso, vea [Creación de un almacén de conocimiento mediante REST](knowledge-store-create-rest.md).

```json
"knowledgeStore": {
  "storageConnectionString": "<an Azure storage connection string>",
  "projections" : [
    {
      "tables": [ ]
    },
    {
      "objects": [
        {
          "storageContainer": "containername",
          "source": "/document/EnrichedShape/",
          "key": "/document/Id"
        }
      ]
    }
  ]
}
```

Puede optar por guardar los documentos enriquecidos como tablas con relaciones jerárquicas conservadas o como documentos JSON en Blob Storage. La salida de cualquiera de las aptitudes del conjunto de aptitudes puede servir como entrada de la proyección. Si quiere proyectar los datos en una determinada forma, la [aptitud Conformador](cognitive-search-skill-shaper.md) actualizada ahora puede modelar tipos complejos para su uso. 

<a name="next-step"></a>

## <a name="next-steps"></a>Pasos siguientes

Ahora que está familiarizado con la canalización de enriquecimiento y los conjuntos de aptitudes, continúe con [How to reference annotations in a skillset](cognitive-search-concept-annotations-syntax.md) (Referencia a las anotaciones en un conjunto de aptitudes) o [How to map outputs to fields in an index](cognitive-search-output-field-mapping.md) (Asignación de salidas a campos en un índice).