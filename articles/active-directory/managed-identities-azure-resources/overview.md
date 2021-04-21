---
title: Identidades administradas de recursos de Azure
description: Información general sobre Managed Identities for Azure Resources.
services: active-directory
documentationcenter: ''
author: barclayn
manager: daveba
editor: ''
ms.assetid: 0232041d-b8f5-4bd2-8d11-27999ad69370
ms.service: active-directory
ms.subservice: msi
ms.devlang: ''
ms.topic: overview
ms.custom: mvc
ms.date: 04/07/2021
ms.author: barclayn
ms.collection: M365-identity-device-management
ms.openlocfilehash: 7fabb8bbdb42212dffd3781f4e98204abb518e6b
ms.sourcegitcommit: 5f482220a6d994c33c7920f4e4d67d2a450f7f08
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/08/2021
ms.locfileid: "107105585"
---
# <a name="what-are-managed-identities-for-azure-resources"></a>¿Qué son las identidades administradas de recursos de Azure?

Una dificultad común para los desarrolladores es la administración de los secretos y las credenciales usados para proteger la comunicación entre los distintos componentes que constituyen una solución. Las identidades administradas eliminan la necesidad de que los desarrolladores administren las credenciales. Proporcionan una identidad que usan las aplicaciones al conectarse a recursos que admiten la autenticación de Azure Active Directory (Azure AD). Las aplicaciones pueden usar la identidad administrada para obtener tokens de Azure AD. Por ejemplo, una aplicación puede usar una identidad administrada para acceder a recursos como [Azure Key Vault](../../key-vault/general/overview.md) donde los desarrolladores pueden almacenar credenciales de forma segura o tener acceso a cuentas de almacenamiento.

¿Para qué se puede usar una identidad administrada?</br>

