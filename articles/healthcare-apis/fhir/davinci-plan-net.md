---
title: 'Tutorial: DaVersiones Plan Net - Azure API for FHIR'
description: Este tutorial le guía a través de la configuración de la Azure API for FHIR para pasar pruebas de Touchstone para la Guía de implementación de Da Dag Payer Data Exchange.
services: healthcare-apis
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: tutorial
ms.reviewer: matjazl
ms.author: cavoeg
author: modillon
ms.date: 06/25/2021
ms.openlocfilehash: bb7b7e3813a317aa1b1a9f97ab4f80650eb2fd1e
ms.sourcegitcommit: 8b7d16fefcf3d024a72119b233733cb3e962d6d9
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 07/16/2021
ms.locfileid: "114285895"
---
# <a name="da-vinci-plan-net"></a>Da Vinci Plan Net

En este tutorial, le guiaremos por la configuración de la Azure API for FHIR para pasar las pruebas de [Touchstone](https://touchstone.aegis.net/touchstone/) para la Guía de implementación de DaDex PDEX Payer Network (Plan-Net).

## <a name="touchstone-capability-statement"></a>Instrucción de funcionalidad de Touchstone

La primera prueba en la que nos centraremos es probar el Azure API for FHIR con la instrucción [de funcionalidad de Plan-Net da da.](https://touchstone.aegis.net/touchstone/testdefinitions?selectedTestGrp=/FHIRSandbox/DaVinci/FHIR4-0-1-Test/PDEX/PlanNet/00-Capability&activeOnly=false&contentEntry=TEST_SCRIPTS) Si ejecuta esta prueba sin ninguna actualización, se producirá un error en la prueba debido a la falta de parámetros de búsqueda y a la falta de perfiles.

## <a name="define-search-parameters"></a>Definición de parámetros de búsqueda

Como parte de Da York Plan-Net IG, deberá definir [](./how-to-do-custom-search.md) seis nuevos parámetros de búsqueda para los recursos Healthcare Service, Insurance Plan, Practitioner Role, Organization y Organization Affiliation. Los seis se prueban en la instrucción de funcionalidad:

* [Área de cobertura del servicio sanitario](http://hl7.org/fhir/us/davinci-pdex-plan-net/STU1/SearchParameter-healthcareservice-coverage-area.html)
* [Área de cobertura del plan de seguros](http://hl7.org/fhir/us/davinci-pdex-plan-net/STU1/SearchParameter-insuranceplan-coverage-area.html)
* [Tipo de plan de plan de seguros](http://hl7.org/fhir/us/davinci-pdex-plan-net/STU1/SearchParameter-insuranceplan-plan-type.html)
* [Área de cobertura de la organización](http://hl7.org/fhir/us/davinci-pdex-plan-net/STU1/SearchParameter-organization-coverage-area.html)
* [Red de afiliación de la organización](http://hl7.org/fhir/us/davinci-pdex-plan-net/STU1/SearchParameter-organizationaffiliation-network.html)
* [Red de rol profesional](http://hl7.org/fhir/us/davinci-pdex-plan-net/STU1/SearchParameter-practitionerrole-network.html)

> [!NOTE]
> En el código JSON sin formato para estos parámetros de búsqueda, el nombre se establece en `Plannet_sp_<Resource Name>_<SearchParameter Name>` . La prueba de Touchstone espera que el nombre de estos sea solo `SearchParameter Name` el (área de cobertura, tipo de plan o red).

El resto de los parámetros de búsqueda necesarios para la Plan-Net IG de Da Dag se definen mediante la especificación base y ya están disponibles en el Azure API for FHIR sin ninguna actualización adicional.

## <a name="store-profiles"></a>Almacenamiento de perfiles

Fuera de definir parámetros de búsqueda, debe cargar los [perfiles y](./validation-against-profiles.md#storing-profiles) extensiones necesarios para superar esta prueba. Hay nueve perfiles que se usan como parte de la Plan-Net DA:

* [Punto de conexión de Plan-Net](http://hl7.org/fhir/us/davinci-pdex-plan-net/STU1/StructureDefinition-plannet-Endpoint.html)
* [Plan-Net Healthcare Service](http://hl7.org/fhir/us/davinci-pdex-plan-net/STU1/StructureDefinition-plannet-HealthcareService.html)
* [Plan de seguros de Plan-Net](http://hl7.org/fhir/us/davinci-pdex-plan-net/STU1/StructureDefinition-plannet-InsurancePlan.html) 
* [Ubicación de Plan-Net](http://hl7.org/fhir/us/davinci-pdex-plan-net/STU1/StructureDefinition-plannet-Location.html)
* [Plan-Net Network](http://hl7.org/fhir/us/davinci-pdex-plan-net/STU1/StructureDefinition-plannet-Network.html)
* [Organización de Plan-Net](http://hl7.org/fhir/us/davinci-pdex-plan-net/STU1/StructureDefinition-plannet-Organization.html)
* [Organización de Plan-NetAffiliation](http://hl7.org/fhir/us/davinci-pdex-plan-net/STU1/StructureDefinition-plannet-OrganizationAffiliation.html)
* [Profesional de Plan-Net](http://hl7.org/fhir/us/davinci-pdex-plan-net/STU1/StructureDefinition-plannet-Practitioner.html)
* [Plan-Net PractitionerRole](http://hl7.org/fhir/us/davinci-pdex-plan-net/STU1/StructureDefinition-plannet-PractitionerRole.html)

## <a name="sample-rest-file"></a>Archivo de rest de ejemplo

Para ayudar con la creación de estos perfiles y parámetros de búsqueda, tenemos un archivo HTTP de ejemplo en el sitio de código abierto que incluye todos los pasos descritos anteriormente en un único archivo. Una vez que haya cargado todos los perfiles y parámetros de búsqueda necesarios, puede ejecutar la prueba de la instrucción de funcionalidad en Touchstone.

:::image type="content" source="media/davinci-plan-net/davinci-plan-net-test-script-execution-passed.png" alt-text="Se ha superado el script de ejecución de pruebas de rest de ejemplo net del plan Da Dag.":::

## <a name="touchstone-error-handling-test"></a>Prueba de control de errores de Touchstone

La segunda prueba que se recorrerá es probar el [control de errores.](https://touchstone.aegis.net/touchstone/testdefinitions?selectedTestGrp=/FHIRSandbox/DaVinci/FHIR4-0-1-Test/PDEX/PlanNet/01-Error-Codes&activeOnly=false&contentEntry=TEST_SCRIPTS) El único paso que debe realizar es eliminar un recurso HealthcareService de la base de datos y usar el identificador del recurso HealthcareService eliminado en la prueba. El ejemplo [DaVinci_PlanNet.http](https://github.com/microsoft/fhir-server/blob/main/docs/rest/DaVinciPlanNet/DaVinci_PlanNet.http) en el sitio de código abierto proporciona un ejemplo de HealthcareService para publicar y eliminar en este paso.

:::image type="content" source="media/davinci-plan-net/davinci-test-script-execution-passed.png" alt-text="Se ha superado el script de ejecución de pruebas de error de la red de touchstone del plan Da Dag":::

## <a name="touchstone-query-test"></a>Prueba de consulta de Touchstone

La siguiente prueba que veremos es la prueba de [funcionalidades de consulta](https://touchstone.aegis.net/touchstone/testdefinitions?selectedTestGrp=/FHIRSandbox/DaVinci/FHIR4-0-1-Test/PDEX/PlanNet/03-Query&activeOnly=false&contentEntry=TEST_SCRIPTS). Esta prueba está probando la conformidad con los perfiles que cargó en la primera prueba. Deberá tener recursos cargados que se ajusten a los perfiles. La mejor ruta de acceso sería probar los recursos que ya tiene en la base de datos, pero también tenemos el archivo [DaVinci_PlanNet_Sample_Resources.http](https://github.com/microsoft/fhir-server/blob/main/docs/rest/DaVinciPlanNet/DaVinci_PlanNet_Sample_Resources.http) con recursos de ejemplo que se extraían de los ejemplos de la IG que puede usar para crear los recursos y realizar pruebas.  

:::image type="content" source="media/davinci-plan-net/touchstone-query-test-execution-failed.png" alt-text="Error en la prueba de consulta neta del plan Da Dag":::

> [!NOTE]
> Con los recursos de ejemplo proporcionados, debería esperar una tasa de éxito del 98 % de las pruebas de consulta:

> * Hay un problema de GitHub abierto en el servidor FHIR que está causando un error en una de estas pruebas: Recurso devuelto varias veces si cumple los criterios base y _include [· Problema #2037 · microsoft/fhir-server (github.com)](https://github.com/microsoft/fhir-server/issues/2037)

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, hemos recorrido la configuración de la Azure API for FHIR para pasar las pruebas de Touchstone para la Guía de implementación de DaDex PDEX Payer Network (Plan-Net). A continuación, puede obtener información sobre todas las Azure API for FHIR características.

>[!div class="nextstepaction"]
>[Características compatibles](fhir-features-supported.md)