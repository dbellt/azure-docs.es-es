---
title: Ejemplos de búsqueda de Azure API for FHIR
description: Búsqueda mediante diferentes parámetros de búsqueda, modificadores y otras herramientas de búsqueda de FHIR
author: ginalee-dotcom
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: reference
ms.date: 05/21/2021
ms.author: cavoeg
ms.openlocfilehash: 5be1be72e47af10868867e0dce8b747911509381
ms.sourcegitcommit: a434cfeee5f4ed01d6df897d01e569e213ad1e6f
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111810813"
---
# <a name="fhir-search-examples"></a>Ejemplos de búsqueda de FHIR

A continuación se muestran algunos ejemplos de uso de las operaciones de búsqueda de FHIR, incluidos los parámetros y modificadores de búsqueda, la búsqueda en cadena y la cadena inversa, la búsqueda compuesta, la visualización del siguiente conjunto de entradas para los resultados de la búsqueda y la búsqueda con una `POST` solicitud. Para obtener más información sobre la búsqueda, vea [Información general de FHIR Search.](overview-of-search.md)
   
## <a name="search-result-parameters"></a>Parámetros de resultados de la búsqueda

### <a name="_include"></a>_include

`_include` busca en los recursos los que incluyen el parámetro especificado del recurso. Por ejemplo, puede buscar entre recursos para buscar solo los que incluyen información sobre las recetas de un paciente `MedicationRequest` específico, que es el `reference` parámetro `patient` . En el ejemplo siguiente, se extraerán todos los pacientes y a los que se `MedicationRequests` hace referencia desde `MedicationRequests` :

```rest
 GET [your-fhir-server]/MedicationRequest?_include=MedicationRequest:patient

```

> [!NOTE]
> **_include** y **_revinclude** están limitados a 100 elementos.

### <a name="_revinclude"></a>_revinclude

`_revinclude` permite buscar en la dirección opuesta como `_include` . Por ejemplo, puede buscar pacientes y, a continuación, invertir incluir todos los encuentros que hacen referencia a los pacientes:

```rest
GET [your-fhir-server]/Patient?_revinclude=Encounter:subject

```
### <a name="_elements"></a>_elements

`_elements` limita el resultado de la búsqueda a un subconjunto de campos para reducir el tamaño de la respuesta omitiendo los datos innecesarios. El parámetro acepta una lista separada por comas de elementos base:

```rest
GET [your-fhir-server]/Patient?_elements=identifier,active

```

En esta solicitud, recibirá un conjunto de pacientes, pero cada recurso solo incluirá los identificadores y el estado activo del paciente. Los recursos de esta respuesta devuelta contendrán un valor de para indicar que se trata de `meta.tag` un conjunto incompleto de `SUBSETTED` resultados.

## <a name="search-modifiers"></a>Modificadores de búsqueda

### <a name="not"></a>:not

`:not` permite buscar recursos en los que un atributo no es true. Por ejemplo, podría buscar pacientes en los que el sexo no sea mujer:

```rest
GET [your-fhir-server]/Patient?gender:not=female

```

Como valor devuelto, se obtienen todas las entradas de pacientes en las que el género no es mujer, incluidos los valores vacíos (entradas especificadas sin género). Esto es diferente de buscar Pacientes en los que el sexo es varón, ya que eso no incluiría las entradas sin un género específico.

### <a name="missing"></a>:missing

`:missing` devuelve todos los recursos que no tienen un valor para el elemento especificado cuando el valor es y devuelve todos los recursos que contienen el elemento especificado cuando el `true` valor es `false` . En el caso de los elementos de tipo de datos simples, coincidirá con todos los recursos en los que el elemento esté presente con `:missing=true` extensiones, pero tenga un valor vacío. Por ejemplo, si desea encontrar todos los recursos a los que falta información sobre la fecha de `Patient` nacimiento, puede hacer lo siguiente:

```rest
GET [your-fhir-server]/Patient?birthdate:missing=true

```

### <a name="exact"></a>:exact
`:exact` se usa para los parámetros y devuelve resultados que coinciden con el parámetro con precisión, como en la concatenación de caracteres `string` y mayúsculas y minúsculas.

```rest
GET [your-fhir-server]/Patient?name:exact=Jon

```

Esta solicitud devuelve `Patient` recursos que tienen el nombre exactamente igual que `Jon` . Si el recurso tuviera pacientes con nombres como o , la búsqueda omitiría y omitiría el recurso, ya que no coincide exactamente con `Jonathan` `joN` el valor especificado.

