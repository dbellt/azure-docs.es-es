---
author: baanders
description: como incluir el archivo que describe una la limitación entre inquilinos con Azure Digital Twins
ms.service: digital-twins
ms.topic: include
ms.date: 4/13/2021
ms.author: baanders
ms.openlocfilehash: 16684f8c5947f8b6a09a9a785968dabf3e644c18
ms.sourcegitcommit: 272351402a140422205ff50b59f80d3c6758f6f6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/17/2021
ms.locfileid: "107589388"
---
Como resultado, las solicitudes a las API de Azure Digital Twins requieren un usuario o una entidad de servicio que forme parte del mismo inquilino donde reside la instancia de Azure Digital Twins. Para evitar exámenes malintencionados de los puntos de conexión de Azure Digital Twins, las solicitudes con tokens de acceso desde fuera del inquilino de origen recibirán un mensaje de error "404 No se ha encontrado el subdominio". Este error se devolverá *incluso si* al usuario o a la entidad de servicio se le ha concedido un [rol](../articles/digital-twins/concepts-security.md) de propietario de datos o de lector de datos de Azure Digital Twins mediante la colaboración de [Azure AD B2B](../articles/active-directory/external-identities/what-is-b2b.md). 