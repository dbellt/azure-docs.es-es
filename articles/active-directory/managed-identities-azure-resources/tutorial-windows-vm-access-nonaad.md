---
title: 'Tutorial: Uso de identidades administradas para acceder a Azure Key Vault (Windows): Azure AD'
description: Este tutorial contiene directrices acerca de cómo utilizar una identidad administrada asignada por el sistema de una máquina virtual Windows para acceder a Azure Key Vault.
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
ms.date: 12/10/2020
ms.author: barclayn
ms.collection: M365-identity-device-management
ms.openlocfilehash: 92ded2d327fd7c75633bf7ef6b7dd3041725c921
ms.sourcegitcommit: 2654d8d7490720a05e5304bc9a7c2b41eb4ae007
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107378268"
---
# <a name="tutorial-use-a-windows-vm-system-assigned-managed-identity-to-access-azure-key-vault"></a>Tutorial: Uso de las identidades administradas asignadas por el sistema de una máquina virtual Windows para acceder a Azure Key Vault 

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

En este tutorial se muestra cómo puede usar una máquina virtual Windows una identidad administrada asignada por el sistema para acceder a [Azure Key Vault](../../key-vault/general/overview.md). Actuando como un arranque, Key Vault hace posible que la aplicación cliente use un secreto para acceder a recursos que no están protegidos por Azure Active Directory (AD). Las identidades de servicio administradas son administradas automáticamente por Azure y le permiten autenticarse con servicios que admiten la autenticación de Azure AD sin necesidad de incluir información de autenticación en el código.

Aprenderá a:

> [!div class="checklist"]
> * Concesión de acceso a la máquina virtual a un secreto almacenado en un almacén de claves
> * Obtener un token de acceso mediante la identidad de máquina virtual y usarlo para recuperar el secreto de Key Vault 

## <a name="prerequisites"></a>Prerrequisitos