### <a name="contains"></a>:contains
`:contains` se usa para los parámetros y busca recursos con coincidencias parciales del valor especificado en cualquier parte de la cadena dentro `string` del campo en el que se está buscando. `contains` no tiene en cuenta las mayúsculas y minúsculas y permite la concatenación de caracteres. Por ejemplo:

```rest
GET [your-fhir-server]/Patient?address:contains=Meadow

```

Esta solicitud devolvería todos los `Patient` recursos con campos que tienen valores que contienen la cadena `address` "Tiempo". Esto significa que podría tener direcciones que incluyan valores como "Ers" o "59 St. Deserción" devueltos como resultados de la búsqueda.

## <a name="chained-search"></a>Búsqueda encadenada 

Para realizar una serie de operaciones de búsqueda que cubren varios parámetros de referencia, puede "encadenar" la serie de parámetros de referencia anexándolos a la solicitud de servidor uno por uno mediante un punto `.` . Por ejemplo, si desea ver todos los recursos con `DiagnosticReport` una referencia a un recurso que incluye un determinado `subject` `Patient` `name` :  

```rest
 GET [your-fhir-server]/DiagnosticReport?subject:Patient.name=Sarah

```

Esta solicitud devolvería todos los `DiagnosticReport` recursos con un paciente llamado "Sarah". El período `.` después de que el campo realice la búsqueda `Patient` encadenada en el parámetro de referencia del `subject` parámetro .

Otro uso común de una búsqueda normal (no una búsqueda encadenada) es encontrar todos los encuentros de un paciente específico. `Patient`a menudo, los tendrán uno o `Encounter` varios con un asunto. Para buscar todos los `Encounter` recursos de un con el `Patient` `id` proporcionado:

```rest
GET [your-fhir-server]/Encounter?subject=Patient/78a14cbe-8968-49fd-a231-d43e6619399f

```

Mediante la búsqueda encadenada, puede encontrar todos los recursos que coincidan con un fragmento de información `Encounter` `Patient` determinado, como `birthdate` :

```rest
GET [your-fhir-server]/Encounter?subject:Patient.birthdate=1987-02-20

```

Esto permitiría no solo buscar recursos para un solo paciente, sino en todos los pacientes que tengan el valor de fecha `Encounter` de nacimiento especificado. 

Además, la búsqueda encadenada se puede realizar más de una vez en una solicitud mediante el símbolo , que permite buscar varias `&` condiciones en una solicitud. En tales casos, la búsqueda encadenada busca "independientemente" cada parámetro, en lugar de buscar condiciones que solo cumplan todas las condiciones a la vez:

```rest
GET [your-fhir-server]/Patient?general-practitioner:Practitioner.name=Sarah&general-practitioner:Practitioner.address-state=WA

```

Esto devolvería todos los recursos que tienen "Sarah" como y tienen un que `Patient` tiene la dirección con el wa de `generalPractitioner` `generalPractitioner` estado. En otras palabras, si un paciente tuviera a Sarah del estado NY y Bill del estado WA a los que se hace referencia como la del paciente, se `generalPractitioner` devolvería .

Para los escenarios en los que la búsqueda tiene que ser una operación AND que cubre todas las condiciones como grupo, consulte el ejemplo de **búsqueda** compuesta siguiente.

## <a name="reverse-chain-search"></a>Búsqueda en cadena inversa

La búsqueda en cadena permite buscar recursos en función de las propiedades de los recursos a los que hacen referencia. El uso de la búsqueda en cadena inversa le permite hacerlo al revés. Puede buscar recursos en función de las propiedades de los recursos que hacen referencia a ellos, mediante el `_has` parámetro . Por ejemplo, `Observation` el recurso tiene un parámetro de búsqueda que hace referencia a un recurso `patient` patient. Para buscar todos los recursos de pacientes a los que hace referencia `Observation` con un elemento `code` específico:

```rest
GET [base]/Patient?_has:Observation:patient:code=527

```

Esta solicitud devuelve los recursos de pacientes a los que hace `Observation` referencia con el código `527` . 

Además, la búsqueda en cadena inversa puede tener una estructura recursiva. Por ejemplo, si desea buscar todos los pacientes que tienen donde la observación tiene un evento de auditoría de un usuario `Observation` `janedoe` específico, podría hacer lo siguiente:

