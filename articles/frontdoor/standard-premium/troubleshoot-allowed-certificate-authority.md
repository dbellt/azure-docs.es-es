---
title: Entidades de certificación permitidas para Azure Front Door Standard/Premium (versión preliminar)
description: En este artículo se enumeran todas las entidades de certificación que se permiten al crear su propio certificado.
services: frontdoor
author: duongau
ms.service: frontdoor
ms.topic: troubleshooting
ms.date: 02/18/2021
ms.author: qixwang
ms.openlocfilehash: e31ad0734630fbd0b4960c26f8d562cea66ae675
ms.sourcegitcommit: e39ad7e8db27c97c8fb0d6afa322d4d135fd2066
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112003634"
---
# <a name="allowed-certificate-authorities-for-azure-front-door-standardpremium-preview"></a>Entidades de certificación permitidas para Azure Front Door Standard/Premium (versión preliminar)

Cuando habilite la característica HTTPS con su propio certificado para un dominio personalizado de Azure Front Door Standard/Premium. Necesita una entidad de certificación (CA) permitida para crear el certificado TLS/SSL. De lo contrario, si usa una entidad de certificación no permitida o un certificado autofirmado, la solicitud se rechazará.

[!INCLUDE [cdn-front-door-allowed-ca](../../../includes/cdn-front-door-allowed-ca.md)]
