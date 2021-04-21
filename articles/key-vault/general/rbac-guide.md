---
title: Concesión de permisos para que las aplicaciones accedan a una instancia de Azure Key Vault mediante Azure RBAC | Microsoft Docs
description: Aprenda a proporcionar acceso a claves, secretos y certificados mediante el control de acceso basado en rol de Azure.
services: key-vault
author: msmbaldwin
ms.service: key-vault
ms.subservice: general
ms.topic: how-to
ms.date: 04/15/2021
ms.author: mbaldwin
ms.custom: devx-track-azurepowershell, devx-track-azurecli
ms.openlocfilehash: 966f704bd47b4b238ed72579a6103bd2e4348849
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107772224"
---
# <a name="provide-access-to-key-vault-keys-certificates-and-secrets-with-an-azure-role-based-access-control"></a>Acceso a las claves, los certificados y los secretos de Key Vault con un control de acceso basado en rol de Azure

> [!NOTE]
> El proveedor de recursos de Key Vault admite dos tipos de recursos: **almacenes** y **HSM administrados**. El control de acceso descrito en este artículo solo se aplica a los **almacenes**. Para más información sobre el control de acceso para HSM administrado, consulte [Control de acceso de HSM administrado](../managed-hsm/access-control.md).

El control de acceso basado en rol de Azure es un sistema de autorización basado en [Azure Resource Manager](../../azure-resource-manager/management/overview.md) que proporciona administración de acceso específico a los recursos de Azure.

Azure RBAC permite a los usuarios administrar los permisos de las claves, secretos y certificados. Proporciona un lugar para administrar todos los permisos en todos los almacenes de claves. 

El modelo de Azure RBAC brinda la posibilidad de establecer permisos en diferentes niveles de ámbito: grupo de administración, suscripción, grupo de recursos o recursos individuales.  Azure RBAC para Key Vault también brinda la posibilidad de tener permisos independientes en las claves, los secretos y los certificados independientes

Para más información, consulte [Control de acceso basado en rol de Azure (Azure RBAC)](../../role-based-access-control/overview.md).

## <a name="best-practices-for-individual-keys-secrets-and-certificates"></a>Procedimientos recomendados para claves, secretos y certificados individuales

Nuestra recomendación es usar un almacén por aplicación y entorno (desarrollo, preproducción y producción).

Los permisos de claves, secretos y certificados individuales solo deben usarse para escenarios concretos:

-   Aplicaciones multicapa que necesitan un control de acceso independiente entre las distintas capas

-   Compartir un secreto individual entre varias aplicaciones

Para más información acerca de las directrices de administración de Azure Key Vault, consulte:

- [Introducción a la seguridad de Azure Key Vault](security-overview.md)
- [Límites de servicio Azure Key Vault](service-limits.md)

## <a name="azure-built-in-roles-for-key-vault-data-plane-operations"></a>Roles integrados de Azure para operaciones del plano de datos de Key Vault
> [!NOTE]
> El rol `Key Vault Contributor` es para que las operaciones del plano de administración administren almacenes de claves; no permite el acceso a claves, secretos y certificados.

| Rol integrado | Descripción | ID |
| --- | --- | --- |
| Administrador de Key Vault| Permite realizar todas las operaciones de plano de datos en un almacén de claves y en todos los objetos que contiene, incluidos los certificados, las claves y los secretos. No permite administrar los recursos del almacén de claves ni administrar las asignaciones de roles. Solo funciona para almacenes de claves que usan el modelo de permisos "Control de acceso basado en rol de Azure". | 00482a5a-887f-4fb3-b363-3b7fe8e74483 |
| Agente de certificados de Key Vault | Permite realizar cualquier acción en los certificados de un almacén de claves, excepto administrar permisos. Solo funciona para almacenes de claves que usan el modelo de permisos "Control de acceso basado en rol de Azure". | a4417e6f-fecd-4de8-b567-7b0420556985 |
| Agente criptográfico de Key Vault | Permite realizar cualquier acción en las claves de un almacén de claves, excepto administrar permisos. Solo funciona para almacenes de claves que usan el modelo de permisos "Control de acceso basado en rol de Azure". | 14b46e9e-c2b7-41b4-b07b-48a6ebf60603 |
| Usuario de cifrado de servicio criptográfico de Key Vault | Permite leer los metadatos de las claves y realizar operaciones de encapsulado/desencapsulado. Solo funciona para almacenes de claves que usan el modelo de permisos "Control de acceso basado en rol de Azure". | e147488a-f6f5-4113-8e2d-b22465e65bf6 |
| Usuario criptográfico de Key Vault  | Permite realizar operaciones criptográficas mediante claves. Solo funciona para almacenes de claves que usan el modelo de permisos "Control de acceso basado en rol de Azure". | 12338af0-0e69-4776-bea7-57ae8d297424 |
| Lector de Key Vault | Permite leer metadatos de almacenes de claves y sus certificados, claves y secretos. No se pueden leer valores confidenciales, como el contenido de los secretos o el material de las claves. Solo funciona para almacenes de claves que usan el modelo de permisos "Control de acceso basado en rol de Azure". | 21090545-7ca7-4776-b22c-e363652d74d2 |
| Responsable de secretos de Key Vault| Permite realizar cualquier acción en los secretos de un almacén de claves, excepto administrar permisos. Solo funciona para almacenes de claves que usan el modelo de permisos "Control de acceso basado en rol de Azure". | b86a8fe4-44ce-4948-aee5-eccb2c155cd7 |
| Usuario de secretos de Key Vault | Permite leer el contenido de los secretos. Solo funciona para almacenes de claves que usan el modelo de permisos "Control de acceso basado en rol de Azure". | 4633458b-17de-408a-b874-0445c86b69e6 |

