---
title: Autenticación de una identidad administrada con Azure Active Directory
description: En este artículo se proporciona información sobre cómo autenticar una identidad administrada con Azure Active Directory para acceder a Azure SignalR Service
author: terencefan
ms.author: tefa
ms.date: 08/03/2020
ms.service: signalr
ms.topic: conceptual
ms.openlocfilehash: b2766495c89b9be1a89e0648bcec83c160c1e0a8
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110479449"
---
# <a name="authenticate-a-managed-identity-with-azure-active-directory-to-access-azure-signalr-resources"></a>Autenticación de una identidad administrada con Azure Active Directory para acceder a recursos de Azure SignalR
Azure SignalR Service admite la autenticación de Azure Active Directory (Azure AD) con [identidades administradas de recursos de Azure](../active-directory/managed-identities-azure-resources/overview.md). Las identidades administradas de recursos de Azure pueden autorizar el acceso a los recursos de Azure SignalR Service con credenciales de Azure AD desde aplicaciones que se ejecutan en máquinas virtuales (VM) de Azure, aplicaciones de funciones, conjuntos de escalado de máquinas virtuales y otros servicios. Si usa identidades administradas para recursos de Azure junto con autenticación de Azure AD, puede evitar el almacenamiento de credenciales con las aplicaciones que se ejecutan en la nube.

En este artículo se muestra cómo autorizar el acceso a una instancia de Azure SignalR Service con una identidad administrada.

## <a name="enable-managed-identities"></a>Habilitar identidades administradas
Para poder usar identidades administradas de recursos de Azure para autorizar el acceso a recursos de Azure SignalR Service, primero debe habilitar las identidades administradas de recursos de Azure. 
### <a name="for-azure-resources-on-a-vm"></a>Para recursos de Azure en una VM
Para aprender a habilitar las identidades administradas para los recursos de Azure en una VM, consulte uno de estos artículos:

- [Azure Portal](../active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm.md)
  1. Vaya a **Configuración** en Azure Portal y seleccione **Identidad**. 
  2. Seleccione el **Estado** que va a estar **Activado**. 
  3. Seleccione **Guardar** para guardar la configuración. 

      ![Identidad administrada de una máquina virtual](./media/authenticate/identity-virtual-machine.png)
- [Azure PowerShell](../active-directory/managed-identities-azure-resources/qs-configure-powershell-windows-vm.md)
- [CLI de Azure](../active-directory/managed-identities-azure-resources/qs-configure-cli-windows-vm.md)
- [Plantilla de Azure Resource Manager](../active-directory/managed-identities-azure-resources/qs-configure-template-windows-vm.md)
- [Bibliotecas cliente de Azure Resource Manager](../active-directory/managed-identities-azure-resources/qs-configure-sdk-windows-vm.md)

### <a name="for-azure-functions"></a>Para Azure Functions

Consulte [Cómo usar identidades administradas para App Service y Azure Functions](../app-service/overview-managed-identity.md).

## <a name="grant-permissions-to-a-managed-identity-in-azure-ad"></a>Concesión de permisos a una identidad administrada en Azure AD
Para autorizar una solicitud a Azure SignalR Service desde una identidad administrada de la aplicación, primero configure los valores de control de acceso basado en rol (RBAC) de esa identidad administrada. Azure SignalR Service define roles RBAC que incluyen permisos para adquirir `AccessKey` o `ClientToken`. Cuando el rol RBAC se asigna a una identidad administrada, a esta se le concede acceso a Azure SignalR Service en el ámbito adecuado.

Siga estas instrucciones para administrar las asignaciones de roles:

