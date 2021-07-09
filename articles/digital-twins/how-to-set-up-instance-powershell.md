---
title: Configuración de una instancia y autenticación (PowerShell)
titleSuffix: Azure Digital Twins
description: Consulte cómo configurar una instancia del servicio Azure Digital Twins mediante Azure PowerShell.
author: baanders
ms.author: baanders
ms.date: 12/16/2020
ms.topic: how-to
ms.service: digital-twins
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: c52cd013d1a5d2cbde0ab88eac24d2fd3ef77d46
ms.sourcegitcommit: 6323442dbe8effb3cbfc76ffdd6db417eab0cef7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110613533"
---
# <a name="set-up-an-azure-digital-twins-instance-and-authentication-powershell"></a>Configuración de una instancia de Azure Digital Twins y autenticación (PowerShell)

[!INCLUDE [digital-twins-setup-selector.md](../../includes/digital-twins-setup-selector.md)]

En este artículo se describen los pasos para **configurar una nueva instancia de Azure Digital Twins**, incluidas la creación de la instancia y la configuración de la autenticación. Después de completar este artículo, tendrá una instancia de Azure Digital Twins lista para empezar a programar.

En esta versión de este artículo se realizan los pasos manualmente, uno por uno, mediante [Azure PowerShell](/powershell/azure/new-azureps-module-az).

* Para seguir estos pasos manualmente con Azure Portal, consulte la versión del portal de este artículo: [Procedimiento: Configuración de una instancia y autenticación (Azure Portal)](how-to-set-up-instance-portal.md) .
* Para ejecutar una configuración automatizada mediante un script de implementación de ejemplo, consulte la versión con scripts de este artículo: [Procedimiento: Configuración de una instancia y autenticación (con scripts)](how-to-set-up-instance-scripted.md) .

[!INCLUDE [digital-twins-setup-steps.md](../../includes/digital-twins-setup-steps.md)]

## <a name="prepare-your-environment"></a>Preparación del entorno

