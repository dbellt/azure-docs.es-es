---
title: Ejecución de un trabajo de reindexación en Azure API for FHIR
description: En este artículo se describe cómo ejecutar un trabajo de reindexación para indexar cualquier parámetro de búsqueda u ordenación que aún no se haya indexado en la base de datos.
author: stevewohl
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: reference
ms.date: 4/23/2021
ms.author: cavoeg
ms.openlocfilehash: 0332582f65ea59f43cc55064f9cdacefe4beefe4
ms.sourcegitcommit: 52491b361b1cd51c4785c91e6f4acb2f3c76f0d5
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108322568"
---
# <a name="running-a-reindex-job"></a>Ejecución de un trabajo de reindexación

Hay escenarios en los que puede tener parámetros de búsqueda u ordenación en Azure API for FHIR que aún no se han indexado. Esto es especialmente relevante cuando se definen sus propios parámetros de búsqueda. Hasta que no se indexe el parámetro de búsqueda, no se puede usar en la búsqueda. En este artículo se describe información general sobre cómo ejecutar un trabajo de reindexación para indexar cualquier parámetro de búsqueda u ordenación que aún no se haya indexado en la base de datos.

> [!Warning]
> Es importante que lea todo este artículo antes de empezar. Un trabajo de reindexación puede ser muy intensivo en el rendimiento. En este artículo se incluyen opciones para limitar y controlar el trabajo de reindexación.

## <a name="how-to-run-a-reindex-job"></a>Ejecución de un trabajo de reindexación 

Para iniciar un trabajo de reindexación, use el ejemplo de código siguiente:

```json
POST {{FHIR URL}}/$reindex 

{ 

“resourceType”: “Parameters”,  

“parameter”: [] 

}
 ```

Si la solicitud se realiza correctamente, se devuelve el estado **201 Creado.** El resultado de este mensaje tendrá el siguiente aspecto:

```json
HTTP/1.1 201 Created 
Content-Location: https://cv-cosmos1.azurewebsites.net/_operations/reindex/560c7c61-2c70-4c54-b86d-c53a9d29495e 

{
  "resourceType": "Parameters",
  "id": "560c7c61-2c70-4c54-b86d-c53a9d29495e",
  "meta": {
    "versionId": "\"4c0049cd-0000-0100-0000-607dc5a90000\""
  },
  "parameter": [
    {
      "name": "id",
      "valueString": "560c7c61-2c70-4c54-b86d-c53a9d29495e"
    },
    {
      "name": "queuedTime",
      "valueDateTime": "2021-04-19T18:02:17.0118558+00:00"
    },
    {
      "name": "totalResourcesToReindex",
      "valueDecimal": 0.0
    },
    {
      "name": "resourcesSuccessfullyReindexed",
      "valueDecimal": 0.0
    },
    {
      "name": "progress",
      "valueDecimal": 0.0
    },
    {
      "name": "status",
      "valueString": "Queued"
    },
    {
      "name": "maximumConcurrency",
      "valueDecimal": 1.0
    },
    {
      "name": "resources",
      "valueString": ""
    },
    {
      "name": "searchParams",
      "valueString": ""
    }
  ]
}
```

> [!NOTE]
> Para comprobar el estado de o para cancelar un trabajo de reindexación, necesitará el identificador de reindexación. Este es el identificador del recurso Parameters resultante (mostrado anteriormente) y también se puede encontrar como GUID al final de la cadena Content-Location:

`https://{{FHIR URL}}/_operations/reindex/560c7c61-2c70-4c54-b86d-c53a9d29495e`

 ## <a name="how-to-check-the-status-of-a-reindex-job"></a>Comprobación del estado de un trabajo de reindexación

Una vez que haya iniciado un trabajo de reindexación, puede comprobar el estado del trabajo mediante lo siguiente:

`GET {{FHIR URL}}/_operations/reindex/{{reindexJobId}`

A continuación se muestra el estado del resultado del trabajo de reindexación:

```json
{

  "resourceType": "Parameters",
  "id": "b65fd841-1c62-47c6-898f-c9016ced8f77",
  "meta": {

    "versionId": "\"1800f05f-0000-0100-0000-607a1a7c0000\""
  },
  "parameter": [

    {

      "name": "id",
      "valueString": "b65fd841-1c62-47c6-898f-c9016ced8f77"
    },
    {

      "name": "startTime",
      "valueDateTime": "2021-04-16T23:11:35.4223217+00:00"
    },
    {

      "name": "queuedTime",
      "valueDateTime": "2021-04-16T23:11:29.0288163+00:00"
    },
    {

      "name": "totalResourcesToReindex",
      "valueDecimal": 262544.0
    },
    {

      "name": "resourcesSuccessfullyReindexed",
      "valueDecimal": 5754.0
    },
    {

      "name": "progress",
      "valueDecimal": 2.0
    },
    {

      "name": "status",
      "valueString": "Running"
    },
    {

      "name": "maximumConcurrency",
      "valueDecimal": 1.0
    },
    {

      "name": "resources",
      "valueString": 
      "{LIST OF IMPACTED RESOURCES}"
    },
    {
```

