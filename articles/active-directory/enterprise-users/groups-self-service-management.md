---
title: Configuración de la administración de grupos de autoservicio en Azure Active Directory | Microsoft Docs
description: Creación y administración de grupos de seguridad o grupos de Microsoft 365 en Azure Active Directory y solicitud de la pertenencia a estos grupos
services: active-directory
documentationcenter: ''
author: curtand
manager: daveba
editor: ''
ms.service: active-directory
ms.subservice: enterprise-users
ms.workload: identity
ms.topic: how-to
ms.date: 05/18/2021
ms.author: curtand
ms.reviewer: krbain
ms.custom: it-pro;seo-update-azuread-jan
ms.collection: M365-identity-device-management
ms.openlocfilehash: 47e3ea0a8ea5dc8dbb01d532a52436ed581311e7
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110089904"
---
# <a name="set-up-self-service-group-management-in-azure-active-directory"></a>Configuración de la administración de grupos de autoservicio en Azure Active Directory 

Puede permitir que los usuarios creen y administren sus propios grupos de seguridad o grupos de Microsoft 365 en Azure Active Directory (Azure AD). El propietario del grupo puede aprobar o rechazar solicitudes de pertenencia y puede delegar el control de la pertenencia a grupos. Las características de administración de grupos de autoservicio no están disponibles para grupos de seguridad habilitados para correo electrónico o listas de distribución.

## <a name="self-service-group-membership-defaults"></a>Valores predeterminados de pertenencia a grupos de autoservicio

