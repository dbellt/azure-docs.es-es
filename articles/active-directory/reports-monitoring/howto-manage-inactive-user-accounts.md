---
title: Administración de cuentas de usuario inactivas en Azure AD | Microsoft Docs
description: Obtenga información sobre cómo detectar y administrar cuentas de usuario en Azure AD que han quedado obsoletas.
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: mtillman
editor: ''
ms.assetid: ada19f69-665c-452a-8452-701029bf4252
ms.service: active-directory
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.subservice: report-monitor
ms.date: 05/06/2021
ms.author: markvi
ms.reviewer: besiler
ms.collection: M365-identity-device-management
ms.openlocfilehash: 1c15bc161874d9c09423fe4070958538ec2148ca
ms.sourcegitcommit: 89c4843ec85d1baea248e81724781d55bed86417
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108795103"
---
# <a name="how-to-manage-inactive-user-accounts-in-azure-ad"></a>Instrucciones: Administración de cuentas de usuario inactivas en Azure AD

En entornos de gran tamaño, las cuentas de usuario no siempre se eliminan cuando los empleados salen de la organización. Como administrador de TI, quiere detectar y administrar estas cuentas de usuario obsoletas porque representan un riesgo para la seguridad.

En este artículo se explica un método para administrar las cuentas de usuario obsoletas en Azure AD. 

> [!IMPORTANT]
> Las API de la versión `/beta` de Microsoft Graph están sujetas a cambios. No se admite su uso en aplicaciones de producción. Para determinar si una API está disponible en v1.0, use el selector **Versión**.

## <a name="what-are-inactive-user-accounts"></a>¿Qué son las cuentas de usuario inactivas?

Las cuentas inactivas son cuentas de usuario que los miembros de su organización ya no necesitan para obtener acceso a los recursos. Un identificador clave para las cuentas inactivas es que no se han usado *durante un tiempo* para iniciar sesión en su entorno. Dado que las cuentas inactivas están vinculadas a la actividad de inicio de sesión, puede usar la marca de tiempo del último inicio de sesión que se ha realizado correctamente para detectarlas. 

El reto de este método es definir qué significa *durante un tiempo* en el caso de su entorno. Por ejemplo, es posible que los usuarios no inicien sesión en un entorno *durante un tiempo*, ya que están en vacaciones. Al definir el diferencial de las cuentas de usuario inactivas, debe tener en cuenta todos los motivos legítimos para no iniciar sesión en su entorno. En muchas organizaciones, el diferencial de las cuentas de usuario inactivas se encuentra entre 90 y 180 días. 

El último inicio de sesión correcto ofrece información potencial sobre la necesidad continuada del usuario de obtener acceso a los recursos.  Puede ayudar a determinar si el acceso a la aplicación o la pertenencia al grupo sigue siendo necesario o se puede quitar. Para la administración de usuarios externos, puede saber si un usuario externo sigue activo en el inquilino o si debe limpiarse. 

    
## <a name="how-to-detect-inactive-user-accounts"></a>Cómo detectar cuentas de usuario inactivas

Las cuentas inactivas se detectan mediante la evaluación de la propiedad **lastSignInDateTime** expuesta por el tipo de recurso **signInActivity** de **Microsoft Graph** API. Con esta propiedad, puede implementar una solución para los escenarios siguientes:

- **Usuarios por nombre**: en este escenario, busca un usuario específico por nombre, lo que le permite evaluar el valor de lastSignInDateTime: `https://graph.microsoft.com/beta/users?$filter=startswith(displayName,'markvi')&$select=displayName,signInActivity`

- **Usuarios por fecha**: En este escenario, solicita una lista de usuarios con un valor de lastSignInDateTime anterior a una fecha especificada: `https://graph.microsoft.com/beta/users?filter=signInActivity/lastSignInDateTime le 2019-06-01T00:00:00Z`

> [!NOTE]
> Puede que sea necesario generar un informe de la última fecha de inicio de sesión de todos los usuarios, en caso de que pueda usar el siguiente escenario.
> **Fecha y hora del último inicio de sesión para todos los usuarios**: en este escenario, se solicita una lista de todos los usuarios y el último lastSignInDateTime para cada usuario respectivo: `https://graph.microsoft.com/beta/users?$select=displayName,signInActivity` 

## <a name="what-you-need-to-know"></a>Lo que necesita saber

En esta sección se enumera lo que necesita saber sobre la propiedad lastSignInDateTime.

### <a name="how-can-i-access-this-property"></a>¿Cómo se puede acceder a esta propiedad?

La propiedad **lastSignInDateTime** se expone por el tipo de recurso [signInActivity](/graph/api/resources/signinactivity?view=graph-rest-beta&preserve-view=true) de [Microsoft Graph API REST](/graph/overview#whats-in-microsoft-graph).   

### <a name="is-the-lastsignindatetime-property-available-through-the-get-azureaduser-cmdlet"></a>¿La propiedad lastSignInDateTime está disponible a través del cmdlet Get-AzureAdUser?

No.

### <a name="what-edition-of-azure-ad-do-i-need-to-access-the-property"></a>¿Qué edición de Azure AD debo tener para acceder a la propiedad?

Para acceder a esta propiedad, necesita una edición de Azure Active Directory Prémium.

### <a name="what-permission-do-i-need-to-read-the-property"></a>¿Qué permisos necesito para leer la propiedad?

Para leerla, debe conceder los siguientes derechos: 

- AuditLogs.Read.All
- Organisation.Read.All  


### <a name="when-does-azure-ad-update-the-property"></a>¿Cuándo actualiza Azure AD la propiedad?

Cada inicio de sesión interactivo que se haya realizado correctamente genera una actualización del almacén de datos subyacente. Normalmente, los inicios de sesión correctos se muestran en el informe de inicio de sesión relacionado en un plazo de 10 minutos.
 

### <a name="what-does-a-blank-property-value-mean"></a>¿Qué significa un valor de propiedad en blanco?

Para generar la marca de tiempo de lastSignInDateTime, se necesita un inicio de sesión correcto. Como la propiedad lastSignInDateTime es una característica nueva, su valor puede estar en blanco si:

- El último inicio de sesión correcto de un usuario tuvo lugar antes de abril de 2020.
- La cuenta de usuario afectada nunca se usó para un inicio de sesión correcto.

## <a name="next-steps"></a>Pasos siguientes

* [Obtención de datos mediante la API de generación de informes de Azure Active Directory con certificados](tutorial-access-api-with-certificates.md)
* [Referencia de la API de auditoría](/graph/api/resources/directoryaudit) 
* [Referencia de la API de generación de informes de actividad de inicio de sesión](/graph/api/resources/signin)
