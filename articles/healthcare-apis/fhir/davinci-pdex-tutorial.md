---
title: 'Tutorial: DaIque PDex - Azure API for FHIR'
description: Este tutorial le guía a través de la configuración de Azure API for FHIR para pasar pruebas para la Guía de implementación del intercambio de datos del pagador de Da Tests.
services: healthcare-apis
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: tutorial
ms.reviewer: matjazl
ms.author: cavoeg
author: caitlinv39
ms.date: 06/07/2021
ms.openlocfilehash: 454b049a90bcf6e1d1793606a8759222698e5697
ms.sourcegitcommit: 8bca2d622fdce67b07746a2fb5a40c0c644100c6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111751116"
---
# <a name="da-vinci-pdex"></a>DaIque PDex

En este tutorial, le guiaremos a través de la configuración de Azure API for FHIR para pasar las pruebas de [Touchstone](https://touchstone.aegis.net/touchstone/) para la Guía de implementación de intercambio de datos del pagador de [Da Tests](http://hl7.org/fhir/us/davinci-pdex/toc.html) (PDex IG).

> [!NOTE]
> Para todas estas pruebas, las ejecutaremos en las pruebas JSON. El Azure API for FHIR admite JSON y XML, pero no tiene puntos de conexión independientes para acceder a JSON o XML. Por este problema, se producirá un error en todas las pruebas XML. Si desea ver la instrucción de funcionalidad en XML, simplemente pase el parámetro \_ format: \` GET {fhirurl}/metadata? \_ format=xml\`

## <a name="touchstone-capability-statement"></a>Instrucción de funcionalidad de Touchstone

El primer conjunto de pruebas en el que nos centraremos es probar el Azure API for FHIR con la instrucción de funcionalidad de IG de PDex. Esto incluye tres pruebas:

* La primera prueba valida la instrucción de funcionalidad básica con respecto a los requisitos de IG y se pasará sin ninguna actualización.

* La segunda prueba valida que se han agregado todos los perfiles para US Core. Esta prueba se superará sin actualizaciones, pero incluirá una serie de advertencias. Para quitar estas advertencias, debe [cargar los perfiles de US Core](validation-against-profiles.md). Hemos creado un archivo [HTTP de ejemplo que](https://github.com/microsoft/fhir-server/blob/main/docs/rest/PayerDataExchange/USCore.http) le guía a través de la creación de todos los perfiles. También puede obtener los [perfiles directamente](http://hl7.org/fhir/us/core/STU3.1.1/profiles.html#profiles) desde el sitio HL7, que tendrá las versiones más recientes.

* La tercera prueba valida que se [admite la $patient-todo.](patient-everything.md) En este momento, se producirá un error en esta prueba. La operación estará disponible a mediados de junio de 2021 en el Azure API for FHIR y ahora está disponible en el servidor FHIR de código abierto en Cosmos DB. Sin embargo, falta en la instrucción de funcionalidad, por lo que esta prueba producirá un error hasta que se libere una corrección al error [1989](https://github.com/microsoft/fhir-server/issues/1989). 

 
:::image type="content" source="media/cms-tutorials/davinci-pdex-test-script-failed.png" alt-text="Error en la ejecución de Da Dag PDex.":::

## <a name="touchstone-member-match-test"></a>Prueba de coincidencia $member de touchstone

La [segunda prueba de](https://touchstone.aegis.net/touchstone/testdefinitions?selectedTestGrp=/FHIRSandbox/DaVinci/FHIR4-0-1-Test/PDEX/PayerExchange/01-Member-Match&activeOnly=false&contentEntry=TEST_SCRIPTS) la sección Intercambio de datos del pagador comprueba la existencia de la $member de búsqueda de [coincidencias](http://hl7.org/fhir/us/davinci-hrex/2020Sep/OperationDefinition-member-match.html). Puede obtener más información sobre la operación de $member coincidencia en nuestra información [general $member operación de coincidencia de coincidencias.](tutorial-member-match.md)

En esta prueba, deberá cargar algunos datos de ejemplo para que la prueba se pase. Aquí tenemos un archivo rest [con](https://github.com/microsoft/fhir-server/blob/main/docs/rest/PayerDataExchange/membermatch.http) el paciente y la cobertura vinculada que necesitará para la prueba. Una vez cargados estos datos, podrá superar correctamente esta prueba. Si no se cargan los datos, recibirá una respuesta 422 debido a que no encuentra una coincidencia exacta.

:::image type="content" source="media/cms-tutorials/davinci-pdex-test-script-passed.png" alt-text="Se ha superado el script de prueba de Da Dag PDex.":::

## <a name="touchstone-patient-by-reference"></a>Paciente de Touchstone por referencia

Las siguientes pruebas que revisaremos son las pruebas [de paciente por referencia.](https://touchstone.aegis.net/touchstone/testdefinitions?selectedTestGrp=/FHIRSandbox/DaVinci/FHIR4-0-1-Test/PDEX/PayerExchange/02-PatientByReference&activeOnly=false&contentEntry=TEST_SCRIPTS) Este conjunto de pruebas validan que puede encontrar un paciente en función de varios criterios de búsqueda. La mejor manera de probar al paciente por referencia será realizar pruebas [](https://github.com/microsoft/fhir-server/blob/main/docs/rest/PayerDataExchange/PDex_Sample_Data.http) con sus propios datos, pero hemos cargado un archivo de recursos de ejemplo que también puede cargar para usarlo.

:::image type="content" source="media/cms-tutorials/davinci-pdex-test-execution-passed.png" alt-text="Se ha pasado la ejecución de Da Dag PDex.":::

## <a name="touchstone-patienteverything-test"></a>Prueba de pacientes/$everything touchstone

La prueba final que se va a realizar es probar patient-everything. Para esta prueba, deberá cargar un paciente y, a continuación, usará el identificador de ese paciente para probar que puede usar la operación $everything para extraer todos los datos relacionados con el paciente.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, se explica cómo superar las pruebas de Exchange del pagador en Touchstone. A continuación, puede obtener información sobre todas las Azure API for FHIR características.

>[!div class="nextstepaction"]
>[Características compatibles](fhir-features-supported.md)  