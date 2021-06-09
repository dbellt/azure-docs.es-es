---
title: Uso de patient-everything en Azure API for FHIR
description: En este artículo se explica cómo usar la operación patient-everything en el Azure API for FHIR
services: healthcare-apis
author: caitlinv39
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: conceptual
ms.date: 06/04/2021
ms.author: cavoeg
ms.openlocfilehash: fee544c318df318e09c75ebaf18fc0837cb4af0d
ms.sourcegitcommit: 832e92d3b81435c0aeb3d4edbe8f2c1f0aa8a46d
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/07/2021
ms.locfileid: "111562864"
---
# <a name="patient-everything-in-fhir"></a>Patient-everything in FHIR

La [$patient todo](https://www.hl7.org/fhir/patient-operation-everything.html) se creó para proporcionar a un paciente acceso a todo su registro o para que un proveedor u otro usuario realice una descarga masiva de datos. Esta operación se usa para devolver toda la información relacionada con uno o varios pacientes descritos en el recurso o contexto en el que se invoca esta operación.  

## <a name="use-patient-everything"></a>Uso de patient-everything
Para llamar a patient-everything, use el siguiente comando:

```json
GET {FHIRURL}/Patient/{ID}/$everything
```
El Azure API for FHIR valida que puede encontrar el paciente que coincide con el identificador de paciente proporcionado. Si se encuentra un resultado, la respuesta será un conjunto de tipo "searchset" con la siguiente información: 
* [Recurso de pacientes](https://www.hl7.org/fhir/patient.html) 
*  Recursos a los que hace referencia directamente el recurso Patient (excepto el vínculo) 
*  Recursos en el compartimiento del [paciente](https://www.hl7.org/fhir/compartmentdefinition-patient.html)
*  [Recursos de dispositivo](https://www.hl7.org/fhir/device.html) que hacen referencia al recurso Patient  

 
> [!Note]
> $patient todo está disponible en el servidor FHIR de código abierto con el respaldo de Cosmos DB y estará disponible en Azure API for FHIR antes del 1 de julio. Falta compatibilidad con la instrucción de funcionalidad del servidor de FHIR para $patient todo, cuyo seguimiento se realiza aquí: Problema [1989.](https://github.com/microsoft/fhir-server/issues/1989) 


## <a name="patient-everything-parameters"></a>Parámetros patient-everything
Azure API for FHIR admite los siguientes parámetros de consulta. Todos estos parámetros son opcionales:

|Parámetro de consulta        |  Descripción|
|-----------------------|------------|
| \_type | Permite especificar qué tipos de recursos se incluirán en la respuesta. Por ejemplo, \_ type=Encounter devolvería solo `Encounter` los recursos asociados al paciente. |
| \_since | Devolverá solo los recursos que se han modificado desde el momento proporcionado. |
| start | Si se especifica la fecha de inicio, se extraerán los recursos en los que haya una fecha de inicio posterior a la fecha de inicio especificada. Si no se proporciona ninguna fecha de inicio, todos los registros anteriores a la fecha de finalización están en el ámbito. |
| end | Si se especifica la fecha de finalización, se extraerán los recursos en los que haya una fecha de ensayo anterior a la fecha de finalización especificada. Si no se proporciona ninguna fecha de finalización, todos los registros después de la fecha de inicio están en el ámbito. |

> [!Note]
> Debe especificar un identificador para un paciente específico. Si necesita todos los datos para todos los pacientes, [consulte $export](export-data.md). 


## <a name="examples-of-patient-everything"></a>Ejemplos de $patient todo 

A continuación se muestran algunos ejemplos adicionales del uso de la $patient de todo. 

Para usar $patient todo para consultar el "todo" de un paciente entre 2010 y 2020, use la siguiente llamada: 

```json
GET {FHIRURL}/Patient/{ID}/$everything?start=2010&end=2020
``` 

Para usar $patient todo para consultar la observación y el encuentro de un paciente, use la siguiente llamada: 
```json
GET {FHIRURL}/Patient/{ID}/$everything_type=Observation,Encounter 
```

Para usar $patient-everything para consultar el "todo" de un paciente desde el 2021-05-27T05:00:00Z, use la siguiente llamada: 

```json
GET {FHIRURL}/Patient/{ID}/$everything?_since=2021-05-27T05:00:00Z 
```

Si se encuentra un paciente para cada una de estas llamadas, se obtiene una respuesta 200 con un paquete de los recursos correspondientes.

## <a name="next-step"></a>Paso siguiente
Ahora que sabe cómo usar la operación patient-everything, puede obtener más opciones de búsqueda en la introducción a la guía de búsqueda.

>[!div class="nextstepaction"]
>[Introducción a la búsqueda de FHIR](overview-of-search.md)