- Conocimientos sobre identidades administradas. Si no está familiarizado con la característica Managed Identities for Azure Resources, consulte esta [introducción](overview.md). 
- Una cuenta de Azure, [regístrese para obtener una cuenta gratuita](https://azure.microsoft.com/free/).
- Para realizar la creación de los recursos necesarios y los pasos de administración de roles, debe tener permisos de "Propietario" en el ámbito adecuado (la suscripción o el grupo de recursos). Si necesita ayuda con la asignación de roles, consulte [Asignación de roles de Azure para administrar el acceso a los recursos de una suscripción de Azure](../../role-based-access-control/role-assignments-portal.md).
- También necesita una máquina virtual Windows que tenga habilitadas las identidades administradas asignadas por el sistema.
  - Si necesita crear una máquina virtual para este tutorial, puede seguir el artículo titulado [Configurar identidades administradas para recursos de Azure en una VM mediante Azure Portal](./qs-configure-portal-windows-vm.md#system-assigned-managed-identity).

## <a name="create-a-key-vault"></a>Creación de un almacén de claves  

En esta sección se muestra cómo conceder acceso a la máquina virtual a un secreto almacenado en un almacén de claves. Mediante Managed Identities for Azure Resources, el código puede obtener tokens de acceso para autenticarse en aquellos recursos que admitan la autenticación de Azure AD.Sin embargo, no todos los servicios de Azure admiten la autenticación de Azure AD. Para usar Managed Identities for Azure Resources con esos servicios, almacene las credenciales del servicio en Azure Key Vault y use una identidad administrada de la máquina virtual a fin de acceder a Key Vault y recuperar las credenciales.

En primer lugar, es necesario crear un almacén de claves y conceder a la identidad administrada asignada por el sistema de la máquina virtual acceso al mismo.

1. Abra [Azure Portal](https://portal.azure.com/).
1. En la parte superior de la barra de navegación izquierda, seleccione **Crear un recurso**.  
1. En el cuadro de texto **Buscar en el Marketplace**, escriba **Key Vault** y presione **Entrar**.  
1. En la lista de resultados, seleccione **Key Vault**.
1. Seleccione **Crear**
1. Proporcione un **nombre** para el nuevo almacén de claves.

    ![Pantalla de creación de un almacén de claves](./media/msi-tutorial-windows-vm-access-nonaad/create-key-vault.png)

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

   ![Crear un secreto](./media/msi-tutorial-windows-vm-access-nonaad/create-secret.png)

## <a name="grant-access"></a>Conceder acceso

Es necesario conceder acceso a la identidad administrada que usa la máquina virtual para leer el secreto que se almacenará en el almacén de claves.

1. Vaya al almacén de claves recién creado.
1. Seleccione **Directiva de acceso** en el menú de la izquierda.
1. Seleccione **Agregar directiva de acceso**.

   ![Pantalla de creación de directivas de acceso de Key Vault](./media/msi-tutorial-windows-vm-access-nonaad/key-vault-access-policy.png)

1. En la sección **Agregar directiva de acceso** situada en **Configurar a partir de una plantilla (opcional)** , elija **Administración de secretos** en el menú desplegable.
1. Elija **Seleccionar la entidad de seguridad** y, en el campo de búsqueda, escriba el nombre de la máquina virtual que creó anteriormente.  Seleccione la máquina virtual de la lista de resultados y elija **Seleccionar**.
1. Seleccione **Agregar**.
1. Seleccione **Guardar**.


## <a name="access-data"></a>Acceso a los datos  

En esta sección se muestra cómo obtener un token de acceso mediante la identidad de la máquina virtual y usarlo para recuperar el secreto del almacén de claves. Si no tiene PowerShell 4.3.1 o superior instalado, necesitará [descargar e instalar la versión más reciente](/powershell/azure/).

En primer lugar, vamos a usar la identidad administrada asignada por el sistema de la máquina virtual a fin de obtener un token de acceso para autenticarse en Key Vault:
 
1. En el portal, vaya a **Virtual Machines** y diríjase a la máquina virtual Windows. A continuación, en **Introducción**, haga clic en **Conectar**.
2. Escriba su **nombre de usuario** y **contraseña** que agregó cuando creó la **máquina virtual Windows**.  
3. Ahora que ha creado una **conexión a Escritorio remoto** con la máquina virtual, abra PowerShell en la sesión remota.  
4. En PowerShell, invoque la solicitud web en el inquilino para obtener el token del host local en el puerto específico de la máquina virtual.  

La solicitud de PowerShell:

```powershell
$Response = Invoke-RestMethod -Uri 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fvault.azure.net' -Method GET -Headers @{Metadata="true"} 
```

Puede ver cómo es la respuesta a continuación:

![Solicitud con respuesta de token](./media/msi-tutorial-windows-vm-access-nonaad/token.png)

Luego, extraiga el token de acceso de la respuesta.  

```powershell
   $KeyVaultToken = $Response.access_token
```

Finalmente, use el comando Invoke-WebRequest de PowerShell para recuperar el secreto que creó anteriormente en el almacén de claves y pasar el token de acceso en el encabezado de autenticación.  Necesitará la dirección URL de su almacén de claves, que se encuentra en la sección de **Información esencial** de la página **Introducción** del almacén de claves.  

```powershell
Invoke-RestMethod -Uri https://<your-key-vault-URL>/secrets/<secret-name>?api-version=2016-10-01 -Method GET -Headers @{Authorization="Bearer $KeyVaultToken"}
```

La respuesta tendrá un aspecto similar al siguiente: 

```powershell
  value       id                                                                                    attributes
  -----       --                                                                                    ----------
  'My Secret' https://mi-lab-vault.vault.azure.net/secrets/mi-test/50644e90b13249b584c44b9f712f2e51 @{enabled=True; created=16…
```

Cuando haya recuperado el secreto del almacén de claves, podrá usarlo para autenticarse en un servicio que requiere un nombre y una contraseña.

## <a name="clean-up-resources"></a>Limpieza de recursos

Para limpiar los recursos, visite [Azure Portal](https://portal.azure.com), seleccione **Grupos de recursos**, busque y seleccione el grupo de recursos que se creó en el proceso de este tutorial (por ejemplo, `mi-test`) y, a continuación, use el comando **Eliminar grupo de recursos**.

Como alternativa, también puede realizar esto mediante [PowerShell o la CLI](../../azure-resource-manager/management/delete-resource-group.md).

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a utilizar una identidad administrada asignada por el sistema de una máquina virtual Windows para acceder a Azure Key Vault.  Para obtener más información sobre Azure Key Vault, vea:

> [!div class="nextstepaction"]
>[Azure Key Vault](../../key-vault/general/overview.md)