Para más información sobre las definiciones de roles de Azure, consulte [Roles integrados en Azure](../../role-based-access-control/built-in-roles.md).

## <a name="using-azure-rbac-secret-key-and-certificate-permissions-with-key-vault"></a>Uso de los permisos de secretos, claves y certificados de Azure RBAC con Key Vault

El nuevo modelo de permisos de Azure RBAC para el almacén de claves proporciona una alternativa al modelo de permisos de la directiva de acceso del almacén. 

### <a name="prerequisites"></a>Requisitos previos

Para agregar asignaciones de roles, debe tener:

- Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.
- Permisos `Microsoft.Authorization/roleAssignments/write` y `Microsoft.Authorization/roleAssignments/delete`, como [Administrador de acceso de usuarios](../../role-based-access-control/built-in-roles.md#user-access-administrator) o [propietario](../../role-based-access-control/built-in-roles.md#owner)

### <a name="enable-azure-rbac-permissions-on-key-vault"></a>Habilitación de los permisos de Azure RBAC en Key Vault

> [!NOTE]
> El cambio del modelo de permiso requiere el permiso "Microsoft.Authorization/roleAssignments/write", que forma parte de los roles [Propietario](../../role-based-access-control/built-in-roles.md#owner) y [Administrador de acceso de usuario](../../role-based-access-control/built-in-roles.md#user-access-administrator). No se admiten roles de administrador de suscripciones clásicas como "Administrador de servicios" y "Coadministrador".

1.  Habilitación de los permisos de Azure RBAC en un almacén de claves nuevo:

    ![Habilitación de los permisos de Azure RBAC: almacén nuevo](../media/rbac/image-1.png)

2.  Habilitación de los permisos de Azure RBAC en un almacén de claves existente:

    ![Habilitación de los permisos de Azure RBAC: almacén existente](../media/rbac/image-2.png)

> [!IMPORTANT]
> El establecimiento del modelo de permisos de Azure RBAC invalida todos los permisos de las directivas de acceso. Puede provocar interrupciones cuando los roles de Azure equivalentes no se asignen.

### <a name="assign-role"></a>Asignación de un rol

> [!Note]
> Se recomienda usar el identificador de rol único, en lugar del nombre de rol en los scripts. Por consiguiente, si se cambia el nombre de un rol, los scripts seguirían funcionando. En este documento, el nombre de rol se usa solo para facilitar la legibilidad.

Ejecute el siguiente comando para crear una asignación de roles:

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)
```azurecli
az role assignment create --role <role_name_or_id> --assignee <assignee> --scope <scope>
```
# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azurepowershell)

```azurepowershell
#Assign by User Principal Name
New-AzRoleAssignment -RoleDefinitionName <role_name> -SignInName <assignee_upn> -Scope <scope>

#Assign by Service Principal ApplicationId
New-AzRoleAssignment -RoleDefinitionName Reader -ApplicationId <applicationId> -Scope <scope>
```
---

En Azure Portal, la pantalla de asignaciones de roles de Azure está disponible para todos los recursos en la pestaña Control de acceso (IAM).

![Asignación de roles (IAM)](../media/rbac/image-3.png)

### <a name="resource-group-scope-role-assignment"></a>Asignación de roles en el ámbito de un grupo de recursos

1.  Vaya al grupo de recursos del almacén de claves.
    ![Asignación de roles: grupo de recursos](../media/rbac/image-4.png)

2.  Haga clic en Control de acceso (IAM) \> Agregar asignación de roles\>Agregar

3.  Creación de un rol de lector de Key Vault "Lector de Key Vault" para el usuario actual

    ![Adición de rol: grupo de recursos](../media/rbac/image-5.png)

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)
```azurecli
az role assignment create --role "Key Vault Reader" --assignee {i.e user@microsoft.com} --scope /subscriptions/{subscriptionid}/resourcegroups/{resource-group-name}
```
# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azurepowershell)

