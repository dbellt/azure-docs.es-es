---
title: Introducción a la búsqueda en Azure API for FHIR
description: En este artículo se describe una introducción a la búsqueda de FHIR que se implementa en Azure API for FHIR
author: stevewohl
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: reference
ms.date: 5/17/2021
ms.author: cavoeg
ms.openlocfilehash: 93468d2be1c9ab8fbdbce1e82a58f04f0676f260
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110071189"
---
# <a name="overview-of-fhir-search"></a>Introducción a la búsqueda de FHIR

La especificación de FHIR define los aspectos básicos de la búsqueda de recursos de FHIR. Este artículo le guiará a través de algunos aspectos clave para buscar recursos en FHIR. Para obtener detalles completos sobre cómo buscar recursos de FHIR, consulte [Búsqueda en](https://www.hl7.org/fhir/search.html) la especificación de HL7 FHIR. A lo largo de este artículo, se proporcionan ejemplos de sintaxis de búsqueda. Cada búsqueda se realizará en el servidor de FHIR, que normalmente tiene una dirección URL de `https://<FHIRSERVERNAME>.azurewebsites.net` . En los ejemplos, usaremos el marcador de posición {{FHIR_URL}} para esta dirección URL. 

Las búsquedas de FHIR pueden estar en un tipo de recurso específico, en un [compartimiento](https://www.hl7.org/fhir/compartmentdefinition.html)especificado o en todos los recursos. La manera más sencilla de ejecutar una búsqueda en FHIR es usar una `GET` solicitud. Por ejemplo, si desea extraer todos los pacientes de la base de datos, puede usar la siguiente solicitud: 

```rest
GET {{FHIR_URL}}/Patient
```

También puede buscar mediante `POST` , lo que resulta útil si la cadena de consulta es demasiado larga. Para realizar búsquedas `POST` mediante , los parámetros de búsqueda se pueden enviar como cuerpo del formulario. Esto permite una serie más larga y compleja de parámetros de consulta que pueden ser difíciles de ver y comprender en una cadena de consulta.

Si la solicitud de búsqueda se realiza correctamente, recibirá una respuesta de agrupación de FHIR con el tipo `searchset` . Si se produce un error en la búsqueda, encontrará los detalles del error en para ayudarle a comprender por qué se produjo un error `OperationOutcome` en la búsqueda.

En las secciones siguientes, se tratarán los distintos aspectos implicados en la búsqueda. Una vez que haya revisado estos [](search-samples.md) detalles, consulte nuestra página de ejemplos que contiene ejemplos de búsquedas que puede realizar en el Azure API for FHIR.

## <a name="search-parameters"></a>Parámetros de búsqueda

Cuando realice una búsqueda, buscará en función de varios atributos del recurso. Estos atributos se denominan parámetros de búsqueda. Cada recurso tiene un conjunto de parámetros de búsqueda definidos. El parámetro de búsqueda se debe definir e indexar en la base de datos para que pueda buscarlo correctamente.

Cada parámetro de búsqueda tiene un tipo de [datos definido.](https://www.hl7.org/fhir/search.html#ptypes) A continuación se describe la compatibilidad con los distintos tipos de datos:


| **Tipo de parámetro de búsqueda**  | **Admitida: PaaS** | **Admitida: OSS (SQL)** | **Admitida: OSS (Cosmos DB)** | **Comentario**|
| -------------------------  | -------------------- | ------------------------- | ------------------------------- |------------|
|  number                    | Sí                  | Sí                       | Sí                             |
|  date                      | Sí                  | Sí                       | Sí                             |
|  string                    | Sí                  | Sí                       | Sí                             |
|  token                     | Sí                  | Sí                       | Sí                             |
|  reference                 | Sí                  | Sí                       | Sí                             |
|  compuestos                 | Parcial              | Parcial                   | Parcial                         | La lista de tipos compuestos admitidos se describe más adelante en este artículo. |
|  quantity                  | Sí                  | Sí                       | Sí                             |
|  uri                       | Sí                  | Sí                       | Sí                             |
|  especial                   | No                   | No                        | No                              |

### <a name="common-search-parameters"></a>Parámetros de búsqueda comunes

Hay parámetros [de búsqueda comunes que](https://www.hl7.org/fhir/search.html#all) se aplican a todos los recursos. Estos se enumeran a continuación, junto con su compatibilidad dentro del Azure API for FHIR:

| **Parámetro de búsqueda común** | **Admitida: PaaS** | **Admitida: OSS (SQL)** | **Admitida: OSS (Cosmos DB)** | **Comentario** |
| --------------------------  | -------------------- | ------------------------- | ------------------------------- | ------------|
| _id                         | Sí                  | Sí                       | Sí                             |             |
| _lastUpdated                | Sí                  | Sí                       | Sí                             |             |
| _tag                        | Sí                  | Sí                       | Sí                             |             | 
| _type                       | Sí                  | Sí                       | Sí                             |             |
| _security                   | Sí                  | Sí                       | Sí                             |             |
| _profile                    | Sí                  | Sí                       | Sí                             | Si creó la base de datos R4 antes del 20 de febrero de 2021, deberá ejecutar un trabajo de [reindexación](how-to-run-a-reindex.md) para habilitar **_profile**.|
| _has                        | Parcial              | Sí                       | Parcial                         | La compatibilidad _has MVP en el Azure API for FHIR y la versión del sistema operativo con el respaldo de Cosmos DB. En la sección de encadenamiento siguiente se incluyen más detalles. |
| _query                      | No                   | No                        | No                              |             |
| _filter                     | No                   | No                        | No                              |             |
| _list                       | No                   | No                        | No                              |             |
| _text                       | No                   | No                        | No                              |             |
| _content                    | No                   | No                        | No                              |             |

### <a name="resource-specific-parameters"></a>Parámetros específicos del recurso

Con la Azure API for FHIR, se admiten casi todos [los](https://www.hl7.org/fhir/searchparameter-registry.html) parámetros de búsqueda específicos del recurso definidos por la especificación de FHIR. Los únicos parámetros de búsqueda que no se admiten están disponibles en los vínculos siguientes:

* [Parámetros de búsqueda no admitidos de STU3](https://github.com/microsoft/fhir-server/blob/main/src/Microsoft.Health.Fhir.Core/Data/Stu3/unsupported-search-parameters.json)

* [Parámetros de búsqueda no admitidos de R4](https://github.com/microsoft/fhir-server/blob/main/src/Microsoft.Health.Fhir.Core/Data/R4/unsupported-search-parameters.json)

También puede ver la compatibilidad actual con los parámetros de búsqueda en la instrucción de funcionalidad de [FHIR](https://www.hl7.org/fhir/capabilitystatement.html) con la siguiente solicitud:

```rest
GET {{FHIR_URL}}/metadata
```

Para ver los parámetros de búsqueda en la instrucción de funcionalidad, vaya a para ver los parámetros de búsqueda de cada recurso y para buscar los parámetros de `CapabilityStatement.rest.resource.searchParam` búsqueda de todos los `CapabilityStatement.rest.searchParam` recursos.

> [!NOTE]
> El Azure API for FHIR no crea ni indexa automáticamente los parámetros de búsqueda que no están definidos por la especificación de FHIR. Sin embargo, se proporciona compatibilidad para definir sus propios parámetros [de búsqueda.](how-to-do-custom-search.md)

### <a name="composite-search-parameters"></a>Parámetros de búsqueda compuestos
La búsqueda compuesta permite buscar en pares de valores. Por ejemplo, si estaba buscando una observación de altura en la que la persona tenía 60 pulgadas, le  gustaría asegurarse de que un solo componente de la observación contenía el código de alto y el valor de 60. No le gustaría obtener una observación en la que se almacenara un peso de 60 y una altura de 48, aunque la observación tendría entradas que calificaran para el valor de 60 y el código de alto, solo en secciones de componentes diferentes. 

Con la Azure API for FHIR, se admiten los siguientes emparejamientos de tipo de parámetro de búsqueda:

* Referencia, token
* Token, Fecha
* Token, Number, Number
* Token, Cantidad
* Token, String
* Token, Token

Para obtener más información, vea Los parámetros de búsqueda compuesta [hl7](https://www.hl7.org/fhir/search.html#composite). 

> [!NOTE]
> Los parámetros de búsqueda compuestos no admiten modificadores según la especificación de FHIR.

 ### <a name="modifiers--prefixes"></a>Modificadores & prefijos

[Los modificadores](https://www.hl7.org/fhir/search.html#modifiers) permiten modificar el parámetro de búsqueda. A continuación se muestra información general de todos los modificadores de FHIR y la compatibilidad de Azure API for FHIR. 

| **Modificadores** | **Admitida: PaaS** | **Admitida: OSS (SQL)** | **Admitida: OSS (Cosmos DB)** |
| ------------- | -------------------- | ------------------------- | ------------------------------- |
|  :missing     | Sí                  | Sí                       | Sí                             |
|  :exact       | Sí                  | Sí                       | Sí                             | 
|  :contains    | Sí                  | Sí                       | Sí                             | 
|  :text        | Sí                  | Sí                       | Sí                             | 
|  :type (referencia) | Sí             | Sí                       | Sí                             | 
|  :not         | Sí                  | Sí                       | Sí                             | 
|  :below (uri) | Sí                  | Sí                       | Sí                             |  
|  :above (uri) | Sí                  | Sí                       | Sí                             | 
|  :in (token)  | No                   | No                        | No                              | 
|  :below (token) | No                 | No                        | No                              | 
|  :above (token) | No                 | No                        | No                              | 
|  :not-in (token) | No                | No                        | No                              | 

Para los parámetros de búsqueda que tienen un orden específico (números, fechas y cantidades), puede usar un prefijo en el parámetro para ayudar [a](https://www.hl7.org/fhir/search.html#prefix) buscar coincidencias. El Azure API for FHIR admite todos los prefijos.

 ### <a name="search-result-parameters"></a>Parámetros de resultados de la búsqueda
Para ayudar a administrar los recursos devueltos, hay parámetros de resultados de búsqueda que puede usar en la búsqueda. Para más información sobre cómo usar cada uno de los parámetros de resultados de búsqueda, consulte el sitio [web de HL7.](https://www.hl7.org/fhir/search.html#return) 

| **Parámetros de resultados de la búsqueda**  | **Admitida: PaaS** | **Admitida: OSS (SQL)** | **Admitida: OSS (Cosmos DB)** | **Comentarios**                 |
| ----------------------------  | -------------------- | ------------------------- | ------------------------------- | -----------------------------|
| _elements                     | Sí                  | Sí                       | Sí                             |                                |
| _count                        | Sí                  | Sí                       | Sí                             | _count está limitado a 1000 recursos. Si se establece en un valor superior a 1000, solo se devolverá 1000 y se devolverá una advertencia en la agrupación.                               |
| _include                      | Sí                  | Sí                       | Sí                             | Los elementos incluidos se limitan a 100. _include en PaaS y OSS en Cosmos DB no incluyen compatibilidad con :iterate [(#1313).](https://github.com/microsoft/fhir-server/issues/1313)                               |
| _revinclude                   | Sí                  | Sí                       | Sí                             |  Los elementos incluidos se limitan a 100. _revinclude en PaaS y OSS en Cosmos DB no incluye compatibilidad con :iterate [(#1313).](https://github.com/microsoft/fhir-server/issues/1313)  Problema [n.º 1319](https://github.com/microsoft/fhir-server/issues/1319)                            |
| _summary                      | Sí             | Sí                   | Sí                        |                               |
| _total                        | Parcial              | Parcial                   | Parcial                         | _total=none y _total=accurate                               |
| _sort                         | Parcial              | Parcial                   | Parcial                         | se admite sort=_lastUpdated. Para Azure API for FHIR y OSS Cosmos DB bases de datos creadas después del 20 de abril de 2021, la ordenación también se admite en el nombre, el apellido y la fecha de la historia.                               |
| _contained                    | No                   | No                        | No                              |                                |
| _containedType                | No                   | No                        | No                              |                                |
| _score                        | No                   | No                        | No                              |                                |

De forma predeterminada, el Azure API for FHIR se establece en control lenient. Esto significa que el servidor omitirá los parámetros desconocidos o no admitidos. Si desea usar un control estricto, puede usar el encabezado **Prefer** y establecer `handling=strict` .

 ## <a name="chained--reverse-chained-searching"></a>Búsqueda encadenada & búsqueda inversa encadenada

Una [búsqueda encadenada](https://www.hl7.org/fhir/search.html#chaining) permite buscar mediante un parámetro de búsqueda en un recurso al que hace referencia otro recurso. Por ejemplo, si desea encontrar encuentras donde el nombre del paciente es Jane, use:

`GET {{FHIR_URL}}/Encounter?subject:Patient.name=Jane`

De forma similar, puede realizar una búsqueda encadenada inversa. Esto le permite obtener recursos donde especifique criterios en otros recursos que hacen referencia a ellos. Para obtener más ejemplos de búsqueda encadenada e inversa, consulte la página de ejemplos de [búsqueda de FHIR.](search-samples.md) 

> [!NOTE]
> En el Azure API for FHIR y el código abierto con el respaldo de Cosmos DB, hay una limitación por la que cada subconsulta necesaria para las búsquedas encadenadas y encadenadas inversas solo devolverá 100 elementos. Si se han encontrado más de 100 elementos, recibirá el siguiente mensaje de error: "Las subconsecciones de una expresión encadenada no pueden devolver más de 100 resultados, use un criterio más selectivo". Para obtener una consulta correcta, debe ser más específico en lo que busca.

## <a name="pagination"></a>Paginación

Como se mencionó anteriormente, los resultados de una búsqueda serán un conjunto paginado. De forma predeterminada, la búsqueda devolverá 10 resultados por página, pero esto se puede aumentar (o disminuir) especificando `_count` . Dentro de la agrupación, habrá un vínculo propio que contiene el resultado actual de la búsqueda. Si hay coincidencias adicionales, la agrupación contendrá un vínculo siguiente. Puede seguir usando el vínculo siguiente para obtener las páginas de resultados posteriores. `_count` está limitado a 1000 elementos o menos. 

Actualmente, el Azure API for FHIR solo admite el vínculo siguiente en las agrupaciones y no admite los vínculos primero, último o anterior.

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha aprendido los conceptos básicos de la búsqueda, consulte la página de ejemplos de búsqueda para obtener más información sobre cómo realizar búsquedas con diferentes parámetros de búsqueda, modificadores y otros escenarios de búsqueda de FHIR.

>[!div class="nextstepaction"]
>[Ejemplos de búsqueda de FHIR](search-samples.md)
