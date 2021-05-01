---
title: Cómo realizar búsquedas personalizadas en Azure API for FHIR
description: En este artículo se describe cómo puede definir sus propios parámetros de búsqueda personalizados que se usarán en la base de datos.
author: stevewohl
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: reference
ms.date: 4/23/2021
ms.author: cavoeg
ms.openlocfilehash: 1482bd7f054c75c4f26b77907d50a6471c389d68
ms.sourcegitcommit: 52491b361b1cd51c4785c91e6f4acb2f3c76f0d5
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108322569"
---
# <a name="defining-custom-search-parameters"></a>Definición de parámetros de búsqueda personalizados

La especificación de FHIR define un conjunto de parámetros de búsqueda para todos los recursos y parámetros de búsqueda que son específicos de los recursos. Sin embargo, hay escenarios en los que es posible que desee buscar en un campo de un recurso que no está definido por la especificación como un parámetro de búsqueda estándar. En este artículo se describe cómo puede definir sus propios parámetros [de](https://www.hl7.org/fhir/searchparameter.html) búsqueda que se usarán en el servidor de FHIR.

> [!NOTE]
> Cada vez que cree, actualice o elimine un parámetro de búsqueda, deberá ejecutar un trabajo [de reindexación](how-to-run-a-reindex.md) para habilitar el cambio en la base de datos.

> [!Warning]
> Si actualiza o elimina un parámetro de búsqueda, asegúrese de ejecutar inmediatamente un [trabajo de reindexación](how-to-run-a-reindex.md). Existe la posibilidad de que la base de datos esté en un estado anómalo con parámetros de búsqueda actualizados o de eliminación que siguen apareciendo activos debido a la necesidad de indexar los cambios. 

## <a name="create-new-search-parameter"></a>Creación de un nuevo parámetro de búsqueda

Para crear un nuevo parámetro de búsqueda, debe `POST` crear un nuevo parámetro de búsqueda en la base de datos. En el ejemplo de código siguiente se muestra cómo agregar el parámetro de búsqueda [US Core Race](http://hl7.org/fhir/us/core/STU3.1.1/SearchParameter-us-core-race.html) al recurso Patient.

```json
POST {fhirurl}/SearchParameter
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
> El nuevo parámetro de búsqueda aparecerá en la instrucción de funcionalidad después de publicarlo en el servidor de FHIR y volver a indexar la base de datos. No se podrá usar hasta que ejecute un trabajo [de reindexación](how-to-run-a-reindex.md). Esta es la única manera en este momento de saber si se admite un parámetro de búsqueda o no en la base de datos. Si puede encontrar el parámetro de búsqueda buscando el parámetro de búsqueda pero no puede verlo en la instrucción de funcionalidad, ejecute un trabajo de reindexación. Puede publicar varios parámetros de búsqueda antes de desencadenar una operación de reindexación.

Descripciones de campos importantes:

* **url:** se trata de una clave única para describir el parámetro de búsqueda. Muchas organizaciones, como HL7, usan un formato estándar para las direcciones URL que definen, como se muestra anteriormente en el parámetro de búsqueda de carreras de US Core.

* **code**: el valor aquí es lo que se usará al buscar. En el ejemplo anterior, buscaría con para `GET {FHIR URL}/Patient?race=2028-9` obtener todos los pacientes asiáticos. Este valor debe ser único para los recursos a los que se aplica.

* **base:** describe a qué recursos se aplica el parámetro de búsqueda. Si se aplica a todos los recursos, solo puede usar Recurso. De lo contrario, puede enumerar todos los recursos pertinentes.
 
* **type**: describe el tipo de datos para el parámetro de búsqueda.

* **expression**: al describir un parámetro de búsqueda, debe incluir la expresión, aunque técnicamente no sea necesaria para la especificación. Esto se debe a que necesita la sintaxis de expresión o xpath y Azure API for FHIR la sintaxis xpath en este momento. Aquí se describe cómo buscar el valor de la búsqueda. 

> [!NOTE]
> "Tipo" está limitado por la compatibilidad con el Azure API for FHIR. Esto significa que no se puede definir un parámetro de búsqueda de tipo Especial ni definir un parámetro de búsqueda compuesto [a](overview-of-search.md) menos que sea de un tipo que se admita.

Una vez que haya agregado los parámetros de búsqueda, ejecute o programe el trabajo de reindexación para que los parámetros de búsqueda se puedan usar en el servidor de FHIR.

## <a name="update-a-search-parameter"></a>Actualización de un parámetro de búsqueda

Para actualizar un parámetro de búsqueda, use `PUT` para crear una nueva versión del parámetro de búsqueda.

`PUT {fhirurl}/SearchParameter/{SearchParameter ID}`

Debe incluir en `SearchParameter ID` el campo ID del cuerpo de la solicitud y en la `PUT` `PUT` llamada.

> [!NOTE]
> Si no conoce el identificador del parámetro de búsqueda, puede buscarlo. El uso de devolverá todos los parámetros de búsqueda personalizados y puede desplazarse por el parámetro de `GET {fhirurl}/SearchParameter` búsqueda para encontrar el parámetro de búsqueda que necesita. También puede limitar la búsqueda por nombre. Con el ejemplo siguiente, puede buscar el nombre mediante `USCoreRace: GET {fhirurl}/SearchParameter?name=USCoreRace` .

```json
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
> Tenga cuidado al actualizar SearchParameters que ya se han indexado en la base de datos. Cambiar el comportamiento de searchParameter existente podría afectar al comportamiento esperado. 

## <a name="delete-a-search-parameter"></a>Eliminación de un parámetro de búsqueda

Si necesita eliminar un parámetro de búsqueda, use lo siguiente:

`Delete {fhirurl}/SearchParameter/{SearchParameter ID}`

> [!Warning]
> Tenga cuidado al eliminar SearchParameters que ya se han indexado en la base de datos. Cambiar el comportamiento de searchParameter existente podría afectar al comportamiento esperado.

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha aprendido a crear un parámetro de búsqueda. Para obtener información sobre cómo volver a indexar un trabajo, consulte

>[!div class="nextstepaction"]
>[Ejecución de un trabajo de reindexación](how-to-run-a-reindex.md)