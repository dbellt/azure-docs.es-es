---
title: Cómo realizar búsquedas personalizadas en Azure API for FHIR
description: En este artículo se describe cómo puede definir sus propios parámetros de búsqueda personalizados que se usarán en la base de datos.
author: ginalee-dotcom
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: reference
ms.date: 05/03/2021
ms.author: cavoeg
ms.openlocfilehash: 5453b11cb49bb48c48e6c949a00654a797c89202
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110476672"
---
# <a name="defining-custom-search-parameters"></a>Definición de parámetros de búsqueda personalizados

La especificación FHIR define un conjunto de parámetros de búsqueda para todos los recursos y parámetros de búsqueda que son específicos de los recursos. Sin embargo, hay escenarios en los que es posible que quiera buscar en un elemento de un recurso que no está definido por la especificación FHIR como parámetro de búsqueda estándar. En este artículo se describe cómo puede definir sus propios parámetros [de](https://www.hl7.org/fhir/searchparameter.html) búsqueda que se usarán en el Azure API for FHIR.

> [!NOTE]
> Cada vez que cree, actualice o elimine un parámetro de búsqueda, deberá ejecutar un trabajo de [reindexación](how-to-run-a-reindex.md) para permitir que el parámetro de búsqueda se utilice en producción. A continuación se describe cómo puede probar los parámetros de búsqueda antes de volver a indexar todo el servidor de FHIR.

## <a name="create-new-search-parameter"></a>Creación de un nuevo parámetro de búsqueda

Para crear un nuevo parámetro de búsqueda, debe `POST` usar el recurso en la base de `SearchParameter` datos. En el ejemplo de código siguiente se muestra cómo agregar [US Core Race SearchParameter](http://hl7.org/fhir/us/core/STU3.1.1/SearchParameter-us-core-race.html) al `Patient` recurso.

```rest
POST {{FHIR_URL}}/SearchParameter

{
  "resourceType" : "SearchParameter",
  "id" : "us-core-race",
  "url" : "http://hl7.org/fhir/us/core/SearchParameter/us-core-race",
  "version" : "3.1.1",
  "name" : "USCoreRace",
  "status" : "active",
  "date" : "2019-05-21",
  "publisher" : "US Realm Steering Committee",
  "contact" : [
    {
      "telecom" : [
        {
          "system" : "other",
          "value" : "http://www.healthit.gov/"
        }
      ]
    }
  ],
  "description" : "Returns patients with a race extension matching the specified code.",
  "jurisdiction" : [
    {
      "coding" : [
        {
          "system" : "urn:iso:std:iso:3166",
          "code" : "US",
          "display" : "United States of America"
        }
      ]
    }
  ],
  "code" : "race",
  "base" : [
    "Patient"
  ],
  "type" : "token",
  "expression" : "Patient.extension.where(url = 'http://hl7.org/fhir/us/core/StructureDefinition/us-core-race').extension.value.code"
}

``` 

> [!NOTE]
> El nuevo parámetro de búsqueda aparecerá en la instrucción de funcionalidad del servidor de FHIR después de publicar el parámetro de búsqueda en la base de datos y **volver** a indexar la base de datos. Ver en la instrucción de funcionalidad es la única manera de saber si se admite un parámetro de `SearchParameter` búsqueda en el servidor de FHIR. Si puede encontrar el parámetro de búsqueda buscando el parámetro de búsqueda, pero no puede verlo en la instrucción de funcionalidad, debe indexar el parámetro de búsqueda. Puede publicar varios parámetros de búsqueda antes de desencadenar una operación de reindexación.

Elementos importantes de `SearchParameter` :

* **url:** clave única para describir el parámetro de búsqueda. Muchas organizaciones, como HL7, usan un formato de dirección URL estándar para los parámetros de búsqueda que definen, como se muestra anteriormente en el parámetro de búsqueda de carreras de US Core.

* **code**: el valor almacenado en **el código** es lo que se usará al buscar. En el ejemplo anterior, buscaría con para obtener todos `GET {FHIR_URL}/Patient?race=<code>` los pacientes de una carrera específica. El código debe ser único para los recursos a los que se aplica el parámetro de búsqueda.

* **base:** describe a qué recursos se aplica el parámetro de búsqueda. Si el parámetro de búsqueda se aplica a todos los recursos, puede usar ; de lo `Resource` contrario, puede enumerar todos los recursos pertinentes.
 
* **type**: describe el tipo de datos para el parámetro de búsqueda. El tipo está limitado por la compatibilidad con el Azure API for FHIR. Esto significa que no puede definir un parámetro de búsqueda de tipo Especial ni definir un parámetro de búsqueda [compuesto](overview-of-search.md) a menos que sea una combinación admitida.

* **expression**: describe cómo calcular el valor de la búsqueda. Al describir un parámetro de búsqueda, debe incluir la expresión, aunque la especificación no lo requiera. Esto se debe a que necesita la expresión o la sintaxis xpath y el Azure API for FHIR omite la sintaxis xpath.

## <a name="test-search-parameters"></a>Parámetros de búsqueda de prueba

Aunque no puede usar los parámetros de búsqueda en producción hasta que ejecute un trabajo de reindexación, hay varias maneras de probar los parámetros de búsqueda antes de volver a indexar toda la base de datos. 

En primer lugar, puede probar el nuevo parámetro de búsqueda para ver qué valores se devolverán. Al ejecutar el comando siguiente en una instancia de recurso específica (mediante la entrada de su identificador), se obtiene una lista de pares de valores con el nombre del parámetro de búsqueda y el valor almacenado para el paciente específico. Esto incluirá todos los parámetros de búsqueda del recurso y puede desplazarse por para buscar el parámetro de búsqueda que ha creado. La ejecución de este comando no cambiará ningún comportamiento en el servidor de FHIR. 

```rest
GET https://{{FHIR_URL}}/{{RESOURCE}}/{{RESOUCE_ID}}/$reindex

```
Por ejemplo, para buscar todos los parámetros de búsqueda de un paciente:

```rest
GET https://{{FHIR_URL}}/Patient/{{PATIENT_ID}}/$reindex

```

El resultado tendrá este aspecto:

```json
{
  "resourceType": "Parameters",
  "id": "8be24e78-b333-49da-a861-523491c3437a",
  "meta": {
    "versionId": "1"
  },
  "parameter": [
    {
      "name": "deceased",
      "valueString": "http://hl7.org/fhir/special-values|false"
    },
    {
      "name": "language",
      "valueString": "urn:ietf:bcp:47|en-US"
    },
    {
      "name": "race",
      "valueString": "2028-9"
    },
...
```
Una vez que vea que el parámetro de búsqueda se muestra según lo previsto, puede volver a indexar un único recurso para probar la búsqueda con el elemento . En primer lugar, volverá a indexar un único recurso:

```rest
POST https://{{FHIR_URL}/{{RESOURCE}}/{{RESOURCE_ID}}/$reindex
```

Al ejecutar esto, se establecen los índices de los parámetros de búsqueda para el recurso específico que definió para ese tipo de recurso. Esto realiza una actualización en el servidor de FHIR. Ahora puede buscar y establecer el encabezado use partial indices (usar índices parciales) en true, lo que significa que devolverá resultados donde cualquiera de los recursos tenga indexado el parámetro de búsqueda, aunque no todos los recursos de ese tipo lo tengan indexado. 

Siguiendo con el ejemplo anterior, podría indexar un paciente para habilitar la carrera principal de EE. `SearchParameter` UU.:

```rest
POST https://{{FHIR_URL}/Patient/{{PATIENT_ID}}/$reindex
```

Y, a continuación, busque pacientes que tengan una carrera específica:

```rest
GET https://{{FHIR_URL}}/Patient?race=2028-9
x-ms-use-partial-indices: true
```

Una vez que haya probado y esté satisfecho con que el parámetro de búsqueda funciona según lo previsto, ejecute o programe el trabajo de reindexación para que los parámetros de búsqueda se puedan usar en el servidor de FHIR para los casos de uso de producción.

## <a name="update-a-search-parameter"></a>Actualización de un parámetro de búsqueda

Para actualizar un parámetro de búsqueda, use `PUT` para crear una nueva versión del parámetro de búsqueda. Debe incluir en `SearchParameter ID` el elemento del cuerpo de la solicitud y en la `id` `PUT` `PUT` llamada.

> [!NOTE]
> Si no conoce el identificador del parámetro de búsqueda, puede buscarlo. El uso de devolverá todos los parámetros de búsqueda personalizados y puede desplazarse por el parámetro de `GET {{FHIR_URL}}/SearchParameter` búsqueda para encontrar el parámetro de búsqueda que necesita. También puede limitar la búsqueda por nombre. Con el ejemplo siguiente, puede buscar el nombre mediante `USCoreRace: GET {{FHIR_URL}}/SearchParameter?name=USCoreRace` .

```rest
PUT {{FHIR_ULR}}/SearchParameter/{SearchParameter ID}

{
  "resourceType" : "SearchParameter",
  "id" : "SearchParameter ID",
  "url" : "http://hl7.org/fhir/us/core/SearchParameter/us-core-race",
  "version" : "3.1.1",
  "name" : "USCoreRace",
  "status" : "active",
  "date" : "2019-05-21",
  "publisher" : "US Realm Steering Committee",
  "contact" : [
    {
      "telecom" : [
        {
          "system" : "other",
          "value" : "http://www.healthit.gov/"
        }
      ]
    }
  ],
  "description" : "New Description!",
  "jurisdiction" : [
    {
      "coding" : [
        {
          "system" : "urn:iso:std:iso:3166",
          "code" : "US",
          "display" : "United States of America"
        }
      ]
    }
  ],
  "code" : "race",
  "base" : [
    "Patient"
  ],
  "type" : "token",
  "expression" : "Patient.extension.where(url = 'http://hl7.org/fhir/us/core/StructureDefinition/us-core-race').extension.value.code"
}

```

El resultado será actualizado `SearchParameter` y la versión se incrementará.

> [!Warning]
> Tenga cuidado al actualizar SearchParameters que ya se han indexado en la base de datos. Cambiar el comportamiento de un elemento SearchParameter existente podría afectar al comportamiento esperado. Se recomienda ejecutar un trabajo de reindexación inmediatamente.

## <a name="delete-a-search-parameter"></a>Eliminación de un parámetro de búsqueda

Si necesita eliminar un parámetro de búsqueda, use lo siguiente:

```rest
Delete {{FHIR_URL}}/SearchParameter/{SearchParameter ID}
```

> [!Warning]
> Tenga cuidado al eliminar SearchParameters que ya se han indexado en la base de datos. Cambiar el comportamiento de un elemento SearchParameter existente podría afectar al comportamiento esperado. Se recomienda ejecutar un trabajo de reindexación inmediatamente.

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha aprendido a crear un parámetro de búsqueda. A continuación, puede aprender a volver a indexar el servidor de FHIR.

>[!div class="nextstepaction"]
>[Procedimientos para ejecutar un trabajo de reindexación](how-to-run-a-reindex.md)