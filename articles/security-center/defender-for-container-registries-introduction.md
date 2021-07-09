---
title: 'Azure Defender para registros de contenedor: ventajas y características'
description: Obtenga información sobre las ventajas y características de Azure Defender para registros de contenedor.
author: memildin
ms.author: memildin
ms.date: 04/07/2021
ms.topic: overview
ms.service: security-center
manager: rkarlin
ms.openlocfilehash: 282777a692b7b0fe54415ef09c2cf1879868dd3b
ms.sourcegitcommit: 67cdbe905eb67e969d7d0e211d87bc174b9b8dc0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111854439"
---
# <a name="introduction-to-azure-defender-for-container-registries"></a>Introducción a Azure Defender para registros de contenedor

Azure Container Registry (ACR) es un servicio de registro de Docker privado y administrado que almacena y administra las imágenes de contenedor de las implementaciones de Azure en un registro central. Se basa en el registro 2.0 de Docker de código abierto.

Para proteger todos los registros basados en Azure Resource Manager de su suscripción, habilite **Azure Defender para registros de contenedor** en el nivel de suscripción. A continuación, Security Center examinará las imágenes insertadas en el registro o importadas en el registro, así como las imágenes extraídas en los últimos 30 días. Esta característica se cobra por imagen.

[!INCLUDE [Defender for container registries availability info](../../includes/security-center-availability-defender-for-container-registries.md)]

## <a name="what-are-the-benefits-of-azure-defender-for-container-registries"></a>¿Cuáles son las ventajas de Azure Defender para registros de contenedor?

Security Center identifica registros de ACR basados Azure Resource Manager en la suscripción y brinda una función fluida de evaluación y administración de vulnerabilidades nativa de Azure para las imágenes del registro.

**Azure Defender para registros de contenedor** incluye un detector de vulnerabilidades para examinar las imágenes de los registros de Azure Container Registry basados en Azure Resource Manager y proporciona una mayor visibilidad de las vulnerabilidades de las imágenes. El detector integrado lo proporciona el proveedor de análisis de vulnerabilidades líder del sector, Qualys.