La siguiente información se muestra en el resultado del trabajo de reindexación:

* **totalResourcesToReindex:** incluye el número total de recursos que se van a volver a indexar como parte del trabajo.

* **resourcesSuccessfullyReindexed:** el total que ya se ha reindexado correctamente.

* **progress:** se ha completado el porcentaje de trabajos de reindexación. Es igual a resourcesSuccessfullyReindexed/totalResourcesToReindex x 100.

* **status:** se mostrará si el trabajo de reindexación se pone en cola, se ejecuta, se completa, se produce un error o se cancela.

* **resources:** muestra todos los tipos de recursos afectados por el trabajo de reindexación.

## <a name="delete-a-reindex-job"></a>Eliminación de un trabajo de reindexación

Si necesita cancelar un trabajo de reindexación, use una llamada de eliminación y especifique el identificador del trabajo de reindexación:

`Delete {{FHIR URL}}/_operations/reindex/{{reindexJobId}`

## <a name="performance-considerations"></a>Consideraciones de rendimiento

Un trabajo de reindexación puede ser muy intensivo en el rendimiento. Hemos implementado algunos controles de limitación para ayudarle a administrar cómo se ejecutará un trabajo de reindexación en la base de datos.

> [!NOTE]
> No es raro en grandes conjuntos de datos que un trabajo de reindexación se ejecute durante días. En el caso de una base de datos con 30 000 000 millones de recursos, hemos observado que se tardaron entre 4 y 5 días en volver a indexar toda la base de datos en 100 000 RU.

A continuación se muestra una tabla en la que se delinea los parámetros disponibles, los valores predeterminados y los intervalos recomendados. Puede usar estos parámetros para acelerar el proceso (usar más proceso) o ralentizar el proceso (usar menos proceso). Por ejemplo, podría ejecutar el trabajo de reindexación en un tiempo de tráfico bajo y aumentar el proceso para que se haga más rápido. En su lugar, podría usar la configuración para garantizar un uso muy bajo del proceso y hacer que se ejecute durante días en segundo plano. 

| **Parámetro**                     | **Descripción**              | **Valor predeterminado**        | **Intervalo recomendado**           |
| --------------------------------- | ---------------------------- | ------------------ | ------------------------------- |
| QueryDelayIntervalInMilliseconds  | Este es el retraso entre cada lote de recursos que se va a iniciar durante el trabajo de reindexación. | 500 MS (.5 segundos) | De 50 a 5000: 50 acelerará el trabajo de reindexación y 5000 lo ralentizará desde el valor predeterminado. |
| MaximumResourcesPerQuery  | Este es el número máximo de recursos incluidos en el lote de recursos que se van a volver a indexar.  | 100 | 1-500 |
| MaximumConcurreny  | Este es el número de lotes que se realizan a la vez.  | 1 | 1-5 |
| targetDataStoreUsagePercentrage | Esto le permite especificar qué porcentaje del almacén de datos se va a usar para el trabajo de reindexación. Por ejemplo, podría especificar el 50 % y eso garantizaría que, como máximo, el trabajo de reindexación usaría el 50 % de las RU disponibles en Cosmos DB.  | No existe, lo que significa que se puede usar hasta el 100 %. | 1-100 |

Si desea usar cualquiera de los parámetros anteriores, puede pasarlos al recurso Parámetros al iniciar el trabajo de reindexación.

```json
{
  "resourceType": "Parameters",
  "parameter": [
    {
      "name": "maximumConcurrency",
      "valueInteger": "3"
    },
    {
      "name": "targetDataStoreUsagePercentage",
      "valueInteger": "20"
    },
    {
      "name": "queryDelayIntervalInMilliseconds",
      "valueInteger": "1000"
    },
    {
      "name": "maximumNumberOfResourcesPerQuery",
      "valueInteger": "1"
    }
  ]
}
```

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha aprendido a iniciar un trabajo de reindexación. Para obtener información sobre cómo definir nuevos parámetros de búsqueda que requieren el trabajo de reindexación, consulte 

>[!div class="nextstepaction"]
>[Definición de parámetros de búsqueda personalizados](how-to-do-custom-search.md)

         
     