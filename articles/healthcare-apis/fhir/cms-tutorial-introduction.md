---
title: 'Tutorial: Introducción a centers for Centers for Centers for Centers and Centers (CMS) -Azure API for FHIR'
description: En esta introducción se presenta una serie de tutoriales que pertenecen a la regla de interoperabilidad y acceso a pacientes de Center for Domain Services (CMS).
services: healthcare-apis
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: tutorial
ms.reviewer: matjazl
ms.author: cavoeg
author: caitlinv39
ms.date: 05/27/2021
ms.openlocfilehash: 837fdb74cbbeb976641b9ee86970d87b80134bc9
ms.sourcegitcommit: ff1aa951f5d81381811246ac2380bcddc7e0c2b0
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/07/2021
ms.locfileid: "111572108"
---
# <a name="centers-for-medicare-and-medicaid-services-cms-interoperability-and-patient-access-rule-introduction"></a>Introducción a la regla de interoperabilidad y acceso a pacientes de Centers for Centers and Centers (CMS)

En esta serie de tutoriales, se trata un resumen de alto nivel de la regla de interoperabilidad y acceso a los pacientes del Centro para la interoperabilidad y los servicios de Technical Services (CMS), así como los requisitos técnicos descritos en esta regla. Se le guiará por las distintas guías de implementación a las que se hace referencia para esta regla. También se proporcionarán detalles sobre cómo configurar el Azure API for FHIR para admitir estas guías de implementación.


## <a name="rule-overview"></a>Introducción a las reglas