Si Qualys o Security Center encuentran incidencias, recibirá una notificación en el panel de Security Center. Por cada vulnerabilidad, Security Center proporciona recomendaciones útiles, junto con una clasificación de gravedad e información sobre cómo corregir el problema. Para más información sobre las recomendaciones de Security Center para los contenedores, consulte la [lista de recomendaciones de referencia](recommendations-reference.md#recs-container).

Security Center filtra y clasifica los resultados del análisis. Cuando una imagen es correcta, Security Center la marca como tal. Security Center solo genera recomendaciones de seguridad para las imágenes que tienen incidencias sin resolver. Security Center proporciona detalles de cada vulnerabilidad detectada y una clasificación de gravedad. Asimismo, ofrece instrucciones sobre cómo corregir las vulnerabilidades específicas encontradas en cada imagen.

Al enviar notificaciones solo cuando hay problemas, Security Center reduce las alertas informativas no deseadas.


> [!TIP]
> Para obtener más información sobre las características de seguridad de los contenedores de Security Center, consulte:
>
> - [Seguridad de los contenedores y Azure Security Center](container-security.md)
> - [Introducción a Azure Defender para Kubernetes](defender-for-kubernetes-introduction.md)

## <a name="when-are-images-scanned"></a>¿Cuándo se examinan las imágenes?

Hay tres desencadenadores para un examen de imagen:

- **Al insertar**: cada vez que se inserta una imagen en el registro, Security Center la examina de forma automática. Para desencadenar el examen de una imagen, insértela en el repositorio.

- **Extraído recientemente**: dado que se detectan nuevas vulnerabilidades cada día, **Azure Defender para registros de contenedor** también examina semanalmente cualquier imagen que se haya extraído en los últimos 30 días. No se aplican cargos adicionales por los nuevos exámenes; como se ha mencionado anteriormente, se le facturará una vez por imagen.

- **Al importar**: Azure Container Registry tiene herramientas de importación para incorporar imágenes al registro desde Docker Hub, Microsoft Container Registry u otro registro de contenedor de Azure. **Azure Defender para registros de contenedor** examina las imágenes admitidas que importe. Obtenga más información en [Importación de imágenes de contenedor en un registro de contenedor](../container-registry/container-registry-import-images.md).
 
El análisis se completa normalmente en 2 minutos, pero puede tardar hasta 15 minutos. Los resultados se publican como recomendaciones de Security Center, como en este caso:

[![Ejemplo de recomendación Azure Security Center sobre puntos vulnerables detectados en una imagen hospedada de Azure Container Registry (ACR)](media/azure-container-registry-integration/container-security-acr-page.png)](media/azure-container-registry-integration/container-security-acr-page.png#lightbox)


## <a name="how-does-security-center-work-with-azure-container-registry"></a>Funcionamiento de Security Center con Azure Container Registry

A continuación se muestra un diagrama detallado de los componentes y las ventajas de proteger los registros con Security Center.

![Introducción de alto nivel de Azure Security Center y Azure Container Registry (ACR)](./media/azure-container-registry-integration/aks-acr-integration-detailed.png)




## <a name="faq---azure-container-registry-image-scanning"></a>Preguntas frecuentes: Análisis de imágenes de Azure Container Registry

### <a name="how-does-security-center-scan-an-image"></a>¿De qué forma analiza Security Center las imágenes?
Security Center extrae la imagen del registro y la ejecuta en un espacio aislado con el detector de Qualys. El detector extrae una lista de vulnerabilidades conocidas.

Security Center filtra y clasifica los resultados del análisis. Cuando una imagen es correcta, Security Center la marca como tal. Security Center solo genera recomendaciones de seguridad para las imágenes que tienen incidencias sin resolver. Al enviar notificaciones solo cuando hay problemas, Security Center reduce las posibles alertas informativas no deseadas.

### <a name="can-i-get-the-scan-results-via-rest-api"></a>¿Se pueden obtener los resultados del examen mediante la API REST?
Sí. Los resultados se encuentran en [API REST de valoración secundaria](/rest/api/securitycenter/subassessments/list/). Asimismo, puede usar Azure Resource Graph (ARG), la API similar a Kusto para todos los recursos: una consulta puede recuperar un análisis específico.

### <a name="what-registry-types-are-scanned-what-types-are-billed"></a>¿Qué tipos de registros se analizan? ¿Qué tipos se facturan?
Para ver los tipos de registros de contenedor que admite Azure Defender para registros de contenedor, consulte [Disponibilidad](#availability).

Si conecta registros no admitidos a su suscripción de Azure, Azure Defender no los examinará y no se facturarán.

### <a name="can-i-customize-the-findings-from-the-vulnerability-scanner"></a>¿Puedo personalizar los resultados del detector de vulnerabilidades?
Sí. Si tiene una necesidad organizativa de omitir un resultado, en lugar de corregirlo, tiene la opción de deshabilitarlo. Los resultados deshabilitados no afectan a la puntuación de seguridad ni generan ruido no deseado.

[Aprenda a crear reglas para deshabilitar los resultados desde la herramienta integrada de valoración de vulnerabilidades](defender-for-container-registries-usage.md#disable-specific-findings-preview).

### <a name="why-is-security-center-alerting-me-to-vulnerabilities-about-an-image-that-isnt-in-my-registry"></a>¿Por qué me está avisando Security Center de las vulnerabilidades de una imagen que no está en mi registro?
Security Center proporciona evaluaciones de vulnerabilidades de cada imagen que se inserta o se extrae en un registro. Algunas imágenes pueden volver a usar etiquetas de una imagen que ya se ha examinado. Por ejemplo, puede reasignar la etiqueta "Más reciente" cada vez que se agrega una imagen a un código hash. En esos casos, la imagen "antigua" todavía existe en el registro y es posible que todavía se pueda extraer mediante su código hash. Si la imagen tiene resultados de seguridad y se extrae, mostrará vulnerabilidades de seguridad.

### <a name="what-should-i-do-if-the-scan-results-for-my-image-arent-shown"></a>¿Qué debo hacer si no se muestran los resultados del examen de la imagen?
Espere unos minutos y, si los resultados no aparecen, intente insertar la imagen de nuevo para desencadenar un nuevo examen. Si aún no aparecen, póngase en contacto con nuestro equipo de soporte técnico. 

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Detección de vulnerabilidades en sus imágenes](defender-for-container-registries-usage.md)
