---
title: Protección de identidades administradas en Azure Active Directory
description: Explicación de cómo buscar, evaluar y aumentar la seguridad de las identidades administradas.
services: active-directory
author: BarbaraSelden
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: fundamentals
ms.topic: conceptual
ms.date: 3/1/2021
ms.author: baselden
ms.reviewer: ajburnle
ms.custom: it-pro, seodec18
ms.collection: M365-identity-device-management
ms.openlocfilehash: 0a08c464e5c6a1fb5d4337e0902c9bd84b3b7772
ms.sourcegitcommit: eda26a142f1d3b5a9253176e16b5cbaefe3e31b3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2021
ms.locfileid: "109732927"
---
# <a name="securing-managed-identities"></a>Protección de identidades administradas

Los desarrolladores suelen enfrentarse al desafío de tener que administrar los secretos y las credenciales utilizados para proteger la comunicación entre diferentes servicios. Las identidades administradas son identidades de Azure Active Directory (Azure AD) seguras creadas para proporcionar identidades para los recursos de Azure.

## <a name="benefits-of-using-managed-identities-for-azure-resources"></a>Ventajas del uso de identidades administradas para los recursos de Azure

A continuación se detallan las ventajas del uso de identidades administradas:

* No es necesario administrar credenciales. Con las identidades administradas, Azure administra totalmente, rota y protege las credenciales. Las identidades se proporcionan y eliminan automáticamente con los recursos de Azure. Las identidades administradas permiten a los recursos de Azure comunicarse con todos los servicios que admiten la autenticación de Azure AD.

* Nadie (incluido cualquier administrador global) tiene acceso a las credenciales, por lo que no se pueden filtrar de forma accidental, por ejemplo, mediante su incorporación en el código.

## <a name="when-to-use-managed-identities"></a>¿Cuándo se deben usar las identidades administradas?

Las identidades administradas son más útiles para las comunicaciones entre los servicios que admiten la autenticación de Azure AD. 

