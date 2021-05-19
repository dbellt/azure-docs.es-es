---
title: 'Tutorial`:` Uso de identidades administradas para acceder a Azure Key Vault (Linux): Azure AD'
description: Este tutorial contiene directrices para utilizar una identidad administrada asignada por el sistema de una máquina virtual Linux para acceder a Azure Resource Manager.
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
ms.date: 12/16/2020
ms.author: barclayn
ms.collection: M365-identity-device-management
ms.openlocfilehash: 5ac851c54a17893082d70bec5d96e8b8cf63b482
ms.sourcegitcommit: 38d81c4afd3fec0c56cc9c032ae5169e500f345d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109517899"
---
# <a name="tutorial-use-a-linux-vm-system-assigned-managed-identity-to-access-azure-key-vault"></a>Tutorial: Uso de identidades administradas asignadas por el sistema de una máquina virtual Linux para acceder a Azure Key Vault 

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

En este tutorial se muestra cómo puede usar una máquina virtual Linux una identidad administrada asignada por el sistema para acceder a [Azure Key Vault](../../key-vault/general/overview.md). Actuando como un arranque, Key Vault hace posible que la aplicación cliente use un secreto para acceder a recursos que no están protegidos por Azure Active Directory (AD). Las identidades de servicio administradas son administradas automáticamente por Azure y le permiten autenticarse con servicios que admiten la autenticación de Azure AD sin necesidad de incluir información de autenticación en el código.

Aprenderá a:

> [!div class="checklist"]
> * Concesión de acceso a la máquina virtual a un secreto almacenado en un almacén de claves 
> * Obtención de un token de acceso mediante la identidad de máquina virtual y usarlo para recuperar el secreto de Key Vault 
 
## <a name="prerequisites"></a>Prerrequisitos

