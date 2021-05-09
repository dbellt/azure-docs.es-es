---
title: 'Preguntas más frecuentes sobre los servicios de FHIR en Azure: Azure API for FHIR'
description: Obtenga respuestas a las preguntas más frecuentes sobre Azure API for FHIR, como la ubicación de almacenamiento de datos detrás de las API de FHIR y la compatibilidad con versiones.
services: healthcare-apis
author: caitlinv39
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: reference
ms.date: 04/30/2021
ms.author: cavoeg
ms.openlocfilehash: 14328179a297ebcc3994bf627b1690195774b2b1
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108741138"
---
# <a name="frequently-asked-questions-about-the-azure-api-for-fhir"></a>Preguntas más frecuentes sobre Azure API for FHIR

## <a name="azure-api-for-fhir-the-basics"></a>Azure API for FHIR: Conceptos básicos

### <a name="what-is-fhir"></a>¿Qué es FHIR?
Recursos Rápidos de Interoperabilidad en Salud (FHIR, pronunciado como "fire" en inglés) es un estándar de interoperabilidad diseñado para permitir el intercambio de datos del sector sanitario entre diferentes sistemas sanitarios. Este estándar fue desarrollado por la organización HL7 y lo adoptan organizaciones del sector sanitario de todo el mundo. La versión más reciente de FHIR disponible es la R4 (versión 4). Azure API for FHIR admite la versión R4 y también es compatible con la versión anterior STU3 (Standard for Trial Use 3). Para más información sobre FHIR, visite [HL7.org](http://hl7.org/fhir/summary.html).

### <a name="is-the-data-behind-the-fhir-apis-stored-in-azure"></a>¿Están los datos detrás de las API de FHIR almacenados en Azure?

Sí, los datos se almacenan en bases de datos administradas en Azure. Azure API for FHIR no proporciona el acceso directo al almacén de datos subyacente.

### <a name="what-identity-provider-do-you-support"></a>¿Qué proveedor de identidades admite?

Actualmente se admite Microsoft Azure Active Directory como proveedor de identidades.

### <a name="what-is-the-recovery-point-objective-rpo-for-the-azure-api-for-fhir"></a>¿Cuál es el objetivo de punto de recuperación (RPO) de Azure API for FHIR?
El servicio Azure API for FHIR está respaldado por Cosmos DB como nuestro proveedor de persistencia. Por este motivo, el RPO del servicio es igual a [Cosmos DB (región única)](../../cosmos-db/consistency-levels.md) y <240 minutos.

### <a name="what-fhir-version-do-you-support"></a>¿Qué versión de FHIR admite?

Se admiten las versiones 4.0.0 y 3.0.1 tanto en Azure API for FHIR (PaaS) como en FHIR Server for Azure (código abierto).

Para obtener detalles, consulte las [características admitidas](fhir-features-supported.md). Lea sobre lo que ha cambiado entre las versiones de FHIR (es decir, de STU3 a R4) en el [historial de versiones de HL7 STU3](https://hl7.org/fhir/R4/history.html).

El Conector de Azure IoT para FHIR (versión preliminar) solo admite actualmente la versión R4 de FHIR y solo es visible en instancias de R4 de Azure API for FHIR.

### <a name="whats-the-difference-between-microsoft-fhir-server-for-azure-and-the-azure-api-for-fhir"></a>¿Cuál es la diferencia entre "Microsoft FHIR Server para Azure" y "Azure API for FHIR"?

Azure API for FHIR es una versión hospedada y administrada de Microsoft FHIR Server for Azure de código abierto. En el servicio administrado, Microsoft proporciona todo el mantenimiento y las actualizaciones. 

Al ejecutar FHIR Server para Azure, tiene acceso directo a los servicios subyacentes, pero también es responsable de mantener y actualizar el servidor y todo el trabajo de cumplimiento requerido si almacena datos de PHI.

Desde el punto de vista del desarrollo, todas las características que no se apliquen solo al servicio administrado se implementan primero en el software de código abierto Microsoft FHIR Server para Azure. Una vez que se haya validado en el código abierto, se lanzará en la solución Azure API for FHIR de PaaS. El tiempo entre el lanzamiento en el código abierto y PaaS depende de la complejidad de la característica y de otras prioridades del mapa de ruta. Este es el mismo proceso para todos nuestros servicios, como el Conector de Azure IoT para FHIR (versión preliminar).

### <a name="in-which-regions-is-azure-api-for-fhir-available"></a>¿En qué regiones está disponible Azure API for FHIR?

Actualmente, tenemos disponibilidad general tanto para el público como para la administración pública en [varias regiones geográficas](https://azure.microsoft.com/global-infrastructure/services/?products=azure-api-for-fhir&regions=non-regional,us-east,us-east-2,us-central,us-north-central,us-south-central,us-west-central,us-west,us-west-2,canada-east,canada-central,usgov-non-regional,us-dod-central,us-dod-east,usgov-arizona,usgov-texas,usgov-virginia). Para información sobre los servicios en la nube de la administración pública de Microsoft, consulte [Servicios de Azure de FedRAMP](../../azure-government/compliance/azure-services-in-fedramp-auditscope.md).

### <a name="where-can-i-see-what-is-releasing-into-the-azure-api-for-fhir"></a>¿Dónde puedo ver lo que se está publicando en Azure API for FHIR?

Para saber algo de lo que se publica en Azure API for FHIR, vea la [versión](https://github.com/microsoft/fhir-server/releases)del servidor FHIR de código abierto. Se ha trabajado para etiquetar los elementos de Azure API for FHIR si se van a publicar en el servicio administrado y suelen estar disponibles dos semanas después de aparecer en la página de versión en código abierto. También se han incluido instrucciones sobre cómo probar la compilación [aquí](https://github.com/microsoft/fhir-server/blob/master/docs/Testing-Releases.md) si se quiere probar en el entorno propio. Se está evaluando cómo compartir mejor las actualizaciones adicionales del servicio administrado.

### <a name="what-is-smart-on-fhir"></a>¿Qué es SMART on FHIR?

SMART (Substitutable Medical Applications and Reusable Technology) on FHIR es un conjunto de especificaciones de código abierto para integrar aplicaciones de asociados con servidores de FHIR y otros sistemas de TI sanitarios, como los registros sanitarios electrónicos y los intercambios de información sanitaria. Al crear una aplicación SMART on FHIR, puede garantizar que una gran cantidad de sistemas diferentes pueden acceder a la aplicación y aprovecharla.
Autenticación y Azure API for FHIR. Para obtener más información acerca de SMART, visite [SMART Health IT](https://smarthealthit.org/).

### <a name="where-can-i-find-what-version-of-fhir-is-running-on-my-database"></a>¿Dónde puedo encontrar qué versión de FHIR se está ejecutando en mi base de datos? 

Puede encontrar la versión exacta de FHIR expuesta en la instrucción de funcionalidad en la propiedad "fhirVersion".

## <a name="fhir-implementations-and-specifications"></a>Implementaciones y especificaciones de FHIR

### <a name="can-i-create-a-custom-fhir-resource"></a>¿Puedo crear un recurso personalizado de FHIR?

No se permiten recursos de FHIR personalizados. Si necesita un recurso FHIR personalizado, puede crear un recurso personalizado sobre el [recurso básico](http://www.hl7.org/fhir/basic.html) con extensiones. 

### <a name="are-extensions-supported-on-azure-api-for-fhir"></a>¿Se admiten las [extensiones](https://www.hl7.org/fhir/extensibility.html) en Azure API for FHIR?

Está permitido cargar datos JSON de FHIR válidos en el servidor. Si desea almacenar la definición de la estructura que define la extensión, puede guardarla como un recurso de definición de estructura. Actualmente, no se puede buscar en las extensiones.

### <a name="what-is-the-limit-on-_count"></a>¿Cuál es el límite en recuento?

El límite actual de _count es 1000. Si establece _count en más de 1000, recibirá una advertencia en la agrupación que indica que solo se mostrarán 1000 registros.

### <a name="are-there-any-limitations-on-the-group-export-functionality"></a>¿Hay alguna limitación en la funcionalidad de exportación de grupos?

Para la exportación de grupos, solo se exportan las referencias incluidas del grupo, no todas las características del [recurso de grupo](https://www.hl7.org/fhir/group.html).

### <a name="can-i-post-a-bundle-to-the-azure-api-for-fhir"></a>¿Puedo publicar un conjunto en Azure API for FHIR?

Actualmente se admite la publicación de [conjuntos por lotes](https://www.hl7.org/fhir/valueset-bundle-type.html), pero no la de conjuntos de transacciones, en Azure API for FHIR. Puede usar el servidor FHIR de código abierto respaldado por SQL para publicar conjuntos de transacciones.

### <a name="how-can-i-get-all-resources-for-a-single-patient-in-the-azure-api-for-fhir"></a>¿Cómo puedo obtener todos los recursos de un solo paciente en Azure API for FHIR?

En Azure API for FHIR se admite la [búsqueda de compartimientos](https://www.hl7.org/fhir/compartmentdefinition.html). Esto permite obtener todos los recursos relacionados con un paciente determinado. Tenga en cuenta que de momento un compartimiento incluye todos los recursos relacionados con el paciente, pero no al propio paciente, por lo que también hay que buscar para obtener el paciente si se necesita ese recurso en los resultados.

A continuación se muestran algunos ejemplos de eso:

* GET Patient/<id>/*
* GET Patient/<id>/Observation
* GET Patient/<id>/Observation?code=8302-2

### <a name="what-is-the-default-sort-when-searching-for-resources-in-azure-api-for-fhir"></a>¿Cuál es el orden predeterminado al buscar recursos en Azure API for FHIR?

Se admite la ordenación por la fecha de la última actualización: _sort=_lastUpdated. Para obtener más información sobre otros parámetros de búsqueda admitidos, vea [Información general de FHIR Search.](overview-of-search.md)

### <a name="does-the-azure-api-for-fhir-support-everything"></a>¿Azure API for FHIR admite $everything? 

No. En este momento no se admite $everything. Sin embargo, se puede lograr con dos llamadas API. Por ejemplo, para obtener Patient$everything, puede tomar primero el registro del paciente mediante /Patient/[ID] y, a continuación, una segunda llamada para recuperar todos los datos del paciente mediante /Patient/[ID]/*.

Puede ver más detalles en esta [publicación de la comunidad](https://chat.fhir.org/#narrow/stream/179166-implementers/topic/.24everything.20with.20_type). 

### <a name="how-does-export-work"></a>¿Cómo funciona $export?

$export forma parte de la especificación de FHIR: https://hl7.org/fhir/uv/bulkdata/export/index.html. Si el servicio FHIR se configura con una identidad administrada y una cuenta de almacenamiento, y si la identidad administrada tiene acceso a esa cuenta de almacenamiento, puede llamar simplemente a $export en la API de FHIR y todos los recursos de FHIR se exportarán a la cuenta de almacenamiento. Para más información, consulte el [artículo sobre $export](export-data.md).

### <a name="is-de-identified-export-available-at-patient-and-group-level-as-well"></a>¿La exportación sin identificación también está disponible en el nivel de paciente y grupo?
La exportación anonimizada solo se admite actualmente en caso de exportación completa del sistema (/$export), no de exportación de pacientes (/Patient/$export). Se está trabajando para que también esté disponible en el nivel de paciente.

## <a name="using-azure-api-for-fhir"></a>Uso de Azure API for FHIR

### <a name="how-do-i-enable-log-analytics-for-azure-api-for-fhir"></a>¿Cómo habilitar Log Analytics para Azure API for FHIR?

Se habilita el registro de diagnóstico y se permite la revisión de consultas de ejemplo para estos registros. Para más información sobre cómo habilitar los registros de auditoría y las consultas de ejemplo, consulte [esta sección](enable-diagnostic-logging.md). Si quiere incluir información adicional en los registros, consulte [Uso de encabezados HTTP personalizados](use-custom-headers.md).

### <a name="where-can-i-see-some-examples-of-using-the-azure-api-for-fhir-within-a-workflow"></a>¿Dónde puedo ver algunos ejemplos de uso de Azure API for FHIR dentro de un flujo de trabajo?

Hay una colección de arquitecturas de referencia disponible en la página de [GitHub de arquitectura sanitaria](https://github.com/microsoft/health-architectures).

### <a name="where-can-i-see-an-example-of-connecting-a-web-application-to-azure-api-for-fhir"></a>¿Dónde puedo ver un ejemplo de conexión de una aplicación web a Azure API for FHIR?

Tenemos una [página de GitHub de la arquitectura de estado](https://aka.ms/health-architectures) que contiene aplicaciones y escenarios de ejemplo. En ella se muestra cómo conectar una aplicación web a Azure API for FHIR.  

## <a name="azure-api-for-fhir-features-and-services"></a>Características y servicios de Azure API for FHIR 

### <a name="is-there-a-way-to-encrypt-my-data-using-my-personal-key-not-a-default-key"></a>¿Hay alguna manera de cifrar los datos con mi clave personal y no con una clave predeterminada?

Sí, Azure API for FHIR permite configurar claves administradas por el cliente aprovechando la compatibilidad de Cosmos DB. Para más información sobre cómo cifrar los datos con una clave personal, consulte [esta sección](customer-managed-key.md).

## <a name="azure-api-for-fhir-preview-features"></a>Azure API for FHIR: Características en vista previa

### <a name="can-i-configure-scaling-capacity-for-azure-iot-connector-for-fhir-preview"></a>¿Puedo configurar la capacidad de escalado para el conector de Azure IoT para FHIR (versión preliminar)?

Dado que el conector de Azure IoT para FHIR es gratuito durante la versión preliminar pública, su capacidad de escalado es fija y limitada. Se espera que la configuración del conector de Azure IoT para FHIR disponible en la versión preliminar pública proporcione un rendimiento de aproximadamente 200 mensajes por segundo. En la versión de disponibilidad general (GA) habrá una manera de configurar la capacidad de los recursos.

### <a name="why-cant-i-install-azure-iot-connector-for-fhir-preview-when-private-link-is-enabled-on-azure-api-for-fhir"></a>¿Por qué no se puede instalar el conector de Azure IoT para FHIR (versión preliminar) cuando está habilitado Private Link en Azure API for FHIR?

El conector de Azure IoT para FHIR no admite la funcionalidad Private Link en este momento. Por lo tanto, si tiene Private Link habilitado en Azure API for FHIR, no podrá instalar el conector de Azure IoT para FHIR y viceversa. Se espera que esta limitación desaparezca cuando el conector de Azure IoT para FHIR esté en versión de disponibilidad general (GA).
