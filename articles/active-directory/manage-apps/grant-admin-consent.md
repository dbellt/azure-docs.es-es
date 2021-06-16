---
title: Concesión del consentimiento del administrador para todo el inquilino a una aplicación - Azure AD
description: Obtenga información sobre cómo conceder consentimiento para todo el inquilino a una aplicación de modo que no se les solicite consentimiento a los usuarios finales al iniciar sesión en una aplicación.
services: active-directory
author: mtillman
manager: CelesteDG
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: how-to
ms.date: 11/04/2019
ms.author: mtillman
ms.reviewer: phsignor
ms.collection: M365-identity-device-management
ms.openlocfilehash: 836636dfa5cc817685ca815485aa4dba980f1e5a
ms.sourcegitcommit: 3bb9f8cee51e3b9c711679b460ab7b7363a62e6b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112081360"
---
# <a name="grant-tenant-wide-admin-consent-to-an-application"></a>Concesión del consentimiento del administrador para todo el inquilino a una aplicación

  Obtenga información acerca de cómo conceder el consentimiento del administrador para todo el inquilino a una aplicación. En este artículo se proporcionan las distintas formas de lograrlo.

Para más información acerca de cómo dar consentimiento a las aplicaciones, consulte [Marco de consentimiento de Azure Active Directory](../develop/consent-framework.md).

## <a name="prerequisites"></a>Prerrequisitos