- Conocimientos básicos sobre identidades administradas. Si no está familiarizado con la característica Managed Identities for Azure Resources, consulte esta [introducción](overview.md). 
- Una cuenta de Azure, [regístrese para obtener una cuenta gratuita](https://azure.microsoft.com/free/).
- Para realizar la creación de los recursos necesarios y los pasos de administración de roles, debe tener permisos de "Propietario" en el ámbito adecuado (la suscripción o el grupo de recursos). Si necesita ayuda con la asignación de roles, consulte [Asignación de roles de Azure para administrar el acceso a los recursos de una suscripción de Azure](../../role-based-access-control/role-assignments-portal.md).
- También necesita una máquina virtual Linux que tenga habilitadas las identidades administradas asignadas por el sistema.
  - Si necesita crear una máquina virtual para este tutorial, puede seguir el artículo titulado [Inicio rápido: Creación de una máquina virtual Linux en Azure Portal](../../virtual-machines/linux/quick-create-portal.md#create-virtual-machine).


## <a name="create-a-key-vault"></a>Creación de un almacén de claves  

En esta sección se muestra cómo conceder acceso a la máquina virtual a un secreto almacenado en un almacén de claves. Mediante Managed Identities for Azure Resources, el código puede obtener tokens de acceso para autenticarse en aquellos recursos que admitan la autenticación de Azure AD.Sin embargo, no todos los servicios de Azure admiten la autenticación de Azure AD. Para usar Managed Identities for Azure Resources con esos servicios, almacene las credenciales del servicio en Azure Key Vault y use una identidad administrada de la máquina virtual a fin de acceder a Key Vault y recuperar las credenciales.

En primer lugar, es necesario crear un almacén de claves y conceder acceso a él a la identidad administrada asignada por el sistema de la máquina virtual.

1. Abra [Azure Portal](https://portal.azure.com/).
1. En la parte superior de la barra de navegación izquierda, seleccione **Crear un recurso**.  
1. En el cuadro de texto **Buscar en el Marketplace**, escriba **Key Vault** y presione **Entrar**.  
1. En la lista de resultados, seleccione **Key Vault**.
1. Seleccione **Crear**
1. Proporcione un **nombre** para el nuevo almacén de claves.

    ![Pantalla de creación de un almacén de claves](./media/tutorial-linux-vm-access-nonaad/create-key-vault.png)

1. Rellene toda la información necesaria y asegúrese de elegir la suscripción y el grupo de recursos donde creó la máquina virtual que usa para este tutorial.
1. Seleccione **Revisar y crear**.
1. Seleccione **Crear**

### <a name="create-a-secret"></a>Crear un secreto

A continuación, agregue un secreto al almacén de claves, de forma que más adelante pueda recuperarlo mediante código que se ejecuta en la máquina virtual. Para este tutorial utilizamos PowerShell, pero los mismos conceptos se aplican a cualquier código que se ejecute en esta máquina virtual.

1. Vaya al almacén de claves recién creado.
1. Seleccione **Secretos** y haga clic en **Agregar**.
1. Seleccione **Generar o importar**.
1. En la pantalla **Crear un secreto** desde **Opciones de carga**, deje la opción **Manual** seleccionada.
1. Escriba un nombre y un valor para el secreto.  El valor puede ser cualquiera de su elección. 
1. Deje la fecha de activación y la fecha de expiración y deje la opción **Habilitado** en **Sí**. 
1. Haga clic en **Crear** para crear el secreto.

   ![Crear un secreto](./media/tutorial-linux-vm-access-nonaad/create-secret.png)

## <a name="grant-access"></a>Conceder acceso

Es necesario conceder acceso a la identidad administrada que usa la máquina virtual para leer el secreto que se almacenará en el almacén de claves.

1. Vaya al almacén de claves recién creado.
1. Seleccione **Directiva de acceso** en el menú de la izquierda.
1. Seleccione **Agregar directiva de acceso**.

   ![Pantalla de creación de directivas de acceso de Key Vault](./media/tutorial-linux-vm-access-nonaad/key-vault-access-policy.png)

1. En la sección **Agregar directiva de acceso** situada en **Configurar a partir de una plantilla (opcional)** , elija **Administración de secretos** en el menú desplegable.
1. Elija **Seleccionar la entidad de seguridad** y, en el campo de búsqueda, escriba el nombre de la máquina virtual que creó anteriormente.  Seleccione la máquina virtual de la lista de resultados y elija **Seleccionar**.
1. Seleccione **Agregar**.
1. Seleccione **Guardar**.

## <a name="access-data"></a>Acceso a los datos

Para completar estos pasos, necesitará un cliente SSH.  Si usa Windows, puede usar el cliente SSH en el [Subsistema de Windows para Linux](/windows/wsl/about). Si necesita ayuda para configurar las claves del cliente de SSH, consulte [Uso de SSH con Windows en Azure](../../virtual-machines/linux/ssh-from-windows.md) o [Creación y uso de un par de claves SSH pública y privada para máquinas virtuales Linux en Azure](../../virtual-machines/linux/mac-create-ssh-keys.md).

>[!IMPORTANT]
> Todos los SDK de Azure admiten la biblioteca Azure.Identity, lo que facilita la adquisición de tokens de Azure AD para acceder a los servicios de destino. Obtenga más información sobre [los SDK de Azure](https://azure.microsoft.com/downloads/) y saque provecho de la biblioteca Azure.Identity.
> - [.NET](/dotnet/api/overview/azure/identity-readme)
> - [JAVA](/java/api/overview/azure/identity-readme)
> - [Javascript](/javascript/api/overview/azure/identity-readme)
> - [Python](/python/api/overview/azure/identity-readme)


1. En el portal, vaya a la máquina virtual Linux y, en **Información general**, haga clic en **Conectar**. 
2. **Conéctese** a la máquina virtual con el cliente SSH que elija. 
3. En la ventana del terminal, utilice CURL para realizar una solicitud al punto de conexión de la característica Managed Identities for Azure Resources y obtener un token de acceso para Azure Key Vault.  
 
    La solicitud CURL para el token de acceso está debajo.  
    
    ```bash
    curl 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fvault.azure.net' -H Metadata:true  
    ```
    La respuesta incluye el token de acceso que necesita para acceder a Resource Manager. 
    
    Respuesta:  
    
    ```bash
    {"access_token":"eyJ0eXAi...",
    "refresh_token":"",
    "expires_in":"3599",
    "expires_on":"1504130527",
    "not_before":"1504126627",
    "resource":"https://vault.azure.net",
    "token_type":"Bearer"} 
    ```
    
    Puede usar este token de acceso para autenticarse en Azure Key Vault.  La siguiente solicitud CURL muestra cómo leer un secreto desde el almacén de claves usando CURL y la API de REST de Key Vault.  Necesitará la dirección URL de su almacén de claves, que se encuentra en la sección **Información esencial** de la página **Información general** del almacén de claves.  También necesitará el token de acceso que obtuvo en la llamada anterior. 
        
    ```bash
    curl 'https://<YOUR-KEY-VAULT-URL>/secrets/<secret-name>?api-version=2016-10-01' -H "Authorization: Bearer <ACCESS TOKEN>" 
    ```
    
    La respuesta tendrá un aspecto similar al siguiente: 
    
    ```bash
    {"value":"p@ssw0rd!","id":"https://mytestkeyvault.vault.azure.net/secrets/MyTestSecret/7c2204c6093c4d859bc5b9eff8f29050","attributes":{"enabled":true,"created":1505088747,"updated":1505088747,"recoveryLevel":"Purgeable"}} 
    ```
    
Cuando haya recuperado el secreto del almacén de claves, podrá usarlo para autenticarse en un servicio que requiera un nombre y una contraseña.

## <a name="clean-up-resources"></a>Limpieza de recursos

Para limpiar los recursos, visite [Azure Portal](https://portal.azure.com), seleccione **Grupos de recursos**, busque y seleccione el grupo de recursos que se creó en el proceso de este tutorial (por ejemplo, `mi-test`) y, a continuación, use el comando **Eliminar grupo de recursos**.

Como alternativa, también puede realizar esto mediante [PowerShell o la CLI](../../azure-resource-manager/management/delete-resource-group.md).

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a utilizar una identidad administrada asignada por el sistema de una máquina virtual Linux para acceder a Azure Key Vault.  Para obtener más información sobre Azure Key Vault, vea:

> [!div class="nextstepaction"]
>[Azure Key Vault](../../key-vault/general/overview.md)