---
title: Creación de una consulta
titleSuffix: Azure Cognitive Search
description: Obtenga información sobre cómo construir una solicitud de consulta de Cognitive Search, qué herramientas y API usar para realizar pruebas y codificaciones, y cómo las decisiones de consulta comienzan con el diseño de índices.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 02/03/2021
ms.openlocfilehash: 53428a0185b003e22fd0ad68001b2b1588f994b1
ms.sourcegitcommit: 8bca2d622fdce67b07746a2fb5a40c0c644100c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111750756"
---
# <a name="creating-queries-in-azure-cognitive-search"></a>Creación de consultas en Azure Cognitive Search

Si va a compilar una consulta por primera vez, en este artículo se describen los enfoques y métodos para configurar consultas. También presenta una solicitud de consulta y explica cómo los atributos de campo y los analizadores lingüísticos pueden afectar a los resultados de la consulta.

## <a name="whats-a-query-request"></a>¿Qué es una solicitud de consulta?

Una consulta es una solicitud de solo lectura en la colección de documentos de un índice de búsqueda único. Especifica un parámetro "search" que contiene la expresión de consulta, formada por términos, expresiones entre comillas y operadores.

Los parámetros adicionales de la solicitud proporcionan más definición a la consulta y la respuesta. Por ejemplo, "searchFields" limita el ámbito de ejecución de consultas a campos específicos, "select" especifica los campos que se devolverán en los resultados y "count" devuelve el número de coincidencias encontradas en el índice.

En el ejemplo siguiente se ofrece una idea general de una solicitud de consulta al mostrar algunos de los parámetros disponibles. Para obtener más información sobre la composición de consultas, vea [Tipos de consultas y composiciones](search-query-overview.md) y [Documentos de búsqueda (REST)](/rest/api/searchservice/search-documents).

```http
POST https://[service name].search.windows.net/indexes/hotels-sample-index/docs/search?api-version=2020-06-30
{
    "search": "NY +view",
    "queryType": "simple",
    "searchMode": "all",
    "searchFields": "HotelName, Description, Address/City, Address/StateProvince, Tags",
    "select": "HotelName, Description, Address/City, Address/StateProvince, Tags",
    "count": "true"
}
```

## <a name="choose-a-client"></a>Elección de un cliente

Necesitará una herramienta como Azure Portal o Postman, o bien código que cree una instancia de un cliente de consulta mediante las API. Se recomienda Azure Portal o las API REST para realizar las primeras pruebas de desarrollo y de prueba de concepto.

### <a name="permissions"></a>Permisos

