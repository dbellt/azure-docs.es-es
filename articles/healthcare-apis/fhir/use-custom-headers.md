---
title: 'Incorporación de datos a los registros de auditoría mediante encabezados personalizados: Azure API for FHIR'
description: En este artículo se describe cómo agregar datos a los registros de auditoría mediante el uso de encabezados HTTP personalizados en Azure API for FHIR.
services: healthcare-apis
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: tutorial
ms.reviewer: dseven
ms.author: zxue
author: matjazl
ms.date: 10/13/2019
ms.openlocfilehash: 574b2bcbf145441b98f8d1cd634316dc9a7654f3
ms.sourcegitcommit: 91fdedcb190c0753180be8dc7db4b1d6da9854a1
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/17/2021
ms.locfileid: "112292040"
---
# <a name="add-data-to-audit-logs-by-using-custom-http-headers"></a>Incorporación de datos a los registros de auditoría mediante encabezados HTTP personalizados

En la API de Recursos Rápidos de Interoperabilidad en Salud (FHIR) de Azure, un usuario puede querer incluir información adicional en los registros, que proceda del sistema de llamada.

Por ejemplo, cuando el usuario de la API se autentica mediante un sistema externo, el sistema reenvía la llamada a la API de FHIR. En la capa de la API de FHIR, la información sobre el usuario original se ha perdido porque la llamada se ha reenviado. Es posible que esta información de usuario tenga que registrarse y conservarse con fines de auditoría o administración. El sistema que realiza la llamada puede proporcionar la identidad del usuario, la ubicación del autor de la llamada u otra información necesaria en los encabezados HTTP, que se pasarán cuando se reenvíe la llamada.

Puede ver este flujo de datos en el diagrama siguiente:

:::image type="content" source="media/custom-headers/custom-headers-diagram.png" alt-text="Diagrama de encabezados personalizados":::

Puede usar los encabezados personalizados para capturar varios tipos de información. Por ejemplo:

* Información de identidad o autorización
* Origen del autor de la llamada
* Organización de origen
* Detalles del sistema de cliente (registro médico electrónico, portal de pacientes)

> [!IMPORTANT]
> Tenga en cuenta que la información enviada en los encabezados personalizados se almacena en un sistema de registro interno de Microsoft durante los 30 días siguientes a estar disponible en la supervisión de registros de Azure. Se recomienda cifrar cualquier información antes de agregarla a los encabezados personalizados. No debe pasar ninguna información médica protegida (PHI) a través de los encabezados de cliente.

Tiene que usar la siguiente convención de nomenclatura para los encabezados HTTP: X-MS-AZUREFHIR-AUDIT-\<name>.

Estos encabezados HTTP se incluyen en un contenedor de propiedades que se agrega al registro. Por ejemplo:

* X-MS-AZUREFHIR-AUDIT-USERID: 1234 
* X-MS-AZUREFHIR-AUDIT-USERLOCATION: XXXX
* X-MS-AZUREFHIR-AUDIT-XYZ: 1234

A continuación, esta información se serializa en JSON cuando se agrega a la columna de propiedades del registro. Por ejemplo:

```json
{ "X-MS-AZUREFHIR-AUDIT-USERID" : "1234",
"X-MS-AZUREFHIR-AUDIT-USERLOCATION" : "XXXX",
"X-MS-AZUREFHIR-AUDIT-XYZ" : "1234" }
```
 
Como con cualquier encabezado HTTP, el mismo nombre de encabezado se puede repetir con valores diferentes. Por ejemplo:

* X-MS-AZUREFHIR-AUDIT-USERLOCATION: HospitalA
* X-MS-AZUREFHIR-AUDIT-USERLOCATION: Emergencia

Cuando se agregan al registro, los valores se combinan con una lista delimitada por comas. Por ejemplo:

{ "X-MS-AZUREFHIR-AUDIT-USERLOCATION" : "HospitalA, Emergencia" }
 
Puede Agregar un máximo de 10 encabezados únicos (las repeticiones del mismo encabezado con valores diferentes cuentan solo como uno). La longitud total máxima del valor de cualquier encabezado es de 2048 caracteres.

Si usa la biblioteca de la API de cliente de Firefly C#, el código tiene un aspecto similar al siguiente:

```C#
FhirClient client;
client = new FhirClient(serverUrl);
client.OnBeforeRequest += (object sender, BeforeRequestEventArgs e) =>
{
    // Add custom headers to be added to the logs
    e.RawRequest.Headers.Add("X-MS-AZUREFHIR-AUDIT-UserLocation", "HospitalA");
};
client.Get("Patient");
```
## <a name="next-steps"></a>Pasos siguientes
En este artículo ha aprendido a agregar datos a los registros de auditoría mediante el uso de encabezados personalizados en Azure API for FHIR. A continuación, obtenga información sobre otros valores adicionales que puede configurar en Azure API for FHIR.
 
>[!div class="nextstepaction"]
>[Configuración adicional](azure-api-for-fhir-additional-settings.md)
