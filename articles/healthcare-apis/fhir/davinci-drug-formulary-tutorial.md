---
title: 'Tutorial: DaVinci Drug Formulary ( Azure API for FHIR'
description: Este tutorial le guía a través de la configuración Azure API for FHIR para pasar las pruebas de Touchstone en la guía de implementación de DaVinci Health Formulary.
services: healthcare-apis
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: tutorial
ms.reviewer: matjazl
ms.author: cavoeg
author: modillon
ms.date: 06/01/2021
ms.openlocfilehash: 22a4f42c0b63c42a50824301bcd9c056e7883a38
ms.sourcegitcommit: 832e92d3b81435c0aeb3d4edbe8f2c1f0aa8a46d
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/07/2021
ms.locfileid: "111562888"
---
# <a name="davinci-drug-formulary"></a>DaVinci Drug Formulary

En este tutorial, le guiaremos por la configuración de la Azure API for FHIR para pasar las pruebas de [Touchstone](https://touchstone.aegis.net/touchstone/) para la Guía de implementación de formularios de la fórmula de la meca para el intercambio de datos del pagador de [DaVinci](http://hl7.org/fhir/us/Davinci-drug-formulary/)para Estados Unidos.

## <a name="touchstone-capability-statement"></a>Instrucción de funcionalidad de Touchstone

La primera prueba en la que nos centraremos es probar el Azure API for FHIR con la instrucción de funcionalidad [de DaVinci Drug Formulary](https://touchstone.aegis.net/touchstone/testdefinitions?selectedTestGrp=/FHIRSandbox/DaVinci/FHIR4-0-1-Test/PDEX/Formulary/00-Capability&activeOnly=false&contentEntry=TEST_SCRIPTS). Si ejecuta esta prueba sin ninguna actualización, se producirá un error en la prueba debido a la falta de parámetros de búsqueda y a la falta de perfiles.

### <a name="define-search-parameters"></a>Definición de parámetros de búsqueda

Como parte de davinci Health Formulary IG, deberá definir tres nuevos parámetros de búsqueda [para](how-to-do-custom-search.md) el recurso FormularyDrug. Los tres se prueban en la instrucción de funcionalidad.

-   [DrugTier](http://hl7.org/fhir/us/davinci-drug-formulary/STU1.0.1/SearchParameter-DrugTier.json.html)

-   [Plan de consumo](http://hl7.org/fhir/us/davinci-drug-formulary/STU1.0.1/SearchParameter-DrugPlan.json.html)

-   [Nombre de la mesocoma](http://hl7.org/fhir/us/davinci-drug-formulary/STU1.0.1/SearchParameter-DrugName.json.html)

El resto de los parámetros de búsqueda necesarios para daVinci Drug Formulary IG se definen mediante la especificación base y ya están disponibles en el Azure API for FHIR sin más actualizaciones.

### <a name="store-profiles"></a>Almacenamiento de perfiles

Fuera de definir los parámetros de búsqueda, la única otra actualización que debe realizar para superar esta prueba es cargar los [perfiles necesarios.](validation-against-profiles.md) Hay dos perfiles que se usan como parte del IG de DaVinci Health Formulary.

-   [Formulary Drug](http://hl7.org/fhir/us/davinci-drug-formulary/STU1.0.1/StructureDefinition-usdf-FormularyDrug.html)

-   [Plan de cobertura de formularios](http://hl7.org/fhir/us/davinci-drug-formulary/STU1.0.1/StructureDefinition-usdf-CoveragePlan.html)

### <a name="sample-rest-file"></a>Archivo de rest de ejemplo

Para ayudar con la creación de estos perfiles y parámetros de búsqueda, tenemos el archivo HTTP de ejemplo de [DaVinci Formulary](https://github.com/microsoft/fhir-server/blob/main/docs/rest/DaVinciFormulary/DaVinciFormulary.http) en el sitio de código abierto que incluye todos los pasos descritos anteriormente en un solo archivo. Una vez que haya cargado todos los perfiles y parámetros de búsqueda necesarios, puede ejecutar la prueba de la instrucción de funcionalidad en Touchstone. Debe obtener una ejecución correcta:

:::image type="content" source="media/cms-tutorials/davinci-test-script-execution.png" alt-text="Ejecución del script de prueba de DaVinci.":::

## <a name="touchstone-query-test"></a>Prueba de consulta de Touchstone

La segunda prueba son las [funcionalidades de consulta](https://touchstone.aegis.net/touchstone/testdefinitions?selectedTestGrp=/FHIRSandbox/DaVinci/FHIR4-0-1-Test/PDEX/Formulary/01-Query&activeOnly=false&contentEntry=TEST_SCRIPTS). Esta prueba valida que puede buscar recursos específicos del plan de cobertura y de la salud mediante varios parámetros. La mejor ruta de acceso sería probar con los recursos que ya tiene en la base de datos, pero también tenemos el archivo HTTP de DaVinciFormulary_Sample_Resources disponible con recursos de ejemplo que se extraían [de](https://github.com/microsoft/fhir-server/blob/main/docs/rest/DaVinciFormulary/DaVinciFormulary_Sample_Resources.http) los ejemplos de la IG que puede usar para crear los recursos y realizar pruebas.

:::image type="content" source="media/cms-tutorials/davinci-test-execution-results.png" alt-text="Resultados de la ejecución de pruebas de DaVinci.":::

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, se explica cómo pasar el formulario de davinci payer data exchange us drug en Touchstone. A continuación, puede obtener información sobre cómo probar la Guía de implementación de DaDex PDex en Touchstone.

>[!div class="nextstepaction"]
>[DaVinci PDex](davinci-pdex-tutorial.md)