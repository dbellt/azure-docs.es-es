---
title: 'Inicio rápido: Creación de un índice de búsqueda en JavaScript'
titleSuffix: Azure Cognitive Search
description: En este inicio rápido de JavaScript, se explica cómo crear un índice, cargar datos y ejecutar consultas en Azure Cognitive Search mediante JavaScript.
author: dereklegenzoff
manager: luisca
ms.author: delegenz
ms.devlang: javascript
ms.service: cognitive-search
ms.topic: quickstart
ms.date: 06/11/2021
ms.custom: devx-track-js
ms.openlocfilehash: f3ca078bd712d711f4c8883360eb121287662a16
ms.sourcegitcommit: 942a1c6df387438acbeb6d8ca50a831847ecc6dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112018358"
---
# <a name="quickstart-create-an-azure-cognitive-search-index-using-the-javascript-sdk"></a>Inicio rápido: Creación de un índice de Azure Cognitive Search mediante el SDK de JavaScript
> [!div class="op_single_selector"]
> * [JavaScript](search-get-started-javascript.md)
> * [C#](search-get-started-dotnet.md)
> * [Portal](search-get-started-portal.md)
> * [PowerShell](./search-get-started-powershell.md)
> * [Python](search-get-started-python.md)
> * [REST](search-get-started-rest.md)


Use el [SDK de JavaScript/TypeScript para Azure Cognitive Search](/javascript/api/overview/azure/search-documents-readme) para crear una aplicación de Node.js en JavaScript que cree, cargue y consulte un indice de búsqueda.

En este artículo se muestra cómo crear la aplicación paso a paso. También puede [descargar el código fuente y los datos](https://github.com/Azure-Samples/azure-search-javascript-samples/tree/master/quickstart/v11) y ejecutar la aplicación desde la línea de comandos.

## <a name="prerequisites"></a>Requisitos previos

Antes de empezar, tenga preparadas las herramientas y los servicios siguientes:

+ Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/).

+ Un servicio de Azure Cognitive Search. [Cree un servicio](search-create-service-portal.md) o [busque uno existente](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Search%2FsearchServices). Puede usar un servicio gratuito para este inicio rápido. 

+ [Node.js](https://nodejs.org) y [NPM](https://www.npmjs.com)

+ [Visual Studio Code](https://code.visualstudio.com) u otro IDE


## <a name="set-up-your-project"></a>Configurar su proyecto

Empiece por obtener el punto de conexión y la clave del servicio de búsqueda. Después, cree un proyecto con NPM como se describe a continuación.

<a name="get-service-info"></a>

### <a name="copy-a-key-and-endpoint"></a>Copia de una clave y un punto de conexión

Las llamadas al servicio requieren un punto de conexión de URL y una clave de acceso en cada solicitud. Como primer paso, busque la clave de API y la dirección URL que se vayan a agregar al proyecto. Deberá especificar ambos valores al crear el cliente en un paso posterior.

1. [Inicie sesión en Azure Portal](https://portal.azure.com/) y en la página **Introducción** del servicio de búsqueda, obtenga la dirección URL. Un punto de conexión de ejemplo podría ser similar a `https://mydemo.search.windows.net`.

2. En **Configuración** > **Claves**, obtenga una clave de administración para los derechos completos en el servicio, esta es necesaria si va a crear o eliminar objetos. Hay dos claves intercambiables una principal y otra secundaria. Puede usar cualquiera de ellas.

   ![Obtención de una clave de acceso y un punto de conexión HTTP](media/search-get-started-rest/get-url-key.png "Obtención de una clave de acceso y un punto de conexión HTTP")

Todas las solicitudes requieren una clave de API en cada solicitud enviada al servicio. Tener una clave válida genera la confianza, solicitud a solicitud, entre la aplicación que envía la solicitud y el servicio que se encarga de ella.

### <a name="create-a-new-npm-project"></a>Creación de un proyecto de NPM

Para empezar, abra VS Code y su [terminal integrado](https://code.visualstudio.com/docs/editor/integrated-terminal) u otro terminal como el símbolo del sistema de Node.js.

1. Cree un directorio de desarrollo y asígnele el nombre `quickstart`:

    ```cmd
    mkdir quickstart
    cd quickstart
    ```

2. Inicialice un proyecto vacío con NPM mediante la ejecución de 

    ```cmd
    npm init
    ```
     Acepte los valores predeterminados, excepto la licencia, que tiene que establecerse en “MIT”. 

3. Instale `@azure/search-documents`, el [SDK de JavaScript/TypeScript para Azure Cognitive Search](/javascript/api/overview/azure/search-documents-readme).

    ```cmd
    npm install @azure/search-documents
    ```

4. Instale `dotenv`, que se usa para importar las variables de entorno, como nuestro nombre de servicio y clave de API.
    ```cmd
    npm install dotenv
    ```

5. Confirme que ha configurado los proyectos y sus dependencias, para lo que debe comprobar que el archivo **package.json** es similar al siguiente código JSON:

    ```json
    {
      "name": "quickstart",
      "version": "1.0.0",
      "description": "Azure Cognitive Search Quickstart",
      "main": "index.js",
      "scripts": {
        "test": "echo \"Error: no test specified\" && exit 1"
      },
      "keywords": [
        "Azure",
        "Search"
      ],
      "author": "Your Name",
      "license": "MIT",
      "dependencies": {
        "@azure/search-documents": "^11.0.3",
        "dotenv": "^8.2.0"
      }
    }
    ```

6. Cree un archivo **.env** que contenga los parámetros del servicio de búsqueda:

    ```
    SEARCH_API_KEY=<search-admin-key>
    SEARCH_API_ENDPOINT=https://<search-service-name>.search.windows.net
    ```

Reemplace el valor `<search-service-name>` por el nombre del servicio de búsqueda. Reemplace `<search-admin-key>` por el valor de la clave que anotó anteriormente. 

### <a name="create-indexjs-file"></a>Creación del archivo index.js

A continuación, se crea un archivo **index.js**, que es el archivo principal que hospedará el código.

Al principio de este archivo, se importa la biblioteca `@azure/search-documents`:

```javascript
const { SearchIndexClient, SearchClient, AzureKeyCredential, odata } = require("@azure/search-documents");
```

A continuación, es preciso requerir que el paquete `dotenv` lea los parámetros del archivo **.env** como se indica a continuación:

```javascript
// Load the .env file if it exists
require("dotenv").config();

// Getting endpoint and apiKey from .env file
const endpoint = process.env.SEARCH_API_ENDPOINT || "";
const apiKey = process.env.SEARCH_API_KEY || "";
```

Con las importaciones y las variables de entorno incorporadas, ya se puede definir la función principal.

La mayor parte de la funcionalidad del SDK es asincrónica, por lo que `async` es la función principal. También se incluye `main().catch()` debajo de la función principal para detectar y registrar los errores encontrados:

```javascript
async function main() {
    console.log(`Running Azure Cognitive Search Javascript quickstart...`);
    if (!endpoint || !apiKey) {
        console.log("Make sure to set valid values for endpoint and apiKey with proper authorization.");
        return;
    }

    // remaining quickstart code will go here
}

main().catch((err) => {
    console.error("The sample encountered an error:", err);
});
```

A continuación, se puede crear un índice.

## <a name="1---create-index"></a>1 - Creación de un índice 

Cree el archivo **hotels_quickstart_index.json**.  Este archivo define la forma en que Azure Cognitive Search funciona con los documentos que cargará en el paso siguiente. Cada campo se identificará mediante un elemento `name` y tendrá un elemento especificado `type`. Cada campo también tiene una serie de atributos del índice que especifican si Azure Cognitive Search puede buscar, filtrar, ordenar y cambiar las facetas del campo. La mayoría de los campos son tipos de datos simples; pero algunos, como `AddressType`, son tipos complejos que le permiten crear estructuras de datos enriquecidos en el índice.  Puede leer más sobre los [tipos de datos admitidos](/rest/api/searchservice/supported-data-types) y los atributos de índice descritos en [Creación de un índice (REST)](/rest/api/searchservice/create-index). 

Agregue lo siguiente a **hotels_quickstart_index.json**, o bien [descargue el archivo](https://github.com/Azure-Samples/azure-search-javascript-samples/blob/master/quickstart/v11/hotels_quickstart_index.json). 

```json
{
    "name": "hotels-quickstart",
    "fields": [
        {
            "name": "HotelId",
            "type": "Edm.String",
            "key": true,
            "filterable": true
        },
        {
            "name": "HotelName",
            "type": "Edm.String",
            "searchable": true,
            "filterable": false,
            "sortable": true,
            "facetable": false
        },
        {
            "name": "Description",
            "type": "Edm.String",
            "searchable": true,
            "filterable": false,
            "sortable": false,
            "facetable": false,
            "analyzer": "en.lucene"
        },
        {
            "name": "Description_fr",
            "type": "Edm.String",
            "searchable": true,
            "filterable": false,
            "sortable": false,
            "facetable": false,
            "analyzer": "fr.lucene"
        },
        {
            "name": "Category",
            "type": "Edm.String",
            "searchable": true,
            "filterable": true,
            "sortable": true,
            "facetable": true
        },
        {
            "name": "Tags",
            "type": "Collection(Edm.String)",
            "searchable": true,
            "filterable": true,
            "sortable": false,
            "facetable": true
        },
        {
            "name": "ParkingIncluded",
            "type": "Edm.Boolean",
            "filterable": true,
            "sortable": true,
            "facetable": true
        },
        {
            "name": "LastRenovationDate",
            "type": "Edm.DateTimeOffset",
            "filterable": true,
            "sortable": true,
            "facetable": true
        },
        {
            "name": "Rating",
            "type": "Edm.Double",
            "filterable": true,
            "sortable": true,
            "facetable": true
        },
        {
            "name": "Address",
            "type": "Edm.ComplexType",
            "fields": [
                {
                    "name": "StreetAddress",
                    "type": "Edm.String",
                    "filterable": false,
                    "sortable": false,
                    "facetable": false,
                    "searchable": true
                },
                {
                    "name": "City",
                    "type": "Edm.String",
                    "searchable": true,
                    "filterable": true,
                    "sortable": true,
                    "facetable": true
                },
                {
                    "name": "StateProvince",
                    "type": "Edm.String",
                    "searchable": true,
                    "filterable": true,
                    "sortable": true,
                    "facetable": true
                },
                {
                    "name": "PostalCode",
                    "type": "Edm.String",
                    "searchable": true,
                    "filterable": true,
                    "sortable": true,
                    "facetable": true
                },
                {
                    "name": "Country",
                    "type": "Edm.String",
                    "searchable": true,
                    "filterable": true,
                    "sortable": true,
                    "facetable": true
                }
            ]
        }
    ],
    "suggesters": [
        {
            "name": "sg",
            "searchMode": "analyzingInfixMatching",
            "sourceFields": [
                "HotelName"
            ]
        }
    ]
}
```

Con la definición del índice implementada, hay que importar **hotels_quickstart_index.json** al principio de **index.js**, con el fin de que la función principal pueda acceder a la definición del índice.

```javascript
const indexDefinition = require('./hotels_quickstart_index.json');
```

Dentro de la función principal, se crea `SearchIndexClient`, que se usa para crear y administrar los índices de Azure Cognitive Search. 

```javascript
const indexClient = new SearchIndexClient(endpoint, new AzureKeyCredential(apiKey));
```

Luego, se puede eliminar el índice si ya existe. Esta es una práctica común para el código de prueba o demostración.

Para ello, se define una función simple que intenta eliminar el índice.

```javascript
async function deleteIndexIfExists(indexClient, indexName) {
    try {
        await indexClient.deleteIndex(indexName);
        console.log('Deleting index...');
    } catch {
        console.log('Index does not exist yet.');
    }
}
```

Para ejecutar la función, se extrae el nombre del índice de la definición del índice y se pasa `indexName`, junto con `indexClient`, a la función `deleteIndexIfExists()`.

```javascript
const indexName = indexDefinition["name"];

console.log('Checking if index exists...');
await deleteIndexIfExists(indexClient, indexName);
```

Después, se puede crear el índice con el método `createIndex()`.

```javascript
console.log('Creating index...');
let index = await indexClient.createIndex(indexDefinition);

console.log(`Index named ${index.name} has been created.`);
```

### <a name="run-the-sample"></a>Ejecución del ejemplo

En este punto, ya está listo para ejecutar el ejemplo. Use una ventana de terminal para ejecutar el siguiente comando:

```cmd
node index.js
```

Si [descargó el código fuente](https://github.com/Azure-Samples/azure-search-javascript-samples/tree/master/quickstart/v11) y aún no ha instalado los paquetes necesarios, ejecute `npm install` primero.

Verá una serie de mensajes en los que se describen las acciones realizadas por el programa. 

Abra la página **Información general** del servicio de búsqueda en Azure Portal. Seleccione la pestaña **Índices**. Debe ver algo parecido a lo siguiente:

:::image type="content" source="media/search-get-started-javascript/create-index-no-data.png" alt-text="Captura de pantalla de Azure Portal, información general del servicio de búsqueda, pestaña Índices" border="false":::

En el paso siguiente, agregará datos al índice. 

## <a name="2---load-documents"></a>2 - Carga de documentos 


En Azure Cognitive Search, los documentos son estructuras de datos que se pueden usar como entradas para la indexación y como salidas de las consultas. Estos datos se pueden insertar en el índice, o bien se puede usar un [indexador](search-indexer-overview.md). En ese caso, los documentos se insertarán mediante programación en el índice.

Las entradas de documentos pueden ser filas de una base de datos, blobs en Blob Storage o, como en este ejemplo, documentos JSON en el disco. Puede descargar [hotels.json](https://github.com/Azure-Samples/azure-search-javascript-samples/blob/master/quickstart/v11/hotels.json), o bien puede crear su propio archivo **hotels.json** con el contenido siguiente:

```json
{
    "value": [
        {
            "HotelId": "1",
            "HotelName": "Secret Point Motel",
            "Description": "The hotel is ideally located on the main commercial artery of the city in the heart of New York. A few minutes away is Time's Square and the historic centre of the city, as well as other places of interest that make New York one of America's most attractive and cosmopolitan cities.",
            "Description_fr": "L'hôtel est idéalement situé sur la principale artère commerciale de la ville en plein cœur de New York. A quelques minutes se trouve la place du temps et le centre historique de la ville, ainsi que d'autres lieux d'intérêt qui font de New York l'une des villes les plus attractives et cosmopolites de l'Amérique.",
            "Category": "Boutique",
            "Tags": ["pool", "air conditioning", "concierge"],
            "ParkingIncluded": false,
            "LastRenovationDate": "1970-01-18T00:00:00Z",
            "Rating": 3.6,
            "Address": {
                "StreetAddress": "677 5th Ave",
                "City": "New York",
                "StateProvince": "NY",
                "PostalCode": "10022"
            }
        },
        {
            "HotelId": "2",
            "HotelName": "Twin Dome Motel",
            "Description": "The hotel is situated in a  nineteenth century plaza, which has been expanded and renovated to the highest architectural standards to create a modern, functional and first-class hotel in which art and unique historical elements coexist with the most modern comforts.",
            "Description_fr": "L'hôtel est situé dans une place du XIXe siècle, qui a été agrandie et rénovée aux plus hautes normes architecturales pour créer un hôtel moderne, fonctionnel et de première classe dans lequel l'art et les éléments historiques uniques coexistent avec le confort le plus moderne.",
            "Category": "Boutique",
            "Tags": ["pool", "free wifi", "concierge"],
            "ParkingIncluded": "false",
            "LastRenovationDate": "1979-02-18T00:00:00Z",
            "Rating": 3.6,
            "Address": {
                "StreetAddress": "140 University Town Center Dr",
                "City": "Sarasota",
                "StateProvince": "FL",
                "PostalCode": "34243"
            }
        },
        {
            "HotelId": "3",
            "HotelName": "Triple Landscape Hotel",
            "Description": "The Hotel stands out for its gastronomic excellence under the management of William Dough, who advises on and oversees all of the Hotel’s restaurant services.",
            "Description_fr": "L'hôtel est situé dans une place du XIXe siècle, qui a été agrandie et rénovée aux plus hautes normes architecturales pour créer un hôtel moderne, fonctionnel et de première classe dans lequel l'art et les éléments historiques uniques coexistent avec le confort le plus moderne.",
            "Category": "Resort and Spa",
            "Tags": ["air conditioning", "bar", "continental breakfast"],
            "ParkingIncluded": "true",
            "LastRenovationDate": "2015-09-20T00:00:00Z",
            "Rating": 4.8,
            "Address": {
                "StreetAddress": "3393 Peachtree Rd",
                "City": "Atlanta",
                "StateProvince": "GA",
                "PostalCode": "30326"
            }
        },
        {
            "HotelId": "4",
            "HotelName": "Sublime Cliff Hotel",
            "Description": "Sublime Cliff Hotel is located in the heart of the historic center of Sublime in an extremely vibrant and lively area within short walking distance to the sites and landmarks of the city and is surrounded by the extraordinary beauty of churches, buildings, shops and monuments. Sublime Cliff is part of a lovingly restored 1800 palace.",
            "Description_fr": "Le sublime Cliff Hotel est situé au coeur du centre historique de sublime dans un quartier extrêmement animé et vivant, à courte distance de marche des sites et monuments de la ville et est entouré par l'extraordinaire beauté des églises, des bâtiments, des commerces et Monuments. Sublime Cliff fait partie d'un Palace 1800 restauré avec amour.",
            "Category": "Boutique",
            "Tags": ["concierge", "view", "24-hour front desk service"],
            "ParkingIncluded": true,
            "LastRenovationDate": "1960-02-06T00:00:00Z",
            "Rating": 4.6,
            "Address": {
                "StreetAddress": "7400 San Pedro Ave",
                "City": "San Antonio",
                "StateProvince": "TX",
                "PostalCode": "78216"
            }
        }
    ]
}
```

De forma parecida a como se hizo con indexDefinition, también es preciso importar `hotels.json` al principio de **index.js**, con el fin de que se pueda acceder a los datos en la función principal.

```javascript
const hotelData = require('./hotels.json');
```


Para indexar los datos en el índice de búsqueda, ahora es preciso crear `SearchClient`. Mientras que `SearchIndexClient` se usa para crear y administrar un índice, `SearchClient` se usa para cargar documentos y realizar consultas en el índice.

Hay dos formas de crear una `SearchClient`. La primera opción consiste en crear `SearchClient` desde cero:

```javascript
 const searchClient = new SearchClient(endpoint, indexName, new AzureKeyCredential(apiKey));
```

Como alternativa, puede utilizar el método `getSearchClient()` de `SearchIndexClient` para crear `SearchClient`:

```javascript
const searchClient = indexClient.getSearchClient(indexName);
```

Ahora que el cliente está definido, cargue los documentos en el índice de búsqueda. En este caso, usamos el método `mergeOrUploadDocuments()`, que cargará los documentos o los combinará con un documento existente si ya existe un documento con la misma clave.

```javascript
console.log('Uploading documents...');
let indexDocumentsResult = await searchClient.mergeOrUploadDocuments(hotelData['value']);

console.log(`Index operations succeeded: ${JSON.stringify(indexDocumentsResult.results[0].succeeded)}`);
```

Vuelva a ejecutar el programa con `node index.js`. Verá un conjunto de mensajes ligeramente distintos de los que ha visto en el paso 1. Esta vez, el índice *sí* existe y debería ver un mensaje relativo a su eliminación antes de que la aplicación cree el nuevo y publique datos en él. 

Antes de ejecutar las consultas en el paso siguiente, defina una función para que el programa espere un segundo. Esta operación se realiza con fines de demostración o prueba, con el fin de garantizar que la indexación finaliza y que los documentos están disponibles en el índice para nuestras consultas.

```javascript
function sleep(ms) {
    var d = new Date();
    var d2 = null;
    do {
        d2 = new Date();
    } while (d2 - d < ms);
}
```

Para que el programa espere un segundo, llame a la función `sleep` como se indica a continuación:

```javascript
sleep(1000);
```

## <a name="3---search-an-index"></a>3 - Búsqueda en un índice

Una vez que haya creado un índice y se hayan cargado los documentos, estará listo para enviarle consultas. En esta sección, enviaremos cinco consultas diferentes al índice de búsqueda para mostrar diferentes partes de la funcionalidad de consulta que tiene a su disposición.

Las consultas se escriben en una función `sendQueries()`, a la que llamaremos en la función principal como se indica a continuación:

```javascript
await sendQueries(searchClient);
```

Para enviar las consultas, se usa el método `search()` de `searchClient`. El primer parámetro es el texto de búsqueda y el segundo parámetro es cualquier opción de búsqueda adicional.

La primera consulta busca `*`, lo que equivale a buscar todo y seleccionar tres de los campos del índice. Es aconsejable usar `select` solo en los campos en que sea necesario, ya que la extracción de datos innecesarios puede agregar latencia a las consultas.

En esta consulte, `searchOptions` también tiene `includeTotalCount` establecido en `true`, lo que devolverá el número de resultados coincidentes que se han encontrado.

```javascript
async function sendQueries(searchClient) {
    console.log('Query #1 - search everything:');
    let searchOptions = {
        includeTotalCount: true,
        select: ["HotelId", "HotelName", "Rating"]
    };

    let searchResults = await searchClient.search("*", searchOptions);
    for await (const result of searchResults.results) {
        console.log(`${JSON.stringify(result.document)}`);
    }
    console.log(`Result count: ${searchResults.count}`);

    // remaining queries go here
}
```

Las restantes consultas que se describen a continuación también se deben agregar a la función `sendQueries()`. Aquí aparecen separadas para mejorar la legibilidad.

En la siguiente consulta, se especifica el término de búsqueda `"wifi"` y también se incluye un filtro para que se devuelvan solo los resultados en los que el estado sea igual a `'FL'`. Los resultados también se ordenan por el valor de `Rating` del hotel.

```javascript
console.log('Query #2 - Search with filter, orderBy, and select:');
let state = 'FL';
searchOptions = {
    filter: odata`Address/StateProvince eq ${state}`,
    orderBy: ["Rating desc"],
    select: ["HotelId", "HotelName", "Rating"]
};

searchResults = await searchClient.search("wifi", searchOptions);
for await (const result of searchResults.results) {
    console.log(`${JSON.stringify(result.document)}`);
}
```

Luego, la búsqueda se limita a un único campo mediante el uso del parámetro `searchFields`. Esta es una excelente opción si desea que la consulta sea más eficaz, pero solo si sabe que está interesado en las coincidencias de determinados campos. 

```javascript
console.log('Query #3 - Limit searchFields:');
searchOptions = {
    select: ["HotelId", "HotelName", "Rating"],
    searchFields: ["HotelName"]
};

searchResults = await searchClient.search("sublime cliff", searchOptions);
for await (const result of searchResults.results) {
    console.log(`${JSON.stringify(result.document)}`);
}
console.log();
```

Otra opción común que se incluye en las consultas es `facets`. Las facetas le permiten crear filtros en la interfaz de usuario, con el fin de que los usuarios sepan fácilmente por qué valores pueden filtrar.

```javascript
console.log('Query #4 - Use facets:');
searchOptions = {
    facets: ["Category"],
    select: ["HotelId", "HotelName", "Rating"],
    searchFields: ["HotelName"]
};

searchResults = await searchClient.search("*", searchOptions);
for await (const result of searchResults.results) {
    console.log(`${JSON.stringify(result.document)}`);
}
```

La consulta final utiliza el método `getDocument()` de `searchClient`. Esto le permite recuperar de forma eficaz un documento por su clave. 

```javascript
console.log('Query #5 - Lookup document:');
let documentResult = await searchClient.getDocument(key='3')
console.log(`HotelId: ${documentResult.HotelId}; HotelName: ${documentResult.HotelName}`)
```

### <a name="run-the-sample"></a>Ejecución del ejemplo

Ejecute el programa con `node index.js`. Ahora, además de los pasos anteriores, se enviarán las consultas y los resultados se escribirán en la consola.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando trabaje con su propia suscripción, es una buena idea al final de un proyecto identificar si todavía se necesitan los recursos que ha creado. Los recursos que se dejan en ejecución pueden costarle mucho dinero. Puede eliminar los recursos de forma individual o eliminar el grupo de recursos para eliminar todo el conjunto de recursos.

Puede encontrar y administrar recursos en el portal, mediante el vínculo **Todos los recursos** o **Grupos de recursos** en el panel de navegación izquierdo.

Si está usando un servicio gratuito, recuerde que está limitado a tres índices, indexadores y orígenes de datos. Puede eliminar elementos individuales en el portal para mantenerse por debajo del límite. 

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido de JavaScript, ha realizado una serie de tareas para crear un índice, cargar documentos en él y ejecutar consultas. Para seguir aprendiendo, pruebe el siguiente tutorial y el ejemplo como trampolín para probar los proveedores de sugerencias (escritura anticipada o autocompletar consultas), los filtros y la navegación por facetas. 

+ [Tutorial: Incorporación de funcionalidad de búsqueda a las aplicaciones web](tutorial-csharp-overview.md)

+ [Ejemplo: Creación de un front-end de React para Azure Cognitive Search](https://github.com/dereklegenzoff/azure-search-react-template)
