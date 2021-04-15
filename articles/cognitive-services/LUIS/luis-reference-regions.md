---
title: 'Regiones de publicación y puntos de conexión: LUIS'
description: La región especificada en Azure Portal es la misma donde se publicará la aplicación LUIS y se generará una dirección URL de punto de conexión para esta misma región.
ms.service: cognitive-services
ms.subservice: language-understanding
author: aahill
ms.author: aahi
ms.topic: reference
ms.date: 04/07/2021
ms.custom: references_regions
ms.openlocfilehash: fd02bf5c3291569b71416392b651967e0da701d9
ms.sourcegitcommit: b28e9f4d34abcb6f5ccbf112206926d5434bd0da
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/09/2021
ms.locfileid: "107226674"
---
# <a name="authoring-and-publishing-regions-and-the-associated-keys"></a>Creación y publicación de regiones y las claves asociadas

El portal de LUIS admite las regiones de creación de LUIS. Para publicar una aplicación de LUIS en más de una región, necesitará al menos una clave por región.

<a name="luis-website"></a>

## <a name="luis-authoring-regions"></a>Regiones de creación de LUIS

[!INCLUDE [portal consolidation](includes/portal-consolidation.md)]

LUIS tiene un portal que se puede usar independientemente de la región, [www.luis.ai](https://www.luis.ai). Aún debe crear y publicar el contenido en la misma región.

Las regiones de creación tienen [regiones de conmutación por error emparejadas](../../best-practices-availability-paired-regions.md).

<a name="regions-and-azure-resources"></a>

## <a name="publishing-regions-and-azure-resources"></a>Regiones de publicación y recursos de Azure

La aplicación se publica en todas las regiones asociadas a los recursos de LUIS agregados en el portal de LUIS. Por ejemplo, para una aplicación creada en [www.luis.ai][www.luis.ai], si crea un recurso de LUIS o de Cognitive Service en **westus** y lo [agrega a la aplicación como recurso](luis-how-to-azure-subscription.md), la aplicación se publica en dicha región.

## <a name="public-apps"></a>Aplicaciones públicas
Se publica una aplicación pública en todas las regiones para que un usuario con una clave de recurso de LUIS basada en regiones pueda tener acceso a la aplicación en cualquier región que esté asociado a su clave de recurso.

<a name="publishing-regions"></a>

## <a name="publishing-regions-are-tied-to-authoring-regions"></a>Las regiones de publicación están asociadas a las regiones de creación

La aplicación de regiones de creación solo se puede publicar en la región de publicación correspondiente. Si la aplicación está en la región de creación incorrecta, expórtela e impórtela en la región de creación correcta para su región de publicación.

> [!NOTE]
> Las aplicaciones de LUIS creadas en https://www.luis.ai ahora se pueden publicar en todos los puntos de conexión, incluidas las regiones de [Europa](#publishing-to-europe) y [Australia](#publishing-to-australia).

## <a name="publishing-to-europe"></a>Publicación en Europa

 Región global | Región de la API de creación | Región de publicación y de consulta<br>`API region name`   |  Formato de dirección URL de punto de conexión   |
|-----|------|------|------|
| Europa | `westeurope`| Centro de Francia<br>`francecentral`     | `https://francecentral.api.cognitive.microsoft.com/luis/v2.0/apps/YOUR-APP-ID?subscription-key=YOUR-SUBSCRIPTION-KEY`   |
| Europa | `westeurope`| Norte de Europa<br>`northeurope`     | `https://northeurope.api.cognitive.microsoft.com/luis/v2.0/apps/YOUR-APP-ID?subscription-key=YOUR-SUBSCRIPTION-KEY`   |
| Europa | `westeurope`| Oeste de Europa<br>`westeurope`    |  `https://westeurope.api.cognitive.microsoft.com/luis/v2.0/apps/YOUR-APP-ID?subscription-key=YOUR-SUBSCRIPTION-KEY`   |
| Europa | `westeurope`| Sur de Reino Unido 2<br>`uksouth`    |  `https://uksouth.api.cognitive.microsoft.com/luis/v2.0/apps/YOUR-APP-ID?subscription-key=YOUR-SUBSCRIPTION-KEY`   |
| Europa | `westeurope`| Norte de Suiza<br>`switzerlandnorth`    |  `https://switzerlandnorth.api.cognitive.microsoft.com/luis/v2.0/apps/YOUR-APP-ID?subscription-key=YOUR-SUBSCRIPTION-KEY`   |

## <a name="publishing-to-australia"></a>Publicación en Australia

 Región global | Región de la API de creación | Región de publicación y de consulta<br>`API region name`   |  Formato de dirección URL de punto de conexión   |
|-----|------|------|------|
| Australia | `australiaeast` | Este de Australia<br>`australiaeast`     |  `https://australiaeast.api.cognitive.microsoft.com/luis/v2.0/apps/YOUR-APP-ID?subscription-key=YOUR-SUBSCRIPTION-KEY`   |

## <a name="other-publishing-regions"></a>Otras regiones de publicación

 Región global | Región de la API de creación | Región de publicación y de consulta<br>`API region name`   |  Formato de dirección URL de punto de conexión   |
|-----|------|------|------|
| África | `westus`<br>[www.luis.ai][www.luis.ai]| Norte de Sudáfrica<br>`southafricanorth` |  `https://southafricanorth.api.cognitive.microsoft.com/luis/v2.0/apps/YOUR-APP-ID?subscription-key=YOUR-SUBSCRIPTION-KEY` |
| Asia | `westus`<br>[www.luis.ai][www.luis.ai]| Centro de la India<br>`centralindia` |  `https://centralindia.api.cognitive.microsoft.com/luis/v2.0/apps/YOUR-APP-ID?subscription-key=YOUR-SUBSCRIPTION-KEY` |
| Asia | `westus`<br>[www.luis.ai][www.luis.ai]| Este de Asia<br>`eastasia`     |  `https://eastasia.api.cognitive.microsoft.com/luis/v2.0/apps/YOUR-APP-ID?subscription-key=YOUR-SUBSCRIPTION-KEY` |
| Asia | `westus`<br>[www.luis.ai][www.luis.ai]| Japón Oriental<br>`japaneast`     |   `https://japaneast.api.cognitive.microsoft.com/luis/v2.0/apps/YOUR-APP-ID?subscription-key=YOUR-SUBSCRIPTION-KEY` |
| Asia | `westus`<br>[www.luis.ai][www.luis.ai]| Japón Occidental<br>`japanwest`     |   `https://japanwest.api.cognitive.microsoft.com/luis/v2.0/apps/YOUR-APP-ID?subscription-key=YOUR-SUBSCRIPTION-KEY` |
| Asia | `westus`<br>[www.luis.ai][www.luis.ai]| Centro de Corea del Sur<br>`koreacentral`     |   `https://koreacentral.api.cognitive.microsoft.com/luis/v2.0/apps/YOUR-APP-ID?subscription-key=YOUR-SUBSCRIPTION-KEY` |
| Asia | `westus`<br>[www.luis.ai][www.luis.ai]| Sudeste de Asia<br>`southeastasia`     |   `https://southeastasia.api.cognitive.microsoft.com/luis/v2.0/apps/YOUR-APP-ID?subscription-key=YOUR-SUBSCRIPTION-KEY` |
| Asia | `westus`<br>[www.luis.ai][www.luis.ai]| Norte de Emiratos Árabes Unidos<br>`northuae`     |   `https://northuae.api.cognitive.microsoft.com/luis/v2.0/apps/YOUR-APP-ID?subscription-key=YOUR-SUBSCRIPTION-KEY` |
| Norteamérica |`westus`<br>[www.luis.ai][www.luis.ai] | Centro de Canadá<br>`canadacentral`     |   `https://canadacentral.api.cognitive.microsoft.com/luis/v2.0/apps/YOUR-APP-ID?subscription-key=YOUR-SUBSCRIPTION-KEY` |
| Norteamérica |`westus`<br>[www.luis.ai][www.luis.ai] | Centro de EE. UU.<br>`centralus`     |   `https://centralus.api.cognitive.microsoft.com/luis/v2.0/apps/YOUR-APP-ID?subscription-key=YOUR-SUBSCRIPTION-KEY` |
| Norteamérica |`westus`<br>[www.luis.ai][www.luis.ai] | Este de EE. UU.<br>`eastus`      |  `https://eastus.api.cognitive.microsoft.com/luis/v2.0/apps/YOUR-APP-ID?subscription-key=YOUR-SUBSCRIPTION-KEY` |
| Norteamérica | `westus`<br>[www.luis.ai][www.luis.ai] | Este de EE. UU. 2<br>`eastus2`     |  `https://eastus2.api.cognitive.microsoft.com/luis/v2.0/apps/YOUR-APP-ID?subscription-key=YOUR-SUBSCRIPTION-KEY` |
| Norteamérica | `westus`<br>[www.luis.ai][www.luis.ai] | Centro-Norte de EE. UU<br>`northcentralus`  |  `https://northcentralus.api.cognitive.microsoft.com/luis/v2.0/apps/YOUR-APP-ID?subscription-key=YOUR-SUBSCRIPTION-KEY` |
| Norteamérica | `westus`<br>[www.luis.ai][www.luis.ai] | Centro-sur de EE. UU.<br>`southcentralus`  |  `https://southcentralus.api.cognitive.microsoft.com/luis/v2.0/apps/YOUR-APP-ID?subscription-key=YOUR-SUBSCRIPTION-KEY` |
| Norteamérica |`westus`<br>[www.luis.ai][www.luis.ai] | Centro-Oeste de EE. UU.<br>`westcentralus`    |  `https://westcentralus.api.cognitive.microsoft.com/luis/v2.0/apps/YOUR-APP-ID?subscription-key=YOUR-SUBSCRIPTION-KEY` |
| Norteamérica | `westus`<br>[www.luis.ai][www.luis.ai] | Oeste de EE. UU.<br>`westus`  |   `https://westus.api.cognitive.microsoft.com/luis/v2.0/apps/YOUR-APP-ID?subscription-key=YOUR-SUBSCRIPTION-KEY` |
| Norteamérica |`westus`<br>[www.luis.ai][www.luis.ai] | Oeste de EE. UU. 2<br>`westus2`    |  `https://westus2.api.cognitive.microsoft.com/luis/v2.0/apps/YOUR-APP-ID?subscription-key=YOUR-SUBSCRIPTION-KEY` |
| Sudamérica | `westus`<br>[www.luis.ai][www.luis.ai] | Sur de Brasil<br>`brazilsouth`    |  `https://brazilsouth.api.cognitive.microsoft.com/luis/v2.0/apps/YOUR-APP-ID?subscription-key=YOUR-SUBSCRIPTION-KEY` |

## <a name="endpoints"></a>Puntos de conexión

Más información sobre la [creación y predicción de puntos de conexión](developer-reference-resource.md).

## <a name="failover-regions"></a>Regiones de conmutación por error

Cada región tiene una región secundaria a la que conmutar por error. Europa conmuta por error dentro de Europa y Australia conmuta por error dentro de Australia.

Las regiones de creación tienen [regiones de conmutación por error emparejadas](../../best-practices-availability-paired-regions.md).

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Referencia de entidades creadas previamente](./luis-reference-prebuilt-entities.md)

 [www.luis.ai]: https://www.luis.ai