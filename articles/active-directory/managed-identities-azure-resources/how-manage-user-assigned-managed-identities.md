---
title: 'Administración de identidades administradas asignadas por el usuario: Azure AD'
description: Creación de identidades administradas asignadas por el usuario
services: active-directory
author: barclayn
manager: daveba
editor: ''
ms.service: active-directory
ms.subservice: msi
ms.devlang: ''
ms.topic: how-to
ms.workload: identity
ms.date: 06/08/2021
ms.author: barclayn
zone_pivot_groups: identity-mi-methods
ms.openlocfilehash: 46f6b6354bed2ef0e2fc531f00b8380b04f6f0b9
ms.sourcegitcommit: 8bca2d622fdce67b07746a2fb5a40c0c644100c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111758295"
---
# <a name="manage-user-assigned-managed-identities"></a>Administración de identidades administradas asignadas por el usuario


Las identidades administradas para los recursos de Azure eliminan la necesidad de administrar las credenciales en el código. Permiten obtener un token de Azure Active Directory que las aplicaciones pueden usar al acceder a los recursos que admiten la autenticación de Azure Active Directory. Azure administra la identidad para que usted no tenga que hacerlo. Hay dos tipos de identidades administradas: asignadas por el sistema y asignadas por el usuario. La diferencia principal entre los dos tipos es que las identidades administradas asignadas por el sistema tienen un ciclo de vida vinculado al recurso donde se usan. Las identidades administradas asignadas por el usuario se pueden usar en varios recursos. Puede obtener más información al respecto en la [introducción](overview.md) a las identidades administradas. 

::: zone pivot="identity-mi-methods-azp"

En este artículo aprenderá a crear, enumerar, eliminar y asignar un rol a una identidad administrada asignada por el usuario mediante Azure Portal.

## <a name="prerequisites"></a>Requisitos previos

