---
title: Compatibilidad de idiomas con Form Recognizer
titleSuffix: Azure Cognitive Services
description: Conozca mejor los idiomas humanos que están disponibles con Form Recognizer.
services: cognitive-services
author: laujan
manager: nitinme
ms.service: cognitive-services
ms.subservice: forms-recognizer
ms.topic: overview
ms.date: 05/10/2021
ms.author: lajanuar
ms.openlocfilehash: eee8c7d595ff73bf14ac996c030ac938523d1a09
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110374430"
---
# <a name="language-support-for-form-recognizer"></a>Compatibilidad de idiomas con Form Recognizer

 En esta tabla se enumeran los idiomas escritos que admite el servicio Form Recognizer.

<!-- markdownlint-disable MD001 -->
<!-- markdownlint-disable MD024 -->

## <a name="layout-and-custom-model"></a>Diseño y modelo personalizado

### <a name="v21"></a>[v2.1](#tab/v2-1)

|Idioma| Código de lenguaje |
|:-----|:----:|
|Afrikáans|`af`|
|Albanés |`sq`|
|Asturiano |`ast`|
|Vasco  |`eu`|
|Bislama   |`bi`|
|Bretón    |`br`|
|Catalán    |`ca`|
|Cebuano    |`ceb`|
|Chamorro  |`ch`|
|Chino (simplificado) | `zh-Hans`|
|Chino (tradicional) | `zh-Hant`|
|Córnico     |`kw`|
|Corso      |`co`|
|Tártaro de Crimea (Latín)  |`crh`|
|Checo | `cs` |
|Danés | `da` |
|Neerlandés | `nl` |
|Inglés (impreso y manuscrito) | `en` |
|Estonio  |`et`|
|Fiyiano |`fj`|
|Filipino  |`fil`|
|Finés | `fi` |
|Francés | `fr` |
|Friulano  | `fur` |
|Gallego   | `gl` |
|Alemán | `de` |
|Gilbertés    | `gil` |
|Groenlandés   | `kl` |
|Criollo haitiano  | `ht` |
|Hani  | `hni` |
|Hmong Daw (Latín) | `mww` |
|Húngaro | `hu` |
|Indonesio   | `id` |
|Interlingua  | `ia` |
|Inuktitut (latino)  | `iu`  |
|Irlandés    | `ga` |
|Italiano | `it` |
|Japonés | `ja` |
|Javanés | `jv` |
|Quiché  | `quc` |
|Caboverdiano | `kea` |
|Kachin (Latín) | `kac` |
|Kara-Kalpak | `kaa` |
|Casubio | `csb` |
|Khasi  | `kha` |
|Coreano | `ko` |
|Kurdo (latino) | `kur` |
|Luxemburgués  | `lb` |
|Malayo (latino)  | `ms` |
|Manx  | `gv` |
|Napolitano   | `nap` |
|Noruego | `no` |
|Occitano | `oc` |
|Polaco | `pl` |
|Portugués | `pt` |
|Romanche  | `rm` |
|Escocés  | `sco` |
|Gaélico escocés  | `gd` |
|Esloveno  | `slv` |
|Español | `es` |
|Swahili (Latín)  | `sw` |
|Sueco | `sv` | 
|Tártaro (Latín)  | `tat` |
|Tetum    | `tet` |
|Turco | `tr` |
|Alto sorbio  | `hsb` |
|Uzbeko (latino)     | `uz` |
|Volapük   | `vo` |
|Walser    | `wae` |
|Frisón occidental | `fy` |
|Maya Yucateco | `yua` |
|Zhuang | `za` |
|Zulú  | `zu` |

### <a name="v20"></a>[v2.0](#tab/v2-0)

|Idioma| Código de lenguaje |
|:-----|:----:|
Chino (simplificado) | `zh-Hans`|
|Neerlandés | `nl` |
|Inglés (impreso y manuscrito) | `en` |
|Francés | `fr` |
|Alemán | `de` |
|Italiano | `it` |
|Japonés | `ja` |
|Portugués | `pt` |
|Español | `es` |

-----

## <a name="prebuilt-receipt-and-business-card"></a>Recibo compilado previamente y tarjeta de visita

>[!NOTE]
 > No es necesario especificar una configuración regional. Es un parámetro opcional y la tecnología de aprendizaje profundo de Form Recognizer detectará automáticamente el idioma del texto de la imagen.

### <a name="v21"></a>[v2.1](#tab/v2-1)

Los recibos compilados previamente y las tarjetas de visita admiten todas las tarjetas de visita y recibos en inglés con las siguientes configuraciones regionales:

|Lenguaje| Código de configuración regional |
|:-----|:----:|
|Inglés (Australia)|`en-au`|
|Inglés (Canadá)|`en-ca`|
|Inglés (Reino Unido)|`en-gb`|
|Inglés (India)|`en-in`|
|Spanish (Traditional Sort) - Spain| `en-us`|

### <a name="v20"></a>[v2.0](#tab/v2-0)

> [!IMPORTANT]
> Esta característica no está disponible en la versión de API seleccionada.

-----

## <a name="prebuilt-invoice"></a>Factura compilada previamente

### <a name="v21"></a>[v2.1](#tab/v2-1)

Lenguaje| Código de configuración regional |
|:-----|:----:|
|Spanish (Traditional Sort) - Spain|es-es|

### <a name="v20"></a>[v2.0](#tab/v2-0)

> [!IMPORTANT]
> Esta característica no está disponible en la versión de API seleccionada.

-----

## <a name="prebuilt-identity-documents"></a>Documentos de identidad compilados previamente

### <a name="v21"></a>[v2.1](#tab/v2-1)

> [!NOTE]
> Esta tecnología está disponible actualmente para los permisos de conducir de EE. UU. y la página de información personal de los pasaportes internacionales (excepto los visados y otros documentos de viajes).

### <a name="v20"></a>[v2.0](#tab/v2-0)

> [!IMPORTANT]
> Esta característica no está disponible en la versión de API seleccionada.

-----

> [!div class="nextstepaction"]
> [Probar Form Recognizer](https://aka.ms/fott-2.1-ga)
