---
title: 'Tutorial: DaVinci PDex - Azure API for FHIR'
description: Este tutorial le guía a través de la configuración de Azure API for FHIR para pasar pruebas para la Guía de implementación del intercambio de datos del pagador de Da Dag.
services: healthcare-apis
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: tutorial
ms.reviewer: matjazl
ms.author: cavoeg
author: caitlinv39
ms.date: 06/02/2021
ms.openlocfilehash: 0e6ee8ba2288ae61eb1fe0d5e6b76ebc63a64897
ms.sourcegitcommit: 832e92d3b81435c0aeb3d4edbe8f2c1f0aa8a46d
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/07/2021
ms.locfileid: "111562952"
---
# <a name="davinci-pdex"></a>DaVinci PDex

En este tutorial, le guiaremos por la configuración de la Azure API for FHIR para pasar las pruebas de [Touchstone](https://touchstone.aegis.net/touchstone/) para la Guía de implementación de intercambio de datos del pagador [da-and-payer](http://hl7.org/fhir/us/davinci-pdex/toc.html) (PDex IG).

> [!NOTE]
> Para todas estas pruebas, las ejecutaremos en las pruebas JSON. El Azure API for FHIR admite JSON y XML, pero no tiene puntos de conexión independientes para acceder a JSON o XML. Por este problema, se producirá un error en todas las pruebas XML. Si desea ver la instrucción de funcionalidad en XML, simplemente pase el parámetro \_ format: \` GET {fhirurl}/metadata? \_ format=xml\`

## <a name="touchstone-capability-statement"></a>Instrucción de funcionalidad de Touchstone

El primer conjunto de pruebas en el que nos centraremos es probar el Azure API for FHIR con la instrucción de funcionalidad de PDex IG. Estas pruebas tienen tres procesos de validación:

* La primera prueba simplemente valida la instrucción de funcionalidad básica con los requisitos de IG y se superará sin ninguna actualización.

* La segunda prueba valida que se han agregado todos los perfiles para US Core. Esta prueba se superará sin actualizaciones, pero incluirá varias advertencias. Para quitar estas advertencias, debe [cargar los perfiles de US Core](validation-against-profiles.md). Hemos creado un archivo [HTTP de ejemplo que](https://github.com/microsoft/fhir-server/blob/main/docs/rest/PayerDataExchange/USCore.http) le guía por la creación de todos los perfiles. También puede obtener los [perfiles directamente](http://hl7.org/fhir/us/core/STU3.1.1/profiles.html#profiles) desde el sitio HL7, que tendrá las versiones más recientes.

* La tercera prueba valida que se admite la operación patient-everything. En este momento, se producirá un error en esta prueba. La operación estará disponible a mediados de junio de 2021 en el Azure API for FHIR y ahora está disponible en el servidor FHIR de código abierto en Cosmos DB. Sin embargo, falta en la instrucción de funcionalidad, por lo que esta prueba producirá un error hasta que se libere una corrección para el error [1989](https://github.com/microsoft/fhir-server/issues/1989). 

 
:::image type="content" source="media/cms-tutorials/davinci-pdex-test-script-failed.png" alt-text="Error en la ejecución de DaVinci PDex.":::

## <a name="touchstone-member-match-test"></a>Prueba de coincidencia $member touchstone

La [segunda prueba de](https://touchstone.aegis.net/touchstone/testdefinitions?selectedTestGrp=/FHIRSandbox/DaVinci/FHIR4-0-1-Test/PDEX/PayerExchange/01-Member-Match&activeOnly=false&contentEntry=TEST_SCRIPTS) la sección Intercambio de datos de pago comprueba la existencia de la $member de búsqueda de [coincidencias](http://hl7.org/fhir/us/davinci-hrex/2020Sep/OperationDefinition-member-match.html). Puede obtener más información sobre la operación $member coincidencia en nuestra información general $member [de la operación de coincidencia de archivos](tutorial-member-match.md).

En esta prueba, deberá cargar algunos datos de ejemplo para que la prueba se pase. Aquí tenemos un archivo [rest](https://github.com/microsoft/fhir-server/blob/main/docs/rest/PayerDataExchange/membermatch.http) con el paciente y la cobertura vinculada que necesitará para la prueba. Una vez cargados estos datos, podrá superar correctamente esta prueba. Si no se cargan los datos, recibirá una respuesta 422 debido a que no encuentra una coincidencia exacta.

:::image type="content" source="media/cms-tutorials/davinci-pdex-test-script-passed.png" alt-text="Se ha superado el script de prueba de DaVinci PDex.":::

## <a name="touchstone-patient-by-reference"></a>Paciente de Touchstone por referencia

Las siguientes pruebas que revisaremos son las pruebas [de paciente por](https://touchstone.aegis.net/touchstone/testdefinitions?selectedTestGrp=/FHIRSandbox/DaVinci/FHIR4-0-1-Test/PDEX/PayerExchange/02-PatientByReference&activeOnly=false&contentEntry=TEST_SCRIPTS) referencia. Este conjunto de pruebas validan que puede encontrar un paciente en función de varios criterios de búsqueda. La mejor manera de probar al paciente por referencia será realizar pruebas [](https://github.com/microsoft/fhir-server/blob/main/docs/rest/PayerDataExchange/PDex_Sample_Data.http) con sus propios datos, pero hemos cargado un archivo de recursos de ejemplo que también puede cargar para usarlo.

:::image type="content" source="media/cms-tutorials/davinci-pdex-test-execution-passed.png" alt-text="Se ha pasado la ejecución de DaVinci PDex.":::

## <a name="touchstone-patienteverything-test"></a>Prueba de pacientes y $everything touchstone

La prueba final que se va a realizar es probar patient-everything. Para esta prueba, deberá cargar un paciente y, a continuación, usará el identificador de ese paciente para probar que puede usar la operación $everything para extraer todos los datos relacionados con el paciente.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, se explica cómo superar las pruebas de Payer Exchange en Touchstone. A continuación, puede obtener información sobre todas las Azure API for FHIR características.

>[!div class="nextstepaction"]
>[Características compatibles](fhir-features-supported.md)  