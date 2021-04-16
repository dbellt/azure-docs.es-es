---
author: baanders
description: 'Archivo de inclusión para Azure Digital Twins: requisito previo para configurar un de aplicación'
ms.service: digital-twins
ms.topic: include
ms.date: 10/29/2020
ms.author: baanders
ms.openlocfilehash: 2ce534972cf6509cdc1ca026f4b29efd3df91afd
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "96303567"
---
Para autenticar todos los recursos que se usan en este artículo, tendrá que **configurar un registro de aplicación de [Azure Active Directory (Azure AD)](../articles/active-directory/fundamentals/active-directory-whatis.md)** . Siga las instrucciones de [*Procedimiento: Creación de un registro de aplicación*](../articles/digital-twins/how-to-create-app-registration.md) para configurarlo. 

Una vez que tenga un registro de la aplicación, necesitará el **_id. de aplicación (cliente)_** y el **_id. de directorio (inquilino)_** del registro ([se encuentra en Azure Portal](../articles/digital-twins/how-to-create-app-registration.md#collect-client-id-and-tenant-id)). Tome nota de estos valores para usarlos más adelante para conceder acceso a las API de Azure Digital Twins.