1. En primer lugar, elija dónde ejecutar los comandos de este artículo. Puede elegir ejecutar comandos de Azure PowerShell mediante una instalación local de Azure PowerShell o en una ventana del explorador con [Azure Cloud Shell](https://shell.azure.com).
    * Si opta por usar Azure PowerShell en un entorno local:
       1. [Instale el módulo de PowerShell](/powershell/azure/install-az-ps).
       1. Abra una ventana de PowerShell en la máquina.
       1. Conéctese a su cuenta de Azure mediante el cmdlet [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount).
    * Si decide usar Azure Cloud Shell:
        1. Consulte [Introducción a Azure Cloud Shell](../cloud-shell/overview.md) para más información acerca de Cloud Shell.
        1. [Abra una ventana de Cloud Shell en el explorador](https://shell.azure.com).
        1. En la barra de iconos de Cloud Shell, asegúrese de que Cloud Shell está establecido para ejecutar la versión de PowerShell.
    
            :::image type="content" source="media/how-to-set-up-instance/cloud-shell/cloud-shell-powershell.png" alt-text="Captura de pantalla de la ventana de Cloud Shell en Azure Portal que muestra la selección de la versión de PowerShell.":::
    
1. Si tiene varias suscripciones a Azure, elija la suscripción adecuada en la que se debe facturar el recurso. Seleccione una suscripción específica con el cmdlet [Set-AzContext](/powershell/module/az.accounts/set-azcontext).

   ```azurepowershell-interactive
   Set-AzContext -SubscriptionId 00000000-0000-0000-0000-000000000000
   ```

1. Si esta es la primera vez que usa Azure Digital Twins con esta suscripción, debe registrar el proveedor de recursos **Microsoft.DigitalTwins**. (Si no está seguro, es correcto volver a ejecutarlo aunque lo haya hecho en algún momento del pasado).

   ```azurepowershell-interactive
   Register-AzResourceProvider -ProviderNamespace Microsoft.DigitalTwins
   ```

1. Use el siguiente comando para instalar el módulo **Az.DigitalTwins** de PowerShell.
    ```azurepowershell-interactive
    Install-Module -Name Az.DigitalTwins
    ```

> [!IMPORTANT]
> Si bien el módulo **Az.DigitalTwins** de PowerShell está en versión preliminar, se debe instalar por separado mediante el cmdlet `Install-Module` como se ha indicado anteriormente. Una vez que este módulo de PowerShell esté disponible con carácter general, formará parte de las futuras versiones del módulo Az de PowerShell y estará disponible de forma predeterminada en Azure Cloud Shell.

## <a name="create-the-azure-digital-twins-instance"></a>Creación de una instancia de Azure Digital Twins

En esta sección, **creará una instancia de Azure Digital Twins** mediante Azure PowerShell.
Tendrá que proporcionar:

* Un [grupo de recursos de Azure](../azure-resource-manager/management/overview.md) en el que se implementará la instancia. Si no tiene ninguno, puede crear uno nuevo con el cmdlet [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup):

  ```azurepowershell-interactive
  New-AzResourceGroup -Name <name-for-your-resource-group> -Location <region>
  ```

* Una región para la implementación. Para ver qué regiones admiten Azure Digital Twins, visite [Productos de Azure disponibles por región](https://azure.microsoft.com/global-infrastructure/services/?products=digital-twins).
* Un nombre para la instancia. El nombre de la nueva instancia debe ser único dentro de la región de la suscripción. Si su suscripción tiene otra instancia de Azure Digital Twins en la región que ya usa el nombre especificado, se le pedirá que elija un nombre diferente.

Use los valores en el comando siguiente para crear la instancia:

```azurepowershell-interactive
New-AzDigitalTwinsInstance -ResourceGroupName <your-resource-group> -ResourceName <name-for-your-Azure-Digital-Twins-instance> -Location <region>
```

### <a name="verify-success-and-collect-important-values"></a>Comprobación de que la operación es correcta y recopilación de valores importantes

Si la instancia se creó correctamente, el resultado tiene un aspecto similar al de la siguiente salida que contiene información sobre el recurso que ha creado:

```Output
Location   Name                                         Type
--------   ----                                         ----
<region>   <name-for-your-Azure-Digital-Twins-instance> Microsoft.DigitalTwins/digitalTwinsInstances
```

A continuación, muestre las propiedades de la nueva instancia mediante la ejecución de `Get-AzDigitalTwinsInstance` y la canalización a `Select-Object -Property *`, de la siguiente manera:

```azurepowershell-interactive
Get-AzDigitalTwinsInstance -ResourceGroupName <your-resource-group> -ResourceName <name-for-your-Azure-Digital-Twins-instance> |
  Select-Object -Property *
```

> [!TIP]
> Puede usar este comando para ver todas las propiedades de la instancia en cualquier momento.

Anote los valores de **nombre de host**, **nombre** y **grupo de recursos** de la instancia de Azure Digital Twins. Estos son valores importantes que puede necesitar a medida que sigue trabajando con la instancia de Azure Digital Twins para configurar la autenticación y los recursos de Azure relacionados. Si otros usuarios van a programar en la instancia, debe compartirlos con ellos.

Ahora tiene lista una instancia de Azure Digital Twins. A continuación, debe proporcionar los permisos de usuario de Azure adecuados para administrarla.

## <a name="set-up-user-access-permissions"></a>Configuración de permisos de acceso de usuarios

[!INCLUDE [digital-twins-setup-role-assignment.md](../../includes/digital-twins-setup-role-assignment.md)]

### <a name="prerequisites-permission-requirements"></a>Requisitos previos: Requisitos de permisos
[!INCLUDE [digital-twins-setup-permissions.md](../../includes/digital-twins-setup-permissions.md)]

### <a name="assign-the-role"></a>Asignación del rol

Para conceder a un usuario permisos para administrar una instancia de Azure Digital Twins, debe asignarle al rol _**Propietario de datos de Azure Digital Twins**_ dentro de la instancia.

En primer lugar, determine el valor **ObjectId** para la cuenta de Azure AD del usuario al que se debe asignar el rol. Puede encontrar este valor mediante el cmdlet [Get-AzAdUser](/powershell/module/az.resources/get-azaduser) pasando el nombre principal de usuario en la cuenta de Azure AD para recuperar su ObjectId (y otra información de usuario). En la mayoría de los casos, el nombre principal de usuario coincidirá con el correo electrónico del usuario en la cuenta de Azure AD.

```azurepowershell-interactive
Get-AzADUser -UserPrincipalName <Azure-AD-user-principal-name-of-user-to-assign>
```

A continuación, use el valor **ObjectId** en el siguiente comando para asignar el rol. El comando también requiere que escriba el mismo identificador de suscripción, nombre del grupo de recursos y nombre de la instancia de Azure Digital Twins que eligió antes al crear la instancia. Un usuario con [suficientes permisos](#prerequisites-permission-requirements) en la suscripción de Azure debe ser el que ejecute el comando.

```azurepowershell-interactive
$Params = @{
  ObjectId = '<Azure-AD-user-object-ID-of-user-to-assign>'
  RoleDefinitionName = 'Azure Digital Twins Data Owner'
  Scope = '/subscriptions/<subscription-ID>/resourceGroups/<resource-group-name>/providers/Microsoft.DigitalTwins/digitalTwinsInstances/<name-for-your-Azure-Digital-Twins-instance>'
}
New-AzRoleAssignment @Params
```

El resultado de este comando es la información de salida acerca de la asignación de roles que se ha creado.

### <a name="verify-success"></a>Comprobación de que la operación se ha completado correctamente

[!INCLUDE [digital-twins-setup-verify-role-assignment.md](../../includes/digital-twins-setup-verify-role-assignment.md)]

Ahora tiene lista una instancia de Azure Digital Twins y los permisos asignados para administrarla.

## <a name="next-steps"></a>Pasos siguientes

Consulte cómo conectar una aplicación cliente a la instancia mediante el código de autenticación:
* [Procedimiento: Escritura de código de autenticación de aplicación](how-to-authenticate-client.md)
