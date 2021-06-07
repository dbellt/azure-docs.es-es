---
title: 'Tutorial: Guía de implementación de CARIN para el botón&#174; azul ( Azure API for FHIR'
description: Este tutorial le guía por los pasos para configurar el Azure API for FHIR para pasar las pruebas de Touchstone para la Guía de implementación de CARIN para el botón azul (C4BB IG).
services: healthcare-apis
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: tutorial
ms.reviewer: matjazl
ms.author: cavoeg
author: caitlinv39
ms.date: 05/27/2021
ms.openlocfilehash: 25be80678389b59e907caf60529a8e4048a959e0
ms.sourcegitcommit: 832e92d3b81435c0aeb3d4edbe8f2c1f0aa8a46d
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/07/2021
ms.locfileid: "111562912"
---
# <a name="carin-implementation-guide-for-blue-button174"></a>Guía de implementación de CARIN para el botón azul&#174;

En este tutorial, le guiaremos por la configuración del Azure API for FHIR para pasar las pruebas de [Touchstone](https://touchstone.aegis.net/touchstone/) para la Guía de implementación de [CARIN ](https://build.fhir.org/ig/HL7/carin-bb/index.html) para el botón azul (C4BB IG).

## <a name="touchstone-capability-statement"></a>Instrucción de funcionalidad de Touchstone

La primera prueba en la que nos centraremos es probar el Azure API for FHIR con la instrucción de funcionalidad [C4BB IG](https://touchstone.aegis.net/touchstone/testdefinitions?selectedTestGrp=/FHIRSandbox/CARIN/CARIN-4-BlueButton/00-Capability&activeOnly=false&contentEntry=TEST_SCRIPTS). Si ejecuta esta prueba en el Azure API for FHIR sin actualizaciones, se producirá un error en la prueba debido a la falta de parámetros de búsqueda y a la falta de perfiles. 


### <a name="define-search-parameters"></a>Definición de parámetros de búsqueda

Como parte de C4BB IG, deberá definir tres nuevos parámetros [de búsqueda](how-to-do-custom-search.md) para el `ExplanationOfBenefit` recurso. Dos de ellos se prueban en la instrucción de funcionalidad (tipo y fecha de servicio) y uno es necesario para `_include` las búsquedas (mutua).  

* [type](https://build.fhir.org/ig/HL7/carin-bb/SearchParameter-explanationofbenefit-type.json)
* [service-date](https://build.fhir.org/ig/HL7/carin-bb/SearchParameter-explanationofbenefit-service-date.json)
* [Asegurador](https://build.fhir.org/ig/HL7/carin-bb/SearchParameter-explanationofbenefit-insurer.json)

> [!NOTE]
> En el código JSON sin formato para estos parámetros de búsqueda, el nombre se establece en `ExplanationOfBenefit_<SearchParameter Name>` . La prueba de Touchstone espera que el nombre de estos sea **de tipo**, **service-date** y **de la compañía de seguros**.  
 
El resto de los parámetros de búsqueda necesarios para C4BB IG se definen mediante la especificación base y ya están disponibles en el Azure API for FHIR sin ninguna actualización adicional.
 
### <a name="store-profiles"></a>Almacenamiento de perfiles

Fuera de definir los parámetros de búsqueda, la otra actualización que debe realizar para superar esta prueba es cargar los [perfiles necesarios.](validation-against-profiles.md) Hay ocho perfiles definidos dentro de C4BB IG. 

* [Cobertura C4BB](https://build.fhir.org/ig/HL7/carin-bb/StructureDefinition-C4BB-Coverage.html) 

* [C4BB ExplanationOfBenefit Inpatient Institutional](https://build.fhir.org/ig/HL7/carin-bb/StructureDefinition-C4BB-ExplanationOfBenefit-Inpatient-Institutional.html) 

* [Explicación de C4BBOfBenefit Outpatient Institutional](https://build.fhir.org/ig/HL7/carin-bb/StructureDefinition-C4BB-ExplanationOfBenefit-Outpatient-Institutional.html) 

* [Explicación de C4BBOfBenefit Infirible](https://build.fhir.org/ig/HL7/carin-bb/StructureDefinition-C4BB-ExplanationOfBenefit-Pharmacy.html) 

* [C4BB ExplanationOfBenefit Professional NonCcian](https://build.fhir.org/ig/HL7/carin-bb/StructureDefinition-C4BB-ExplanationOfBenefit-Professional-NonClinician.html) 

* [Organización C4BB](https://build.fhir.org/ig/HL7/carin-bb/StructureDefinition-C4BB-Organization.html) 

* [C4BB Patient](https://build.fhir.org/ig/HL7/carin-bb/StructureDefinition-C4BB-Patient.html) 

* [Profesional de C4BB](https://build.fhir.org/ig/HL7/carin-bb/StructureDefinition-C4BB-Practitioner.html) 

### <a name="sample-rest-file"></a>Archivo de rest de ejemplo

Para ayudar con la creación de estos perfiles y parámetros de búsqueda, tenemos un archivo [HTTP](https://github.com/microsoft/fhir-server/blob/main/docs/rest/C4BB/C4BB.http) de ejemplo que incluye todos los pasos descritos anteriormente en un único archivo. Una vez que haya cargado todos los perfiles y parámetros de búsqueda necesarios, puede ejecutar la prueba de la instrucción de funcionalidad en Touchstone.

:::image type="content" source="media/cms-tutorials/capability-test-script-execution-results.png" alt-text="Resultados de la ejecución del script de prueba de funcionalidad.":::

## <a name="touchstone-read-test"></a>Prueba de lectura de Touchstone

Después de probar la instrucción capabilities, probaremos las [funcionalidades](https://touchstone.aegis.net/touchstone/testdefinitions?selectedTestGrp=/FHIRSandbox/CARIN/CARIN-4-BlueButton/01-Read&activeOnly=false&contentEntry=TEST_SCRIPTS) de lectura del Azure API for FHIR con el C4BB IG. Esta prueba está probando la conformidad con los ocho perfiles que cargó en la primera prueba. Deberá tener los recursos cargados que se ajusten a los perfiles. La mejor ruta de acceso sería probar con los recursos que ya tiene en la base de datos, pero también tenemos un archivo [HTTP](https://github.com/microsoft/fhir-server/blob/main/docs/rest/C4BB/C4BB_Sample_Resources.http) disponible con recursos de ejemplo que se extraían de los ejemplos de la IG que puede usar para crear los recursos y realizar pruebas.

:::image type="content" source="media/cms-tutorials/test-execution-results-touchstone.png" alt-text="Resultados de la ejecución de pruebas de lectura de Touchstone.":::

## <a name="touchstone-eob-query-test"></a>Prueba de consulta EOB de Touchstone

La siguiente prueba que revisaremos es la [prueba de consulta EOB](https://touchstone.aegis.net/touchstone/testdefinitions?selectedTestGrp=/FHIRSandbox/CARIN/CARIN-4-BlueButton/02-EOBQuery&activeOnly=false&contentEntry=TEST_SCRIPTS). Si ya ha completado la prueba de lectura, tendrá todos los datos cargados que necesitará. Esta prueba valida que puede buscar pacientes específicos y explicar los recursos de las ventajas mediante varios parámetros.

:::image type="content" source="media/cms-tutorials/test-execution-touchstone-eob-query-test.png" alt-text="Resultados de la ejecución de consultas EOB de Touchstone.":::

## <a name="touchstone-error-handling-test"></a>Prueba de control de errores de Touchstone

La prueba final que se recorrerá es probar el [control de errores.](https://touchstone.aegis.net/touchstone/testdefinitions?selectedTestGrp=/FHIRSandbox/CARIN/CARIN-4-BlueButton/99-ErrorHandling&activeOnly=false&contentEntry=TEST_SCRIPTS) El único paso que debe realizar es eliminar un recurso ExplanationOfBenefit de la base de datos y usar el identificador del recurso ExplanationOfBenfit de eliminación en la prueba.

:::image type="content" source="media/cms-tutorials/test-execution-touchstone-error-handling.png" alt-text="Resultados del control de errores de EOB de Touchstone.":::


## <a name="next-steps"></a>Pasos siguientes

En este tutorial, se explica cómo pasar las pruebas de CARIN IG para el botón azul en Touchstone. A continuación, puede revisar cómo probar las pruebas de formulario de Da Vinci.

>[!div class="nextstepaction"]
>[DaVinci Drug Formulary](davinci-drug-formulary-tutorial.md)       
 