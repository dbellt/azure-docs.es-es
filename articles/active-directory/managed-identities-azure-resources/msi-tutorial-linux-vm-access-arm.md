---
title: Uso de las identidades administradas asignadas por el usuario de una máquina virtual Linux para acceder a Azure Resource Manager
description: Este tutorial contiene directrices acerca de cómo usar una identidad administrada asignada por el usuario de una máquina virtual Linux para acceder a Azure Resource Manager.
services: active-directory
documentationcenter: ''
author: barclayn
manager: daveba
editor: daveba
ms.service: active-directory
ms.subservice: msi
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 12/01/2020
ms.author: barclayn
ROBOTS: NOINDEX,NOFOLLOW
ms.collection: M365-identity-device-management
ms.openlocfilehash: e9c7555235283e892741234b74ddb80ce3a13051
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107784720"
---
# <a name="tutorial-use-a-user-assigned-managed-identity-on-a-linux-vm-to-access-azure-resource-manager"></a>Tutorial: Uso de las identidades administradas asignadas por el usuario de una máquina virtual Linux para acceder a Azure Resource Manager

En este tutorial, se explica el procedimiento para crear una identidad administrada asignada por el usuario, asignar esta identidad a una máquina virtual Linux y utilizarla después para acceder a la API de Azure Resource Manager. Azure administra automáticamente las identidades administradas de Managed Identities for Azure Resources. Estas identidades permiten autenticarse en servicios que admiten la autenticación de Azure AD, sin necesidad de incluir credenciales en el código. 

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear una identidad administrada asignada por el usuario
> * Asignar una identidad administrada asignada por el usuario a una máquina virtual Linux. 
> * Conceder a la identidad administrada asignada por el usuario acceso a un grupo de recursos de Azure Resource Manager. 
> * Obtener un token de acceso mediante la identidad administrada asignada por el usuario y usarla para llamar a Azure Resource Manager. 

## <a name="prerequisites"></a>Prerrequisitos

