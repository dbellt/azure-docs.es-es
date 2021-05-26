---
title: $validate recursos de FHIR en los perfiles de Azure API for FHIR
description: $validate recursos de FHIR en perfiles
author: ginalee-dotcom
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: reference
ms.date: 05/06/2021
ms.author: ginle
ms.openlocfilehash: 2c367dbed14e0dba9a8a95a3ce2709d2415c7cd6
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110466708"
---
# <a name="how-to-validate-fhir-resources-against-profiles"></a>Validación de recursos de FHIR con perfiles

HL7 FHIR define una manera estándar e interoperable de almacenar e intercambiar datos sanitarios. Incluso dentro de la especificación FHIR base, puede ser útil definir reglas o extensiones adicionales en función del contexto que se usa FHIR. Para estos usos específicos del contexto de FHIR, los perfiles **de FHIR** se usan para la capa adicional de especificaciones.

[El perfil de FHIR](https://www.hl7.org/fhir/profiling.html) describe contexto adicional, como restricciones o extensiones, en un recurso representado como `StructureDefinition` . El estándar HL7 FHIR define un conjunto de recursos base y estos recursos base estándar tienen definiciones genéricas. El perfil de FHIR permite restringir y personalizar las definiciones de recursos mediante restricciones y extensiones.

Azure API for FHIR permite validar los recursos en los perfiles para ver si los recursos se ajustan a los perfiles. Este artículo le guía por los conceptos básicos del perfil de FHIR y cómo usar para validar recursos en los perfiles al `$validate` crear y actualizar recursos.

## <a name="fhir-profile-the-basics"></a>Perfil de FHIR: los aspectos básicos

Un perfil establece contexto adicional en el recurso, normalmente representado como `StructureDefinition` un recurso. `StructureDefinition` define un conjunto de reglas sobre el contenido de un recurso o un tipo de datos, como qué campos tiene un recurso y qué valores pueden tomar estos campos. Por ejemplo, los perfiles pueden restringir la cardinalidad (por ejemplo, establecer la cardinalidad máxima en 0 para descartar el elemento), restringir el contenido de un elemento a un único valor fijo o definir las extensiones necesarias para el recurso. También puede especificar restricciones adicionales en un perfil existente. Un `StructureDefinition` se identifica mediante su dirección URL canónica:

```rest
http://hl7.org/fhir/StructureDefinition/{profile}
```

En el campo `{profile}` , especifique el nombre del perfil.

Por ejemplo:

- `http://hl7.org/fhir/StructureDefinition/patient-birthPlace` es un perfil base que requiere información sobre la dirección registrada del nacimiento del paciente.
- `http://hl7.org/fhir/StructureDefinition/bmi` es otro perfil base que define cómo representar observaciones del índice de masa corporal (BMI).
- `http://hl7.org/fhir/us/core/StructureDefinition/us-core-allergyintolerance` es un perfil de US Core que establece las expectativas mínimas para los recursos asociados a un paciente e identifica campos obligatorios, como extensiones y `AllergyIntolerance` conjuntos de valores.

### <a name="base-profile-and-custom-profile"></a>Perfil base y perfil personalizado

Hay dos tipos de perfiles: perfil base y perfil personalizado. Un perfil base es una base a la que un recurso debe ajustarse y se ha definido mediante `StructureDefinition` recursos base como o `Patient` `Observation` . Por ejemplo, un perfil de índice de masa corporal (BMI) `Observation` se iniciaría de la siguiente forma:

```json
{
  "resourceType" : "StructureDefinition",
  "id" : "bmi",
...
}
```

Un perfil personalizado es un conjunto de restricciones adicionales sobre un perfil base, lo que restringe o agrega parámetros de recursos que no forman parte de la especificación base. El perfil personalizado es útil porque puede personalizar sus propias definiciones de recursos especificando las restricciones y extensiones en el recurso base existente. Por ejemplo, es posible que desee crear un perfil que muestre instancias de recursos basadas en géneros, en cuyo caso crearía un perfil personalizado sobre un perfil existente `AllergyIntolerance` `Patient` con `Patient` `AllergyIntolerance` perfil.

> [!NOTE]
> Los perfiles personalizados deben compilarse sobre el recurso base y no pueden estar en conflicto con el recurso base. Por ejemplo, si un elemento tiene una cardinalidad de 1..1, el perfil personalizado no puede hacerlo opcional.

Los perfiles personalizados también se especifican mediante varias guías de implementación. Algunas guías de implementación comunes son:

|Nombre |URL
|---- |----
Us Core |<https://www.hl7.org/fhir/us/core/>
Botón AZUL CARIN |<http://hl7.org/fhir/us/carin-bb/>
Intercambio de datos del pagador da-payer |<http://hl7.org/fhir/us/davinci-pdex/>
Argonauta |<http://www.fhir.org/guides/argonaut/pd/>

## <a name="accessing-profiles-and-storing-profiles"></a>Acceso a perfiles y almacenamiento de perfiles

### <a name="storing-profiles"></a>Almacenamiento de perfiles

Para almacenar perfiles en el servidor, puede realizar una `POST` solicitud:

```rest
POST http://<your FHIR service base URL>/{Resource}
```

En el que el campo se reemplazará por , y tendría el recurso en el `{Resource}` servidor en formato o `StructureDefinition` `StructureDefinition` `POST` `JSON` `XML` . Por ejemplo, si desea almacenar el `us-core-allergyintolerance` perfil, haría lo siguiente:

```rest
POST http://my-fhir-server.azurewebsites.net/StructureDefinition?url=http://hl7.org/fhir/us/core/StructureDefinition/us-core-allergyintolerance
```

Dónde se almacenaría y recuperaría el perfil de us core- ing:

```json
{
    "resourceType" : "StructureDefinition",
    "id" : "us-core-allergyintolerance",
    "text" : {
        "status" : "extensions"
    },
    "url" : "http://hl7.org/fhir/us/core/StructureDefinition/us-core-allergyintolerance",
    "version" : "3.1.1",
    "name" : "USCoreAllergyIntolerance",
    "title" : "US  Core AllergyIntolerance Profile",
    "status" : "active",
    "experimental" : false,
    "date" : "2020-06-29",
        "publisher" : "HL7 US Realm Steering Committee",
    "contact" : [
    {
      "telecom" : [
        {
          "system" : "url",
          "value" : "http://www.healthit.gov"
        }
      ]
    }
  ],
    "description" : "Defines constraints and extensions on the AllergyIntolerance resource for the minimal set of data to query and retrieve allergy information.",

...
```

La mayoría de los perfiles tienen el tipo de recurso , pero también pueden ser de `StructureDefinition` los tipos y , que son recursos de `ValueSet` `CodeSystem` [terminología.](http://hl7.org/fhir/terminologies.html) Por ejemplo, si tiene un perfil en un formulario JSON, el servidor devolverá el perfil almacenado con el asignado para el perfil, tal como lo `POST` `ValueSet` haría con `id` `StructureDefinition` . A continuación se muestra un [](https://www.hl7.org/fhir/valueset-condition-severity.html) ejemplo que se obtiene al cargar un perfil de gravedad de condición, que especifica los criterios para una clasificación de gravedad de condición o diagnóstico:

```json
{
    "resourceType": "ValueSet",
    "id": "35ab90e5-c75d-45ca-aa10-748fefaca7ee",
    "meta": {
        "versionId": "1",
        "lastUpdated": "2021-05-07T21:34:28.781+00:00",
        "profile": [
            "http://hl7.org/fhir/StructureDefinition/shareablevalueset"
        ]
    },
    "text": {
        "status": "generated"
    },
    "extension": [
        {
            "url": "http://hl7.org/fhir/StructureDefinition/structuredefinition-wg",
            "valueCode": "pc"
        }
    ],
    "url": "http://hl7.org/fhir/ValueSet/condition-severity",
    "identifier": [
        {
            "system": "urn:ietf:rfc:3986",
            "value": "urn:oid:2.16.840.1.113883.4.642.3.168"
        }
    ],
    "version": "4.0.1",
    "name": "Condition/DiagnosisSeverity",
    "title": "Condition/Diagnosis Severity",
    "status": "draft",
    "experimental": false,
    "date": "2019-11-01T09:29:23+11:00",
    "publisher": "FHIR Project team",
...
```

Puede ver que es un , y para el perfil también especifica `resourceType` que se trata de un tipo : `ValueSet` `url` `ValueSet` `"http://hl7.org/fhir/ValueSet/condition-severity"` .

### <a name="viewing-profiles"></a>Visualización de perfiles

Puede acceder a los perfiles personalizados existentes en el servidor mediante una `GET` solicitud. Todos los perfiles válidos, como los perfiles con direcciones URL canónicas válidas en las guías de implementación, deben ser accesibles consultando:

```rest
GET http://<your FHIR service base URL>/StructureDefinition?url={canonicalUrl} 
```

Donde el campo `{canonicalUrl}` se reemplazaría por la dirección URL canónica del perfil.

Por ejemplo, si desea ver el perfil de recursos de US `Goal` Core:

```rest
GET http://my-fhir-server.azurewebsites.net/StructureDefinition?url=http://hl7.org/fhir/us/core/StructureDefinition/us-core-goal
```

Esto devolverá el recurso `StructureDefinition` para el perfil de US Core Goal, que comenzará de la siguiente forma:

```json
{
  "resourceType" : "StructureDefinition",
  "id" : "us-core-goal",
  "url" : "http://hl7.org/fhir/us/core/StructureDefinition/us-core-goal",
  "version" : "3.1.1",
  "name" : "USCoreGoalProfile",
  "title" : "US Core Goal Profile",
  "status" : "active",
  "experimental" : false,
  "date" : "2020-07-21",
  "publisher" : "HL7 US Realm Steering Committee",
  "contact" : [
    {
      "telecom" : [
        {
          "system" : "url",
          "value" : "http://www.healthit.gov"
        }
      ]
    }
  ],
  "description" : "Defines constraints and extensions on the Goal resource for the minimal set of data to query and retrieve a patient's goal(s).",
...
```

Nuestro servidor FHIR no devuelve instancias para los perfiles base, pero se pueden encontrar fácilmente en el sitio web `StructureDefinition` hl7, como:

- `http://hl7.org/fhir/Observation.profile.json.html`
- `http://hl7.org/fhir/Patient.profile.json.html`


### <a name="profiles-in-the-capability-statement"></a>Perfiles en la instrucción de funcionalidad

enumera todos los comportamientos posibles del servidor de FHIR que se usarán como una instrucción de la funcionalidad del servidor, como definiciones de estructura y `Capability Statement` conjuntos de valores. Azure API for FHIR actualiza la instrucción de funcionalidad con información sobre los perfiles cargados y almacenados en las formas siguientes:

- `CapabilityStatement.rest.resource.profile`
- `CapabilityStatement.rest.resource.supportedProfile`

Estos mostrarán toda la especificación del perfil que describe la compatibilidad general con el recurso, incluidas las restricciones de cardinalidad, enlaces, extensiones u otras restricciones. Por lo tanto, cuando un perfil en forma de y los metadatos de recursos para ver la instrucción de funcionalidad completa, verá junto al parámetro todos los detalles sobre el perfil que `POST` `StructureDefinition` `GET` `supportedProfiles` carizó.

Por ejemplo, si tiene un `POST` perfil de us Core Patient, que comienza de esta forma:

```json
{
  "resourceType": "StructureDefinition",
  "id": "us-core-patient",
  "url": "http://hl7.org/fhir/us/core/StructureDefinition/us-core-patient",
  "version": "3.1.1",
  "name": "USCorePatientProfile",
  "title": "US Core Patient Profile",
  "status": "active",
  "experimental": false,
  "date": "2020-06-27",
  "publisher": "HL7 US Realm Steering Committee",
...
```

Y envíe una `GET` solicitud para `metadata` :

```rest
GET http://<your FHIR service base URL>/metadata
```

Se le devolverá un valor de que incluye la siguiente información en el perfil de us Core Patient que `CapabilityStatement` carizó en el servidor de FHIR:

```json
...
{
    "type": "Patient",
    "profile": "http://hl7.org/fhir/StructureDefinition/Patient",
    "supportedProfile":[
        "http://hl7.org/fhir/us/core/StructureDefinition/us-core-patient"
    ],
...
```

## <a name="validating-resources-against-the-profiles"></a>Validación de recursos en los perfiles

Los recursos de FHIR, como `Patient` o , pueden expresar su conformidad con `Observation` perfiles específicos. Esto permite que nuestro servidor de FHIR **valide determinados** recursos con los perfiles asociados o los perfiles especificados. Validar un recurso con perfiles significa comprobar si el recurso se ajusta a los perfiles, incluidas las especificaciones enumeradas en `Resource.meta.profile` o en una guía de implementación.

Hay dos maneras de validar el recurso. En primer lugar, puede `$validate` usar la operación en un recurso que ya está en el servidor de FHIR. En segundo lugar, `POST` puede hacerlo en el servidor como parte de un recurso u `Update` `Create` operación. En ambos casos, puede decidir a través de la configuración del servidor de FHIR qué hacer cuando el recurso no se ajusta al perfil deseado.

### <a name="using-validate"></a>Uso de $validate

La operación comprueba si el perfil proporcionado es válido y `$validate` si el recurso se ajusta al perfil especificado. Como se mencionó en las [especificaciones de HL7 FHIR,](https://www.hl7.org/fhir/resource-operation-validate.html)también puede especificar `mode` para , como y `$validate` `create` `update` :

- `create`: el servidor comprueba que el contenido del perfil es único de los recursos existentes y que es aceptable que se cree como un nuevo recurso.
- `update`: comprueba que el perfil es una actualización con respecto al recurso existente designado (por ejemplo, que no se realiza ningún cambio en los campos inmutables).

El servidor siempre devolverá un como `OperationOutcome` resultado de la validación.

#### <a name="validating-an-existing-resource"></a>Validación de un recurso existente

Para validar un recurso existente, use `$validate` en una `GET` solicitud:

```rest
GET http://<your FHIR service base URL>/{resource}/{resource ID}/$validate
```

Por ejemplo:

```rest
GET http://my-fhir-server.azurewebsites.net/Patient/a6e11662-def8-4dde-9ebc-4429e68d130e/$validate
```

En el ejemplo anterior, validaría el recurso `Patient` `a6e11662-def8-4dde-9ebc-4429e68d130e` existente. Si es válido, recibirá un como `OperationOutcome` el siguiente:

```json
{
    "resourceType": "OperationOutcome",
    "issue": [
        {
            "severity": "information",
            "code": "informational",
            "diagnostics": "All OK"
        }
    ]
}
```

Si el recurso no es válido, recibirá un código de error y un mensaje de error con detalles sobre por qué el recurso no es válido. Un error o significa que no se pudo realizar la validación en sí y se desconoce si `4xx` el recurso es válido o `5xx` no. Un ejemplo `OperationOutcome` devuelto con mensajes de error podría ser parecido al siguiente:

```json
{
    "resourceType": "OperationOutcome",
    "issue": [
        {
            "severity": "error",
            "code": "invalid",
            "details": {
                "coding": [
                    {
                        "system": "http://hl7.org/fhir/dotnet-api-operation-outcome",
                        "code": "1028"
                    }
                ],
                "text": "Instance count for 'Patient.identifier.value' is 0, which is not within the specified cardinality of 1..1"
            },
            "location": [
                "Patient.identifier[1]"
            ]
        },
        {
            "severity": "error",
            "code": "invalid",
            "details": {
                "coding": [
                    {
                        "system": "http://hl7.org/fhir/dotnet-api-operation-outcome",
                        "code": "1028"
                    }
                ],
                "text": "Instance count for 'Patient.gender' is 0, which is not within the specified cardinality of 1..1"
            },
            "location": [
                "Patient"
            ]
        }
    ]
}
```

En este ejemplo anterior, el recurso no se ajustaba al perfil proporcionado que requería un `Patient` valor de identificador de paciente y un sexo.

Si desea especificar un perfil como parámetro, puede especificar la dirección URL canónica con la que se va a validar el perfil, como el ejemplo siguiente con el perfil de US Core y un `Patient` perfil base para `heartrate` :

```rest
GET http://<your FHIR service base URL>/{Resource}/{Resource ID}/$validate?profile={canonicalUrl}
```

Por ejemplo:

```rest
GET http://my-fhir-server.azurewebsites.net/Patient/a6e11662-def8-4dde-9ebc-4429e68d130e/$validate?profile=http://hl7.org/fhir/us/core/StructureDefinition/us-core-patient
GET http://my-fhir-server.azurewebsites.net/Observation/12345678/$validate?profile=http://hl7.org/fhir/StructureDefinition/heartrate
```

#### <a name="validating-a-new-resource"></a>Validación de un nuevo recurso

Si desea validar un nuevo recurso que va a cargar en el servidor, puede realizar una `POST` solicitud:

```rest
POST http://<your FHIR service base URL>/{Resource}/$validate
```

Por ejemplo:

```rest
POST http://my-fhir-server.azurewebsites.net/Patient/$validate 
```

Esta solicitud creará el nuevo recurso que está especificando en la carga de la solicitud, tanto si está en formato JSON como XML, y validará el recurso cargado. A continuación, devolverá `OperationOutcome` como resultado de la validación en el nuevo recurso.

### <a name="validate-on-resource-create-or-resource-update"></a>Validación en el recurso CREATE o resource UPDATE

Puede elegir cuándo desea validar el recurso, como en el recurso CREATE o UPDATE. Puede especificarlo en la configuración del servidor, en `CoreFeatures` :

```json
{
   "FhirServer": {
      "CoreFeatures": {
            "ProfileValidationOnCreate": true,
            "ProfileValidationOnUpdate": false
        }
}
```

Si el recurso se ajusta al proporcionado y el perfil está presente en el sistema, el servidor actuará según la configuración `Resource.meta.profile` anterior. Si el perfil proporcionado no está presente en el servidor, la solicitud de validación se omitirá y se deja en `Resource.meta.profile` .
La validación suele ser una operación costosa, por lo que normalmente solo se ejecuta en servidores de prueba o en un pequeño subconjunto de recursos, por lo que es importante tener estas maneras de activar o desactivar la validación en el lado servidor. Si la configuración del servidor especifica que no se valida en el recurso Create/Update, el usuario puede invalidar el comportamiento si lo especifica en la de la `header` solicitud Create/Update:

```rest
x-ms-profile-validation: true
```

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha aprendido sobre los perfiles de FHIR y cómo validar los recursos en los perfiles mediante $validate. Para obtener información sobre Azure API for FHIR otras características admitidas, consulte:

>[!div class="nextstepaction"]
>[Características compatibles con FHIR](fhir-features-supported.md)
