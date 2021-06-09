---
title: 'Tutorial: DaAnalysis Drug Formulary ( Azure API for FHIR'
description: Este tutorial le guía a través de la configuración Azure API for FHIR para pasar las pruebas de Touchstone en la guía de implementación de DaVinci Health Formulary.
services: healthcare-apis
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: tutorial
ms.reviewer: matjazl
ms.author: cavoeg
author: modillon
ms.date: 06/07/2021
ms.openlocfilehash: fc66de13c02d6d04e8a156937de70775b9053da3
ms.sourcegitcommit: 8bca2d622fdce67b07746a2fb5a40c0c644100c6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111751098"
---
# <a name="da-vinci-drug-formulary"></a>DaCoso de la fórmula de la receta

En este tutorial, le guiaremos por la configuración de Azure API for FHIR para pasar las pruebas de [Touchstone](https://touchstone.aegis.net/touchstone/) para la Guía de implementación de formularios de la fórmula de consumo de cáncer de Ee. UU. de [Da York Payer Data Exchange](http://hl7.org/fhir/us/Davinci-drug-formulary/).

## <a name="touchstone-capability-statement"></a>Instrucción de funcionalidad de Touchstone

La primera prueba en la que nos centraremos es probar el Azure API for FHIR con la instrucción de funcionalidad De la fórmula de la receta de cáncer [de da- da.](https://touchstone.aegis.net/touchstone/testdefinitions?selectedTestGrp=/FHIRSandbox/DaVinci/FHIR4-0-1-Test/PDEX/Formulary/00-Capability&activeOnly=false&contentEntry=TEST_SCRIPTS) Si ejecuta esta prueba sin ninguna actualización, se producirá un error en la prueba debido a la falta de parámetros de búsqueda y a la falta de perfiles.

### <a name="define-search-parameters"></a>Definición de parámetros de búsqueda

Como parte de la ig del fórmulado de productos de Da York, deberá definir tres nuevos parámetros de búsqueda [para](how-to-do-custom-search.md) el recurso FormularyDrug. Los tres se prueban en la instrucción de funcionalidad.

* [DrugTier](http://hl7.org/fhir/us/davinci-drug-formulary/STU1.0.1/SearchParameter-DrugTier.json.html)
* [Plan de consumo](http://hl7.org/fhir/us/davinci-drug-formulary/STU1.0.1/SearchParameter-DrugPlan.json.html)
* [Nombre de la mesocoma](http://hl7.org/fhir/us/davinci-drug-formulary/STU1.0.1/SearchParameter-DrugName.json.html)

El resto de los parámetros de búsqueda necesarios para la fórmula de dación en la fórmula de la receta de cáncer se definen mediante la especificación base y ya están disponibles en el Azure API for FHIR sin más actualizaciones.

### <a name="store-profiles"></a>Almacenamiento de perfiles

Fuera de definir parámetros de búsqueda, la única otra actualización que debe realizar para superar esta prueba es cargar los [perfiles necesarios.](validation-against-profiles.md) Hay dos perfiles que se usan como parte de DaEtt Drug Formulary IG.

* [Formulary Drug](http://hl7.org/fhir/us/davinci-drug-formulary/STU1.0.1/StructureDefinition-usdf-FormularyDrug.html)
* [Plan de cobertura de formularios](http://hl7.org/fhir/us/davinci-drug-formulary/STU1.0.1/StructureDefinition-usdf-CoveragePlan.html)

### <a name="sample-rest-file"></a>Archivo de rest de ejemplo

Para ayudar con la creación de estos perfiles y parámetros de búsqueda, tenemos el archivo HTTP de ejemplo de la fórmula [DaEtt](https://github.com/microsoft/fhir-server/blob/main/docs/rest/DaVinciFormulary/DaVinciFormulary.http) en el sitio de código abierto que incluye todos los pasos descritos anteriormente en un solo archivo. Una vez que haya cargado todos los perfiles y parámetros de búsqueda necesarios, puede ejecutar la prueba de la instrucción de funcionalidad en Touchstone. Debe obtener una ejecución correcta:

:::image type="content" source="media/cms-tutorials/davinci-test-script-execution.png" alt-text="Ejecución del script de prueba de Da Dag.":::

## <a name="touchstone-query-test"></a>Prueba de consulta de Touchstone

La segunda prueba son las [funcionalidades de consulta](https://touchstone.aegis.net/touchstone/testdefinitions?selectedTestGrp=/FHIRSandbox/DaVinci/FHIR4-0-1-Test/PDEX/Formulary/01-Query&activeOnly=false&contentEntry=TEST_SCRIPTS). Esta prueba valida que puede buscar recursos específicos del plan de cobertura y de la salud mediante varios parámetros. La mejor ruta de acceso sería probar los recursos que ya tiene en la base de datos, pero también tenemos el archivo HTTP [da VinciFormulary_Sample_Resources](https://github.com/microsoft/fhir-server/blob/main/docs/rest/DaVinciFormulary/DaVinciFormulary_Sample_Resources.http) disponible con los recursos de ejemplo que se extraían de los ejemplos de la IG que puede usar para crear los recursos y realizar pruebas.

:::image type="content" source="media/cms-tutorials/davinci-test-execution-results.png" alt-text="Resultados de la ejecución de pruebas de Da Dag.":::

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, se explica cómo pasar el formulario de intercambio de datos del pagador da da a estados unidos en Touchstone. A continuación, puede obtener información sobre cómo probar la Guía de implementación de DaIque PDex en Touchstone.

>[!div class="nextstepaction"]
>[DaIque PDex](davinci-pdex-tutorial.md)