La concesión del consentimiento del administrador para todo el inquilino requiere que inicie sesión como un usuario que esté autorizado para dar su consentimiento en nombre de la organización. Esto incluye a los roles de [administrador global](../roles/permissions-reference.md#global-administrator) y [administrador de roles con privilegios](../roles/permissions-reference.md#privileged-role-administrator) y, para algunas aplicaciones, [administrador de aplicaciones](../roles/permissions-reference.md#application-administrator) y [administrador de aplicaciones en la nube](../roles/permissions-reference.md#cloud-application-administrator). También se puede autorizar a un usuario para conceder consentimiento para todo el inquilino si se le asigna un [rol de directorio personalizado](../roles/custom-create.md) que incluya el [permiso para conceder permisos a las aplicaciones](../roles/custom-consent-permissions.md).

> [!WARNING]
> La concesión del consentimiento del administrador para todo el inquilino a una aplicación concederá acceso a la aplicación y al publicador de la aplicación a los datos de la organización. Antes de conceder el consentimiento, revise con atención los permisos que solicita la aplicación.

> [!IMPORTANT]
> Cuando a una aplicación se le concede consentimiento del administrador para todo el inquilino, todos los usuarios podrán iniciar sesión en la aplicación a menos que se haya configurado para requerir la asignación de usuarios. Para restringir qué usuarios pueden iniciar sesión en una aplicación, debe requerir la asignación de usuarios y, luego, asignar usuarios o grupos a la aplicación. Para más información, consulte [Métodos para asignar usuarios y grupos](./assign-user-or-group-access-portal.md).

## <a name="grant-admin-consent-from-the-azure-portal"></a>Concesión del consentimiento del administrador desde Azure Portal

### <a name="grant-admin-consent-in-enterprise-apps"></a>Concesión del consentimiento del administrador en aplicaciones empresariales

Puede conceder el consentimiento del administrador para todo el inquilino a través de *Aplicaciones empresariales* si la aplicación ya se ha aprovisionado en el inquilino. Por ejemplo, se podría aprovisionar una aplicación en el inquilino si al menos un usuario ya ha dado su consentimiento a la aplicación. Para más información, vea [Cómo y por qué se agregan aplicaciones a Azure AD](../develop/active-directory-how-applications-are-added.md).

Para conceder el consentimiento del administrador para todo el inquilino a una de las aplicaciones mostradas en **Aplicaciones empresariales**:

1. Inicie sesión en [Azure Portal](https://portal.azure.com) como [Administrador global](../roles/permissions-reference.md#global-administrator), [Administrador de aplicaciones](../roles/permissions-reference.md#application-administrator) o [Administrador de aplicaciones en la nube](../roles/permissions-reference.md#cloud-application-administrator).
2. Seleccione **Azure Active Directory** y después **Aplicaciones empresariales**.
3. Seleccione la aplicación a la que quiera conceder el consentimiento del administrador para todo el inquilino.
4. Seleccione **Permisos** y, a continuación, haga clic en **Conceder consentimiento de administrador**.
5. Revise cuidadosamente los permisos que requiere la aplicación.
6. Si está de acuerdo con los permisos que requiere la aplicación, conceda el consentimiento. En caso contrario, haga clic en **Cancelar** o cierre la ventana.

> [!WARNING]
> La concesión del consentimiento de administrador para todo el inquilino a través de **aplicaciones empresariales** revocará todos los permisos concedidos previamente para todo el inquilino. Los permisos concedidos previamente por los usuarios en su propio nombre no se verán afectados. 

### <a name="grant-admin-consent-in-app-registrations"></a>Concesión del consentimiento del administrador en Registros de aplicaciones

Para las aplicaciones desarrolladas por la organización, o bien las que se hayan registrado directamente en el inquilino de Azure AD, también puede conceder el consentimiento del administrador para todo el inquilino desde **Registros de aplicaciones** en Azure Portal.

Para conceder el consentimiento del administrador para todo el inquilino desde **Registros de aplicaciones**:

1. Inicie sesión en [Azure Portal](https://portal.azure.com) como [Administrador global](../roles/permissions-reference.md#global-administrator), [Administrador de aplicaciones](../roles/permissions-reference.md#application-administrator) o [Administrador de aplicaciones en la nube](../roles/permissions-reference.md#cloud-application-administrator).
2. Seleccione **Azure Active Directory** y después **Registros de aplicaciones**.
3. Seleccione la aplicación a la que quiera conceder el consentimiento del administrador para todo el inquilino.
4. Seleccione **Permisos de API** y, después, haga clic en **Conceder consentimiento del administrador**.
5. Revise cuidadosamente los permisos que requiere la aplicación.
6. Si está de acuerdo con los permisos que requiere la aplicación, conceda el consentimiento. En caso contrario, haga clic en **Cancelar** o cierre la ventana.

> [!WARNING]
> La concesión del consentimiento de administrador para todo el inquilino a través de **registros de aplicaciones** revocará todos los permisos concedidos previamente para todo el inquilino. Los permisos concedidos previamente por los usuarios en su propio nombre no se verán afectados. 

## <a name="construct-the-url-for-granting-tenant-wide-admin-consent"></a>Construcción de la dirección URL para conceder el consentimiento del administrador para todo el inquilino

Al conceder el consentimiento del administrador para todo el inquilino con cualquiera de los métodos descritos antes, se abre una ventana desde Azure Portal para solicitar el consentimiento del administrador para todo el inquilino. Si conoce el identificador de cliente (también conocido como id. de la aplicación) de la aplicación, puede crear la misma dirección URL para conceder consentimiento del administrador para todo el inquilino.

La dirección URL de consentimiento del administrador para todo el inquilino tiene el formato siguiente:

```http
https://login.microsoftonline.com/{tenant-id}/adminconsent?client_id={client-id}
```

donde:

* `{client-id}` es el identificador de cliente de la aplicación (también conocido como id. de la aplicación).
* `{tenant-id}` es el identificador de inquilino de la organización o cualquier nombre de dominio comprobado.

Como siempre, antes de conceder el consentimiento, revise con atención los permisos que solicita la aplicación.

> [!WARNING]
> La concesión del consentimiento de administrador para todo el inquilino a través de esta dirección URL revocará todos los permisos concedidos previamente para todo el inquilino. Los permisos concedidos previamente por los usuarios en su propio nombre no se verán afectados. 

## <a name="next-steps"></a>Pasos siguientes

[Configuración del consentimiento de los usuarios finales a las aplicaciones](configure-user-consent.md)

[Configuración del flujo de trabajo de consentimiento del administrador](configure-admin-consent-workflow.md)

[Permisos y consentimiento en la plataforma de identidad de Microsoft](../develop/v2-permissions-and-consent.md)

[Azure AD en Microsoft Q&A](/answers/topics/azure-active-directory.html)
