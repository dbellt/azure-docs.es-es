---
title: Configuración de un indexador de blobs
titleSuffix: Azure Cognitive Search
description: Configure un indexador de blobs de Azure para automatizar la indexación de contenido de blobs en las operaciones de búsqueda de texto completo en Azure Cognitive Search.
manager: nitinme
author: MarkHeff
ms.author: maheff
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 03/22/2021
ms.custom: contperf-fy21q3
ms.openlocfilehash: 40e3e804d62bfd0ece669ebd46891f0c72dcb71e
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108140342"
---
# <a name="how-to-configure-blob-indexing-in-cognitive-search"></a>Configuración de una indexación de blobs en Cognitive Search

Un indexador de blobs se usa para la ingesta de contenido de Azure Blob Storage a un índice de Cognitive Search. Los indexadores de blobs se usan con frecuencia en el [Enriquecimiento de IA](cognitive-search-concept-intro.md), donde un [conjunto de aptitudes](cognitive-search-working-with-skillsets.md) agrega una imagen y un procesamiento de lenguaje natural para crear el contenido de búsqueda. Sin embargo, también puede usar indexadores de blob sin enriquecimiento de IA para ingerir contenido de documentos basados en texto, como PDF, documentos de Microsoft Office y formatos de archivo.

En este artículo se muestra cómo configurar un indexador de blobs para cualquiera de los escenarios descritos. Si no está familiarizado con los conceptos de indexador, comience con [Indexadores de Azure Cognitive Search](search-indexer-overview.md) y [Creación de un indexador de búsqueda](search-howto-create-indexers.md) antes de profundizar en la indexación de blobs.

<a name="SupportedFormats"></a>

## <a name="supported-document-formats"></a>Formatos de documento admitidos

El indexador de blobs de Azure Cognitive Search puede extraer texto de los siguientes formatos de documento:

[!INCLUDE [search-blob-data-sources](../../includes/search-blob-data-sources.md)]

## <a name="data-source-definitions"></a>Definiciones de origen de datos

La diferencia principal entre un indexador de blobs y cualquier otro indexador es la definición de origen de datos que se asigna al indexador. La definición del origen de datos especifica el tipo de origen de datos ("type": "azureblob"), así como otras propiedades de autenticación y conexión con el contenido que se va a indexar.

Una definición de origen de datos de blob es similar al ejemplo siguiente:

```http
{
    "name" : "my-blob-datasource",
    "type" : "azureblob",
    "credentials" : { "connectionString" : "DefaultEndpointsProtocol=https;AccountName=<account name>;AccountKey=<account key>;" },
    "container" : { "name" : "my-container", "query" : "<optional-virtual-directory-name>" }
}
```

La propiedad `"credentials"` puede ser una cadena de conexión, como se muestra en el ejemplo anterior, o uno de los enfoques alternativos descritos en la sección siguiente. La propiedad `"container"` proporciona la ubicación del contenido dentro de Azure Storage y `"query"` se utiliza para especificar una subcarpeta en el contenedor. Para obtener más información sobre las definiciones de origen de datos, vea [Creación de un origen de datos (REST)](/rest/api/searchservice/create-data-source).

<a name="Credentials"></a>

## <a name="credentials"></a>Credenciales

Puede proporcionar las credenciales para el contenedor de blobs de una de estas maneras:

**Cadena de conexión de identidad administrada**: `{ "connectionString" : "ResourceId=/subscriptions/<your subscription ID>/resourceGroups/<your resource group name>/providers/Microsoft.Storage/storageAccounts/<your storage account name>/;" }`

Esta cadena de conexión no requiere una clave de cuenta, pero es preciso seguir las instrucciones de [Configuración de una conexión a una cuenta de Azure Storage mediante una identidad administrada](search-howto-managed-identities-storage.md).

**Cadena de conexión de la cuenta de almacenamiento de acceso total**: `{ "connectionString" : "DefaultEndpointsProtocol=https;AccountName=<your storage account>;AccountKey=<your account key>;" }`

Para obtener la cadena de conexión del portal de Azure, vaya a la hoja de la cuenta de almacenamiento > Configuración > Claves (para las cuentas de almacenamiento del modelo clásico) o Configuración > Claves de acceso (para las cuentas de almacenamiento de Azure Resource Manager).