```rest
GET [base]/Patient?_has:Observation:patient:_has:AuditEvent:entity:agent:Practitioner.name=janedoe

``` 

> [!NOTE]
> En la Azure API for FHIR servidor de FHIR de código abierto con el respaldo de Cosmos, la búsqueda encadenada y la búsqueda encadenada inversa es una implementación de MVP. Para realizar búsquedas encadenadas en Cosmos DB, la implementación recorre la expresión de búsqueda y emite subconsediciones para resolver los recursos coincidentes. Esto se hace para cada nivel de la expresión. Si alguna consulta devuelve más de 100 resultados, se producirá un error.

## <a name="composite-search"></a>Búsqueda compuesta

Para buscar recursos que cumplan varias condiciones a la vez, use la búsqueda compuesta que une una secuencia de valores de parámetro único con un símbolo `$` . El resultado devuelto sería la intersección de los recursos que coinciden con todas las condiciones especificadas por los parámetros de búsqueda unidos. Estos parámetros de búsqueda se denominan parámetros de búsqueda compuestos y definen un nuevo parámetro que combina los varios parámetros en una estructura anidada. Por ejemplo, si desea buscar todos los recursos que contienen con un valor de parámetro menor o `DiagnosticReport` `Observation` igual que 9.2:

```rest
GET [your-fhir-server]/DiagnosticReport?result.code-value-quantity=2823-3$lt9.2

``` 

Esta solicitud especifica el componente que contiene un código de , que en `2823-3` este caso sería un elemento de tiempo. Después del símbolo, especifica el intervalo del valor para el componente que usa para "menor o igual que" y para el intervalo de valores `$` `lt` de `9.2` enumeración. 

## <a name="search-the-next-entry-set"></a>Buscar en el siguiente conjunto de entradas

El número máximo de entradas que se pueden devolver por una sola consulta de búsqueda es 1000. Sin embargo, es posible que tenga más de 1000 entradas que coincidan con la consulta de búsqueda y que desee ver el siguiente conjunto de entradas después de las primeras 1000 entradas devueltas. En tal caso, usaría el valor del token de `url` continuación en `searchset` como en el `Bundle` resultado siguiente:

```json
    "resourceType": "Bundle",
    "id": "98731cb7-3a39-46f3-8a72-afe945741bd9",
    "meta": {
        "lastUpdated": "2021-04-22T09:58:16.7823171+00:00"
    },
    "type": "searchset",
    "link": [
        {
            "relation": "next",
            "url": "[your-fhir-server]/Patient?_sort=_lastUpdated&ct=WzUxMDAxNzc1NzgzODc5MjAwODBd"
        },
        {
            "relation": "self",
            "url": "[your-fhir-server]/Patient?_sort=_lastUpdated"
        }
    ],

```

Y realizaría una solicitud GET para la dirección URL proporcionada en el campo `relation: next` :

```rest
GET [your-fhir-server]/Patient?_sort=_lastUpdated&ct=WzUxMDAxNzc1NzgzODc5MjAwODBd

```

Esto devolverá el siguiente conjunto de entradas para el resultado de la búsqueda. es el conjunto completo de entradas de resultados de búsqueda y el token de continuación es el vínculo proporcionado por el servidor para que pueda recuperar las entradas que no se muestran en el primer conjunto debido a la restricción en el número máximo de entradas devueltas para una consulta de `searchset` `url` búsqueda.

## <a name="search-using-post"></a>Búsqueda mediante POST

Todos los ejemplos de búsqueda mencionados anteriormente han usado `GET` solicitudes. También puede realizar operaciones de búsqueda mediante `POST` solicitudes mediante `_search` :

```rest
POST [your-fhir-server]/Patient/_search?_id=45

```

Esta solicitud devolvería todos `Patient` los recursos con el valor de `id` 45. Al igual que en las solicitudes GET, el servidor determina cuál de los recursos cumple las condiciones y devuelve un recurso de agrupación en la respuesta HTTP.

Otro ejemplo de búsqueda mediante POST donde los parámetros de consulta se envían como cuerpo del formulario es:

```rest
POST [your-fhir-server]/Patient/_search
content-type: application/x-www-form-urlencoded

name=John

```
## <a name="next-steps"></a>Pasos siguientes

>[!div class="nextstepaction"]
>[Introducción a la búsqueda de FHIR](overview-of-search.md)