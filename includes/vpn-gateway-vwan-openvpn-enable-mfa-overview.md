---
title: archivo de inclusión
description: archivo de inclusión
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 02/14/2020
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 8fba8aa577bcb3b5ef44d57c388a1f1de7494782
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "95995629"
---
Si quiere que se solicite a los usuarios un segundo factor de autenticación antes de conceder acceso, puede configurar Azure AD Multi-Factor Authentication (MFA). Puede configurar MFA por usuario o bien aprovechar MFA a través del [acceso condicional](../articles/active-directory/conditional-access/overview.md).

* MFA por usuario puede habilitarse sin ningún costo adicional. Al habilitar MFA por usuario, se le solicitará al usuario un segundo factor de autenticación en todas las aplicaciones vinculadas al inquilino de Azure AD. Consulte la [Opción 1](#peruser) para ver los pasos.
* El acceso condicional permite un control más específico sobre cómo se debe promover un segundo factor. Puede permitir la asignación de MFA únicamente a VPN, así como excluir otras aplicaciones vinculadas al inquilino de Azure AD. Consulte la [Opción 2](#conditional) para ver los pasos.