```azurepowershell
#Assign by User Principal Name
New-AzRoleAssignment -RoleDefinitionName 'Key Vault Reader' -SignInName {i.e user@microsoft.com} -Scope /subscriptions/{subscriptionid}/resourcegroups/{resource-group-name}

#Assign by Service Principal ApplicationId
New-AzRoleAssignment -RoleDefinitionName 'Key Vault Reader' -ApplicationId {i.e 8ee5237a-816b-4a72-b605-446970e5f156} -Scope /subscriptions/{subscriptionid}/resourcegroups/{resource-group-name}
```
---

La asignación de roles anterior brinda la posibilidad de enumerar los objetos de almacén de claves que hay en el almacén de claves.

### <a name="key-vault-scope-role-assignment"></a>Asignación de roles del ámbito de Key Vault

1. Vaya a la pestaña Key Vault \> Control de acceso (IAM).

2. Haga clic en Agregar asignación de roles\>Agregar.

3. Cree el rol de responsable de secretos de claves "Responsable de secretos de Key Vault" para el usuario actual.

    ![Asignación de roles: almacén de claves](../media/rbac/image-6.png)

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)
```azurecli
az role assignment create --role "Key Vault Secrets Officer" --assignee {i.e jalichwa@microsoft.com} --scope /subscriptions/{subscriptionid}/resourcegroups/{resource-group-name}/providers/Microsoft.KeyVault/vaults/{key-vault-name}
```
# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azurepowershell)

```azurepowershell
#Assign by User Principal Name
New-AzRoleAssignment -RoleDefinitionName 'Key Vault Secrets Officer' -SignInName {i.e user@microsoft.com} -Scope /subscriptions/{subscriptionid}/resourcegroups/{resource-group-name}/providers/Microsoft.KeyVault/vaults/{key-vault-name}

#Assign by Service Principal ApplicationId
New-AzRoleAssignment -RoleDefinitionName 'Key Vault Secrets Officer' -ApplicationId {i.e 8ee5237a-816b-4a72-b605-446970e5f156} -Scope /subscriptions/{subscriptionid}/resourcegroups/{resource-group-name}/providers/Microsoft.KeyVault/vaults/{key-vault-name}
```
---

Después de crear la asignación de roles anterior puede crear, actualizar o eliminar los secretos.

4. Cree un secreto (Secrets \> +Generate/Import) [(Secretos > +Generar o importar)] para probar la asignación de roles de nivel de secreto.

    ![Adición de rol: almacén de claves](../media/rbac/image-7.png)

### <a name="secret-scope-role-assignment"></a>Asignación de roles del ámbito de secreto

1. Abra uno de los secretos creados previamente y observe las opciones Información general y Control de acceso (IAM). 

2. Haga clic en la pestaña Control de acceso (IAM).

    ![Asignación de roles: secretos](../media/rbac/image-8.png)

3. Cree el rol de responsable de secretos de claves "Responsable de secretos de Key Vault" para el usuario actual de la misma forma que lo hizo antes para Key Vault.

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)
```azurecli
az role assignment create --role "Key Vault Secrets Officer" --assignee {i.e user@microsoft.com} --scope /subscriptions/{subscriptionid}/resourcegroups/{resource-group-name}/providers/Microsoft.KeyVault/vaults/{key-vault-name}/secrets/RBACSecret
```
# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azurepowershell)