Un sistema de origen solicita acceso a un servicio de destino. Cualquier recurso de Azure puede actuar como un sistema de origen. Por ejemplo, una máquina virtual de Azure, una instancia de Azure Functions y las instancias de Azure App Service admiten identidades administradas.

   > [!VIDEO https://www.youtube.com/embed/5lqayO_oeEo]

### <a name="how-authentication-and-authorization-work"></a>Funcionamiento de la autenticación y la autorización

Con las identidades administradas, el sistema de origen puede obtener un token de Azure AD sin que el propietario del origen tenga que administrar las credenciales. Azure administra las credenciales. El token obtenido por el sistema de origen se presenta al sistema de destino para la autenticación. 

El sistema de destino debe autenticarse (identificarse) y autorizar al sistema de origen antes de permitir el acceso. Cuando el servicio de destino admite la autenticación basada en Azure AD, acepta un token de acceso emitido por Azure AD. 

Azure tiene un plano de control y un plano de datos. Los recursos se crean en el plano de control y se accede a ellos en el plano de datos. Por ejemplo, puede crear una base de datos de Cosmos DB en el plano de control, pero consultarla en el plano de datos.

Una vez que el sistema de destino acepta el token para la autenticación, puede admitir distintos mecanismos de autorización para el plano de control y el plano de datos.

Todas las operaciones del plano de control de Azure se administran mediante [Azure Resource Manager](../../azure-resource-manager/management/overview.md) y usan el [control de acceso basado en rol de Azure](../../role-based-access-control/overview.md). En el plano de datos, cada sistema de destino tiene su propio mecanismo de autorización. Azure Storage admite Azure RBAC en el plano de datos. Por ejemplo, las aplicaciones que usan Azure App Service pueden leer los datos de Azure Storage, y las aplicaciones que usan Azure Kubernetes Service pueden leer los secretos almacenados en Azure Key Vault.

Para más información sobre el plano de control y el plano de datos, vea [Operaciones en el plano de control y el plano de datos: Azure Resource Manager](../../azure-resource-manager/management/control-plane-and-data-plane.md).

Finalmente, todos los servicios de Azure admitirán identidades administradas. Para más información, vea [Servicios que admiten identidades administradas para recursos de Azure](../managed-identities-azure-resources/services-support-managed-identities.md).

## <a name="types-of-managed-identities"></a>Tipos de identidades administradas

Hay dos tipos de identidades administradas: asignadas por el sistema y asignadas por el usuario.

Las identidades administradas asignadas por el sistema tienen las siguientes propiedades:

* Tienen una relación 1:1 con el recurso de Azure. Por ejemplo, hay una identidad administrada única asociada a cada máquina virtual.

* Están vinculadas al ciclo de vida de los recursos de Azure. Cuando se elimina el recurso, la identidad administrada asociada a él se elimina automáticamente, de tal forma que se elimina el riesgo asociado a las cuentas huérfanas. 

Las identidades administradas asignadas por el usuario tienen las siguientes propiedades:

* El ciclo de vida de estas identidades es independiente de un recurso de Azure, y se debe administrar el ciclo de vida. Cuando se elimina el recurso de Azure, la identidad administrada asignada por el usuario asignada no se elimina automáticamente.

* Una única identidad administrada asignada por el usuario se puede asignar a cero o a más recursos de Azure.

* Se puede crear de antemano y, a continuación, asignarse a un recurso.

## <a name="find-managed-identity-service-principals-in-azure-ad"></a>Búsqueda de entidades de servicio de identidades administradas en Azure AD

Hay varias maneras de buscar identidades administradas:

* Uso de la página Aplicaciones empresariales en Azure Portal

* Uso de Microsoft Graph

### <a name="using-the-azure-portal"></a>Uso de Azure Portal

1. En Azure Active Directory, seleccione Aplicaciones empresariales.

2. Seleccione el filtro para "Identidades administradas". 

   ![Imagen de la pantalla Todas las aplicaciones con la lista desplegable Tipo de aplicación, donde la opción "Identidades administradas" aparece resaltada.](./media/securing-service-accounts/service-accounts-managed-identities.png)

 

### <a name="using-microsoft-graph"></a>Uso de Microsoft Graph

Puede obtener una lista de todas las identidades administradas en el inquilino con la siguiente solicitud GET a Microsoft Graph:

`https://graph.microsoft.com/v1.0/servicePrincipals?$filter=(servicePrincipalType eq 'ManagedIdentity') `

Puede filtrar estas solicitudes. Para más información, vea la documentación de Graph para [Obtener servicePrincipal](/graph/api/serviceprincipal-get).

## <a name="assess-the-security-of-managed-identities"></a>Evaluación de la seguridad de las identidades administrads 

Puede evaluar la seguridad de las identidades administradas de las siguientes maneras:

* Examine los privilegios y asegúrese de que está seleccionado el modelo con menos privilegios. Use el siguiente cmdlet de PowerShell para obtener los permisos asignados a las identidades administradas.

   ` Get-AzureADServicePrincipal | % { Get-AzureADServiceAppRoleAssignment -ObjectId $_ }`

 
* Asegúrese de que la identidad administrada no forma parte de ningún grupo con privilegios, como un grupo de administradores.  
‎Para ello, puede enumerar los miembros de los grupos con privilegios elevados mediante PowerShell.

   `Get-AzureADGroupMember -ObjectId <String> [-All <Boolean>] [-Top <Int32>] [<CommonParameters>]`

* [Asegúrese de que sabe a qué recursos tiene acceso la identidad administrada](../../role-based-access-control/role-assignments-list-powershell.md).

## <a name="move-to-managed-identities"></a>Traslado a identidades administradas

Si usa una entidad de servicio o una cuenta de usuario de Azure AD, evalúe si, en su lugar, puede usar una identidad administrada para eliminar la necesidad de proteger, rotar y administrar credenciales. 

## <a name="next-steps"></a>Pasos siguientes

**Para más información sobre cómo crear identidades administradas, vea:** 

[Creación de una identidad administrada asignada por el usuario](../managed-identities-azure-resources/how-to-manage-ua-identity-portal.md) 

[Habilitación de una identidad administrada asignada por el sistema durante la creación del recurso](../managed-identities-azure-resources/qs-configure-portal-windows-vm.md)

[Habilitación de una identidad administrada asignada por el sistema en un recurso existente](../managed-identities-azure-resources/qs-configure-portal-windows-vm.md)

**Para más información sobre las cuentas de servicio, vea:**

[Introducción a las cuentas de servicio de Azure Active Directory](service-accounts-introduction-azure.md)

[Protección de entidades de servicio](service-accounts-principal.md)

[Gobernanza de las cuentas de servicio de Azure](service-accounts-governing-azure.md)

[Introducción a las cuentas de servicio locales](service-accounts-on-premises.md)

 

 

