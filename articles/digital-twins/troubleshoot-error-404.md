---
title: 'Error de solicitud de Azure Digital Twins con estado: 404 subdominio no encontrado'
description: 'Causas y soluciones de "Service request failed. Estado: 404 subdominio no encontrado" en Azure Digital Twins.'
ms.service: digital-twins
author: baanders
ms.author: baanders
ms.topic: troubleshooting
ms.date: 4/13/2021
ms.openlocfilehash: bc635d742fb9edf53a6cd3244b7e8aa34c8d7148
ms.sourcegitcommit: a5dd9799fa93c175b4644c9fe1509e9f97506cc6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108205669"
---
# <a name="service-request-failed-status-404-sub-domain-not-found"></a>Error en la solicitud del servicio. Estado: 404 subdominio no encontrado

En este artículo se describen las causas y los pasos de resolución al recibir un error 404 de las solicitudes de servicio a Azure Digital Twins. 

## <a name="symptoms"></a>Síntomas

Este error puede producirse al acceder a una instancia de Azure Digital Twins mediante una entidad de servicio o una cuenta de usuario que pertenece a un [inquilino de Azure Active Directory (Azure AD)](../active-directory/develop/quickstart-create-new-tenant.md) diferente de la instancia. Al parecer, los [roles](concepts-security.md) correctos se asignan a la identidad, pero las solicitudes de API no se pueden completar con un estado de error de `404 Sub-Domain not found`.

## <a name="causes"></a>Causas

### <a name="cause-1"></a>Causa 1

Azure Digital Twins requiere que todos los usuarios que se autentican pertenezcan al mismo inquilino de Azure AD que la instancia de Azure Digital Twins.

[!INCLUDE [digital-twins-tenant-limitation](../../includes/digital-twins-tenant-limitation.md)]

## <a name="solutions"></a>Soluciones

### <a name="solution-1"></a>Solución 1

Para resolver este problema, puede hacer que cada identidad federada de otro inquilino solicite un **token** al inquilino "principal" de la instancia de Azure Digital Twins. 

[!INCLUDE [digital-twins-tenant-solution-1](../../includes/digital-twins-tenant-solution-1.md)]

### <a name="solution-2"></a>Solución 2

Si usa la clase `DefaultAzureCredential` en el código y sigue detectando este problema después de obtener un token, puede especificar el inquilino principal en las opciones de `DefaultAzureCredential` para aclarar el inquilino, incluso cuando la autenticación tome otro tipo como valor predeterminado.

[!INCLUDE [digital-twins-tenant-solution-2](../../includes/digital-twins-tenant-solution-2.md)]

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre la seguridad y los permisos de Azure Digital Twins:
* [Conceptos: Seguridad para las soluciones de Azure Digital Twins](concepts-security.md)
