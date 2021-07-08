---
title: Unidades administrativas en Azure Active Directory | Microsoft Docs
description: Utilice unidades administrativas para una delegación más detallada de permisos en Azure Active Directory.
services: active-directory
documentationcenter: ''
author: rolyon
manager: daveba
ms.service: active-directory
ms.topic: overview
ms.subservice: roles
ms.workload: identity
ms.date: 11/04/2020
ms.author: rolyon
ms.reviewer: anandy
ms.custom: oldportal;it-pro;
ms.collection: M365-identity-device-management
ms.openlocfilehash: e7071d180efe9f7da088f30106c879fd10ed9051
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110087119"
---
# <a name="administrative-units-in-azure-active-directory"></a>Unidades administrativas en Azure Active Directory

En este artículo se describen las unidades administrativas en Azure Active Directory (Azure AD). Una unidad administrativa es un recurso de Azure AD que puede ser un contenedor para otros recursos de Azure AD. Una unidad administrativa solo puede contener usuarios y grupos.

Las unidades administrativas restringen los permisos de un rol a cualquier parte de la organización que defina. Por ejemplo, podría usar unidades administrativas para delegar el rol [Administrador del departamento de soporte técnico](permissions-reference.md#helpdesk-administrator) a los especialistas de soporte técnico regionales, para que solo puedan administrar los usuarios de la región en cuestión.

## <a name="deployment-scenario"></a>Escenario de implementación

Puede resultar útil para restringir el ámbito administrativo mediante el uso de unidades administrativas en organizaciones compuestas de divisiones independientes de cualquier tipo. Considere el ejemplo de una universidad grande que se compone de muchas escuelas autónomas (Escuela de negocios, Escuela de ingeniería, etc.). Cada escuela tiene un equipo de administradores de TI que controlan el acceso, administran los usuarios y establecen directivas para su escuela.

Un administrador central podría:

- Crear un rol con permisos administrativos solo sobre los usuarios de Azure AD de la unidad administrativa de la Escuela de negocios.
- Crear una unidad administrativa para la Escuela de negocios.
- Rellenar la unidad administrativa solo con los alumnos y el personal de la Escuela de negocios.
- Agregar el equipo de TI de la Escuela de negocios al rol, junto con su ámbito.

## <a name="license-requirements"></a>Requisitos de licencia

El uso de unidades administrativas requiere una licencia de Azure AD Premium P1 para cada administrador de unidad administrativa y licencias de Azure AD Free para los miembros de las unidades administrativas. Para obtener la licencia correcta para sus requisitos, consulte [Comparación de las características con disponibilidad general de las ediciones Gratis y Prémium](https://azure.microsoft.com/pricing/details/active-directory/).

## <a name="manage-administrative-units"></a>Administración de unidades administrativas

Puede administrar las unidades administrativas mediante Azure Portal, cmdlets y scripts de PowerShell o Microsoft Graph. Para más información, consulte:

- [Crear, quitar, rellenar y agregar roles a las unidades administrativas](admin-units-manage.md): incluye procedimientos completos.
- [Trabajo con unidades administrativas](/powershell/azure/active-directory/working-with-administrative-units): se describe cómo trabajar con unidades administrativas mediante PowerShell.
- [Soporte de Graph en unidades administrativas](/graph/api/resources/administrativeunit): proporciona documentación detallada sobre Microsoft Graph para unidades administrativas.

### <a name="plan-your-administrative-units"></a>Planeamiento de las unidades administrativas

Puede utilizar unidades administrativas para agrupar recursos de Azure AD de manera lógica. Una organización cuyo departamento de TI está disperso a nivel global podría crear unidades administrativas que definan los límites geográficos pertinentes. En otro escenario, en el que una organización multinacional tiene distintas suborganizaciones que son semiautónomas en las operaciones, las unidades administrativas pueden representar a cada suborganización.

Los requisitos únicos de una organización guían los criterios según los cuales se crean las unidades administrativas. Las unidades administrativas son una forma habitual de definir la estructura en todos los servicios de Microsoft 365. Se recomienda que prepare las unidades administrativas teniendo en mente su uso en los servicios de Microsoft 365. Puede aprovechar al máximo las unidades administrativas cuando puede asociar recursos comunes en Microsoft 365 bajo una unidad administrativa.

Puede esperar que la creación de las unidades administrativas en la organización pase por las fases siguientes:

1. **Adopción inicial**: la organización empezará a crear unidades administrativas en función de los criterios iniciales y el número de unidades administrativas aumentará a medida que se perfeccionan los criterios.
1. **Eliminación**: una vez que los criterios están definidos, se eliminarán las unidades administrativas que ya no se requieran.
1. **Estabilización**: la estructura de la organización está definida y el número de unidades administrativas no va a cambiar significativamente a corto plazo.

## <a name="currently-supported-scenarios"></a>Escenarios admitidos actualmente

Como administrador global o administrador de roles con privilegios, puede usar Azure Portal para:

- Crear unidades administrativas
- Agregar usuarios y grupos como miembros de unidades administrativas
- Asignar personal de TI a los roles de administrador con ámbito de unidad administrativa.

Los administradores con ámbito de unidad administrativa pueden usar el centro de administración de Microsoft 365 para la administración básica de los usuarios de sus unidades administrativas. Un administrador de grupo con ámbito de unidad administrativa puede administrar grupos mediante PowerShell, Microsoft Graph y el centro de administración de Microsoft 365.

>[!Note]
>Solo están disponibles en el centro de administración de Microsoft 365 las características descritas en esta sección. No hay características de nivel de organización disponibles para un rol de Azure AD con ámbito de unidad administrativa.

En las secciones siguientes se describe la compatibilidad actual con los escenarios de unidades administrativas.

### <a name="administrative-unit-management"></a>Administración de unidades administrativas

| Permisos |   Graph y PowerShell   | Portal de Azure | Centro de administración de Microsoft 365 |
| --- | --- | --- | --- |
| Crear y eliminar unidades administrativas   |    Compatible    |   Compatible   |    No compatible |
| Agregar y quitar miembros de las unidades administrativas de manera individual    |   Compatible    |   Compatible   |    No compatible |
| Agregar o eliminar miembros de las unidades administrativas de forma masiva con archivos .csv   |    No compatible     |  Compatible   |    Sin planes de compatibilidad |
| Asignar administradores con ámbito de unidad administrativa  |     Compatible    |   Compatible    |   No compatible |
| Agregar y eliminar miembros de las unidades administrativas de manera dinámica en función de los atributos | No compatible | No compatible | No compatible

### <a name="user-management"></a>Administración de usuarios

| Permisos |   Graph y PowerShell   | Portal de Azure | Centro de administración de Microsoft 365 |
| --- | --- | --- | --- |
| Administración con ámbito de unidad administrativa de las propiedades de usuario, contraseñas y licencias   |    Compatible     |  Compatible   |   Compatible |
| Bloqueo y desbloqueo con ámbito de unidad administrativa de los inicios de sesión de usuario    |   Compatible   |    Compatible   |    Compatible |
| Administración con ámbito de unidad administrativa de las credenciales de autenticación multifactor del usuario   |    Compatible   |   Compatible   |   No compatible |

### <a name="group-management"></a>Administración de grupos

| Permisos |   Graph y PowerShell   | Portal de Azure | Centro de administración de Microsoft 365 |
| --- | --- | --- | --- |
| Administración con ámbito de unidad administrativa de los miembros y las propiedades de un grupo     |  Compatible   |    Compatible    |  No compatible |
| Administración con ámbito de unidad administrativa de las licencias de grupo   |    Compatible  |    Compatible   |   No compatible |

Las unidades administrativas solo aplican el ámbito a los permisos de administración. No impiden que los miembros o los administradores usen sus [permisos de usuario predeterminados](../fundamentals/users-default-permissions.md) para examinar otros usuarios, grupos o recursos fuera de la unidad administrativa. En el centro de administración de Microsoft 365, se filtran los usuarios que están fuera de las unidades administrativas de un administrador con ámbito. Sin embargo, puede examinar otros usuarios en Azure Portal, PowerShell y otros servicios de Microsoft.

## <a name="next-steps"></a>Pasos siguientes

- [Administración de unidades administrativas](admin-units-manage.md)
- [Adición y administración de usuarios en las unidades administrativas en Azure Active Directory](admin-units-add-manage-users.md)
- [Adición y administración de grupos en las unidades administrativas en Azure Active Directory](admin-units-add-manage-groups.md)
- [Asignación de roles con ámbito a una unidad administrativa](admin-units-assign-roles.md)