- Conocimientos sobre identidades administradas. Si no está familiarizado con la característica Managed Identities for Azure Resources, consulte esta [introducción](overview.md). 
- Una cuenta de Azure, [regístrese para obtener una cuenta gratuita](https://azure.microsoft.com/free/).
- También necesita una máquina virtual Linux. Si necesita crear una máquina virtual para este tutorial, puede seguir el artículo titulado [Inicio rápido: Creación de una máquina virtual Linux en Azure Portal](../../virtual-machines/linux/quick-create-portal.md#create-virtual-machine).
- Para ejecutar los scripts de ejemplo, tiene dos opciones:
    - Use [Azure Cloud Shell](../../cloud-shell/overview.md), que puede abrir mediante el botón **Probar** en la esquina superior derecha de los bloques de código.
    - Ejecute scripts localmente instalando la versión más reciente de la [CLI de Azure](/cli/azure/install-azure-cli) y, a continuación, inicie sesión en Azure con [az login](/cli/azure/reference-index#az_login).

## <a name="create-a-user-assigned-managed-identity"></a>Crear una identidad administrada asignada por el usuario

Cree una identidad administrada asignada por el usuario mediante [az identity create](/cli/azure/identity#az_identity_create). El parámetro `-g` especifica el grupo de recursos en el que se crea la identidad administrada asignada por el usuario, mientras que el parámetro `-n` especifica su nombre. Asegúrese de reemplazar los valores de los parámetros `<RESOURCE GROUP>` y `<UAMI NAME>` por sus propios valores:
    
[!INCLUDE [ua-character-limit](~/includes/managed-identity-ua-character-limits.md)]

```azurecli-interactive
az identity create -g <RESOURCE GROUP> -n <UAMI NAME>
```

La respuesta contiene detalles de la identidad administrada asignada por el usuario que se ha creado, tal y como se muestra en el siguiente ejemplo. Anote el valor `id` de la identidad asignada administrada por el usuario, ya que los utilizará en el paso siguiente:

```json
{
"clientId": "73444643-8088-4d70-9532-c3a0fdc190fz",
"clientSecretUrl": "https://control-westcentralus.identity.azure.net/subscriptions/<SUBSCRIPTON ID>/resourcegroups/<RESOURCE GROUP>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<UAMI NAME>/credentials?tid=5678&oid=9012&aid=12344643-8088-4d70-9532-c3a0fdc190fz",
"id": "/subscriptions/<SUBSCRIPTON ID>/resourcegroups/<RESOURCE GROUP>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<UAMI NAME>",
"location": "westcentralus",
"name": "<UAMI NAME>",
"principalId": "9012",
"resourceGroup": "<RESOURCE GROUP>",
"tags": {},
"tenantId": "733a8f0e-ec41-4e69-8ad8-971fc4b533bl",
"type": "Microsoft.ManagedIdentity/userAssignedIdentities"
}
```

## <a name="assign-an-identity-to-your-linux-vm"></a>Asignación de una identidad a la máquina virtual Linux

Los clientes pueden usar las identidades administradas asignadas por el usuario en varios recursos de Azure. Utilice los comandos siguientes para asignar la identidad administrada asignada por el usuario a una única máquina virtual. Use la propiedad `Id` devuelta en el paso anterior para el parámetro `-IdentityID`.

Asigne la identidad administrada asignada por el usuario a la máquina virtual Linux utilizando el cmdlet [az vm identity-assign](/cli/azure/vm). Asegúrese de reemplazar los valores de los parámetros `<RESOURCE GROUP>` y `<VM NAME>` con sus propios valores. Use la propiedad `id` devuelta en el paso anterior para el valor del parámetro `--identities`.

```azurecli-interactive
az vm identity assign -g <RESOURCE GROUP> -n <VM NAME> --identities "/subscriptions/<SUBSCRIPTION ID>/resourcegroups/<RESOURCE GROUP>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<UAMI NAME>"
```

## <a name="grant-access-to-a-resource-group-in-azure-resource-manager"></a>Concesión de acceso a un grupo de recursos en Azure Resource Manager

Managed Identities for Azure Resources dispone de identidades que el código puede utilizar para solicitar tokens de acceso que le permitan autenticarse en las API de recursos compatibles con la autenticación de Azure AD. En este tutorial, el código accederá a la API de Azure Resource Manager.  

Antes de que el código pueda acceder a la API, debe conceder a la identidad acceso a un recurso en Azure Resource Manager. En este caso, se trata del grupo de recursos que contiene la máquina virtual. Actualice los valores de `<SUBSCRIPTION ID>` y `<RESOURCE GROUP>` según corresponda en su entorno. No olvide reemplazar `<UAMI PRINCIPALID>` por la propiedad `principalId` devuelta por el comando `az identity create` en [Creación de una identidad administrada asignada por el usuario](#create-a-user-assigned-managed-identity):

```azurecli-interactive
az role assignment create --assignee <UAMI PRINCIPALID> --role 'Reader' --scope "/subscriptions/<SUBSCRIPTION ID>/resourcegroups/<RESOURCE GROUP> "
```

La respuesta contiene detalles de la asignación de roles que se ha creado, de forma similar al ejemplo siguiente:

```json
{
  "id": "/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>/providers/Microsoft.Authorization/roleAssignments/b402bd74-157f-425c-bf7d-zed3a3a581ll",
  "name": "b402bd74-157f-425c-bf7d-zed3a3a581ll",
  "properties": {
    "principalId": "f5fdfdc1-ed84-4d48-8551-999fb9dedfbl",
    "roleDefinitionId": "/subscriptions/<SUBSCRIPTION ID>/providers/Microsoft.Authorization/roleDefinitions/acdd72a7-3385-48ef-bd42-f606fba81ae7",
    "scope": "/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>"
  },
  "resourceGroup": "<RESOURCE GROUP>",
  "type": "Microsoft.Authorization/roleAssignments"
}

```

## <a name="get-an-access-token-using-the-vms-identity-and-use-it-to-call-resource-manager"></a>Obtenga un token de acceso con la identidad de la máquina virtual y úselo para llamar a Resource Manager 

En el resto del tutorial, vamos a trabajar desde la máquina virtual que se creó anteriormente.

Para completar estos pasos, necesitará un cliente SSH. Si usa Windows, puede usar el cliente SSH en el [Subsistema de Windows para Linux](/windows/wsl/about). 

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. En el portal, vaya a **Máquinas virtuales** y diríjase a la máquina virtual Linux y, en **Introducción**, haga clic en **Conectar**. Copie la cadena para conectarse a la máquina virtual.
3. Conéctese a la máquina virtual con el cliente SSH que elija. Si usa Windows, puede usar el cliente SSH en el [Subsistema de Windows para Linux](/windows/wsl/about). Si necesita ayuda para configurar las claves del cliente de SSH, consulte [Uso de SSH con Windows en Azure](~/articles/virtual-machines/linux/ssh-from-windows.md) o [Creación y uso de un par de claves SSH pública y privada para máquinas virtuales Linux en Azure](~/articles/virtual-machines/linux/mac-create-ssh-keys.md).
4. En la ventana del terminal, con CURL, realice una solicitud con el punto de conexión de Azure Instance Metadata Service (IMDS) para obtener un token de acceso para Azure Resource Manager.  

   En el ejemplo siguiente, se muestra la solicitud CURL para adquirir un token de acceso. No olvide reemplazar `<CLIENT ID>` por la propiedad `clientId` devuelta por el comando `az identity create` en [Creación de una identidad administrada asignada por el usuario](#create-a-user-assigned-managed-identity): 
    
   ```bash
   curl -H Metadata:true "http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fmanagement.azure.com/&client_id=<UAMI CLIENT ID>"   
   ```
    
    > [!NOTE]
    > El valor del parámetro `resource` debe coincidir exactamente con el que Azure AD espera. Al usar el identificador de recurso de Azure Resource Manager, debe incluir la barra diagonal final en el URI. 
    
    La respuesta incluye el token de acceso que necesita para acceder a Azure Resource Manager. 
    
    Ejemplo de respuesta:  

    ```bash
    {
    "access_token":"eyJ0eXAiOi...",
    "refresh_token":"",
    "expires_in":"3599",
    "expires_on":"1504130527",
    "not_before":"1504126627",
    "resource":"https://management.azure.com",
    "token_type":"Bearer"
    } 
    ```

5. Utilice este token de acceso para acceder a Azure Resource Manager y leer las propiedades del grupo de recursos al que previamente concedió acceso a la identidad administrada asignada por el usuario. No olvide reemplazar `<SUBSCRIPTION ID>` y `<RESOURCE GROUP>` por los valores especificados anteriormente, y `<ACCESS TOKEN>` por el token devuelto en el paso anterior.

    > [!NOTE]
    > La dirección URL distingue mayúsculas de minúsculas, por lo tanto, asegúrese de que usa las mismas mayúsculas y minúsculas que al asignar el nombre al grupo de recursos, así como la "G" mayúscula de `resourceGroups`.  

    ```bash 
    curl https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>?api-version=2016-09-01 -H "Authorization: Bearer <ACCESS TOKEN>" 
    ```

    La respuesta contiene la información específica del grupo de recursos, de forma similar al ejemplo siguiente: 

    ```bash
    {
    "id":"/subscriptions/<SUBSCRIPTION ID>/resourceGroups/DevTest",
    "name":"DevTest",
    "location":"westus",
    "properties":{"provisioningState":"Succeeded"}
    } 
    ```
    
## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a crear una identidad administrada asignada por el usuario y a asociarla a una máquina virtual Linux para acceder a la API de Azure Resource Manager.  Para obtener más información sobre Azure Resource Manager, vea:

> [!div class="nextstepaction"]
>[Azure Resource Manager](../../azure-resource-manager/management/overview.md)
