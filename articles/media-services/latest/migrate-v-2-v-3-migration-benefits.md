---
title: Las ventajas de migrar a la API de Media Services v3
description: En este artículo se enumeran las ventajas de la migración de Media Services v2 a v3.
services: media-services
author: IngridAtMicrosoft
manager: femila
ms.service: media-services
ms.topic: conceptual
ms.workload: media
ms.date: 03/25/2021
ms.author: inhenkel
ms.openlocfilehash: 9dd3525f4efec3c49950839306ee5419c7850c69
ms.sourcegitcommit: 02bc06155692213ef031f049f5dcf4c418e9f509
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/03/2021
ms.locfileid: "106275420"
---
# <a name="step-1---understand-the-benefits-of-migrating-to-media-services-api-v3"></a>Paso 1: Entender las ventajas de migrar a la API de Media Services V3

![logotipo de la guía de migración](./media/migration-guide/azure-media-services-logo-migration-guide.svg)

<hr color="#5ea0ef" size="10">

![pasos de migración 2](./media/migration-guide/steps-1.svg)

## <a name="use-the-latest-api"></a>Uso de la API más reciente

Le recomendamos que empiece a usar la versión 2020-05-01 de la API de Azure Media Services V3 ahora para obtener las ventajas porque las nuevas características, funcionalidad y optimizaciones de rendimiento solo están disponibles en la API V3 actual.

Puede cambiar la versión de la API en el portal, los SDK más recientes, la CLI más reciente y la API de REST con la cadena de versión correcta.

Se han realizado mejoras significativas en Media Services con V3.  

## <a name="benefits-of-media-services-v3"></a>Ventajas de Media Services v3

| **V3 (característica)** | **Prestación** |
| --- | --- |
| **Azure Portal** | |
| Actualizaciones de Azure Portal | Azure Portal se ha actualizado para incluir la administración de las entidades de la API V3. Permite a los clientes usar el portal para iniciar streaming en vivo, enviar trabajos de transformación de V3, administrar directivas de protección de contenido, puntos de conexión de streaming, obtener acceso de API, administrar cuentas de almacenamiento vinculadas y realizar tareas de supervisión. |
| **Cuentas y almacenamiento** | |
| Control de acceso basado en rol (RBAC) de Azure | Los clientes ahora pueden definir sus propios roles y controlar el acceso a cada entidad de la API de ARM de Media Services. Esto ayuda a controlar el acceso a los recursos por parte de las cuentas de AAD. |
| Identidades administradas | Las identidades administradas eliminan la necesidad de que los desarrolladores administren credenciales, al proporcionar una identidad para el recurso de Azure en Azure AD. Consulte los detalles sobre las identidades administradas [aquí](../../active-directory/managed-identities-azure-resources/overview.md). |
| Compatibilidad con vínculos privados | Los clientes tendrán acceso a los puntos de conexión de Media Services para la entrega de claves, LiveEvents y StreamingEndpoints a través de un elemento PrivateEndpoint en su red virtual. |
| Compatibilidad con las [claves administradas por el cliente](concept-use-customer-managed-keys-byok.md) o Bring Your Own Key (BYOK) | Los clientes pueden cifrar los datos en su cuenta de Media Services mediante una clave en su instancia de Azure Key Vault. |
| **Recursos** | |
| Un recurso puede tener varios [localizadores de streaming](stream-streaming-locators-concept.md) con diferentes configuraciones de [empaquetado dinámico](encode-dynamic-packaging-concept.md) y cifrado dinámico. | Hay un límite de 100 localizadores de streaming permitidos en cada recurso. Los clientes pueden almacenar una sola copia del contenido multimedia en el recurso, pero compartir diversas direcciones URL de streaming con diferentes directivas de streaming o directivas de protección de contenido basadas en una audiencia de destino.
| **Procesamiento de trabajos** ||
| V3 presenta el concepto de  [transformaciones](transform-jobs-concept.md)  para el procesamiento de trabajos basados en archivos. | Una transformación puede utilizarse para crear configuraciones reutilizables, crear plantillas de Azure Resource Manager y aislar los valores de procesamiento entre varios clientes o inquilinos. |
| Para el procesamiento de trabajos basados en archivos, puede usar una dirección URL de HTTP(S) como entrada. | No es necesario tener contenido ya almacenado en Azure, ni es necesario crear recursos de entrada. |
| **Eventos en directo** ||
| Objetos LiveEvent 1080p prémium | La nueva SKU de LiveEvent permite a los clientes obtener codificación en la nube con una salida de hasta 1080p en la resolución. |
| Nueva compatibilidad de streaming en vivo de [baja latencia](live-event-latency-reference.md) en objetos LiveEvent. | Esto permite a los usuarios ver eventos en directo más próximos a tiempo real que si no tenían esta opción habilitada. |
| La versión preliminar de LiveEvent admite  [empaquetado dinámico](encode-dynamic-packaging-concept.md)  y cifrado dinámico. | Esto habilita la protección de contenido en la versión preliminar, así como el empaquetado DASH y HLS. |
| Los objetos LiveOutput reemplazan entidades Program | La salida en directo es más fácil de usar que la entidad Program de las API v2. |
| Se ha mejorado la ingesta de RTMP para eventos en directo, con compatibilidad con más codificadores | Aumenta la estabilidad y proporciona flexibilidad del codificador de origen. |
| Los objetos LiveEvent pueden transmitirse ininterrumpidamente | Puede hospedar un objeto LiveEvent y mantener a la audiencia alerta durante períodos más largos. |
| Transcripción en directo en objetos LiveEvent | La transcripción en directo permite a los clientes transcribir automáticamente el idioma hablado en texto en tiempo real durante la difusión de eventos en directo. Esto mejora significativamente la accesibilidad de los eventos en directo. |
| [Modo de espera](live-event-outputs-concept.md#standby-mode) en objetos LiveEvent | Los eventos en directo que están en estado de espera son menos costosos que la ejecución de eventos en directo. Esto permite a los clientes mantener un conjunto de eventos en directo que están preparados para iniciarse en cuestión de segundos a un costo más bajo que el mantenimiento de un conjunto de eventos en directo en ejecución. Los precios reducidos de los eventos en directo en espera entrarán en vigor en febrero de 2021 en la mayoría de las regiones, mientras que el resto lo hará en abril de 2021.
|**Protección de contenido** ||
| La [protección de contenido](drm-content-key-policy-concept.md)  es compatible con características de varias claves. | Los clientes ahora pueden usar varias claves de cifrado de contenido en sus localizadores de streaming. |
| **Supervisión** | |
| Compatibilidad con las notificaciones de[Azure EventGrid](monitoring/reacting-to-media-services-events.md) | Las notificaciones de EventGrid cuentan con un mayor número de características. Hay más tipos de notificaciones, compatibilidad con SDK más amplia para recibir las notificaciones en su propia aplicación y más servicios de Azure existentes que pueden actuar como controladores de eventos. |
| [Soporte técnico de Azure Monitor e integración en Azure Portal](monitoring/monitor-events-portal-how-to.md) | Esto permite a los clientes visualizar el uso de cuota de cuenta de Media Services, las estadísticas en tiempo real de los puntos de conexión de streaming y la ingesta y el archivado de estadísticas para eventos en directo. Los clientes ahora pueden establecer alertas y realizar las acciones necesarias en función de los datos de métricas en tiempo real. |