- Si no está familiarizado con las identidades administradas de los recursos de Azure, consulte la [sección de introducción](overview.md). **No olvide revisar la [diferencia entre una identidad administrada asignada por el sistema y una identidad administrada asignada por el usuario](overview.md#managed-identity-types)** .
- Si aún no tiene una cuenta de Azure, [regístrese para una cuenta gratuita](https://azure.microsoft.com/free/) antes de continuar.

## <a name="create-a-user-assigned-managed-identity"></a>Crear una identidad administrada asignada por el usuario

Para crear una identidad administrada asignada por el usuario, la cuenta requiere la asignación del rol [Colaborador de identidades administradas](../../role-based-access-control/built-in-roles.md#managed-identity-contributor).

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta asociada a la suscripción de Azure para crear la identidad administrada asignada por el usuario.
2. En el cuadro de búsqueda, escriba *Identidades administradas* y en **Servicios**, haga clic en **Identidades administradas**.
3. Haga clic en **Agregar** y escriba valores en los campos siguientes del panel **Crear identidad administrada asignada por el usuario**:
    - **Suscripción**: elija la suscripción donde crear la identidad administrada asignada por el usuario.
    - **Grupo de recursos**: elija un grupo de recursos en el que crear la identidad administrada asignada por el usuario o haga clic en **Crear nuevo** para crear un nuevo grupo de recursos.
    - **Región**: elija una región para implementar la identidad administrada asignada por el usuario, por ejemplo, **Oeste de EE. UU.** .
    - **Name**: este es el nombre de la identidad administrada asignada por el usuario, por ejemplo, UAI1.
    ![Creación de una identidad administrada asignada por el usuario](media/how-to-manage-ua-identity-portal/create-user-assigned-managed-identity-portal.png)
4. Haga clic en **Revisar y crear** para revisar los cambios.
5. Haga clic en **Crear**.

## <a name="list-user-assigned-managed-identities"></a>Enumerar identidades administradas asignadas por el usuario

Para crear o leer una identidad administrada asignada por el usuario, la cuenta requiere la asignación del rol [Operador de identidades administradas ](../../role-based-access-control/built-in-roles.md#managed-identity-operator) o [Colaborador de identidades administradas](../../role-based-access-control/built-in-roles.md#managed-identity-contributor).

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta asociada a la suscripción de Azure para enumerar las identidades administradas asignadas por el usuario.
2. En el cuadro de búsqueda, escriba *Identidades administradas* y en Servicios, haga clic en **Identidades administradas**.
3. Se devuelve una lista de las identidades administradas asignadas por el usuario de la suscripción.  Para ver los detalles de una identidad administrada asignada por el usuario, haga clic en su nombre.

![Enumerar identidades administradas asignadas por el usuario](media/how-to-manage-ua-identity-portal/list-user-assigned-managed-identity-portal.png)

## <a name="delete-a-user-assigned-managed-identity"></a>Eliminar una identidad administrada asignada por el usuario

Para eliminar una identidad administrada asignada por el usuario, la cuenta requiere la asignación del rol [Colaborador de identidades administradas](../../role-based-access-control/built-in-roles.md#managed-identity-contributor).

La eliminación de una identidad asignada por el usuario no la quita de ninguna VM o recurso a la que estuviera asignada.  Para quitar la identidad asignada por el usuario desde una máquina virtual, consulte [Eliminación de una identidad administrada asignada por el usuario de una VM](qs-configure-portal-windows-vm.md#remove-a-user-assigned-managed-identity-from-a-vm).

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta asociada a la suscripción de Azure para eliminar una identidad administrada asignada por el usuario.
2. Seleccione la identidad administrada asignada por el usuario y haga clic en **Eliminar**.
3. En el cuadro de confirmación, elija **Sí**.

![Eliminar una identidad administradas asignada por el usuario](media/how-to-manage-ua-identity-portal/delete-user-assigned-managed-identity-portal.png)

## <a name="assign-a-role-to-a-user-assigned-managed-identity"></a>Asignación de un rol a una identidad administrada asignada por el usuario 

Para asignar un rol a una identidad administrada asignada por el usuario, la cuenta requiere la asignación del rol [Administrador de acceso de usuario](../../role-based-access-control/built-in-roles.md#user-access-administrator).

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta asociada a la suscripción de Azure para enumerar las identidades administradas asignadas por el usuario.
2. En el cuadro de búsqueda, escriba *Identidades administradas* y en Servicios, haga clic en **Identidades administradas**.
3. Se devuelve una lista de las identidades administradas asignadas por el usuario de la suscripción.  Seleccione la identidad administrada asignada por el usuario a la que desea asignar un rol.
4. Seleccione **Control de acceso (IAM)** y después **Agregar asignación de rol**.

   ![Inicio de Identidad administrada asignada por el usuario](media/how-to-manage-ua-identity-portal/assign-role-screenshot1.png)

5. En la hoja Agregar asignación de roles, configure los siguientes valores y después, haga clic en **Guardar**:
   - **Rol**: el rol que se va a asignar
   - **Asignar acceso a**: el recurso al que se asigna la identidad administrada asignada por el usuario
   - **Seleccionar**: el miembro al que se asigna acceso
   
   ![IAM de identidad administrada asignada por el usuario](media/how-to-manage-ua-identity-portal/assign-role-screenshot2.png)



::: zone-end

::: zone pivot="identity-mi-methods-azcli"

En este artículo aprenderá a crear, enumerar, eliminar y asignar un rol a una identidad administrada asignada por el usuario mediante la CLI de Azure.

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../../includes/azure-cli-prepare-your-environment-no-header.md)]

> [!IMPORTANT]  
> A fin de modificar los permisos de usuario al usar una entidad de servicio de aplicación mediante la CLI, debe proporcionar los permisos adicionales de la entidad de servicio en Graph API de Azure AD como partes de la CLI, realice solicitudes GET en Graph API. De lo contrario, puede acabar recibiendo el mensaje "No tiene privilegios suficientes para completar la operación". Para ello, tendrá que entrar en el Registro de aplicación en Azure Active Directory, seleccionar la aplicación, hacer clic en Permisos de API, desplazarse hacia abajo y seleccionar Azure Active Directory Graph. Desde allí, seleccione Permisos de la aplicación y, a continuación, agregue los permisos adecuados. 

## <a name="create-a-user-assigned-managed-identity"></a>Crear una identidad administrada asignada por el usuario 

Para crear una identidad administrada asignada por el usuario, la cuenta requiere la asignación del rol [Colaborador de identidades administradas](../../role-based-access-control/built-in-roles.md#managed-identity-contributor).

Use el comando [az identity create](/cli/azure/identity#az_identity_create) para crear una identidad administrada asignada por el usuario. El parámetro `-g` especifica el grupo de recursos donde se creará la identidad administrada asignada por el usuario, mientras que el parámetro `-n` especifica su nombre. Reemplace los valores de los parámetros `<RESOURCE GROUP>` y `<USER ASSIGNED IDENTITY NAME>` con sus propios valores:

[!INCLUDE [ua-character-limit](~/includes/managed-identity-ua-character-limits.md)]

```azurecli-interactive
az identity create -g <RESOURCE GROUP> -n <USER ASSIGNED IDENTITY NAME>
```
## <a name="list-user-assigned-managed-identities"></a>Enumerar identidades administradas asignadas por el usuario

Para crear o leer una identidad administrada asignada por el usuario, la cuenta requiere la asignación del rol [Operador de identidades administradas ](../../role-based-access-control/built-in-roles.md#managed-identity-operator) o [Colaborador de identidades administradas](../../role-based-access-control/built-in-roles.md#managed-identity-contributor).

Para enumerar las identidades administradas asignadas por el usuario, use el comando [az identity list](/cli/azure/identity#az_identity_list). Reemplace `<RESOURCE GROUP>` por su propio valor:

```azurecli-interactive
az identity list -g <RESOURCE GROUP>
```

En la respuesta JSON, a las identidades administradas asignadas por el usuario se devuelve el valor `"Microsoft.ManagedIdentity/userAssignedIdentities"` para la clave `type`.

`"type": "Microsoft.ManagedIdentity/userAssignedIdentities"`

## <a name="delete-a-user-assigned-managed-identity"></a>Eliminar una identidad administrada asignada por el usuario

Para eliminar una identidad administrada asignada por el usuario, la cuenta requiere la asignación del rol [Colaborador de identidades administradas](../../role-based-access-control/built-in-roles.md#managed-identity-contributor).

Para eliminar una identidad administrada asignada por el usuario, use el comando [az identity delete](/cli/azure/identity#az_identity_delete).  El parámetro -n especifica el nombre, y el parámetro -g especifica el grupo de recursos donde se creó la identidad administrada asignada por el usuario. Reemplace los valores de los parámetros `<USER ASSIGNED IDENTITY NAME>` y `<RESOURCE GROUP>` con sus propios valores:

```azurecli-interactive
az identity delete -n <USER ASSIGNED IDENTITY NAME> -g <RESOURCE GROUP>
```
> [!NOTE]
> El proceso de eliminación de una identidad administrada asignada por el usuario no quitará la referencia de ningún recurso al que se haya asignado. Quite esas máquinas virtuales y VMSS con el comando `az vm/vmss identity remove`.

## <a name="next-steps"></a>Pasos siguientes

Para obtener una lista completa de comandos de identidad de la CLI de Azure, consulte [az identity](/cli/azure/identity).

Para obtener información sobre cómo asignar una identidad administrada asignada por el usuario a una VM de Azure, consulte [Configure managed identities for Azure resources on an Azure VM using Azure CLI](qs-configure-cli-windows-vm.md#user-assigned-managed-identity) (Configuración de identidades administradas de recursos de Azure en una VM de Azure con la CLI de Azure).


::: zone-end

::: zone pivot="identity-mi-methods-powershell"

En este artículo, aprenderá a crear, enumerar y eliminar una identidad administrada asignada por el usuario con PowerShell.

## <a name="prerequisites"></a>Requisitos previos

- Si no está familiarizado con las identidades administradas de los recursos de Azure, consulte la [sección de introducción](overview.md). **No olvide revisar la [diferencia entre una identidad administrada asignada por el sistema y una identidad administrada asignada por el usuario](overview.md#managed-identity-types)** .
- Si aún no tiene una cuenta de Azure, [regístrese para una cuenta gratuita](https://azure.microsoft.com/free/) antes de continuar.
- Para ejecutar los scripts de ejemplo, tiene dos opciones:
    - Use [Azure Cloud Shell](../../cloud-shell/overview.md), que puede abrir mediante el botón **Probar** en la esquina superior derecha de los bloques de código.
    - Ejecute los scripts localmente con Azure PowerShell, tal como se describe en la sección siguiente.

### <a name="configure-azure-powershell-locally"></a>Configuración de Azure PowerShell de forma local

Para usar Azure PowerShell de forma local en este artículo (en lugar de usar Cloud Shell), complete los pasos siguientes:

1. Instale [la versión más reciente de Azure PowerShell](/powershell/azure/install-az-ps) si aún no lo ha hecho.

1. Inicie de sesión en Azure:

    ```azurepowershell
    Connect-AzAccount
    ```

1. Instalar la [versión más reciente de PowerShellGet](/powershell/scripting/gallery/installing-psget#for-systems-with-powershell-50-or-newer-you-can-install-the-latest-powershellget).

    ```azurepowershell
    Install-Module -Name PowerShellGet -AllowPrerelease
    ```

    Es posible que necesite `Exit` fuera de la sesión de PowerShell actual después de ejecutar este comando para el paso siguiente.

1. Instale la versión preliminar del módulo `Az.ManagedServiceIdentity` con el fin de realizar las operaciones de identidad administrada que haya asignado el usuario, y que se indican en este artículo.

    ```azurepowershell
    Install-Module -Name Az.ManagedServiceIdentity -AllowPrerelease
    ```

## <a name="create-a-user-assigned-managed-identity"></a>Crear una identidad administrada asignada por el usuario

Para crear una identidad administrada asignada por el usuario, la cuenta requiere la asignación del rol [Colaborador de identidades administradas](../../role-based-access-control/built-in-roles.md#managed-identity-contributor).

Para crear una identidad administrada asignada por el usuario, use el comando `New-AzUserAssignedIdentity`. El parámetro `ResourceGroupName` especifica el grupo de recursos donde se creará la identidad administrada asignada por el usuario, mientras que el parámetro `-Name` especifica su nombre. Reemplace los valores de los parámetros `<RESOURCE GROUP>` y `<USER ASSIGNED IDENTITY NAME>` con sus propios valores:

[!INCLUDE [ua-character-limit](~/includes/managed-identity-ua-character-limits.md)]

```azurepowershell-interactive
New-AzUserAssignedIdentity -ResourceGroupName <RESOURCEGROUP> -Name <USER ASSIGNED IDENTITY NAME>
```

## <a name="list-user-assigned-managed-identities"></a>Enumerar identidades administradas asignadas por el usuario

Para crear o leer una identidad administrada asignada por el usuario, la cuenta requiere la asignación del rol [Operador de identidades administradas ](../../role-based-access-control/built-in-roles.md#managed-identity-operator) o [Colaborador de identidades administradas](../../role-based-access-control/built-in-roles.md#managed-identity-contributor).

Para enumerar las identidades administradas asignadas por el usuario, use el comando [Get-AzUserAssigned].  El parámetro `-ResourceGroupName` especifica el grupo de recursos donde se creó la identidad administrada asignada por el usuario. Reemplace `<RESOURCE GROUP>` por su propio valor:

```azurepowershell-interactive
Get-AzUserAssignedIdentity -ResourceGroupName <RESOURCE GROUP>
```

En la respuesta, las identidades administradas asignadas por el usuario reciben el valor `"Microsoft.ManagedIdentity/userAssignedIdentities"` para la clave `Type`.

`Type :Microsoft.ManagedIdentity/userAssignedIdentities`

## <a name="delete-a-user-assigned-managed-identity"></a>Eliminar una identidad administrada asignada por el usuario

Para eliminar una identidad administrada asignada por el usuario, la cuenta requiere la asignación del rol [Colaborador de identidades administradas](../../role-based-access-control/built-in-roles.md#managed-identity-contributor).

Para eliminar una identidad administrada asignada por el usuario, use el comando `Remove-AzUserAssignedIdentity`.  El parámetro `-ResourceGroupName` especifica el grupo de recursos donde se creó la identidad asignada por el usuario, mientras que el parámetro `-Name` especifica su nombre. Reemplace los valores de los parámetros `<RESOURCE GROUP>` y `<USER ASSIGNED IDENTITY NAME>` con sus propios valores:

```azurepowershell-interactive
Remove-AzUserAssignedIdentity -ResourceGroupName <RESOURCE GROUP> -Name <USER ASSIGNED IDENTITY NAME>
```

> [!NOTE]
> El proceso de eliminación de una identidad administrada asignada por el usuario no quitará la referencia de ningún recurso al que se haya asignado. Las asignaciones de identidad deben quitarse por separado.

## <a name="next-steps"></a>Pasos siguientes

Para obtener una lista completa y más detalles sobre las identidades administradas de Azure PowerShell para los comandos de los recursos de Azure, consulte [Az.ManagedServiceIdentity](/powershell/module/az.managedserviceidentity#managed_service_identity).


::: zone-end


::: zone pivot="identity-mi-methods-arm"

En este artículo creará una identidad administrada asignada por el usuario mediante Azure Resource Manager.

No es posible enumerar y eliminar una identidad administrada asignada por el usuario mediante una plantilla de Azure Resource Manager.  Consulte los artículos siguientes para crear y enumerar una identidad administrada asignada por el usuario:

- [Enumeración de identidades administradas asignadas por el usuario](how-to-manage-ua-identity-cli.md#list-user-assigned-managed-identities)
- [Eliminación de identidades administradas asignadas por el usuario](how-to-manage-ua-identity-cli.md#delete-a-user-assigned-managed-identity)

## <a name="prerequisites"></a>Requisitos previos

- Si no está familiarizado con las identidades administradas para recursos de Azure, consulte la [sección de introducción](overview.md). **No olvide revisar la [diferencia entre una identidad administrada asignada por el sistema y una identidad administrada asignada por el usuario](overview.md#managed-identity-types)** .
- Si aún no tiene una cuenta de Azure, [regístrese para una cuenta gratuita](https://azure.microsoft.com/free/) antes de continuar.

## <a name="template-creation-and-editing"></a>Creación y edición de una plantilla

Como con Azure Portal y los scripts, las plantillas de Azure Resource Manager proporcionan la capacidad de implementar recursos nuevos o modificados definidos con un grupo de recursos de Azure. Existen varias opciones para la edición e implementación de plantillas, tanto localmente como basadas en el portal, incluidas:

- Usar una [plantilla personalizada de Azure Marketplace](../../azure-resource-manager/templates/deploy-portal.md#deploy-resources-from-custom-template), que permite crear una plantilla desde cero, o bien basada en una plantilla común existente o en una [plantilla de inicio rápido](https://azure.microsoft.com/documentation/templates/).
- Derivar a partir de un grupo de recursos existente, exportando una plantilla de [la implementación original](../../azure-resource-manager/management/manage-resource-groups-portal.md#export-resource-groups-to-templates) o del [estado actual de la implementación](../../azure-resource-manager/management/manage-resource-groups-portal.md#export-resource-groups-to-templates).
- Usar un [editor de JSON (por ejemplo, VS Code)](../../azure-resource-manager/templates/quickstart-create-templates-use-the-portal.md) local y, a continuación, cargarla e implementarla con PowerShell o la CLI.
- Usar el [proyecto del grupo de recursos de Azure](../../azure-resource-manager/templates/create-visual-studio-deployment-project.md) de Visual Studio tanto para crear como para implementar una plantilla. 

## <a name="create-a-user-assigned-managed-identity"></a>Crear una identidad administrada asignada por el usuario 

Para crear una identidad administrada asignada por el usuario, la cuenta requiere la asignación del rol [Colaborador de identidades administradas](../../role-based-access-control/built-in-roles.md#managed-identity-contributor).

Para crear una identidad administrada asignada por el usuario, use la siguiente plantilla. Reemplace el valor de `<USER ASSIGNED IDENTITY NAME>` por sus propios valores:

[!INCLUDE [ua-character-limit](~/includes/managed-identity-ua-character-limits.md)]

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "resourceName": {
          "type": "string",
          "metadata": {
            "description": "<USER ASSIGNED IDENTITY NAME>"
          }
        }
  },
  "resources": [
    {
      "type": "Microsoft.ManagedIdentity/userAssignedIdentities",
      "name": "[parameters('resourceName')]",
      "apiVersion": "2018-11-30",
      "location": "[resourceGroup().location]"
    }
  ],
  "outputs": {
      "identityName": {
          "type": "string",
          "value": "[parameters('resourceName')]"
      }
  }
}
```
## <a name="next-steps"></a>Pasos siguientes

Para obtener información sobre cómo asignar una identidad administrada asignada por el usuario a una VM de Azure mediante una plantilla de Azure Resource Manager, consulte [Configure managed identities for Azure resources on an Azure VM using a template](qs-configure-template-windows-vm.md) (Configuración de identidades administradas de recursos de Azure en una VM de Azure mediante una plantilla).




::: zone-end


::: zone pivot="identity-mi-methods-rest"

En este artículo, aprenderá a crear, enumerar y eliminar una identidad administrada asignada por el usuario mediante CURL para hacer llamadas a la API REST.

## <a name="prerequisites"></a>Requisitos previos

- Si no está familiarizado con las identidades administradas de los recursos de Azure, consulte la [sección de introducción](overview.md). **No olvide revisar la [diferencia entre una identidad administrada asignada por el sistema y una identidad administrada asignada por el usuario](overview.md#managed-identity-types)** .
- Si aún no tiene una cuenta de Azure, [regístrese para una cuenta gratuita](https://azure.microsoft.com/free/) antes de continuar.
- Puede ejecutar todos los comandos de este artículo, ya sea en la nube o localmente:
    - Para ejecutar en la nube, use [Azure Cloud Shell](../../cloud-shell/overview.md).
    - Para ejecutar localmente, instale [curl](https://curl.haxx.se/download.html) y la [CLI de Azure](/cli/azure/install-azure-cli).

## <a name="obtain-a-bearer-access-token"></a>Obtención de un token de acceso de portador

1. Si se ejecuta localmente, inicie sesión en Azure a través de la CLI de Azure:

    ```
    az login
    ```

1. Obtenga un token de acceso mediante [az account get-access-token](/cli/azure/account#az_account_get_access_token)

    ```azurecli-interactive
    az account get-access-token
    ```

## <a name="create-a-user-assigned-managed-identity"></a>Crear una identidad administrada asignada por el usuario 

Para crear una identidad administrada asignada por el usuario, la cuenta requiere la asignación del rol [Colaborador de identidades administradas](../../role-based-access-control/built-in-roles.md#managed-identity-contributor).

[!INCLUDE [ua-character-limit](~/includes/managed-identity-ua-character-limits.md)]

```bash
curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroup
s/<RESOURCE GROUP>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<USER ASSIGNED IDENTITY NAME>?api-version=2015-08-31-preview' -X PUT -d '{"loc
ation": "<LOCATION>"}' -H "Content-Type: application/json" -H "Authorization: Bearer <ACCESS TOKEN>"
```

```HTTP
PUT https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroup
s/<RESOURCE GROUP>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<USER ASSIGNED IDENTITY NAME>?api-version=2015-08-31-preview HTTP/1.1
```

**Encabezados de solicitud**

|Encabezado de solicitud  |Descripción  |
|---------|---------|
|*Content-Type*     | Necesario. Establézcalo en `application/json`.        |
|*Autorización*     | Necesario. Establézcalo en un token de acceso `Bearer` válido.        |

**Cuerpo de la solicitud**

|Nombre  |Descripción  |
|---------|---------|
|ubicación     | Necesario. Ubicación del recurso        |

## <a name="list-user-assigned-managed-identities"></a>Enumerar identidades administradas asignadas por el usuario

Para crear o leer una identidad administrada asignada por el usuario, la cuenta requiere la asignación del rol [Operador de identidades administradas ](../../role-based-access-control/built-in-roles.md#managed-identity-operator) o [Colaborador de identidades administradas](../../role-based-access-control/built-in-roles.md#managed-identity-contributor).

```bash
curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>/providers/Microsoft.ManagedIdentity/userAssignedIdentities?api-version=2015-08-31-preview' -H "Authorization: Bearer <ACCESS TOKEN>"
```

```HTTP
GET https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>/providers/Microsoft.ManagedIdentity/userAssignedIdentities?api-version=2015-08-31-preview HTTP/1.1
```

|Encabezado de solicitud  |Descripción  |
|---------|---------|
|*Content-Type*     | Necesario. Establézcalo en `application/json`.        |
|*Autorización*     | Necesario. Establézcalo en un token de acceso `Bearer` válido.        |

## <a name="delete-a-user-assigned-managed-identity"></a>Eliminar una identidad administrada asignada por el usuario

Para eliminar una identidad administrada asignada por el usuario, la cuenta requiere la asignación del rol [Colaborador de identidades administradas](../../role-based-access-control/built-in-roles.md#managed-identity-contributor).

> [!NOTE]
> La eliminación de una identidad administrada asignada por el usuario no quitará la referencia de ningún recurso al que se haya asignado. Para quitar una identidad administrada asignada por el usuario desde una máquina virtual mediante CURL, consulte [Eliminar una identidad asignada por el usuario de una máquina virtual de Azure](qs-configure-rest-vm.md#remove-a-user-assigned-managed-identity-from-an-azure-vm).

```bash
curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroup
s/<RESOURCE GROUP>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<USER ASSIGNED IDENTITY NAME>?api-version=2015-08-31-preview' -X DELETE -H "Authorization: Bearer <ACCESS TOKEN>"
```

```HTTP
DELETE https://management.azure.com/subscriptions/80c696ff-5efa-4909-a64d-f1b616f423ca/resourceGroups/TestRG/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<USER ASSIGNED IDENTITY NAME>?api-version=2015-08-31-preview HTTP/1.1
```
|Encabezado de solicitud  |Descripción  |
|---------|---------|
|*Content-Type*     | Necesario. Establézcalo en `application/json`.        |
|*Autorización*     | Necesario. Establézcalo en un token de acceso `Bearer` válido.        |

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre cómo asignar una identidad asignada por el usuario a una máquina virtual o a un conjunto de escalado de máquinas virtuales de Azure mediante CURL, consulte cómo [configurar las identidades administradas para los recursos de Azure en una máquina virtual de Azure mediante las llamadas a API REST](qs-configure-rest-vm.md#user-assigned-managed-identity) y cómo [configurar las identidades administradas para los recursos de Azure en un conjunto de escalado de máquinas virtuales mediante las llamadas a API REST](qs-configure-rest-vmss.md#user-assigned-managed-identity).

::: zone-end


