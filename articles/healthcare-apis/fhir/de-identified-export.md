---
title: Exportación de datos sin identificación (versión preliminar) para Azure API for FHIR
description: En este artículo se indica cómo configurar y usar la exportación sin identificación.
author: ranvijaykumar
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: reference
ms.date: 9/28/2020
ms.author: ranku
ms.openlocfilehash: ba0af51eaf15211c2214f9457235afa250d92b7b
ms.sourcegitcommit: a434cfeee5f4ed01d6df897d01e569e213ad1e6f
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111814452"
---
# <a name="exporting-de-identified-data-preview"></a>Exportación de datos sin identificación (versión preliminar)

> [!Note] 
> Los resultados cuando se usa la exportación sin identificación variarán en función de factores como los datos introducidos y las funciones seleccionadas por el cliente. Microsoft no puede evaluar las salidas de exportación sin identificación ni determinar la aceptabilidad para los casos de uso del cliente y las necesidades de cumplimiento. No se garantiza que la exportación sin identificación cumpla los requisitos legales, normativos o de cumplimiento específicos.

También se puede usar el comando $export para exportar datos sin identificación desde el servidor de FHIR. Usa el motor de anonimización de [Herramientas de FHIR para la anonimización](https://github.com/microsoft/FHIR-Tools-for-Anonymization) y toma los detalles de configuración de anonimización en los parámetros de la consulta. Puede crear su propio archivo de configuración de anonimización o usar el [archivo de configuración de ejemplo](https://github.com/microsoft/FHIR-Tools-for-Anonymization#sample-configuration-file-for-hipaa-safe-harbor-method) para el método Safe Harbor de HIPAA como punto de partida. 

 `https://<<FHIR service base URL>>/$export?_container=<<container_name>>&_anonymizationConfig=<<config file name>>&_anonymizationConfigEtag=<<ETag on storage>>`

> [!Note] 
> En este momento, Azure API for FHIR solo admite la exportación sin identificación en el nivel de sistema ($export).

|Parámetro de consulta            | Ejemplo |Opcionalidad| Descripción|
|---------------------------|---------|-----------|------------|
| _\_anonymizationConfig_   |DemoConfig.json|Obligatorio para la exportación sin identificación |Nombre del archivo de configuración. Consulte el formato del archivo de configuración [aquí](https://github.com/microsoft/FHIR-Tools-for-Anonymization#configuration-file-format). Este archivo debe mantenerse dentro de un contenedor llamado **anonymization** dentro de la misma cuenta de almacenamiento de Azure que está configurada como ubicación de exportación. |
| _\_anonymizationConfigEtag_|"0x8D8494A069489EC"|Opcional para la exportación sin identificación|Este es el valor de Etag del archivo de configuración. Puede obtener el valor de Etag mediante Explorador de Azure Storage desde la propiedad del blob.|

> [!IMPORTANT]
> Tanto la exportación sin formato como la exportación sin identificación escriben en la misma cuenta de almacenamiento de Azure especificada como parte de la configuración de la exportación. Se recomienda que use distintos contenedores que se correspondan con las diferentes configuraciones sin identificación y que administren el acceso de usuarios en el nivel de contenedor.
