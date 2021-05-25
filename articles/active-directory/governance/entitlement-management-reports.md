---
title: 'Visualización de informes y registros en la administración de derechos: Azure AD'
description: Obtenga información sobre cómo ver el informe de asignaciones de usuario y los registros de auditoría en la administración de derechos de Azure Active Directory.
services: active-directory
documentationCenter: ''
author: ajburnle
manager: daveba
editor: jocastel-MSFT
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.subservice: compliance
ms.date: 12/23/2020
ms.author: ajburnle
ms.reviewer: jocastel
ms.collection: M365-identity-device-management
ms.openlocfilehash: ab1cd0da0e44f9b0dd496b72c3dff7630eb9d641
ms.sourcegitcommit: 32ee8da1440a2d81c49ff25c5922f786e85109b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109786274"
---
# <a name="view-reports-and-logs-in-azure-ad-entitlement-management"></a>Consulta de informes y registros en la administración de derechos de Azure AD

Los informes de administración de derechos de Azure AD y el registro de auditoría de Azure AD proporcionan detalles adicionales sobre los recursos a los que tienen acceso los usuarios. Como administrador, puede ver los paquetes de acceso y las asignaciones de recursos para un usuario. Además, puede ver los registros de solicitudes para auditarlos o para determinar el estado de la solicitud de un usuario. En este artículo se describe cómo usar los informes de administración de derechos y los registros de auditoría de Azure AD.

Vea el siguiente vídeo para obtener información sobre cómo ver los recursos a los que tienen acceso los usuarios en la administración de derechos:

>[!VIDEO https://www.youtube.com/embed/omtNJ7ySjS0]

## <a name="view-access-packages-for-a-user"></a>Consulta de los paquetes de acceso de un usuario

Este informe le permite mostrar todos los paquetes de acceso que puede solicitar un usuario y los paquetes de acceso asignados actualmente al usuario.

**Rol necesario:** administrador global, administrador de Identity Governance o administrador de usuarios

1. Haga clic en **Azure Active Directory** y, luego, haga clic en **Gobernanza de identidades**.

1. En el menú izquierdo, haga clic en **Informes**.

1. Haga clic en **Paquetes de acceso de un usuario**.

1. Haga clic en **Seleccionar usuarios** para abrir el panel de selección de usuarios.

1. Busque el usuario en la lista y, a continuación, haga clic en **Seleccionar**.

    En la pestaña **Puede solicitar** se muestra una lista de los paquetes de acceso que puede solicitar el usuario. Esta lista viene determinada por las [directivas de solicitud](entitlement-management-access-package-request-policy.md#for-users-in-your-directory) definidas para los paquetes de acceso. 

    ![Paquetes de acceso de un usuario](./media/entitlement-management-reports/access-packages-report.png)

1. Si hay más de un rol o directiva de recursos para un paquete de acceso, haga clic en la entrada de roles o directivas de recursos para ver detalles de la selección.

1. Haga clic en la pestaña **Asignado** para ver una lista de los paquetes de acceso asignados actualmente al usuario. Si se asigna un paquete de acceso a un usuario, significa que el usuario tiene acceso a todos los roles de recurso del paquete de acceso.

## <a name="view-resource-assignments-for-a-user"></a>Consulta de las asignaciones de recursos de un usuario

Este informe le permite mostrar los recursos asignados actualmente a un usuario en la administración de derechos. Tenga en cuenta que este informe es para recursos administrados con la administración de derechos. El usuario podría tener acceso a otros recursos de su directorio fuera de la administración de derechos.

**Rol necesario:** administrador global, administrador de Identity Governance o administrador de usuarios

1. Haga clic en **Azure Active Directory** y, luego, haga clic en **Gobernanza de identidades**.

1. En el menú izquierdo, haga clic en **Informes**.

1. Haga clic en **Asignaciones de recursos de un usuario**.

1. Haga clic en **Seleccionar usuarios** para abrir el panel de selección de usuarios.

1. Busque el usuario en la lista y, a continuación, haga clic en **Seleccionar**.

    Se muestra una lista de los recursos asignados actualmente al usuario. En la lista también se muestran el paquete de acceso y la directiva de la que se obtuvo el rol de recurso, junto con la fecha de inicio y finalización para el acceso.
    
    Si un usuario obtuvo acceso al mismo recurso en dos o más paquetes, puede hacer clic en una flecha para ver cada paquete y directiva.

    ![Asignaciones de recursos de un usuario](./media/entitlement-management-reports/resource-assignments-report.png)

## <a name="determine-the-status-of-a-users-request"></a>Determinar el estado de la solicitud de un usuario

Para obtener más información sobre cómo un usuario ha solicitado y recibido acceso a un paquete de acceso, puede usar el registro de auditoría de Azure AD. En concreto, puede usar las entradas de registro de las categorías `EntitlementManagement` y `UserManagement` para obtener detalles adicionales sobre los pasos de procesamiento de cada solicitud.  

1. Haga clic en **Azure Active Directory** y, luego, haga clic en **Registros de auditoría**.

1. En la parte superior, cambie el valor de **Categoría** a `EntitlementManagement` o `UserManagement`, según el registro de auditoría que esté buscando.  

1. Haga clic en **Aplicar**.

1. Haga clic en **Descargar** para descargar los registros.

Cuando Azure AD recibe una solicitud nueva, escribe un registro de auditoría, donde el valor de **Categoría** es `EntitlementManagement` y el de **Actividad** suele ser `User requests access package assignment`.  En el caso de una asignación directa creada en Azure Portal, el campo **Actividad** del registro de auditoría es `Administrator directly assigns user to access package` y el usuario que realiza la asignación se identifica mediante el campo **ActorUserPrincipalName**.

Azure AD escribirá registros de auditoría adicionales mientras la solicitud esté en curso, que incluirán:

| Category | Actividad | Estado de la solicitud |
| :---- | :------------ | :------------ |
| `EntitlementManagement` | `Auto approve access package assignment request` | La solicitud no requiere aprobación |
| `UserManagement` | `Create request approval` | La solicitud requiere aprobación |
| `UserManagement` | `Add approver to request approval` | La solicitud requiere aprobación |
| `EntitlementManagement` | `Approve access package assignment request` | Solicitud aprobada |
| `EntitlementManagement` | `Ready to fulfill access package assignment request` |Solicitud aprobada o no requiere aprobación |

Cuando se asigna el acceso a un usuario, Azure AD escribe un registro de auditoría para la categoría `EntitlementManagement` con el valor **Actividad** `Fulfill access package assignment`.  El usuario que recibe el acceso se identifica mediante el campo **ActorUserPrincipalName**.

Si no se ha asignado el acceso, Azure AD escribe un registro de auditoría para la categoría `EntitlementManagement` con **Actividad** establecido en `Deny access package assignment request`, si un aprobador ha denegado la solicitud, o en `Access package assignment request timed out (no approver action taken)`, si la solicitud ha agotado el tiempo de espera antes de un aprobador pueda aprobarla.

Cuando la asignación de paquetes de acceso del usuario expira, la cancela el usuario o la quita un administrador, Azure AD escribe un registro de auditoría para la categoría `EntitlementManagement` con **Actividad** establecido en `Remove access package assignment`.

## <a name="next-steps"></a>Pasos siguientes

- [Archivar informes y registros](entitlement-management-logs-and-reporting.md)
- [Solución de problemas de la administración de derechos de Azure AD](entitlement-management-troubleshoot.md)
- [Escenarios comunes](entitlement-management-scenarios.md)