1. En [Azure Portal](https://portal.azure.com/), vaya al recurso de SignalR.
1. Seleccione **Control de acceso (IAM)** para mostrar la configuración del control de acceso de Azure SignalR. 
1. Seleccione la pestaña **Asignaciones de roles** para ver la lista de asignaciones de roles. Seleccione el botón **Agregar** de la barra de herramientas y luego seleccione **Agregar asignación de roles**. 

    ![Botón Agregar de la barra de herramientas](./media/authenticate/role-assignments-add-button.png)

1. En la página **Agregar asignación de roles**, siga estos pasos:
    1. Seleccione el rol de **servidor de aplicaciones de SignalR**. Tenga en cuenta que esto también se aplica a la **aplicación Azure Functions**.
    1. Realice una búsqueda para localizar la **entidad de seguridad** (usuario, grupo, entidad de servicio) a la que quiere asignar el rol.
    1. Seleccione **Guardar** para guardar la asignación de roles. 

        ![Asignación de un rol a una aplicación](./media/authenticate/assign-role-to-application.png)

    1. La identidad a la que ha asignado el rol aparece enumerada debajo de ese rol. Por ejemplo, la imagen siguiente muestra que los elementos `signalr-dev` y `signalr-service` de la aplicación se encuentran en el rol del servidor de aplicaciones de SignaIR. 
        
        ![Lista de asignación de roles](./media/authenticate/role-assignment-list.png)

Puede realizar este mismo procedimiento para asignar un rol que tenga como ámbito un grupo de recursos o una suscripción. Una vez que defina el rol y su ámbito, puede probar este comportamiento con ejemplos [en esta ubicación de GitHub](https://github.com/Azure/azure-event-hubs/tree/master/samples/DotNet/Microsoft.Azure.EventHubs/Rbac).

Para obtener más información sobre la asignación de roles RBAC, vea [Autenticación con Azure Active Directory para el acceso a recursos de Azure SignalR Service](authorize-access-azure-active-directory.md).

Para más información sobre cómo administrar el acceso a los recursos de Azure con Azure RBAC y Azure Portal, consulte [este artículo](..//role-based-access-control/role-assignments-portal.md). 

## <a name="configure-your-app"></a>Configurar la aplicación
### <a name="app-server"></a>Servidor de aplicaciones

Agregue las siguientes opciones cuando `AddAzureSignalR`:

```C#
services.AddSignalR().AddAzureSignalR(option =>
{
    option.ConnectionString = "Endpoint=https://<name>.signalr.net;AuthType=aad;Version=1.0;";
});
```

### <a name="azure-functions-app"></a>Aplicación Azure Functions

En Azure Portal, agregue una configuración de aplicación con el nombre `AzureSignalRConnectionString` y el valor `Endpoint=https://<name>.signalr.net;AuthType=aad;`.

En el entorno local, en el archivo `local.appsettings.json`, agregue lo siguiente en la sección `Values`:
```json
{
    "Values": {
        "AzureSignalRConnectionString": "Endpoint=https://<name>.signalr.net;AuthType=aad;Version=1.0;"
    }
}
```

## <a name="next-steps"></a>Pasos siguientes
- Para más información acerca de RBAC, consulte [¿Qué es el control de acceso basado en rol (RBAC) de Azure?](../role-based-access-control/overview.md).
- Para información sobre cómo asignar y administrar las asignaciones de roles de Azure con Azure PowerShell, la CLI de Azure o la API de REST, consulte estos artículos:
    - [Administración del control de acceso basado en rol (RBAC) con Azure PowerShell](../role-based-access-control/role-assignments-powershell.md)  
    - [Administración del control de acceso basado en rol (RBAC) con la CLI de Azure](../role-based-access-control/role-assignments-cli.md)
    - [Administración del control de acceso basado en rol (RBAC) con la API de REST](../role-based-access-control/role-assignments-rest.md)
    - [Administración del acceso a los recursos de Azure mediante RBAC y plantillas de Azure Resource Manager](../role-based-access-control/role-assignments-template.md)

Consulte los artículos relacionados siguientes:
- [Autenticación de una aplicación con Azure Active Directory para acceder a Azure SignalR Service](authenticate-application.md)
- [Autorización del acceso a Azure SignalR Service mediante Azure Active Directory](authorize-access-azure-active-directory.md)
