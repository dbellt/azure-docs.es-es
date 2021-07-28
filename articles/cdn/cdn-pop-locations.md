---
title: Ubicaciones POP de Azure CDN por región | Microsoft Docs
description: En este artículo se enumeran las ubicaciones POP de Azure CDN, ordenadas por región, para los productos de Azure CDN.
services: cdn
documentationcenter: ''
author: sohamnchatterjee
manager: danielgi
editor: sohamnchatterjee
ms.assetid: 669ef140-a6dd-4b62-9b9d-3f375a14215e
ms.service: azure-cdn
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/18/2021
ms.author: allensu
ms.custom: references_regions
ms.openlocfilehash: e58ce2b4fa016a79050425dad4e4dd0b784f0c9c
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110472375"
---
# <a name="azure-cdn-coverage-by-metro"></a>Cobertura de Azure CDN según Metro 
> [!div class="op_single_selector"]
> * [Ubicaciones POP por región](cdn-pop-locations.md)
> * [Ubicaciones POP de Verizon por abreviatura](cdn-pop-abbreviations.md)
> * [Ubicaciones POP de Microsoft por abreviatura](microsoft-pop-abbreviations.md)
>


En este artículo se enumeran áreas metropolitanas que contienen ubicaciones de punto de presencia (POP) actuales, ordenadas por región, para los productos de Azure Content Delivery Network (CDN). Cada área metropolitana puede contener más de una ubicación POP. Por ejemplo, Azure CDN de Microsoft tiene 116 ubicaciones POP en 100 ciudades metropolitanas. 

> [!IMPORTANT]
> Cada producto de Azure CDN tiene una manera distinta de crear sus infraestructuras de CDN, por lo que Microsoft recomienda no usar ubicaciones POP para decidir qué producto de Azure CDN usar. En su lugar, debe considerar sus características y el rendimiento del usuario final. Pruebe el rendimiento con cada producto de Azure CDN para elegir el producto adecuado para sus usuarios. 
> 

## <a name="microsoft"></a>Microsoft

[!INCLUDE [front-door-edge-location](../../includes/front-door-edge-locations.md)]

## <a name="partners"></a>Asociados

> [!IMPORTANT]
> Las ubicaciones de ciudades de POP de **Azure CDN de Akamai** no se revelan individualmente.  
> 

| Region | Verizon | Akamai |
|--|--|--|
| Norteamérica | Guadalajara, México<br />Ciudad de México, México<br />Puebla, México<br />Querétaro, México<br />Atlanta, GA, Estados Unidos<br />Boston, MA, Estados Unidos<br />Chicago, IL, Estados Unidos<br />Dallas, TX, Estados Unidos<br />Denver, CO, Estados Unidos<br />Detroit, MI, Estados Unidos<br />Los Ángeles, CA, Estados Unidos<br />Miami, FL, Estados Unidos<br />Nueva York, NY, Estados Unidos<br />Filadelfia, PA, Estados Unidos<br />San José, CA, Estados Unidos<br />Seattle, WA, Estados Unidos<br />Washington, DC, Estados Unidos <br /> Ashburn, VA, Estados Unidos <br /> Phoenix, AZ, Estados Unidos | Canadá<br />México<br />EE. UU. |
| Sudamérica | Buenos Aires, Argentina<br />Río de Janeiro, Brasil<br />San Pablo, Brasil<br />Valparaíso, Chile<br />Bogotá, Colombia<br />Barranquilla, Colombia<br />Medellín, Colombia<br />Quito, Ecuador<br />Lima, Perú | Argentina<br />Brasil<br />Chile<br />Colombia<br />Ecuador<br />Perú<br />Uruguay |
| Europa | Viena, Austria<br />Copenhague, Dinamarca<br />Helsinki, Finlandia<br />Marsella, Francia<br />París, Francia<br />Fráncfort, Alemania<br />Milán, Italia<br />Riga, Letonia<br />Ámsterdam, Países Bajos<br />Varsovia, Polonia<br />Madrid, España<br />Estocolmo, Suecia<br />Londres, Reino Unido <br /> Manchester, Reino Unido | Austria<br />Bulgaria<br />Dinamarca<br />Finlandia<br />Francia<br />Alemania<br />Grecia<br />Irlanda<br />Italia<br />Países Bajos<br />Polonia<br />Rusia<br />España<br />Suecia<br />Suiza<br />Reino Unido |
| África | Johannesburgo (Sudáfrica) <br/> Nairobi, Kenia | Sudáfrica |
| Oriente Medio | Muscat, Omán<br />Fujirah, Emiratos Árabes Unidos | Qatar<br />Emiratos Árabes Unidos |
| India | Bengaluru (Bangalore), India<br />Chennai, India<br />Mumbai (Bombay), India<br />Nueva Delhi, India<br /> | India |
| Asia | RAE de Hong Kong<br />Yakarta, Indonesia<br />Osaka, Japón<br />Tokio, Japón<br />Singapur<br />Seúl, Corea del Sur<br />Kaohsiung, Taiwán<br />Taipei, Taiwán <br />Manila, Filipinas | RAE de Hong Kong<br />Indonesia<br />Israel<br />Japón<br />RAE de Macao<br />Malasia<br />Filipinas<br />Singapur<br />Corea del Sur<br />Taiwán<br />Tailandia<br />Turquía<br />Vietnam |
| Australia y Nueva Zelanda | Melbourne, Australia<br />Sídney, Australia<br />Auckland, Nueva Zelanda | Australia<br />Nueva Zelanda |

## <a name="next-steps"></a>Pasos siguientes

* Para obtener las últimas direcciones IP para agregarlas a la lista de permitidos, consulte la [API de nodos perimetrales de Azure CDN](/rest/api/cdn/cdn/edgenodes).