Una solicitud de consulta requiere permisos de lectura, concedidos a través de una clave de API que se transmite en el encabezado. Cualquier operación, incluidas las solicitudes de consulta, funcionará con una [clave de API de administración ](search-security-api-keys.md), pero las solicitudes de consulta tienen la opción de usar una [clave de API de consulta](search-security-api-keys.md#create-query-keys). Se recomiendan encarecidamente las claves de API de consulta. Puede crear hasta 50 por servicio y asignar diferentes claves a diferentes aplicaciones.

En Azure Portal, es necesario tener un rol de colaborador o superior en el servicio para obtener acceso a herramientas, asistentes y objetos. 

### <a name="use-azure-portal-to-query-an-index"></a>Uso de Azure Portal para consultar un índice

[El explorador de búsqueda (portal)](search-explorer.md) es una interfaz de consulta de Azure Portal que ejecuta consultas en índices del servicio de búsqueda subyacente. Internamente, el portal realiza solicitudes de [búsqueda de documentos](/rest/api/searchservice/search-documents), pero no puede invocar las funciones Autocomplete, Sugerencias ni Búsqueda de documentos. 

Puede seleccionar cualquier versión de índice y API de REST, incluida la versión preliminar. Una cadena de consulta puede utilizar una sintaxis simple o completa, con compatibilidad para todos los parámetros de consulta (filter, select, searchFields, etc.). Cuando abre un índice en el portal, puede trabajar con el explorador de búsqueda junto con la definición JSON del índice en las pestañas en paralelo, para facilitar el acceso a los atributos de campo. Compruebe los campos que se pueden buscar, ordenar, filtrar y clasificar mientras se prueban las consultas.

### <a name="use-a-rest-client"></a>Uso de un cliente REST

Tanto Postman como Visual Studio Code (con una extensión para Azure Cognitive Search) pueden funcionar como un cliente de consulta. Con cualquiera de las dos herramientas puede conectarse al servicio de búsqueda y enviar solicitudes de [búsqueda de documentos (REST)](/rest/api/searchservice/search-documents). Hay numerosos tutoriales y ejemplos que muestran los clientes REST para la consulta de la indización. 

Para obtener información acerca de los distintos clientes, puede empezar por leer estos artículos (ambos incluyen instrucciones para las consultas):

+ [Creación de un índice de búsqueda mediante REST y Postman](search-get-started-rest.md)
+ [Introducción a Visual Studio Code y Azure Cognitive Search](search-get-started-vs-code.md)

Tenga en cuenta que cada solicitud es independiente, por lo que debe proporcionar el punto de conexión, el nombre del índice y la versión de la API en cada solicitud. Otras propiedades, el tipo de contenido y la clave de API, se pasan en el encabezado de solicitud. Para obtener más información, consulte [Búsqueda de documentos (REST)](/rest/api/searchservice/search-documents) y obtenga ayuda con la formulación de solicitudes de consulta.

### <a name="use-an-sdk"></a>Uso de un SDK

En el caso de Cognitive Search, los SDK de Azure implementan características disponibles con carácter general. Como tal, puede utilizar cualquiera de los SDK para consultar un índice. Todos proporcionan un elemento **SearchClient**, que tiene métodos para interactuar con un índice, desde la carga de un índice con documentos de búsqueda hasta la formulación de solicitudes de consulta.

| SDK de Azure | Cliente | Ejemplos |
|-----------|--------|----------|
| .NET | [SearchClient](/dotnet/api/azure.search.documents.searchclient) | [DotNetHowTo](https://github.com/Azure-Samples/search-dotnet-getting-started/tree/master/DotNetHowTo) |
| Java | [SearchClient](/java/api/com.azure.search.documents.searchclient) | [SearchForDynamicDocumentsExample.java](https://github.com/Azure/azure-sdk-for-java/blob/azure-search-documents_11.1.3/sdk/search/azure-search-documents/src/samples/java/com/azure/search/documents/SearchForDynamicDocumentsExample.java) |
| JavaScript | [SearchClient](/javascript/api/@azure/search-documents/searchclient) | Pendiente. |
| Python | [SearchClient](/python/api/azure-search-documents/azure.search.documents.searchclient) | [sample_simple_query.py ](https://github.com/Azure/azure-sdk-for-python/blob/7cd31ac01fed9c790cec71de438af9c45cb45821/sdk/search/azure-search-documents/samples/sample_simple_query.py) |

## <a name="choose-a-query-type-simple--full"></a>Elegir un tipo de consulta: simple | total

Si la consulta es una búsqueda de texto completo, se usará un analizador de consultas para procesar cualquier texto que se pase como términos y frases de búsqueda. Azure Cognitive Search ofrece dos analizadores de consultas. 

+ El analizador simple entiende la [sintaxis de consulta simple](query-simple-syntax.md). Este analizador se seleccionó como valor predeterminado, debido a su velocidad y eficacia en las consultas de texto de forma libre. La sintaxis admite operadores de búsqueda comunes (AND, OR, NOT) en las búsquedas de términos y frases y en la búsqueda de prefijos (`*`), como "sea*" para Seattle y Seaside. Como recomendación general, pruebe primero el analizador simple y, a continuación, pase al analizador completo si los requisitos de la aplicación llaman a consultas más complejas.

+ La [sintaxis de consulta completa de Lucene](query-Lucene-syntax.md#bkmk_syntax) que se habilita al agregar `queryType=full` a la solicitud, se basa en el [analizador Apache Lucene](https://lucene.apache.org/core/6_6_1/queryparser/org/apache/lucene/queryparser/classic/package-summary.html).

La sintaxis completa y la simple se superponen hasta el punto en que ambas admiten el mismo prefijo y las mismas operaciones booleanas, pero la sintaxis completa proporciona más operadores. En su totalidad, hay más operadores para las expresiones booleanas y más operadores para consultas avanzadas como la búsqueda aproximada, la búsqueda con caracteres comodín, la búsqueda de proximidad y las expresiones regulares.

## <a name="choose-query-methods"></a>Elección de los métodos de consulta

La búsqueda es fundamentalmente un ejercicio controlado por el usuario, donde los términos o frases se recopilan de un cuadro de búsqueda o de los eventos de clic de una página. En la tabla siguiente se resumen los mecanismos mediante los cuales puede recopilar las entradas de usuario y la experiencia de búsqueda esperada.

| Entrada | Experiencia |
|-------|---------|
| [Método de búsqueda](/rest/api/searchservice/search-documents) | El usuario escribe términos o frases en un cuadro de búsqueda, con o sin operadores, y hace clic en Buscar para enviar la solicitud. La búsqueda se puede usar con filtros en la misma solicitud, pero no con la opción de autocompletar o con sugerencias. |
| [Método de autocompletar](/rest/api/searchservice/autocomplete) | El usuario escribe unos pocos caracteres y las consultas se inician después de escribir cada carácter nuevo. La respuesta es una cadena completada del índice. Si la cadena proporcionada es válida, el usuario hace clic en Buscar para enviar la consulta al servicio. |
| [Método de sugerencias](/rest/api/searchservice/suggestions) | Igual que sucede con la opción de autocompletar, el usuario escribe unos pocos caracteres y se generan consultas incrementales. La respuesta es una lista desplegable de documentos coincidentes, que se representa normalmente con algunos campos únicos o descriptivos. Si alguna de las selecciones es válida, el usuario hace clic en una y se devuelve el documento coincidente. |
| [Navegación por facetas](/rest/api/searchservice/search-documents#query-parameters) | Una página muestra vínculos de navegación en los que se puede hacer clic o rutas de navegación que restringen el ámbito de la búsqueda. Una estructura de navegación por facetas se compone de forma dinámica en función de una consulta inicial. Por ejemplo, el valor `search=*` se usa para rellenar un árbol de navegación por facetas que se compone de cada categoría posible. Una estructura de navegación por facetas se crea a partir de una respuesta de consulta, pero también es un mecanismo para expresar la consulta siguiente. En una referencia de la API de REST, el valor `facets` se documenta como un parámetro de consulta de una operación de búsqueda de documentos, pero se puede usar sin el parámetro `search`.|
| [Método de filtro](/rest/api/searchservice/search-documents#query-parameters) | Los filtros se usan con las facetas para restringir los resultados. Asimismo, también puede implementar un filtro en el segundo plano de la página, para inicializar, por ejemplo, la página con campos específicos del idioma. En una referencia de la API de REST, el valor `$filter` se documenta como un parámetro de consulta de una operación de búsqueda de documentos, pero se puede usar sin el parámetro `search`.|

## <a name="know-your-field-attributes"></a>Conozca los atributos de campo

Si ya ha revisado los [tipos de consultas y su composición](search-query-overview.md), puede que recuerde que los parámetros de la solicitud de consulta dependen de cómo se atribuyen los campos en un índice. Por ejemplo, si quiere usar un campo en una consulta, un filtro o un criterio de ordenación, este se debe poder *buscar*, *filtrar* y *ordenar*. Del mismo modo, solo pueden aparecer en los resultados los campos marcados como *recuperables*. Cuando empiece a especificar los parámetros `search`, `filter` y `orderby` en la solicitud, asegúrese de comprobar los atributos a medida que avance, para evitar resultados inesperados.

En la captura de pantalla siguiente del portal que muestra el [índice de ejemplo Hotels](search-get-started-portal.md), solo se pueden usar los dos últimos campos "LastRenovationDate" y "Rating" en una sola cláusula `"$orderby"`.

![Definición del índice para el ejemplo del hotel](./media/search-query-overview/hotel-sample-index-definition.png "Definición del índice para el ejemplo del hotel")

Para obtener una descripción de los atributos de campo, consulte [Creación del índice (API de REST)](/rest/api/searchservice/create-index).

## <a name="know-your-tokens"></a>Conozca los tokens

Durante la indización, el motor de búsqueda usa un analizador para realizar el análisis de texto en las cadenas, maximizando así la posibilidad de encontrar coincidencias en el momento de la consulta. Como mínimo, las cadenas están en minúsculas, pero también pueden someterse a una lematización y detener la eliminación de palabras. Las cadenas o palabras compuestas más amplias suelen dividirse con espacios en blanco, guiones o rayas, y se indexan como tokens independientes. 

Lo importante aquí es que lo que cree que contiene su índice, y lo que realmente contiene, puede ser diferente. Si las consultas no devuelven los resultados esperados, puede inspeccionar los tokens que haya creado el analizador a través de la opción [Analizar texto (API de REST)](/rest/api/searchservice/test-analyzer). Para obtener más información acerca de la tokenización y el impacto en las consultas, consulte [Búsqueda de términos y patrones parciales con caracteres especiales](search-query-partial-matching.md).

## <a name="next-steps"></a>Pasos siguientes

Ahora que conoce mejor cómo funcionan las solicitudes de consulta, pruebe las siguientes guías de inicio rápido para obtener experiencia práctica.

+ [Explorador de búsqueda](search-explorer.md)
+ [Consultas con REST](search-get-started-rest.md)
+ [Consultas con .NET](search-get-started-dotnet.md)
+ [Consultas con Python](search-get-started-python.md)
+ [Consultas con JavaScript](search-get-started-javascript.md)