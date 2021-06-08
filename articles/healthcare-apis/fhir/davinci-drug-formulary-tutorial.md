---
title: 'Tutorial: Formulario de davinci sobre el consumo de Azure API for FHIR'
description: Este tutorial le guía a través de la configuración de Azure API for FHIR para pasar las pruebas de Touchstone en la guía de implementación de DaVinci Health Formulary.
services: healthcare-apis
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: tutorial
ms.reviewer: matjazl
ms.author: cavoeg
author: modillon
ms.date: 06/01/2021
ms.openlocfilehash: feeccd8f194397e9c99f19920d09b8bb2056ff08
ms.sourcegitcommit: b11257b15f7f16ed01b9a78c471debb81c30f20c
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/08/2021
ms.locfileid: "111592954"
---
# <a name="davinci-drug-formulary"></a>DaVinci Health Formulary

En este tutorial, le guiaremos a través de la configuración de la Azure API for FHIR para pasar las pruebas de [Touchstone](https://touchstone.aegis.net/touchstone/) para la Guía de implementación de la fórmula de la receta de estados unidos de [DaVinci Payer Data Exchange](http://hl7.org/fhir/us/Davinci-drug-formulary/).

## <a name="touchstone-capability-statement"></a>Instrucción de funcionalidad de Touchstone

La primera prueba en la que nos centraremos es probar el Azure API for FHIR con la instrucción de funcionalidad [de DaVinci Health Formulary](https://touchstone.aegis.net/touchstone/testdefinitions?selectedTestGrp=/FHIRSandbox/DaVinci/FHIR4-0-1-Test/PDEX/Formulary/00-Capability&activeOnly=false&contentEntry=TEST_SCRIPTS). Si ejecuta esta prueba sin actualizaciones, se producirá un error en la prueba debido a que faltan parámetros de búsqueda y perfiles que faltan.

### <a name="define-search-parameters"></a>Definición de parámetros de búsqueda

Como parte del ig de la fórmula de la receta de da york, deberá definir tres nuevos parámetros de búsqueda [para](how-to-do-custom-search.md) el recurso FormularyDrug. Los tres se prueban en la instrucción de funcionalidad.

* [DrugTier](http://hl7.org/fhir/us/davinci-drug-formulary/STU1.0.1/SearchParameter-DrugTier.json.html)
* [Plan de consumo](http://hl7.org/fhir/us/davinci-drug-formulary/STU1.0.1/SearchParameter-DrugPlan.json.html)
* [Nombre DeCombreDeCombre](http://hl7.org/fhir/us/davinci-drug-formulary/STU1.0.1/SearchParameter-DrugName.json.html)

El resto de los parámetros de búsqueda necesarios para la IG de da- mente se definen mediante la especificación base y ya están disponibles en el Azure API for FHIR sin más actualizaciones.

### <a name="store-profiles"></a>Almacenamiento de perfiles

Fuera de definir los parámetros de búsqueda, la única otra actualización que debe realizar para superar esta prueba es cargar los [perfiles necesarios.](validation-against-profiles.md) Hay dos perfiles que se usan como parte de la fórmula de la receta de DaThth, IG.

* [Formulary Drug](http://hl7.org/fhir/us/davinci-drug-formulary/STU1.0.1/StructureDefinition-usdf-FormularyDrug.html)
* [Plan de cobertura de formularios](http://hl7.org/fhir/us/davinci-drug-formulary/STU1.0.1/StructureDefinition-usdf-CoveragePlan.html)

### <a name="sample-rest-file"></a>Archivo de rest de ejemplo

Para ayudar con la creación de estos perfiles y parámetros de búsqueda, tenemos el archivo HTTP de ejemplo de [Da Dag Yacaly](https://github.com/microsoft/fhir-server/blob/main/docs/rest/DaVinciFormulary/DaVinciFormulary.http) en el sitio de código abierto que incluye todos los pasos descritos anteriormente en un único archivo. Una vez que haya cargado todos los perfiles y parámetros de búsqueda necesarios, puede ejecutar la prueba de la instrucción de funcionalidad en Touchstone. Debe obtener una ejecución correcta:

:::image type="content" source="media/cms-tutorials/davinci-test-script-execution.png" alt-text="Ejecución del script de prueba de DaVinci.":::

## <a name="touchstone-query-test"></a>Prueba de consulta de Touchstone

La segunda prueba son las [funcionalidades de consulta](https://touchstone.aegis.net/touchstone/testdefinitions?selectedTestGrp=/FHIRSandbox/DaVinci/FHIR4-0-1-Test/PDEX/Formulary/01-Query&activeOnly=false&contentEntry=TEST_SCRIPTS). Esta prueba valida que puede buscar recursos y `CoveragePlan` específicos `Drug` mediante varios parámetros. La mejor ruta de acceso sería probar los recursos que ya tiene en la base de datos, pero también tenemos el archivo HTTP de [DaVinciFormulary_Sample_Resources](https://github.com/microsoft/fhir-server/blob/main/docs/rest/DaVinciFormulary/DaVinciFormulary_Sample_Resources.http) disponible con recursos de ejemplo que se extraían de los ejemplos de la IG que puede usar para crear los recursos y realizar pruebas.

:::image type="content" source="media/cms-tutorials/davinci-test-execution-results.png" alt-text="Resultados de la ejecución de pruebas de DaVinci.":::

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, se ha recorrido cómo pasar el formulario de intercambio de datos de pago de da payer de Estados Unidos en Touchstone. A continuación, puede aprender a probar la Guía de implementación de Da Dag PDex en Touchstone.

>[!div class="nextstepaction"]
>[DaIque PDex](davinci-pdex-tutorial.md)