```azurepowershell
#Assign by User Principal Name
New-AzRoleAssignment -RoleDefinitionName 'Key Vault Secrets Officer' -SignInName {i.e user@microsoft.com} -Scope /subscriptions/{subscriptionid}/resourcegroups/{resource-group-name}/providers/Microsoft.KeyVault/vaults/{key-vault-name}/secrets/RBACSecret

#Assign by Service Principal ApplicationId
New-AzRoleAssignment -RoleDefinitionName 'Key Vault Secrets Officer' -ApplicationId {i.e 8ee5237a-816b-4a72-b605-446970e5f156} -Scope /subscriptions/{subscriptionid}/resourcegroups/{resource-group-name}/providers/Microsoft.KeyVault/vaults/{key-vault-name}/secrets/RBACSecret
```
---

### <a name="test-and-verify"></a>Prueba y comprobación

> [!NOTE]
> Los exploradores usan el almacenamiento en caché y la actualización de página es necesaria después de quitar las asignaciones de roles.<br>
> Deje varios minutos para que se actualice la asignación de roles

1. Valide la adición de un secreto nuevo sin el rol "Responsable de secretos del almacén de claves" en el nivel del almacén de claves.

Vaya a la pestaña Control de acceso (IAM) del almacén de claves y elimine la asignación de roles "Responsable de secretos de Key Vault" de este recurso.

![Eliminación de asignación: almacén de claves](../media/rbac/image-9.png)

Vaya al secreto que creó anteriormente. Puede ver todas las propiedades del secreto.

![Vista de secreto con acceso](../media/rbac/image-10.png)

Crear secreto (Secrets \> +Generate/Import) [(Secretos > +Generar o importar)] debe mostrar el siguiente error:

   ![Crear un secreto](../media/rbac/image-11.png)

2.  Valide la edición del secreto sin el rol "Responsable de secretos del almacén de claves" en el nivel del secreto.

-   Vaya a la pestaña Control de acceso (IAM) del secreto creado antes y elimine la asignación de roles "Responsable de secretos de Key Vault" de este recurso.

-   Vaya al secreto que creó anteriormente. Puede ver las propiedades del secreto.

   ![Vista de secreto sin acceso](../media/rbac/image-12.png)

3. Valide la lectura de los secretos sin el rol de lectura en el nivel del almacén de claves.

-   Vaya a la pestaña Control de acceso (IAM) del grupo de recursos del almacén de claves y elimine la asignación de roles "Lector de Key Vault".

-   Si va a la pestaña Secretos del almacén de claves, debería aparecer el siguiente error:

   ![Pestaña Secretos: error](../media/rbac/image-13.png)

### <a name="creating-custom-roles"></a>Creación de roles personalizados 

[comando az role definition create](/cli/azure/role/definition#az_role_definition_create)

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)
```azurecli
az role definition create --role-definition '{ \
   "Name": "Backup Keys Operator", \
   "Description": "Perform key backup/restore operations", \
    "Actions": [ 
    ], \
    "DataActions": [ \
        "Microsoft.KeyVault/vaults/keys/read ", \
        "Microsoft.KeyVault/vaults/keys/backup/action", \
         "Microsoft.KeyVault/vaults/keys/restore/action" \
    ], \
    "NotDataActions": [ 
   ], \
    "AssignableScopes": ["/subscriptions/{subscriptionId}"] \
}'
```
# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azurepowershell)

```azurepowershell
$roleDefinition = @"
{ 
   "Name": "Backup Keys Operator", 
   "Description": "Perform key backup/restore operations", 
    "Actions": [ 
    ], 
    "DataActions": [ 
        "Microsoft.KeyVault/vaults/keys/read ", 
        "Microsoft.KeyVault/vaults/keys/backup/action", 
         "Microsoft.KeyVault/vaults/keys/restore/action" 
    ], 
    "NotDataActions": [ 
   ], 
    "AssignableScopes": ["/subscriptions/{subscriptionId}"] 
}
"@

$roleDefinition | Out-File role.json

New-AzRoleDefinition -InputFile role.json
```
---

Para más información sobre cómo crear roles personalizados, consulte:

[Roles personalizados en los recursos de Azure](../../role-based-access-control/custom-roles.md)

## <a name="known-limits-and-performance"></a>Límites conocidos y rendimiento

-   Dos mil asignaciones de roles de Azure por suscripción

-   Latencia de las asignación de roles: con el rendimiento actual esperado, pasarán 10 minutos (600 segundos) después de que la asignación de roles se cambia para que se aplique el rol

## <a name="learn-more"></a>Más información

- [Información general de Azure RBAC](../../role-based-access-control/overview.md)
- [Tutorial de roles personalizados](../../role-based-access-control/tutorial-custom-role-cli.md)