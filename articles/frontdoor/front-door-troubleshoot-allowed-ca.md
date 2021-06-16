---
title: CA permitidas para habilitar HTTPS personalizado en Azure Front Door
description: Si utiliza su propio certificado para habilitar HTTPS en un dominio personalizado de Azure Front Door, debe utilizar una entidad de certificación permitida (CA) para crearlo.
services: frontdoor
documentationcenter: ''
author: duongau
ms.assetid: ''
ms.service: frontdoor
ms.workload: infrastructure-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/30/2020
ms.author: duau
ms.openlocfilehash: 20c5d611272ee2159ce8ddcc2865797a225a7ebb
ms.sourcegitcommit: e39ad7e8db27c97c8fb0d6afa322d4d135fd2066
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112003641"
---
# <a name="allowed-certificate-authorities-for-enabling-custom-https-on-azure-front-door"></a>Entidades de certificación permitidas para habilitar HTTPS personalizado en Azure Front Door

Cuando [habilite la característica HTTPS con su propio certificado](front-door-custom-domain-https.md?tabs=option-2-enable-https-with-your-own-certificate) para un dominio personalizado de Azure Front Door. Necesita una entidad de certificación (CA) permitida para crear el certificado TLS/SSL. De lo contrario, si usa una entidad de certificación no permitida o un certificado autofirmado, la solicitud se rechazará.

[!INCLUDE [cdn-front-door-allowed-ca](../../includes/cdn-front-door-allowed-ca.md)]