> [!VIDEO https://www.youtube.com/embed/5lqayO_oeEo]

Estas son algunas de las ventajas de usar las identidades administradas:

- No es necesario administrar credenciales. Las credenciales ni siquiera están accesible.
- Puede usar identidades administradas para autenticarse en cualquier recurso que admita la [autenticación de Azure Active Directory](../authentication/overview-authentication.md), incluidas sus propias aplicaciones.
- Las identidades administradas se pueden usar sin ningún costo adicional.

> [!NOTE]
> Identidades administradas para recursos de Azure es el nombre con el que ahora se conoce al servicio Managed Service Identity (MSI).

## <a name="managed-identity-types"></a>Tipos de identidad administrada

Hay dos tipos de identidades administradas:

- **Asignada por el sistema**. Algunos servicios de Azure permiten habilitar una identidad administrada directamente en una instancia de servicio. Cuando se habilita una identidad administrada asignada por el sistema, se crea una identidad en Azure AD que está vinculada al ciclo de vida de esa instancia de servicio. Por tanto, cuando se elimina el recurso, Azure elimina automáticamente la identidad. Por diseño, solo ese recurso de Azure puede usar esta identidad para solicitar tokens de Azure AD.
- **Asignada por el usuario**. También es posible crear una identidad administrada como un recurso independiente de Azure. Puede [crear una identidad administrada asignada por el usuario](how-to-manage-ua-identity-portal.md) y asignarla a una o varias instancias de un servicio de Azure. En el caso de las identidades administradas asignadas por el usuario, la identidad se administra independientemente de los recursos que la utilicen. </br></br>

> [!VIDEO https://www.youtube.com/embed/OzqpxeD3fG0]

En la tabla siguiente, se muestran las diferencias entre los dos tipos de identidades administradas.

|  Propiedad    | Identidad administrada asignada por el sistema | Identidad administrada asignada por el usuario |
|------|----------------------------------|--------------------------------|
| Creación |  Se crea como parte de un recurso de Azure (por ejemplo, una máquina virtual de Azure o Azure App Service) | Se crea como un recurso de Azure independiente |
| Ciclo de vida | Se comparte el ciclo de vida con el recurso de Azure con el que se creó la identidad administrada. <br/> Si se elimina el recurso primario, se elimina también la identidad administrada. | Ciclo de vida independiente. <br/> Se debe eliminar explícitamente. |
| Uso compartido de recursos de Azure | No se puede compartir. <br/> Solo se puede asociar con un único recurso de Azure. | Se puede compartir <br/> La misma identidad administrada asignada por el usuario se puede asociar con más de un recurso de Azure. |
| Casos de uso comunes | Cargas de trabajo contenidas en un único recurso de Azure <br/> Cargas de trabajo para las que necesita identidades independientes. <br/> Por ejemplo, una aplicación que se ejecuta en una sola máquina virtual | Cargas de trabajo que se ejecutan en varios recursos y que pueden compartir una única identidad. <br/> Cargas de trabajo que necesitan autorización previa para un recurso seguro como parte de un flujo de aprovisionamiento. <br/> Cargas de trabajo donde los recursos se reciclan con frecuencia, pero los permisos deben permanecer coherentes. <br/> Por ejemplo, una carga de trabajo en la que varias máquinas virtuales tienen que acceder al mismo recurso |

> [!IMPORTANT]
> Independientemente del tipo de identidad elegido, una identidad administrada es una entidad de servicio de un tipo especial que solo se puede usar con recursos de Azure. Cuando se elimina la identidad administrada, se quita automáticamente la entidad de servicio correspondiente.

## <a name="how-can-i-use-managed-identities-for-azure-resources"></a>¿Cómo se usan las identidades administradas de Managed Identities for Azure Resources?

![Algunos ejemplos de cómo un desarrollador puede usar identidades administradas para obtener acceso a los recursos desde el código, sin tener que administrar la información de autenticación.](media/overview/when-use-managed-identities.png)

## <a name="what-azure-services-support-the-feature"></a>¿Qué servicios de Azure admiten la característica?<a name="which-azure-services-support-managed-identity"></a>

Las identidades administradas de Managed Identities for Azure Resources se pueden usar para autenticarse en servicios que admiten la autenticación de Azure AD. Para ver una lista de los servicios de Azure que admiten la característica Managed Identities for Azure Resources, consulte [Services that support managed identities for Azure resources](./services-support-managed-identities.md) (Servicios que admiten la característica Managed Identities for Azure Resources).

## <a name="which-operations-can-i-perform-using-managed-identities"></a>¿Qué operaciones puedo realizar mediante identidades administradas?

Los recursos que admiten identidades administradas asignadas por el sistema le permiten:

- Habilitar o deshabilitar las identidades administradas en el nivel de recurso.
- Usar roles de RBAC para [conceder permisos](howto-assign-access-portal.md).
- Ver las operaciones de creación, lectura, actualización y eliminación (CRUD) en los [registros de actividad de Azure](../../azure-resource-manager/management/view-activity-logs.md).
- Ver la actividad de inicio de sesión en los [registros de inicio de sesión](../reports-monitoring/concept-sign-ins.md) de Azure AD.

Si en su lugar elige una identidad administrada asignada por el usuario:

- Puede [crear, leer, actualizar y eliminar](how-to-manage-ua-identity-portal.md) las identidades.
- Puede usar asignaciones de roles de RBAC para [conceder permisos](howto-assign-access-portal.md).
- Las identidades administradas asignadas por el usuario se pueden usar en más de un recurso.
- Las operaciones CRUD están disponibles para su revisión en los [registros de actividad de Azure](../../azure-resource-manager/management/view-activity-logs.md).
- Ver la actividad de inicio de sesión en los [registros de inicio de sesión](../reports-monitoring/concept-sign-ins.md) de Azure AD.

Las operaciones sobre identidades administradas se pueden realizar mediante una plantilla de Azure Resource Manager (ARM), Azure Portal, la CLI de Azure, PowerShell y las API REST.

## <a name="next-steps"></a>Pasos siguientes

* [Uso de las identidades administradas asignadas por el sistema de una máquina virtual Windows para acceder a Resource Manager](tutorial-windows-vm-access-arm.md)
* [Uso de las identidades administradas asignadas por el sistema de una máquina virtual Linux para acceder a Resource Manager](tutorial-linux-vm-access-arm.md)
* [Cómo usar identidades administradas para App Service y Azure Functions](../../app-service/overview-managed-identity.md)
* [Cómo utilizar una identidad administrada con Azure Container Instances](../../container-instances/container-instances-managed-identity.md)
* [Implementación de identidades administradas para recursos de Microsoft Azure](https://www.pluralsight.com/courses/microsoft-azure-resources-managed-identities-implementing).