Cadena de conexión de la **firma de acceso compartido de la cuenta de almacenamiento** (SAS): `{ "connectionString" : "BlobEndpoint=https://<your account>.blob.core.windows.net/;SharedAccessSignature=?sv=2016-05-31&sig=<the signature>&spr=https&se=<the validity end time>&srt=co&ss=b&sp=rl;" }`

La firma de acceso compartido debe tener permisos de enumeración y lectura sobre los contenedores y objetos (en este caso, los blobs).

**Firma de acceso compartido de contenedor**: `{ "connectionString" : "ContainerSharedAccessUri=https://<your storage account>.blob.core.windows.net/<container name>?sv=2016-05-31&sr=c&sig=<the signature>&se=<the validity end time>&sp=rl;" }`

La firma de acceso compartido debe tener permisos de enumeración y lectura sobre el contenedor. Para más información sobre las firmas de acceso compartido de almacenamiento, consulte [Uso de firmas de acceso compartido (SAS)](../storage/common/storage-sas-overview.md).

> [!NOTE]
> Si usa credenciales SAS, deberá actualizar las credenciales del origen de datos periódicamente con firmas renovadas para evitar que caduquen. Si las credenciales SAS expiran, el indexador produce un error con un mensaje parecido a este: "Las credenciales proporcionadas en la cadena de conexión no son válidas o han expirado".  

## <a name="index-definitions"></a>Definiciones de índice

El índice especifica los campos de un documento, los atributos y otras construcciones que conforman la experiencia de búsqueda. Todos los indexadores requieren que especifique una definición de índice de búsqueda como destino. En el ejemplo siguiente se crea un índice simple mediante [Creación de un índice (API REST)](/rest/api/searchservice/create-index). 

```http
POST https://[service name].search.windows.net/indexes?api-version=2020-06-30
Content-Type: application/json
api-key: [admin key]

{
      "name" : "my-target-index",
      "fields": [
        { "name": "id", "type": "Edm.String", "key": true, "searchable": false },
        { "name": "content", "type": "Edm.String", "searchable": true, "filterable": false, "sortable": false, "facetable": false }
      ]
}
```

Las definiciones de índice requieren un campo en la colección `"fields"` para que actúe como la clave del documento. Las definiciones de índice también deben incluir campos de contenido y metadatos.

Un campo **`content`** es común al contenido de blobs. Contiene el texto extraído de los blobs. La definición de este campo podría ser similar a la anterior. No es necesario usar este nombre, pero esto le permite aprovechar las asignaciones de campos implícitas. El indexador de blobs puede enviar el contenido de los blobs a un campo de contenido Edm.String en el índice, sin necesidad de realizar asignaciones de campos.

