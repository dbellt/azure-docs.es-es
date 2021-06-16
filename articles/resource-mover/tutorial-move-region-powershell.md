---
title: Movimiento de recursos entre regiones mediante PowerShell en Azure Resource Mover
description: Aprenda a mover recursos entre regiones mediante PowerShell en Azure Resource Mover.
manager: evansma
author: rayne-wiselman
ms.service: resource-move
ms.topic: tutorial
ms.date: 02/21/2021
ms.author: raynew
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 3489d7359b7557f0a2305a30ff3d28f05863504d
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110702252"
---
# <a name="move-resources-across-regions-in-powershell"></a>Movimiento de recursos entre regiones en PowerShell

En este artículo se explica cómo mover recursos de Azure a otra región mediante PowerShell en [Azure Resource Mover](overview.md).

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Comprobar los requisitos previos y otras exigencias.
> * Configurar la colección de transferencia de recursos.
> * Agregar recursos a la colección de transferencia de recursos y resolver las dependencias.
> * Preparar y trasladar el grupo de recursos de origen. 
> * Preparar y trasladar los demás recursos.
> * Elegir si desea descartar o confirmar el traslado. 
> * Opcionalmente, quitar los recursos de la región después del traslado.

> [!NOTE]
> Los tutoriales muestran la manera más rápida de probar un escenario y utilizar las opciones predeterminadas. 

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/pricing/free-trial/) antes de empezar. Luego, inicie sesión en el [Portal de Azure](https://portal.azure.com).

## <a name="prerequisites"></a>Requisitos previos
**Requisito** | **Descripción**
--- | ---
**Permisos de suscripción** | Compruebe que tiene acceso de *Propietario* a la suscripción que contiene los recursos que desea trasladar.<br/><br/> **¿Por qué necesito acceso de Propietario?** La primera vez que agregue un recurso de un par de origen y destino específicos a una suscripción de Azure, Azure Resource Mover creará una [identidad administrada asignada por el sistema](../active-directory/managed-identities-azure-resources/overview.md#managed-identity-types), anteriormente llamada identidad de servicio administrado (MSI), en la que confiará la suscripción. Para crear la identidad y asignarle el rol requerido (Colaborador o Administrador de acceso de usuario en la suscripción de origen), la cuenta que utilice para agregar recursos necesita permisos de *propietario* para la suscripción. [Obtenga más información](../role-based-access-control/rbac-and-directory-admin-roles.md#azure-roles) sobre los roles de Azure.
**Compatibilidad con Resource Mover** | [Revise](common-questions.md) las regiones admitidas y otras preguntas comunes.
**Soporte técnico de máquina virtual** |  Compruebe que las máquinas virtuales que desea trasladar estén admitidas.<br/><br/> - [Compruebe](support-matrix-move-region-azure-vm.md#windows-vm-support) las máquinas virtuales Windows admitidas.<br/><br/> - [Compruebe](support-matrix-move-region-azure-vm.md#linux-vm-support) las máquinas virtuales Linux y las versiones de kernel admitidas.<br/><br/> - Compruebe la configuración admitida de [proceso](support-matrix-move-region-azure-vm.md#supported-vm-compute-settings), [almacenamiento](support-matrix-move-region-azure-vm.md#supported-vm-storage-settings) y [redes](support-matrix-move-region-azure-vm.md#supported-vm-networking-settings).
**Soporte técnico de SQL** | Si desea trasladar recursos de SQL, consulte [Comprobación de los requisitos de SQL](tutorial-move-region-sql.md#check-sql-requirements).
**Suscripción de destino** | La suscripción de la región de destino debe tener cuota suficiente para crear los recursos que va a trasladar a la región de destino. Si no dispone de cuota suficiente, [solicite límites adicionales](../azure-resource-manager/management/azure-subscription-service-limits.md).
**Cargos de la región de destino** | Compruebe los precios y los cargos asociados con la región de destino a la que va a trasladar las máquinas virtuales. El uso de la [calculadora de precios](https://azure.microsoft.com/pricing/calculator/) le resultará útil.

### <a name="review-powershell-requirements"></a>Revisión de los requisitos de PowerShell

La mayoría de las operaciones de movimiento de recursos son las mismas, independientemente de si se usa Azure Portal o PowerShell, con un par de excepciones.

**Operación** | **PowerShell** | **Portal**
--- | --- | ---
**Crear una colección de transferencia de recursos** | Se crea automáticamente una colección de transferencia de recursos (una lista de todos los recursos que se van a mover). El portal asigna los permisos de identidad necesarios en el back-end. | Puede utilizar cmdlets de PowerShell para:<br/><br/> - Crear un grupo de recursos para la colección de transferencia de recursos y especificar su ubicación.<br/><br/> - Asignar una identidad administrada a la colección.<br/><br/> - Agregar recursos a la colección.
**Eliminar una colección de transferencia de recursos** | No se puede eliminar directamente una colección de transferencia de recursos en el portal. | Utilice un cmdlet de PowerShell para eliminar una colección de transferencia de recursos.
**Operaciones de movimiento de recursos**<br/><br/> (Preparación, movimiento inicial, confirmación, etc.).| Pasos únicos con validación automática por parte de Resource Mover. | Cmdlets de PowerShell para:<br/><br/> 1) Validar las dependencias.<br/><br/> 2) Realizar el traslado.
**Eliminación de los recursos de origen** | Directamente en el portal de Resource Mover. | Cmdlets de PowerShell en el nivel de tipo de recurso.


### <a name="sample-values"></a>Valores de ejemplo

Usaremos estos valores en los ejemplos de script:

**Configuración** | **Valor** 
--- | ---
Id. de suscripción | subscription-id
Región de origen |  Centro de EE. UU.
Región de destino | Centro-Oeste de EE. UU.
Grupo de recursos (contiene los metadatos de la colección de transferencia de recursos) | RG-MoveCollection-demoRMS
Nombre de la colección de movimiento | PS-centralus-westcentralus-demoRMS
Grupo de recursos (región de origen) | PSDemoRM 
Grupo de recursos (región de destino) | PSDemoRM-target
Ubicación del servicio Resource Mover | Este de EE. UU. 2
Tipo de identidad | Asignada por el sistema
Máquina virtual que se va a mover | PSDemoVM


## <a name="sign-into-azure"></a>Inicio de sesión en Azure

Inicie sesión en la suscripción de Azure con el cmdlet Connect-AzAccount:

```azurepowershell-interactive
Connect-AzAccount – Subscription "<subscription-id>"
```

## <a name="set-up-the-move-collection"></a>Configuración de la colección de transferencia de recursos

El objeto MoveCollection almacena los metadatos y la información de configuración de los recursos que se quieren mover. Para configurar una colección de transferencia de recursos, haga lo siguiente:

- Cree un grupo de recursos para la colección de transferencia de recursos.
- Registre el proveedor de servicios en la suscripción para que se pueda crear el recurso MoveCollection.
- Cree el objeto MoveCollection con una identidad administrada. Para que el objeto MoveCollection tenga acceso a la suscripción en la que se encuentra el servicio Resource Mover, necesita una [identidad administrada asignada por el sistema](../active-directory/managed-identities-azure-resources/overview.md#managed-identity-types) (antes conocida como identidad de servicio administrada (MSI)) en la que confíe la suscripción.
- Conceda acceso a la suscripción de Resource mover para la identidad administrada.

### <a name="create-the-resource-group"></a>Crear el grupo de recursos

Cree un grupo de recursos para los metadatos de la colección de transferencia de recursos y la información de configuración, como se indica a continuación:

```azurepowershell-interactive
New-AzResourceGroup -Name "RG-MoveCollection-demoRMS" -Location "East US 2"
```
**Salida**:

![Texto de salida después de crear el grupo de recursos](./media/tutorial-move-region-powershell/create-metadata-resource-group.png)

### <a name="register-the-resource-provider"></a>Registrar el proveedor de recursos

1. Registre el proveedor de recursos Microsoft.Migrate para que se pueda crear el recurso MoveCollection, como se indica a continuación:

    ```azurepowershell-interactive
    Register-AzResourceProvider -ProviderNamespace Microsoft.Migrate
    
2. Wait for registration:

    ```azurepowershell-interactive 
    While(((Get-AzResourceProvider -ProviderNamespace Microsoft.Migrate)| where {$_.RegistrationState -eq "Registered" -and $_.ResourceTypes.ResourceTypeName -eq "moveCollections"}|measure).Count -eq 0)
    {
        Start-Sleep -Seconds 5
        Write-Output "Waiting for registration to complete."
    }
    ```
### <a name="create-a-movecollection-object"></a>Creación de un objeto MoveCollection

Cree un objeto MoveCollection y asígnele una identidad administrada, como se indica a continuación: 

```azurepowershell-interactive
New-AzResourceMoverMoveCollection -Name "PS-centralus-westcentralus-demoRMS"  -ResourceGroupName "RG-MoveCollection-demoRMS" -SourceRegion "centralus" -TargetRegion "westcentralus" -Location "centraluseuap" -IdentityType "SystemAssigned"
```

**Salida**:

![Texto de salida después de crear la colección de movimiento](./media/tutorial-move-region-powershell/output-move-collection.png)


### <a name="grant-access-to-the-managed-identity"></a>Concesión de acceso a la identidad administrada

Conceda acceso a la suscripción de Resource mover a la identidad administrada como se indica a continuación. El usuario debe ser propietario de la suscripción.

1. Recupere los detalles de la identidad del objeto MoveCollection.

    ```azurepowershell-interactive
    $moveCollection = Get-AzResourceMoverMoveCollection -SubscriptionId $subscriptionId -ResourceGroupName "RG-MoveCollection-demoRMS" -Name "PS-centralus-westcentralus-demoRMS"

    $identityPrincipalId = $moveCollection.IdentityPrincipalId   
    ``` 

2. Asigne los roles necesarios a la identidad para que Azure Resource Mover pueda acceder a la suscripción para ayudar a trasladar recursos.

    ```azurepowershell-interactive
    New-AzRoleAssignment -ObjectId $identityPrincipalId -RoleDefinitionName Contributor -Scope "/subscriptions/$subscriptionId"

    New-AzRoleAssignment -ObjectId $identityPrincipalId -RoleDefinitionName "User Access Administrator" -Scope "/subscriptions/$subscriptionId"
    ``` 

## <a name="add-resources-to-the-move-collection"></a>Adición de recursos a la colección de movimientos

Recupere los identificadores de los recursos de origen existentes que desee trasladar. Cree el objeto de configuración de recursos de destino y, a continuación, agregue recursos a la colección de transferencia de recursos.

> [!NOTE]
> Los recursos agregados a una colección de movimiento deben estar en la misma suscripción, pero pueden estar en distintos grupos de recursos.

Agregue los recursos como se indica a continuación:

1. Obtenga el identificador del recurso de origen:

    ```azurepowershell-interactive
    Get-AzResource -Name PSDemoVM -ResourceGroupName PSDemoRM
    ```

    **Salida**

    ![Texto de salida después de recuperar el identificador de recurso](./media/tutorial-move-region-powershell/output-retrieve-resource.png)

2. Cree el objeto de configuración de recursos de destino de acuerdo con el recurso que va a trasladar. En nuestro caso, es una máquina virtual.

    ```azurepowershell-interactive
    $targetResourceSettingsObj = New-Object Microsoft.Azure.PowerShell.Cmdlets.ResourceMover.Models.Api202101.VirtualMachineResourceSettings
    ```

3. Establezca el tipo de recurso y el nombre del recurso de destino para el objeto.

    ```azurepowershell-interactive
    $targetResourceSettingsObj.ResourceType = "Microsoft.Compute/virtualMachines"
    $targetResourceSettingsObj.TargetResourceName = "PSDemoVM"
    ```
    > [!NOTE]
    > Nuestra máquina virtual de destino tiene el mismo nombre que la máquina virtual de la región de origen. Puede elegir otro nombre.

4. Agregue los recursos de origen a la colección de transferencia de recursos con el identificador de recurso y el objeto de configuración de destino que ha recuperado o creado.

    ```azurepowershell-interactive
    Add-AzResourceMoverMoveResource -ResourceGroupName "RG-MoveCollection-demoRMS" -MoveCollectionName "PS-centralus-westcentralus-demoRMS" -SourceId "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx xxxxxxxxxxxx/resourceGroups/
    PSDemoRM/providers/Microsoft.Compute/virtualMachines/PSDemoVM" -Name "PSDemoVM" -ResourceSetting $targetResourceSettingsObj
    ```

    **Salida** ![Texto de salida después de agregar el recurso](./media/tutorial-move-region-powershell/output-add-resource.png)

## <a name="validate-and-add-dependencies"></a>Validación y adición de las dependencias

Compruebe si los recursos que ha agregado tienen dependencias en otros recursos y agréguelos según sea necesario. 

1. Valide las dependencias de la siguiente manera:

    ```azurepowershell-interactive
    Resolve-AzResourceMoverMoveCollectionDependency -ResourceGroupName "RG-MoveCollection-demoRMS" -MoveCollectionName "PS-centralus-westcentralus-demoRMS"
    ```
    **Salida (cuando existen dependencias)**

    ![Texto de salida después de validar las dependencias](./media/tutorial-move-region-powershell/dependency-output.png)

2. Identidades a las que les faltan dependencias:

    - Para recuperar una lista de todas las dependencias que faltan:

        ```azurepowershell-interactive
        Get-AzResourceMoverUnresolvedDependency -MoveCollectionName "PS-centralus-westcentralus-demoRMS" -ResourceGroupName "RG-MoveCollection-demoRMS" -DependencyLevel Descendant"
        ```
        **Salida** ![Texto de salida después de recuperar una lista de todas las dependencias](./media/tutorial-move-region-powershell/dependencies-list.png)  

    - Para recuperar solo las dependencias de primer nivel (dependencias directas del recurso):

        ```azurepowershell-interactive
        Get-AzResourceMoverUnresolvedDependency -MoveCollectionName "PS-centralus-westcentralus-demoRMS" -ResourceGroupName "RG-MoveCollection-demoRMS" -DependencyLevel Direct
        ```
        **Salida** ![Texto de salida después de recuperar una lista de las dependencias de primer nivel](./media/tutorial-move-region-powershell/dependencies-list-direct.png)  

3. Para agregar las dependencias pendientes que faltan, repita las instrucciones anteriores para [agregar recursos a la colección de transferencia de recursos](#add-resources-to-the-move-collection) y vuelva a validar hasta que no haya recursos pendientes.

> [!NOTE]
> Si, por alguna razón, desea quitar recursos de la colección de recursos, siga las instrucciones de [este artículo](remove-move-resources.md).

## <a name="add-the-source-resource-group"></a>Adición del grupo de recursos de origen

Agregue el grupo de recursos de origen que contiene los recursos que desea trasladar a la colección de transferencia de recursos.

1. Recupere el identificador del grupo de recursos.

    ```azurepowershell-interactive
    Get-AzResourceMoverUnresolvedDependency -MoveCollectionName "PS-centralus-westcentralus-demoRMS" -ResourceGroupName "RG-MoveCollection-demoRMS" -DependencyLevel Direct
    ```
    **Salida** ![Texto de salida después de recuperar el identificador del grupo de recursos de origen](./media/tutorial-move-region-powershell/source-resource-group-id.png)  

    > [!NOTE]
    > Utilizamos un grupo de recursos que ya existe en la región de destino.

 
2. Use el identificador que ha recuperado para agregar el grupo de recursos a la colección.

    ```azurepowershell-interactive
    Add-AzResourceMoverMoveResource -ResourceGroupName "RG-MoveCollection-demoRMS"  -MoveCollectionName "PS-centralus-westcentralus-demoRMS" -SourceId "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourcegroups/psdemorm"  -Name "psdemorm"  -ExistingTargetId "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourcegroups/PSDemoRM-target"
    ```
    **Salida** ![Texto de salida después de agregar el grupo de recursos de origen a la colección de transferencia de recursos](./media/tutorial-move-region-powershell/add-source-resource-group.png)

3. Compruebe las dependencias para asegurarse de que no haya quedado nada después de agregar el grupo de recursos.

    ```azurepowershell-interactive
    Resolve-AzResourceMoverMoveCollectionDependency -ResourceGroupName "RG-MoveCollection-demoRMS" -MoveCollectionName "PS-centralus-westcentralus-demoRMS"
    ```
4. Vemos que no hay dependencias pendientes.

    **Salida** ![Texto de salida después de comprobar las dependencias](./media/tutorial-move-region-powershell/all-dependencies-added.png)


## <a name="prepare-resources"></a>Preparación de recursos

Normalmente, es necesario preparar los recursos de la región de origen antes del traslado. Por ejemplo:

- Para trasladar recursos sin estado, como las redes virtuales de Azure, los adaptadores de red, los equilibradores de carga y los grupos de seguridad de red, es posible que necesite exportar una plantilla de Azure Resource Manager.
- Para trasladar recursos con estado, como máquinas virtuales de Azure y bases de datos SQL, es posible que necesite iniciar la replicación de recursos desde la región de origen a la de destino.

En este tutorial, dado que vamos a trasladar máquinas virtuales, es necesario preparar el grupo de recursos de origen y, a continuación, iniciar y confirmar su traslado, antes de que podamos empezar a preparar las máquinas virtuales.

> [!NOTE]
> Si tiene un grupo de recursos de destino existente, puede confirmar directamente el movimiento para el grupo de recursos de origen y omitir las fases de preparación e inicio del traslado.

  
### <a name="prepare-the-source-resource-group"></a>Preparación del grupo de recursos de origen:

1. Prepare el grupo de recursos:

    ```azurepowershell-interactive
    Invoke-AzResourceMoverPrepare -ResourceGroupName "RG-MoveCollection-demoRMS" -MoveCollectionName "PS-centralus-westcentralus-demoRMS"  -MoveResource “PSDemoRM”
    ```
    **Salida**

    ![Texto de salida después de preparar el grupo de recursos de origen](./media/tutorial-move-region-powershell/prepare-source-resource-group.png)

2. Inicie el traslado del grupo de recursos de origen.

    ```azurepowershell-interactive
    "RG-MoveCollection-demoRMS" -MoveCollectionName "PS-centralus-westcentralus-demoRMS"  -MoveResource “PSDemoRM”
    ```
    ![Texto de salida después de iniciar el traslado del grupo de recursos de origen](./media/tutorial-move-region-powershell/initiate-move-source-resource-group.png)

3. Confirme el traslado del grupo de recursos de origen.

    ```azurepowershell-interactive
    Invoke-AzResourceMoverCommit -ResourceGroupName "RG-MoveCollection-demoRMS" -MoveCollectionName "PS-centralus-westcentralus-demoRMS"  -MoveResource “PSDemoRM”
    ```
    **Salida**

    ![Texto de salida después de confirmar el grupo de recursos de origen](./media/tutorial-move-region-powershell/commit-move-source-resource-group.png)


### <a name="prepare-vm-resources"></a>Preparación de los recursos de máquina virtual

Después de preparar y mover el grupo de recursos de origen, podemos preparar los recursos de máquina virtual para su traslado.

1. Valide las dependencias antes de preparar los recursos de máquina virtual.

    ```azurepowershell-interactive
    $resp = Invoke-AzResourceMoverPrepare -ResourceGroupName "RG-MoveCollection-demoRMS" -MoveCollectionName "PS-centralus-westcentralus-demoRMS"  -MoveResource $('psdemovm') -ValidateOnly
    ```
    **Salida**

    ![Texto de salida después de validar la máquina virtual antes de prepararla](./media/tutorial-move-region-powershell/validate-vm-before-move.png)

2. Obtenga los recursos dependientes que se deben preparar junto con la máquina virtual.

    ```azurepowershell-interactive
    $resp.AdditionalInfo[0].InfoMoveResource
    ```
    **Salida**

    ![Texto de salida después de recuperar los recursos de máquina virtual dependientes](./media/tutorial-move-region-powershell/get-resources-before-prepare.png)

3. Inicie el proceso de preparación de todos los recursos dependientes.

    ```azurepowershell-interactive
    Invoke-AzResourceMoverPrepare -ResourceGroupName "RG-MoveCollection-demoRMS" -MoveCollectionName "PS-centralus-westcentralus-demoRMS"  -MoveResource $('PSDemoVM','psdemovm111', 'PSDemoRM-vnet','PSDemoVM-nsg')
    ```
    **Salida**

    ![Texto de salida después de iniciar la preparación de todos los recursos](./media/tutorial-move-region-powershell/initiate-prepare-all.png)


    > [!NOTE]
    > Puede proporcionar el identificador de recurso de origen en lugar del nombre de recurso como parámetros de entrada para el cmdlet de preparación, así como en los cmdlets de inicio y de confirmación. Para hacerlo, ejecute lo siguiente:


    ```azurepowershell-interactive
        Invoke-AzResourceMoverPrepare -ResourceGroupName "RG-MoveCollection-demoRMS" -MoveCollectionName "PS-centralus-westcentralus-demoRMS" -MoveResourceInputType MoveResourceSourceId  -MoveResource $('/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/PSDemoRMS/providers/Microsoft.Network/networkSecurityGroups/PSDemoVM-nsg')
    ```

## <a name="initiate-move-of-vm-resources"></a>Inicio del traslado de recursos de máquina virtual

1. Compruebe que los recursos de máquina virtual tengan el estado *Initiate Move Pending* (Iniciar movimiento pendiente):

    ```azurepowershell-interactive
    Get-AzResourceMoverMoveResource  -SubscriptionId “ xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx “ -ResourceGroupName “RG-MoveCollection-demoRMS” -MoveCollectionName “PS-centralus-westcentralus-demoRMS ”   | Where-Object {  $_.MoveStatusMoveState -eq “InitiateMovePending” } | Select Name
    ```    

    **Salida**

    ![Texto de salida después de comprobar el estado de inicio](./media/tutorial-move-region-powershell/verify-initiate-move-pending.png)

2. Inicie el traslado:

    ```azurepowershell-interactive
    Invoke-AzResourceMoverInitiateMove -ResourceGroupName "RG-MoveCollection-demoRMS" -MoveCollectionName "PS-centralus-westcentralus-demoRMS"  -MoveResource $('psdemovm111', 'PSDemoRM-vnet','PSDemoVM-nsg', ‘PSDemoVM’) -MoveResourceInputType "MoveResourceId"
    ```    

    **Salida**

    ![Texto de salida después de iniciar el traslado de recursos](./media/tutorial-move-region-powershell/initiate-resources-move.png)


## <a name="discard-or-commit"></a>¿Descartar o confirmar?

Tras la operación inicial de traslado, puede decidir si desea confirmarla o descartarla. 

- **Descartar**: puede descartar un traslado cuando esté haciendo pruebas y no quiera trasladar verdaderamente el recurso de origen. Si el traslado se descarta, el recurso volverá a tener el estado *Initiate move pending* (Iniciar movimiento pendiente). Después, puede volver a iniciar el movimiento si es necesario.
- **Confirmación**: la confirmación completa la operación de traslado a la región de destino. Tras la confirmación, un recurso de origen tendrá el estado *Delete source pending* (Eliminar origen pendiente) y el usuario puede decidir si desea eliminarlo.

### <a name="discard-the-move"></a>Descartar la operación de traslado

Para descartar el movimiento:

```azurepowershell-interactive
Invoke-AzResourceMoverDiscard -ResourceGroupName "RG-MoveCollection-demoRMS" -MoveCollectionName "PS-centralus-westcentralus-demoRMS"  -MoveResource $('psdemovm111', 'PSDemoRM-vnet','PSDemoVM-nsg', ‘PSDemoVM’) -MoveResourceInputType "MoveResourceId"
```
**Salida**

![Texto de salida después de descartar el movimiento](./media/tutorial-move-region-powershell/discard-move.png)


### <a name="commit-the-move"></a>Confirmación de la operación de traslado

1. Confirme el traslado de la siguiente manera:

    ```azurepowershell-interactive
    Invoke-AzResourceMoverCommit -ResourceGroupName "RG-MoveCollection-demoRMS" -MoveCollectionName "PS-centralus-westcentralus-demoRMS"  -MoveResource $('psdemovm111', 'PSDemoRM-vnet','PSDemoVM-nsg', ‘PSDemoVM’) -MoveResourceInputType "MoveResourceId"
    ```
    **Salida**

    ![Texto de salida después de confirmar el movimiento](./media/tutorial-move-region-powershell/commit-move.png)

2. Compruebe que se hayan trasladado todos los recursos a la región de destino:

    ```azurepowershell-interactive
    Get-AzResourceMoverMoveResource  -ResourceGroupName “RG-MoveCollection-demoRMS ” -MoveCollectionName “PS-centralus-westcentralus-demoRMS”   
    ```
    Todos los recursos tienen ahora el estado *Delete Source Pending* (Eliminar origen pendiente) en la región de destino.

## <a name="delete-source-resources"></a>Eliminación de los recursos de origen

Después de confirmar el movimiento y comprobar que los recursos funcionan según lo previsto en la región de destino, puede eliminar cada uno de los recursos de origen mediante [Azure Portal](../azure-resource-manager/management/manage-resources-portal.md#delete-resources), [PowerShell](../azure-resource-manager/management/manage-resources-powershell.md#delete-resources) o la [CLI de Azure](../azure-resource-manager/management/manage-resources-cli.md#delete-resources).

## <a name="next-steps"></a>Pasos siguientes

En este tutorial ha:

> [!div class="checklist"]
> * Trasladado máquinas virtuales de Azure a otra región de Azure mediante PowerShell.
> * Trasladar los recursos asociados con máquinas virtuales a otra región.

Ahora, pruebe a trasladar máquinas virtuales de Azure mediante el portal.

> [!div class="nextstepaction"]
> [Tutorial: Traslado de máquinas virtuales de Azure entre regiones](./tutorial-move-region-virtual-machines.md)