CMS publicó la [regla de interoperabilidad](https://www.cms.gov/Regulations-and-Guidance/Guidance/Interoperability/index) y acceso a pacientes el 1 de mayo de 2020. Esta regla requiere un flujo de datos gratuito y seguro entre todas las partes implicadas en la atención a los pacientes (pacientes, proveedores y pagadores) para permitir que los pacientes accedan a su información sanitaria cuando la necesiten. La interoperabilidad ha asolado el sector sanitario durante décadas, lo que ha dado lugar a datos almacenados en silos que provocan resultados sanitarios negativos con costos más altos e imprevisibles para la atención. CMS usa su autoridad para regular a los emisores del Programa de seguros de salud para niños (CHIP) y el Plan de salud calificado (QHP) en los intercambios facilitados por el gobierno federal (FFE) para aplicar esta regla. 

En agosto de 2020, CMS detalló cómo las organizaciones pueden cumplir el requisito. Para asegurarse de que los datos se pueden intercambiar de forma segura y estandarizada, CMS identificó la versión 4 (R4) de FHIR como el estándar fundamental necesario para el intercambio de datos. 

Hay tres partes principales para la interoperabilidad y el acceso a los pacientes:

* API de acceso de pacientes (obligatorio el 1 de julio de **2021):** los pagadores regulados por CMS (tal y como se definió anteriormente) deben implementar y mantener una API segura basada en estándares que permita a los pacientes acceder fácilmente a sus notificaciones y encontrar información, incluido el costo, así como un subconjunto definido de su información clínica a través de aplicaciones de terceros de su elección.  

* API de directorio del proveedor (obligatorio el 1 de julio de **2021):** esta parte de la regla requiere a los pagadores regulados por CMS para que la información del directorio del proveedor esté disponible públicamente a través de una API basada en estándares. Al hacer que esta información esté disponible, los desarrolladores de aplicaciones de terceros podrán crear servicios que ayuden a los pacientes a encontrar proveedores para necesidades de atención específicas y a los médicos a buscar otros proveedores para la coordinación asistencial.  

* Intercambio de datos de pago a pago (obligatorio el 1 de enero de **2022):** los pagadores regulados por CMS deben intercambiar determinados datos clínicos de pacientes a petición del paciente con otros pagadores. Aunque no es necesario seguir ningún tipo de estándar, se recomienda aplicar FHIR para intercambiar estos datos. 

## <a name="key-fhir-concepts"></a>Conceptos clave de FHIR

Como se mencionó anteriormente, FHIR R4 es necesario para cumplir este requisito. Además, se han desarrollado varias guías de implementación que proporcionan instrucciones para la regla. [Las guías de](https://www.hl7.org/fhir/implementationguide.html) implementación proporcionan contexto adicional sobre la especificación base de FHIR. Esto incluye la definición de parámetros de búsqueda adicionales, perfiles, extensiones, operaciones, conjuntos de valores y sistemas de código.

El Azure API for FHIR tiene las siguientes funcionalidades para ayudarle a configurar la base de datos para las distintas guías de implementación:

* [Compatibilidad con interacciones RESTful](fhir-features-supported.md)
* [Almacenamiento y validación de perfiles](validation-against-profiles.md)
* [Definición e indexación de parámetros de búsqueda personalizados](how-to-do-custom-search.md)
* [Convertir datos](convert-data.md)

## <a name="patient-access-api-implementation-guides"></a>Guías de implementación de API de acceso de pacientes

Patient Access API describe el cumplimiento de cuatro guías de implementación de FHIR.

### <a name="carin-ig-for-blue-button"></a>[CARIN IG para el botón azul®](http://hl7.org/fhir/us/carin-bb/STU1/index.html):

Los pagadores deben hacer las reclamaciones de los pacientes y encuentran datos disponibles según la Guía de implementación del botón azul (C4BB IG) de CARIN IG. La IG de C4BB proporciona un conjunto de recursos que los pagadores pueden mostrar a los consumidores a través de una API de FHIR e incluye los detalles necesarios para los datos de notificaciones en Interoperability and Patient Access API. En esta guía de implementación se usa el recurso ExplanationOfBenefit (EOB) como recurso principal y se extraen otros recursos a medida que se hace referencia a ellos.

### <a name="hl7-fhir-da-vinci-pdex-ig"></a>[HL7 FHIR DaIque PDex IG](http://hl7.org/fhir/us/davinci-pdex/STU1/index.html)

La Guía de implementación del intercambio de datos de pago (PDex IG) se centra en garantizar que los pagadores proporcionen todos los datos clínicos de pacientes pertinentes para cumplir los requisitos de la API de acceso de pacientes. Esto usa los perfiles de US Core en recursos R4 e incluye (como mínimo) encuentros, proveedores, organizaciones, ubicaciones, fechas de servicio, diagnósticos, procedimientos y observaciones. Aunque estos datos pueden estar disponibles en formato FHIR, también pueden proceden de otros sistemas con el formato de datos de notificaciones, mensajes HL7 V2 y documentos C-CDA.

### <a name="hl7-us-core-ig"></a>[HL7 US Core IG](https://www.hl7.org/fhir/us/core/toc.html)

La guía de implementación de HL7 US Core (US Core IG) es la red troncal de la IG de PDex descrita anteriormente. Aunque la IG de PDex limita algunos recursos incluso más allá de la IG básica de EE. UU., muchos recursos solo siguen los estándares de LA IG de US Core.

### <a name="hl7-fhir-da-vinci---pdex-us-drug-formulary-ig"></a>[HL7 FHIR Da Dag - PDex US Health Formulary IG](http://hl7.org/fhir/us/Davinci-drug-formulary/index.html)

Los planes de advantage de la parte D tienen que hacer que la información del formulario esté disponible a través de patient API. Para ello, usan la Guía de implementación de la fórmula de la receta de ESTADOS UNIDOS (USDF IG). La IG de USDF define una interfaz FHIR para la información de los formularios de salud de una médica, que es una lista de productos genéricos y de nombre de marca que una médica está de acuerdo en pagar. El caso de uso principal de esto es para que los pacientes puedan comprender si hay alternativas a una que se les haya prescrito y para comparar los costos de los tratamientos.

## <a name="provider-directory-api-implementation-guide"></a>Guía de implementación de API de directorio de proveedor

La API de directorio del proveedor describe el cumplimiento de una guía de implementación.

### <a name="hl7-da-vinci-pdex-plan-network-ig"></a>[HL7 DaIque PDex Plan Network IG](http://build.fhir.org/ig/HL7/davinci-pdex-plan-net/)

En esta guía de implementación se define una interfaz de FHIR para los planes de seguros de una empresa sanitaria, sus redes asociadas y las organizaciones y proveedores que participan en estas redes.

## <a name="touchstone"></a>Touchstone

Para probar el cumplimiento de las distintas guías de implementación, [Touchstone](https://touchstone.aegis.net/touchstone/) es un excelente recurso. A lo largo de los próximos tutoriales, nos centraremos en garantizar que el Azure API for FHIR esté configurado para pasar correctamente varias pruebas de Touchstone. El sitio de Touchstone tiene una gran documentación que le ayudará a empezar a funcionar.

## <a name="next-steps"></a>Pasos siguientes

Ahora que tiene una comprensión básica de la regla de interoperabilidad y acceso a pacientes, las guías de implementación y la herramienta de pruebas disponibles (Touchstone), le guiará por la configuración del Azure API for FHIR para carin IG para el botón azul. 

>[!div class="nextstepaction"]
>[Guía de implementación de CARIN para el botón azul](carin-implementation-guide-blue-button-tutorial.md)  