También puede agregar campos para los metadatos de blob que quiera en el índice. El indexador puede leer propiedades de metadatos personalizadas, propiedades de [metadatos estándar](#indexing-blob-metadata) y propiedades de [metadatos específicos de contenido](search-blob-metadata-properties.md). Para obtener más información acerca de los índices, vea [Creación de un índice](search-what-is-an-index.md).

<a name="DocumentKeys"></a>

### <a name="defining-document-keys-and-field-mappings"></a>Definición de claves de documento y asignaciones de campos

En un índice de búsqueda, la clave del documento identifica de forma exclusiva cada documento. El campo que elija debe ser de tipo `Edm.String`. Para el contenido de blobs, los mejores candidatos para una clave de documento son propiedades de metadatos en el blob.

+ **`metadata_storage_name`** : esta propiedad es candidata, pero solo si los nombres son únicos en todos los contenedores y carpetas que se indexan. Independientemente de la ubicación del blob, el resultado final es que la clave del documento (nombre) debe ser única en el índice de búsqueda una vez que se ha indexado todo el contenido. 

  Otro posible problema sobre el nombre de almacenamiento es que podría contener caracteres que no son válidos para las claves de documento, como los guiones. Puede tratar los caracteres no válidos mediante la [función de asignación de campos](search-indexer-field-mappings.md#base64EncodeFunction) `base64Encode`. Si lo hace, recuerde codificar también las claves de documento al pasarlas en las llamadas API como [documento de búsqueda (REST)](/rest/api/searchservice/lookup-document). En .NET puede usar el [método UrlTokenEncode](/dotnet/api/system.web.httpserverutility.urltokenencode) para codificar caracteres.

+ **`metadata_storage_path`** : El uso de la ruta de acceso completa garantiza la exclusividad, pero la ruta de acceso contiene siempre caracteres `/` que [no son válidos en una clave de documento](/rest/api/searchservice/naming-rules). Como se indicó anteriormente, puede usar la [función](search-indexer-field-mappings.md#base64EncodeFunction) `base64Encode` para codificar caracteres.

+ Una tercera opción es agregar una propiedad de metadatos personalizados a los blobs. Esta opción requiere que el proceso de carga de blobs agregue dicha propiedad de metadatos a todos los blobs. Dado que la clave es una propiedad obligatoria, todos los blobs a los que les falte esa propiedad no se indexarán.

> [!IMPORTANT]
> Si no hay ninguna asignación explícita para el campo de clave en el índice, Azure Cognitive Search usa automáticamente `metadata_storage_path` como clave y valores de clave de codificaciones Base64 (la segunda opción anterior).
>
> Si usa una propiedad de metadatos personalizada como clave, evite realizar cambios en esa propiedad. Los indexadores agregarán documentos duplicados para el mismo blob si cambia la propiedad de clave.

#### <a name="example"></a>Ejemplo

En el siguiente ejemplo se muestra la sección `metadata_storage_name` como clave de documento. Supongamos que el índice tiene un campo de clave denominado `key` y otro campo denominado `fileSize` para almacenar el tamaño del documento. Las [asignaciones de campos](search-indexer-field-mappings.md) en la definición de indexador establecen asociaciones de campo y `metadata_storage_name` tiene la [función de asignación de campos `base64Encode`](search-indexer-field-mappings.md#base64EncodeFunction) para controlar los caracteres no admitidos.

```http
PUT https://[service name].search.windows.net/indexers/my-blob-indexer?api-version=2020-06-30
Content-Type: application/json
api-key: [admin key]

{
  "dataSourceName" : "my-blob-datasource ",
  "targetIndexName" : "my-target-index",
  "schedule" : { "interval" : "PT2H" },
  "fieldMappings" : [
    { "sourceFieldName" : "metadata_storage_name", "targetFieldName" : "key", "mappingFunction" : { "name" : "base64Encode" } },
    { "sourceFieldName" : "metadata_storage_size", "targetFieldName" : "fileSize" }
  ]
}
```

#### <a name="how-to-make-an-encoded-field-searchable"></a>Cómo hacer que un campo codificado permita búsquedas

Hay ocasiones en que se necesita usar una versión codificada de un campo como `metadata_storage_path` como clave, pero también es preciso poder realizar búsquedas en ese campo (sin codificar) en el índice de búsqueda. Para admitir ambos casos de uso, puede asignar `metadata_storage_path` a dos campos; uno para la clave (codificado) y otro para un campo de ruta de acceso, que cabe suponer que se atribuirá como campo que permite búsquedas en el esquema de índice. En el ejemplo siguiente se muestran dos asignaciones de campos para `metadata_storage_path`.

```http
PUT https://[service name].search.windows.net/indexers/blob-indexer?api-version=2020-06-30
Content-Type: application/json
api-key: [admin key]

{
  "dataSourceName" : " blob-datasource ",
  "targetIndexName" : "my-target-index",
  "schedule" : { "interval" : "PT2H" },
  "fieldMappings" : [
    { "sourceFieldName" : "metadata_storage_path", "targetFieldName" : "key", "mappingFunction" : { "name" : "base64Encode" } },
    { "sourceFieldName" : "metadata_storage_path", "targetFieldName" : "path" }
  ]
}
```

<a name="PartsOfBlobToIndex"></a>

## <a name="index-content-and-metadata"></a>Contenido y metadatos del índice

Los blobs incluyen contenido y metadatos. Puede controlar qué partes de los blobs se indizan mediante el parámetro de configuración `dataToExtract`. Puede tomar los siguientes valores:

+ `contentAndMetadata`: especifica que se indizan todos los metadatos y el contenido textual extraído del blob. Este es el valor predeterminado.

+ `storageMetadata`: especifica que solamente se indizan las [propiedades de blob estándar y los metadatos especificados por el usuario](../storage/blobs/storage-blob-container-properties-metadata.md).

+ `allMetadata`: especifica que las propiedades de blob estándar y los [metadatos para los tipos de contenido encontrados](search-blob-metadata-properties.md) se extraen del contenido del blob y se indexan.

Por ejemplo, para indizar solo los metadatos de almacenamiento, use lo siguiente:

```http
PUT https://[service name].search.windows.net/indexers/[indexer name]?api-version=2020-06-30
Content-Type: application/json
api-key: [admin key]

{
  ... other parts of indexer definition
  "parameters" : { "configuration" : { "dataToExtract" : "storageMetadata" } }
}
```

<a name="how-azure-search-indexes-blobs"></a>

### <a name="indexing-blob-content"></a>Indexación del contenido del blob

De manera predeterminada, los blobs con contenido estructurado, como JSON o CSV, se indexan como un único fragmento de texto. Sin embargo, si los documentos JSON o CSV tienen una estructura interna (delimitadores), puede asignar modos de análisis para generar documentos de búsqueda individuales para cada línea o elemento. Para obtener más información, consulte [Indexación de blobs JSON](search-howto-index-json-blobs.md) e [Indexación de blobs CSV](search-howto-index-csv-blobs.md).

Un documento compuesto o insertado (por ejemplo, un archivo ZIP o un documento de Word con correo electrónico de Outlook insertado que contiene datos adjuntos, o un archivo .MSG con datos adjuntos) también se indexa como un solo documento. Por ejemplo, todas las imágenes extraídas de los datos adjuntos de un archivo MSG se devolverán en el campo normalized_images.

Se extrae el contenido textual en un campo de cadena denominado "`content`".

  > [!NOTE]
  > Azure Cognitive Search limita la cantidad de texto que extrae según el plan de tarifa. Los [límites de servicio](search-limits-quotas-capacity.md#indexer-limits) actuales son de 32 000 caracteres en el nivel Gratis, 64 000 en Básico, 4 millones en Estándar, 8 millones en Estándar S2 y 16 millones en Estándar S3. Se incluye una advertencia en la respuesta de estado del indexador para documentos truncados.  

<a name="indexing-blob-metadata"></a>

### <a name="indexing-blob-metadata"></a>Indexación de metadatos de blob

Los indexadores también pueden indexar metadatos de blobs. En primer lugar, las propiedades de metadatos especificadas por el usuario se pueden extraer textualmente. Para recibir los valores, debe definir el campo en el índice de búsqueda de tipo `Edm.String` con el mismo nombre que la clave de metadatos del blob. Por ejemplo, si un blob tiene una clave de metadatos de `Sensitivity` con el valor `High`, debe definir un campo denominado `Sensitivity` en el índice de búsqueda y se rellenará con el valor `High`.

En segundo lugar, las propiedades de metadatos de blob estándar se pueden extraer en los campos que se indican a continuación. El indexador de blob crea automáticamente asignaciones de campo internas para estas propiedades de metadatos de blob. Todavía tiene que agregar los campos que quiere usar para la definición de índice, pero puede omitir la creación de asignaciones de campos en el indexador.

  + **metadata_storage_name** (`Edm.String`): nombre de archivo del blob. Por ejemplo, si tiene un blob /my-container/my-folder/subfolder/resume.pdf, el valor de este campo es `resume.pdf`.

  + **metadata_storage_path** (`Edm.String`): URI completo del blob, incluida la cuenta de almacenamiento. Por ejemplo: `https://myaccount.blob.core.windows.net/my-container/my-folder/subfolder/resume.pdf`

  + **metadata_storage_content_type** (`Edm.String`): tipo de contenido tal como especifica el código que usó para cargar el blob. Por ejemplo, `application/octet-stream`.

  + **metadata_storage_last_modified** (`Edm.DateTimeOffset`): última marca de tiempo modificada del blob. Azure Cognitive Search emplea esta marca de tiempo para identificar los blobs modificados a fin de evitar volver a indexar todo después de la indexación inicial.

  + **metadata_storage_size** (`Edm.Int64`): tamaño del blob en bytes.

  + **metadata_storage_content_md5** (`Edm.String`): hash MD5 del contenido del blob, si está disponible.

  + **metadata_storage_sas_token** (`Edm.String`): un token de SAS temporal que pueden usar las [aptitudes personalizadas](cognitive-search-custom-skill-interface.md) para obtener acceso al blob. Este token no se debe almacenar para su uso posterior, ya que podría expirar.

Por último, las propiedades de metadatos específicas del formato de documento de los blobs que está indexando también se pueden representar en el esquema de índice. Para obtener más información sobre los metadatos específicos de contenido, vea [Propiedades de metadatos de contenido](search-blob-metadata-properties.md).

Es importante señalar que no es necesario definir campos para todas las propiedades anteriores en el índice de búsqueda, capture solo las propiedades que necesita para la aplicación.

<a name="WhichBlobsAreIndexed"></a>

## <a name="how-to-control-which-blobs-are-indexed"></a>Control de qué blobs se indexan

Puede controlar qué blobs se indexan y cuáles se omiten en el tipo de archivo del blob o mediante el establecimiento de propiedades en el propio blob, lo que hace que el indexador los omita.

### <a name="include-specific-file-extensions"></a>Inclusión de extensiones de archivo concretas

Use `indexedFileNameExtensions` para proporcionar una lista separada por comas de extensiones de archivo que se van a indexar (con un punto inicial). Por ejemplo, para indexar solamente los blobs .PDF y .DOCX, realice el siguiente procedimiento:

```http
PUT https://[service name].search.windows.net/indexers/[indexer name]?api-version=2020-06-30
Content-Type: application/json
api-key: [admin key]

{
  ... other parts of indexer definition
  "parameters" : { "configuration" : { "indexedFileNameExtensions" : ".pdf,.docx" } }
}
```

### <a name="exclude-specific-file-extensions"></a>Exclusión de extensiones de archivo concretas

Use `excludedFileNameExtensions` para proporcionar una lista separada por comas de extensiones de archivo que se van a omitir (también con un punto inicial). Por ejemplo, para indexar todos los blobs excepto aquellos con las extensiones .PNG y .JPEG, realice el siguiente procedimiento:

```http
PUT https://[service name].search.windows.net/indexers/[indexer name]?api-version=2020-06-30
Content-Type: application/json
api-key: [admin key]

{
  ... other parts of indexer definition
  "parameters" : { "configuration" : { "excludedFileNameExtensions" : ".png,.jpeg" } }
}
```

Si los dos parámetros `indexedFileNameExtensions` y `excludedFileNameExtensions` están presentes, el indexador mira primero en `indexedFileNameExtensions` y, luego, en `excludedFileNameExtensions`. Si la misma extensión de archivo está en las dos listas, se excluirá de la indexación.

### <a name="add-skip-metadata-the-blob"></a>Adición de metadatos "skip" al blob

Los parámetros de configuración del indexador se aplican a todos los blobs del contenedor o la carpeta. En ocasiones, quiere controlar cómo indexar *blobs concretos*. Puede hacerlo agregando los siguientes valores y propiedades de metadatos a los blobs en Blob Storage. Cuando el indexador encuentra estas propiedades, omitirá el blob o su contenido en la ejecución de la indexación.

| Nombre de propiedad | Valor de propiedad | Explicación |
| ------------- | -------------- | ----------- |
| `AzureSearch_Skip` |`"true"` |Indica al indizador de blob que pase completamente el blob. No se trata de realizar la extracción de metadatos ni del contenido. Esto es útil cuando se produce un error repetidamente y se interrumpe el proceso de indización de un blob determinado. |
| `AzureSearch_SkipContent` |`"true"` |Esto es equivalente a la configuración `"dataToExtract" : "allMetadata"` descrita [anteriormente](#PartsOfBlobToIndex) en el ámbito de un blob determinado. |

## <a name="index-large-datasets"></a>Indexación de conjuntos de datos grandes

La indización de blobs puede ser un proceso lento. En los casos donde hay millones de blobs que se van a indexar, puede acelerar la indexación dividiendo los datos y utilizando varios indexadores para [procesar los datos en paralelo](search-howto-large-index.md#parallel-indexing). Le mostramos cómo puede configurar esta opción:

+ Divida los datos en varios contenedores de blobs o carpetas virtuales.

+ Configure varios orígenes de datos, uno por cada contenedor o carpeta. Para seleccionar una carpeta de blobs, use el parámetro `query`:

    ```json
    {
        "name" : "blob-datasource",
        "type" : "azureblob",
        "credentials" : { "connectionString" : "<your storage connection string>" },
        "container" : { "name" : "my-container", "query" : "my-folder" }
    }
    ```

+ Cree un indizador correspondiente a cada origen de datos. Todos los indexadores deben apuntar al mismo índice de búsqueda de destino.  

+ Una unidad de búsqueda del servicio puede ejecutar un indexador en cualquier momento dado. La creación de varios indexadores como se ha descrito arriba solo es útil si se ejecutan realmente en paralelo.

  Para ejecutar varios indexadores en paralelo, escale horizontalmente el servicio de búsqueda mediante la creación de un número adecuado de particiones y réplicas. Por ejemplo, si el servicio de búsqueda tiene 6 unidades de búsqueda (por ejemplo, 2 particiones x 3 réplicas), se pueden ejecutar 6 indexadores simultáneamente, lo que produce una multiplicación por seis del rendimiento de indexación. Para más información sobre el escalado y la planificación de capacidad, consulte [Ajuste de la capacidad de un servicio de Azure Cognitive Search](search-capacity-planning.md).

<a name="DealingWithErrors"></a>

## <a name="handling-errors"></a>Control de errores

Los errores que suelen producirse durante la indexación incluyen tipos de contenido no admitidos, contenido que falta o blobs demasiado grandes.

De forma predeterminada, el indizador de blob se detiene cuando encuentra un blob con un tipo de contenido no admitido (por ejemplo, una imagen). Puede usar el parámetro `excludedFileNameExtensions` para omitir determinados tipos de contenido. Sin embargo, es posible que quiera realizar la indexación para continuar aunque se produzcan errores y, a continuación, depurar documentos concretos posteriormente. Para obtener más información sobre los errores del indexador, vea [Solución de problemas comunes con el indexador](search-indexer-troubleshooting.md) y [Errores y advertencias del indexador](cognitive-search-common-errors-warnings.md).

### <a name="respond-to-errors"></a>Respuesta a errores

Hay cuatro propiedades de indexador que controlan la respuesta del indexador cuando se producen errores. En los siguientes ejemplos se muestra cómo establecer estas propiedades en la definición del indexador. Si ya existe un indexador, puede agregar estas propiedades editando la definición en el portal.

#### <a name="maxfaileditems-and-maxfaileditemsperbatch"></a>`"maxFailedItems"` y `"maxFailedItemsPerBatch"`

Continue con la indexación si se producen errores en cualquier punto del procesamiento, mientras se analizan blobs o se agregan documentos a un índice. Establezca estas propiedades en el número de errores aceptables. Un valor de `-1` permite el procesamiento, independientemente del número de errores que se produzcan. De lo contrario, el valor es un entero positivo.

```http
PUT https://[service name].search.windows.net/indexers/[indexer name]?api-version=2020-06-30
Content-Type: application/json
api-key: [admin key]

{
  ... other parts of indexer definition
  "parameters" : { "maxFailedItems" : 10, "maxFailedItemsPerBatch" : 10 }
}
```

#### <a name="failonunsupportedcontenttype-and-failonunprocessabledocument"></a>`"failOnUnsupportedContentType"` y `"failOnUnprocessableDocument"` 

En algunos blobs, Azure Cognitive Search no puede determinar el tipo de contenido o no puede procesar un documento de otro tipo de contenido admitido. Para omitir estas condiciones de error, establezca los parámetros de configuración en `false`:

```http
PUT https://[service name].search.windows.net/indexers/[indexer name]?api-version=2020-06-30
Content-Type: application/json
api-key: [admin key]

{
  ... other parts of indexer definition
  "parameters" : { "configuration" : { "failOnUnsupportedContentType" : false, "failOnUnprocessableDocument" : false } }
}
```

### <a name="relax-indexer-constraints"></a>Reducción de las restricciones del indexador

También puede establecer [propiedades de configuración de blob](/rest/api/searchservice/create-indexer#blob-configuration-parameters) que determinen de forma eficaz si existe una condición de error. La propiedad siguiente puede reducir las restricciones y suprimir los errores que, de otro modo, podrían producirse.

+ `"indexStorageMetadataOnlyForOversizedDocuments"` para indexar los metadatos de almacenamiento del contenido de blob que sea demasiado grande para procesar. Los blobs demasiado grandes se tratan como errores de forma predeterminada. Si quiere obtener información acerca de los límites de tamaño de los blobs, consulte los [límites de servicio](search-limits-quotas-capacity.md).

## <a name="see-also"></a>Consulta también

+ [Indexadores de Azure Cognitive Search](search-indexer-overview.md)
+ [Creación de un indexador](search-howto-create-indexers.md)
+ [Información general sobre el enriquecimiento con IA sobre blobs](search-blob-ai-integration.md)
+ [Información general sobre la búsqueda de blobs](search-blob-storage-integration.md)