---
title: Filtrado geográfico en un dominio para Azure Front Door | Microsoft Docs
description: En este artículo, obtendrá información acerca de la directiva de filtrado geográfico para Azure Front Door
services: frontdoor
documentationcenter: ''
author: duongau
editor: ''
ms.service: frontdoor
ms.workload: infrastructure-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/28/2020
ms.author: duau
ms.reviewer: amsriva
ms.openlocfilehash: 0ab6dfb61800bba91b52edd5b07fa918c85ba73b
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110085013"
---
# <a name="geo-filtering-on-a-domain-for-azure-front-door"></a>Filtrado geográfico en un dominio para Azure Front Door

De forma predeterminada, Azure Front Door responde a todas las solicitudes del usuario, independientemente del lugar de donde venga la solicitud. En algunos escenarios quizá desee restringir el acceso a la aplicación web por país o región. El servicio de firewall de aplicaciones web (WAF) de Front Door permite definir una directiva mediante reglas de acceso personalizadas en una ruta de acceso concreta del punto de conexión para permitir o bloquear el acceso desde determinados países o regiones.

Una directiva de WAF incluye un conjunto de reglas personalizadas. Las reglas constan de condiciones de coincidencia, acciones y prioridades. En las condiciones de coincidencia se definen una variable de coincidencia, un operador y un valor de coincidencia. En el caso de la regla de filtrado geográfico, la variable de coincidencia es REMOTE_ADDR, el operador es GeoMatch y el valor es el código de país o región de dos letras. El código de país "ZZ" o el país "Desconocido" capturan direcciones IP que todavía no están asignadas a un país en nuestro conjunto de datos. Puede agregar ZZ a la condición de coincidencia para evitar falsos positivos. Puede combinar una condición GeoMatch y una condición de coincidencia de la cadena REQUEST_URI para crear una regla de filtrado geográfico basada en la ruta de acceso.


Las directivas de filtrado geográfico para Front Door se pueden configurar mediante [Azure PowerShell](front-door-tutorial-geo-filtering.md) o una [plantilla de inicio rápido](https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.network/front-door-geo-filtering).

## <a name="countryregion-code-reference"></a>Referencia de código de país o región

|Código de país o región | Nombre del país o región |
| ----- | ----- |
| AD | Andorra |
| AE | Emiratos Árabes Unidos|
| AF | Afganistán|
| AG | Antigua y Barbuda|
| AL | Albania|
| AM | Armenia|
| AO | Angola|
| AR | Argentina|
| AS | Samoa Americana|
| AT | Austria|
| AU | Australia|
| AZ | Azerbaiyán|
| BA | Bosnia y Herzegovina|
| BB | Barbados|
| BD | Bangladés|
| BE | Bélgica|
| BF | Burkina Faso|
| BG | Bulgaria|
| BH | Bahréin|
| BI | Burundi|
| BJ | Benín|
| BL | San Bartolomé|
| BN | Brunéi Darussalam|
| BO | Bolivia|
| BR | Brasil|
| BS | Bahamas|
| BT | Bután|
| BW | Botsuana|
| BY | Belarús|
| BZ | Belice|
| CA | Canadá|
| CD | República Democrática del Congo|
| CF | República Centroafricana|
| CH | Suiza|
| CI | República de Côte d’Ivoire|
| CL | Chile|
| CM | Camerún|
| CN | China|
| CO | Colombia|
| CR | Costa Rica|
| CU | Cuba|
| CV | Cabo Verde|
| CY | Chipre|
| CZ | República Checa|
| DE | Alemania|
| DK | Dinamarca|
| DO | República Dominicana|
| DZ | Argelia|
| EC | Ecuador|
| EE | Estonia|
| EG | Egipto|
| ES | España|
| ET | Etiopía|
| FI | Finlandia|
| FJ | Islas Fiji|
| FM | Micronesia, Estados Federados de|
| VF | Francia|
| GB | Reino Unido|
| GE | Georgia|
| GF | Guayana Francesa|
| GH | Ghana|
| GN | Guinea|
| GP | Guadalupe|
| GR | Grecia|
| GT | Guatemala|
| GY | Guayana|
| HK | RAE de Hong Kong|
| HN | Honduras|
| HR | Croacia|
| HT | Haití|
| HU | Hungría|
| id | Indonesia|
| IE | Irlanda|
| IL | Israel|
| IN | India|
| IQ | Iraq|
| IR | Irán, República Islámica de|
| IS | Islandia|
| IT | Italia|
| JM | Jamaica|
| JO | Jordania|
| JP | Japón|
| KE | Kenia|
| KG | Kirguistán|
| KH | Camboya|
| KI | Kiribati|
| KN | San Cristóbal y Nieves|
| KP | Corea, República Popular Democrática de|
| KR | Corea, República de|
| KW | Kuwait|
| KY | Islas Caimán|
| KZ | Kazajistán|
| Los Ángeles | República Democrática Popular de Laos|
| LB | Líbano|
| LI | Liechtenstein|
| LK | Sri Lanka|
| LR | Liberia|
| LS | Lesoto|
| LT | Lituania|
| LU | Luxemburgo|
| LV | Letonia|
| LY | Libia |
| MA | Marruecos|
| MD | Moldavia, República de|
| MG | Madagascar|
| MK | Macedonia del Norte|
| ML | Malí|
| MM | Myanmar|
| MN | Mongolia|
| MO | RAE de Macao|
| MQ | Martinica|
| MR | Mauritania|
| MT | Malta|
| MV | Maldivas|
| MW | Malaui|
| MX | México|
| MY | Malasia|
| MZ | Mozambique|
| N/D | Namibia|
| NE | Níger|
| NG | Nigeria|
| NI | Nicaragua|
| NL | Países Bajos|
| No | Noruega|
| NP | Nepal|
| NR | Nauru|
| NZ | Nueva Zelanda|
| OM | Omán|
| PA | Panamá|
| PE | Perú|
| PH | Filipinas|
| PK | Pakistán|
| PL | Polonia|
| PR | Puerto Rico|
| PT | Portugal|
| PW | Palaos|
| PY | Paraguay|
| QA | Qatar|
| RE | Reunión|
| RO | Rumanía|
| RS | Serbia|
| RU | Federación Rusa|
| RW | Ruanda|
| SA | Arabia Saudí|
| SD | Sudán|
| SE | Suecia|
| SG | Singapur|
| SI | Eslovenia|
| SK | Eslovaquia|
| SN | Senegal|
| SO | Somalia|
| SR | Surinam|
| SS | Sudán del Sur|
| SV | El Salvador|
| SY | República Árabe Siria|
| SZ | Suazilandia|
| TC | Islas Turcas y Caicos|
| TG | Togo|
| TH | Tailandia|
| TN | Túnez|
| TR | Turquía|
| TT | Trinidad y Tobago|
| TW | Taiwán|
| TZ | Tanzania, República Unida de|
| UA | Ucrania|
| UG | Uganda|
| US | Estados Unidos|
| UY | Uruguay|
| UZ | Uzbekistán|
| VC | San Vicente y las Granadinas|
| VE | Venezuela|
| VG | Islas Vírgenes Británicas|
| VI | Islas Vírgenes de EE. UU.|
| VN | Vietnam|
| ZA | Sudáfrica|
| ZM | Zambia|
| ZW | Zimbabue|

## <a name="next-steps"></a>Pasos siguientes

- Aprenda a [crear una instancia de Front Door](quickstart-create-front-door.md).
- Aprenda a [configurar una directiva WAF de filtrado geográfico](front-door-tutorial-geo-filtering.md).
