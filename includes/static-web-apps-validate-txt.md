---
author: burkeholland
ms.service: static-web-apps
ms.topic: include
ms.date: 05/10/2021
ms.author: buhollan
ms.openlocfilehash: 89e61c1cb472f7b6bb62f1f2bd84d083e103d650
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110066123"
---
#### <a name="validate-txt-record"></a>Validación del registro TXT

1. Vuelva a la pantalla _Validar y configurar_ en Azure Portal.

Durante este paso, Azure comprueba automáticamente el registro TXT con el proveedor de DNS. Una vez completado el proceso de validación, aparece un indicador verde junto al dominio agregado.

:::image type="content" source="../articles/static-web-apps/media/custom-domain/txt-record-ready.png" alt-text="Indicador verde que muestra que se ha validado el registro TXT":::

Cuando aparezca el indicador verde junto al dominio en la pantalla _Dominios personalizados_, puede completar el segundo paso, que consiste en agregar un registro ALIAS si va a agregar un dominio raíz o un registro CNAME si va a agregar un subdominio.
