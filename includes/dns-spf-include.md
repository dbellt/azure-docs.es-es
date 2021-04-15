---
author: vhorne
ms.service: dns
ms.topic: include
ms.date: 11/25/2018
ms.author: victorh
ms.openlocfilehash: 9cc650cea17acb8d89933c819c4ca60e2c459d1c
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "67186946"
---
Los registros de marco de directivas de remitente (SPF) se utilizan para especificar los servidores de correo electrónico que tienen permiso para enviar correo en nombre de un nombre de dominio. Es importante configurar correctamente los registros SPF para evitar que los destinatarios marquen el correo como no deseado.

Las RFC de DNS originalmente incluyeron un nuevo tipo de registro SPF que admite este escenario. Para admitir servidores de nombres anteriores, permitían también el uso del tipo de registro TXT a fin de especificar registros SPF. Esta ambigüedad llevó a confusión, que se resolvió mediante [RFC 7208](http://tools.ietf.org/html/rfc7208#section-3.1). Indica que los registros SPF deben crearse con el tipo de registro TXT. También indica que el tipo de registro SPF está en desuso.

**Los registros SPF son compatibles con Azure DNS y deben crearse con el tipo de registro TXT.** No se admite el tipo de registro SPF obsoleto. Al [importar un archivo de zona DNS](../articles/dns/dns-import-export.md), los registros SPF con el tipo de registro SPF se convierten al tipo de registro TXT.