Cuando se crean grupos de seguridad en Azure Portal o con Azure AD PowerShell, solo los propietarios del grupo pueden actualizar pertenencia. Los grupos de seguridad creados mediante autoservicio en el [Panel de acceso](https://account.activedirectory.windowsazure.com/r#/joinGroups) y todos los grupos de Microsoft 365 están disponibles para que se unan todos los usuarios, tanto los aprobados por el propietario como los aprobados automáticamente. En el Panel de acceso, puede cambiar las opciones de pertenencia al crear el grupo.

Grupos creados en | Comportamiento predeterminado del grupo de seguridad | Comportamiento predeterminado del grupo de Microsoft 365
------------------ | ------------------------------- | ---------------------------------
[Azure AD PowerShell](../enterprise-users/groups-settings-cmdlets.md) | Solo los propietarios pueden agregar miembros<br>Visible pero no está disponible para unión en el Panel de acceso | Abierto para unirse todos los usuarios
[Azure Portal](https://portal.azure.com) | Solo los propietarios pueden agregar miembros<br>Visible pero no está disponible para unión en el Panel de acceso<br>El propietario no se asigna automáticamente durante la creación del grupo | Abierto para unirse todos los usuarios
[Panel de acceso](https://account.activedirectory.windowsazure.com/r#/joinGroups) | Abierto para unirse todos los usuarios<br>Las opciones de pertenencia se pueden cambiar cuando se crea el grupo | Abierto para unirse todos los usuarios<br>Las opciones de pertenencia se pueden cambiar cuando se crea el grupo

## <a name="self-service-group-management-scenarios"></a>Escenarios de administración de grupos de autoservicio

* **Administración de grupos delegados** Por ejemplo, un administrador que administra el acceso a una aplicación SaaS que su compañía usa. La administración de estos derechos de acceso se está volviendo compleja, por lo que este administrador solicita al propietario de la empresa la creación de un nuevo grupo. El administrador asigna al nuevo grupo acceso a la aplicación y le agrega todas las personas que ya acceden a la aplicación. Luego, el propietario de la empresa puede agregar más usuarios, y dichos usuarios se aprovisionan automáticamente en la aplicación. No es preciso que el propietario espere a que el administrador administre el acceso de los usuarios. Si el administrador concede el mismo permiso a un administrador de otro grupo de negocios, dicha persona también puede administrar el acceso de sus propios usuarios. Ni el propietario de una empresa ni el administrador pueden ver ni administrar las pertenencias al grupo del otro. El administrador podrá seguir viendo todos los usuarios que tienen acceso a la aplicación y bloquear los derechos de acceso si fuera necesario.
* **Administración de grupos de autoservicio** Por ejemplo, dos usuarios tienen sitios de SharePoint Online configurados de forma independiente. Ellos quieren dar al equipo del otro acceso a sus sitios. Para ello, pueden crear un grupo en Azure AD, y en SharePoint Online cada uno de ellos selecciona dicho grupo para proporcionar acceso a sus sitios. Cuando alguien desea tener acceso, lo solicita en el Panel de acceso, y tras la aprobación obtiene acceso a ambos sitios de SharePoint Online automáticamente. Posteriormente, uno de ellos decide que todas las personas que accedan a su sitio también deben obtener acceso a una aplicación SaaS concreta. El administrador de la aplicación SaaS puede agregar derechos de acceso a la aplicación en el sitio de SharePoint Online. Desde ese momento, todas las solicitudes aprobadas darán acceso tanto a los dos sitios de SharePoint Online como a la aplicación SaaS.

## <a name="make-a-group-available-for-user-self-service"></a>Puesta a disposición de un grupo para el autoservicio del usuario

1. Inicie sesión en el [Centro de administración de Azure AD](https://aad.portal.azure.com) con una cuenta a la que se haya asignado el rol Administrador global o Administrador de roles con privilegios para el directorio.

1. Seleccione **Grupos** y, a continuación, seleccione el valor **General**.

    ![Configuración general de grupos de Azure Active Directory](./media/groups-self-service-management/groups-settings-general.png)

1. Establezca **Los propietarios pueden administrar solicitudes de pertenencia a grupos en el Panel de acceso** en **Sí**.

1. Establezca **Restringir la capacidad del usuario para acceder a las características de grupos del panel de acceso** en **No**.

1. Si establece **Los usuarios pueden crear grupos de seguridad en Azure Portal, la API o PowerShell** o **Los usuarios pueden crear grupos de Microsoft 365 en Azure Portal, la API o PowerShell** en

    - **Sí**: todos los usuarios de la organización de Azure AD pueden crear nuevos grupos de seguridad y agregar miembros a estos grupos en Azure Portal, la API o PowerShell. Estos grupos nuevos también se muestran en el Panel de acceso para los restantes usuarios. Si la configuración de la directiva en el grupo lo permite, otros usuarios pueden crear solicitudes para unirse a dichos grupos.
    - **No**: los usuarios no pueden crear grupos ni cambiar los grupos existentes de los que sean propietarios. Sin embargo, pueden administrar la pertenencia a dichos grupos y aprobar las solicitudes de otros usuarios para unirse a ellos.

    Esta configuración se cambió recientemente para agregar compatibilidad con la API y PowerShell. Para obtener más información sobre este cambio, vea la siguiente sección [Cambio de configuración de grupos](#groups-setting-change).

También puede usar **Owners who can assign members as group owners in the Azure portal** (Los propietarios pueden asignar miembros como propietarios de grupos en Azure Portal) para conseguir un control más pormenorizado sobre la administración de grupos de autoservicio para los usuarios.

Cuando los usuarios puedan crear grupos, todos los usuarios de su organización podrán crear nuevos grupos y, a continuación, podrán, como propietarios predeterminados, agregarles miembros. No puede especificar a personas que puedan crear sus propios grupos. Solo puede especificar a personas para convertir a otro miembro del grupo en propietario del grupo.

> [!NOTE]
> Se requiere una licencia de Azure Active Directory Premium (P1 o P2) para que los usuarios puedan solicitar unirse a un grupo de seguridad o a un grupo de Microsoft 365 y para que los propietarios puedan aprobar o denegar las solicitudes de pertenencia. Sin una licencia de Azure Active Directory Premium, los usuarios todavía pueden administrar sus grupos en el Panel de acceso, pero no pueden crear un grupo que requiera la aprobación del propietario en el Panel de acceso y no podrán solicitar unirse a un grupo.

## <a name="groups-setting-change"></a>Cambio de configuración de grupos

La configuración de los grupos de seguridad actuales y los grupos de Microsoft 365 se están dejando en desuso y reemplazando. La configuración actual se está reemplazando porque solo controla la creación de grupos en Azure Portal y no se aplica a la API ni a PowerShell. La nueva configuración controla la creación de grupos en Azure Portal, así como en la API y en PowerShell.

| Configuración en desuso | Nueva configuración |
| --- | --- |
| Los usuarios pueden crear grupos de seguridad en Azure Portal. | Los usuarios pueden crear grupos de seguridad en Azure Portal, la API o PowerShell. |
| Los usuarios pueden crear grupos de Microsoft 365 en Azure Portal. | Los usuarios pueden crear grupos de Microsoft 365 en Azure Portal, la API o PowerShell. |

Hasta que la configuración actual esté totalmente en desuso, ambas opciones aparecerán en Azure Portal. Debe configurar esta nueva opción antes de finales de **mayo de 2021**. Para configurar los grupos de seguridad, debe tener asignado el rol Administrador global o Administrador de roles con privilegios. 

![Cambio de configuración de grupos de seguridad de Azure Active Directory](./media/groups-self-service-management/security-groups-setting.png)

La tabla siguiente le ayudará a decidir qué valores elegir.

| Si desea: | Elija estos valores |
| --- | --- |
| Los usuarios pueden crear grupos mediante Azure Portal, API o PowerShell | Establezca ambas opciones en **Sí**. Los cambios pueden tardar hasta 15 minutos en surtir efecto. |
| Los usuarios **no pueden** crear grupos mediante Azure Portal, la API ni PowerShell | Establezca ambos valores en **No**. Los cambios pueden tardar hasta 15 minutos en surtir efecto. |
| Los usuarios pueden crear grupos mediante Azure Portal, pero no mediante la API ni PowerShell | No compatible |
| Los usuarios pueden crear grupos mediante la API o PowerShell, pero no mediante Azure Portal | No compatible |

En la tabla siguiente se muestra lo que sucede para los distintos valores de esta configuración. No se recomienda tener la configuración en desuso y la nueva configuración establecidas en valores diferentes.

| Los usuarios pueden crear grupos mediante Azure Portal | Los usuarios pueden crear grupos mediante Azure Portal, API o PowerShell | Efecto en el inquilino |
| :---: | :---: | --- |
| Sí | Sí | Los usuarios pueden crear grupos mediante Azure Portal, la API o PowerShell. Los cambios pueden tardar hasta 15 minutos en surtir efecto.|
| No | No | Los usuarios **no pueden** crear grupos mediante Azure Portal, la API ni PowerShell. Los cambios pueden tardar hasta 15 minutos en surtir efecto. |
| Sí | No | Los usuarios **no pueden** crear grupos mediante Azure Portal, la API ni PowerShell. No se recomienda establecer esta configuración en valores diferentes. Los cambios pueden tardar hasta 15 minutos en surtir efecto. |
| No | Sí | Hasta que la opción **Los usuarios pueden crear grupos mediante Azure Portal** esté totalmente en desuso en **junio de 2021**, los usuarios pueden crear grupos mediante la API o PowerShell, pero no con Azure Portal. A partir de algún día indeterminado de **junio de 2021**, la opción **Los usuarios pueden crear grupos mediante Azure Portal, la API o PowerShell.** se hará efectiva y los usuarios podrán crear grupos mediante Azure Portal, la API o PowerShell. |

## <a name="next-steps"></a>Pasos siguientes

Estos artículos proporcionan información adicional sobre Azure Active Directory.

* [Administración del acceso a recursos con grupos de Azure Active Directory](../fundamentals/active-directory-manage-groups.md)
* [Cmdlets de Azure Active Directory para configurar las opciones de grupo](../enterprise-users/groups-settings-cmdlets.md)
* [Administración de aplicaciones en Azure Active Directory](../manage-apps/what-is-application-management.md)
* [¿Qué es Azure Active Directory?](../fundamentals/active-directory-whatis.md)
* [Integre las identidades locales con Azure Active Directory](../hybrid/whatis-hybrid-identity.md)
