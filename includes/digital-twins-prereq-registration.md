---
author: baanders
description: 'Archivo de inclusión para Azure Digital Twins: requisito previo para configurar un de aplicación'
ms.service: digital-twins
ms.topic: include
ms.date: 10/29/2020
ms.author: baanders
ms.openlocfilehash: a700f45e90334033cfc4463043017166d5f7ffd9
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111993425"
---
Para autenticar todos los recursos que se usan en este artículo, tendrá que **configurar un registro de aplicación de [Azure Active Directory (Azure AD)](../articles/active-directory/fundamentals/active-directory-whatis.md)** . Siga las instrucciones de [Procedimiento: Creación de un registro de aplicación](../articles/digital-twins/how-to-create-app-registration-portal.md) para configurarlo. 

Una vez que tenga un registro de la aplicación, necesitará el **id. de la aplicación (cliente)** , el **id. de directorio (inquilino)** y el **secreto de cliente** del registro ([se encuentran en Azure Portal](../articles/digital-twins/how-to-create-app-registration-portal.md#collect-important-values)). Tome nota de estos valores para usarlos más adelante para conceder acceso a las API de Azure Digital Twins.