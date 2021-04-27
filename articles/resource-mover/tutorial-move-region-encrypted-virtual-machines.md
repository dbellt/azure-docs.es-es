---
title: Traslado de máquinas virtuales de Azure cifradas entre regiones mediante Azure Resource Mover
description: Aprenda a trasladar máquinas virtuales de Azure cifradas a otra región mediante Azure Resource Mover.
manager: evansma
author: rayne-wiselman
ms.service: resource-move
ms.topic: tutorial
ms.date: 02/10/2021
ms.author: raynew
ms.custom: mvc
ms.openlocfilehash: 457c4c4752b4d78434b1fb90710472b1998f1c4e
ms.sourcegitcommit: 950e98d5b3e9984b884673e59e0d2c9aaeabb5bb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/18/2021
ms.locfileid: "107600699"
---
# <a name="tutorial-move-encrypted-azure-vms-across-regions"></a>Tutorial: Traslado de máquinas virtuales de Azure cifradas entre regiones

En este artículo se describe cómo trasladar máquinas virtuales de Azure cifradas a otra región de Azure mediante [Azure Resource Mover](overview.md). 

Las máquinas virtuales cifradas se pueden describir como:

- Máquinas virtuales que tienen discos con Azure Disk Encryption habilitado. Para más información, consulte [Creación y cifrado de una máquina virtual Windows mediante Azure Portal](../virtual-machines/windows/disk-encryption-portal-quickstart.md).
- Máquinas virtuales que usan claves administradas por el cliente (CMK) para el cifrado en reposo o cifrado del lado servidor. Para más información, consulte [Uso de Azure Portal para habilitar el cifrado del lado servidor con claves administradas por el cliente para los discos administrados](../virtual-machines/disks-enable-customer-managed-keys-portal.md).


En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Comprobar los requisitos previos. 
> * Para las máquinas virtuales con Azure Disk Encryption habilitado, copiar las claves y los secretos del almacén de claves de la región de origen al almacén de claves de la región de destino.
> * Preparar las máquinas virtuales para su traslado y seleccionar los recursos de la región de origen desde la que desea realizar el traslado.
> * Resolver las dependencias de recursos.
> * Para las máquinas virtuales con Azure Disk Encryption habilitado, asignar manualmente el almacén de claves de destino. Para las máquinas virtuales que usan el cifrado del lado servidor con claves administradas por el cliente, asignar manualmente un conjunto de cifrado de disco en la región de destino.
> * Trasladar el almacén de claves o el conjunto de cifrado de disco.
> * Preparar y trasladar el grupo de recursos de origen. 
> * Preparar y trasladar los demás recursos.
> * Elegir si desea descartar o confirmar el traslado. 
> * Opcionalmente, quitar los recursos de la región después del traslado.

> [!NOTE]
> En este tutorial se muestra la manera más rápida de probar un escenario. Solo usa las opciones predeterminadas. 

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/pricing/free-trial/) antes de empezar. Luego, inicie sesión en [Azure Portal](https://portal.azure.com).

## <a name="prerequisites"></a>Requisitos previos

Requisito |Detalles
--- | ---
**Permisos de suscripción** | Compruebe que tiene acceso de *Propietario* a la suscripción que contiene los recursos que desea trasladar.<br/><br/> *¿Por qué necesito acceso de Propietario?* La primera vez que agregue un recurso de un par de origen y destino específicos a una suscripción de Azure, Azure Resource Mover creará una [identidad administrada asignada por el sistema](../active-directory/managed-identities-azure-resources/overview.md#managed-identity-types), anteriormente llamada identidad de servicio administrado (MSI). Esta identidad es de confianza para la suscripción. Para crear la identidad y asignarle los roles requeridos (*Colaborador* y *Administrador de acceso de usuario* en la suscripción de origen), la cuenta que utilice para agregar recursos necesita permisos de *Propietario* de la suscripción. Para más información, consulte [Roles de administrador de la suscripción clásica, roles de Azure y roles de Azure AD](../role-based-access-control/rbac-and-directory-admin-roles.md#azure-roles).
**Soporte técnico de máquina virtual** | Para asegurarse de que las máquinas virtuales que desea trasladar son compatibles, haga lo siguiente:<li>[Compruebe](support-matrix-move-region-azure-vm.md#windows-vm-support) las máquinas virtuales Windows admitidas.<li>[Compruebe](support-matrix-move-region-azure-vm.md#linux-vm-support) las máquinas virtuales Linux y las versiones de kernel admitidas.<li>Compruebe la configuración admitida de [proceso](support-matrix-move-region-azure-vm.md#supported-vm-compute-settings), [almacenamiento](support-matrix-move-region-azure-vm.md#supported-vm-storage-settings) y [redes](support-matrix-move-region-azure-vm.md#supported-vm-networking-settings).
**Requisitos del almacén de claves (Azure Disk Encryption)** | Si ha habilitado Azure Disk Encryption para las máquinas virtuales, necesita un almacén de claves en las regiones de origen y de destino. Para más información, consulte [Creación de un almacén de claves](../key-vault/general/quick-create-portal.md).<br/><br/> Para los almacenes de claves de las regiones de origen y de destino, necesita estos permisos:<li>Permisos de claves: operaciones de administración de claves (Get, List) y operaciones criptográficas (Decrypt y Encrypt).<li>Permisos de secretos: operaciones de administración de secretos (Get, List y Set)<li>Certificado (List y Get).
**Conjunto de cifrado de disco (cifrado del lado servidor con CMK)** | Si utiliza máquinas virtuales con cifrado del lado servidor que usa una clave administrada por el cliente, necesita un conjunto de cifrado de disco en las regiones de origen y de destino. Para más información, consulte [Creación de un conjunto de cifrado de disco](../virtual-machines/disks-enable-customer-managed-keys-portal.md#set-up-your-disk-encryption-set).<br/><br/> No se admite el traslado entre regiones si usa claves del módulo de seguridad de hardware (claves HSM) para las claves administradas por el cliente.
**Cuota de la región de destino** | La suscripción necesita tener cuota suficiente para crear los recursos que va a trasladar a la región de destino. Si no dispone de cuota suficiente, [solicite límites adicionales](../azure-resource-manager/management/azure-subscription-service-limits.md).
**Cargos de la región de destino** | Compruebe los precios y los cargos asociados con la región de destino a la que va a trasladar las máquinas virtuales. Use la [calculadora de precios](https://azure.microsoft.com/pricing/calculator/).


## <a name="verify-permissions-in-the-key-vault"></a>Comprobación de los permisos del almacén de claves

Si va a trasladar máquinas virtuales que tienen Azure Disk Encryption habilitado, debe ejecutar un script como se mencionó en la sección [Copia de las claves en el almacén de claves de destino](#copy-the-keys-to-the-destination-key-vault). Los usuarios que ejecutan el script deben tener los permisos adecuados para hacerlo. Para saber qué permisos son necesarios, consulte la tabla siguiente. Para encontrar las opciones para cambiar los permisos, vaya al almacén de claves de Azure Portal. En **Configuración**, seleccione **Directivas de acceso**.

:::image type="content" source="./media/tutorial-move-region-encrypted-virtual-machines/key-vault-access-policies.png" alt-text="Captura de pantalla del vínculo &quot;Directivas de acceso&quot; en el panel Configuración del almacén de claves." lightbox="./media/tutorial-move-region-encrypted-virtual-machines/key-vault-access-policies.png":::

Si los permisos de usuario no están establecidos, seleccione **Agregar directiva de acceso** y, a continuación, especifique los permisos. Si la cuenta de usuario ya tiene una directiva, en **Usuario**, establezca los permisos según las instrucciones de la siguiente tabla.

Las máquinas virtuales de Azure que usan Azure Disk Encryption pueden tener las siguientes variaciones y deberá establecer los permisos según sus componentes correspondientes. Puede que las máquinas virtuales tengan:
- Una opción predeterminada cuando el disco está cifrado solo con secretos.
- Seguridad agregada que usa una [clave de cifrado de claves (KEK)](../virtual-machines/windows/disk-encryption-key-vault.md#set-up-a-key-encryption-key-kek).

### <a name="source-region-key-vault"></a>Almacén de claves de la región de origen

Para los usuarios que ejecutan el script, establezca permisos para los siguientes componentes: 

Componente | Permisos necesarios
--- | ---
Secretos |  *Get* <br></br> Seleccione **Permisos de los secretos** > **Operaciones de administración de secretos** y, después, seleccione **Get**. 
Claves <br></br> Si usa una clave de cifrado de claves, necesitará estos permisos además de los permisos para los secretos. | *Get* y *Decrypt* <br></br> Seleccione **Permisos de las claves** > **Operaciones de administración de claves** y, a continuación, seleccione **Get**. En **Operaciones criptográficas**, seleccione **Decrypt**.

### <a name="destination-region-key-vault"></a>Almacén de claves de la región de destino

En **Directivas de acceso**, asegúrese de que **Azure Disk Encryption para el cifrado de volúmenes** está habilitado. 

Para los usuarios que ejecutan el script, establezca permisos para los siguientes componentes: 

Componente | Permisos necesarios
--- | ---
Secretos |  *Establecimiento* <br></br> Seleccione **Permisos de los secretos** > **Operaciones de administración de secretos** y, después, seleccione **Set**. 
Claves <br></br> Si usa una clave de cifrado de claves, necesitará estos permisos además de los permisos para los secretos. | *Get*, *Create* y *Encrypt* <br></br> Seleccione **Permisos de las claves** > **Operaciones de administración de claves** y, a continuación, seleccione **Get** y **Create**. En **Operaciones criptográficas**, seleccione **Encrypt**.

<br>

Además de los permisos anteriores, en el almacén de claves de destino debe agregar permisos para la [identidad del sistema administrada](./common-questions.md#how-is-managed-identity-used-in-resource-mover) que usa Resource Mover para acceder a los recursos de Azure en su nombre. 

1. En **Configuración**, seleccione **Add Access policies** (Agregar directivas de acceso). 
1. En **Seleccionar la entidad de seguridad**, busque el MSI. El nombre de MSI es ```movecollection-<sourceregion>-<target-region>-<metadata-region>```. 
1. Para la identidad del sistema administrada, agregue los siguientes permisos:

    Componente | Permisos necesarios
    --- | ---
    Secretos|  *Get* y *List* <br></br> Seleccione **Permisos de los secretos** > **Operaciones de administración de secretos** y, después, seleccione **Get** y **List**. 
    Claves <br></br> Si usa una clave de cifrado de claves, necesitará estos permisos además de los permisos para los secretos. | *Get* y *List* <br></br> Seleccione **Permisos de las claves** > **Operaciones de administración de claves** y, a continuación, seleccione **Get** y **List**.

<br>

### <a name="copy-the-keys-to-the-destination-key-vault"></a>Copia de las claves en el almacén de claves de destino

Copie los secretos y las claves de cifrado del almacén de claves de origen en el almacén de claves de destino mediante el uso de un [script](https://raw.githubusercontent.com/AsrOneSdk/published-scripts/master/CopyKeys/CopyKeys.ps1) que se proporciona.

- Ejecute el script en PowerShell. Se recomienda usar la versión más reciente de PowerShell.
- Específicamente, el script requiere estos módulos:
    - Az.Compute
    - Az.KeyVault (versión 3.0.0)
    - Az.Accounts (versión 2.2.3)

Para ejecutar el script, haga lo siguiente:

1. Abra el [script](https://raw.githubusercontent.com/AsrOneSdk/published-scripts/master/CopyKeys/CopyKeys.ps1) en GitHub.
1. Copie el contenido del script en un archivo local y asígnele el nombre *Copy-keys.ps1*.
1. Ejecute el script.
1. Inicie sesión en Azure Portal.
1. En las listas desplegables de la ventana **Entradas de usuario**, seleccione la suscripción de origen, el grupo de recursos y la máquina virtual de origen y, a continuación, seleccione la ubicación de destino y los almacenes de destino para el cifrado de discos y claves.

    :::image type="content" source="./media/tutorial-move-region-encrypted-virtual-machines/script-input.png" alt-text="Captura de pantalla de la ventana &quot;Entradas de usuario&quot; para especificar los valores del script." :::

1. Seleccione el botón **Seleccionar**. 
   
   Cuando el script ha terminado de ejecutarse, un mensaje le notifica que CopyKeys se ha completado correctamente.

## <a name="prepare-vms"></a>Preparación de máquinas virtuales

1. Después de comprobar que las máquinas virtuales cumplen los [requisitos previos](#prerequisites), asegúrese de que estén activadas. Todos los discos de las máquinas virtuales que desea que estén disponibles en la región de destino deben estar conectados e inicializados en la máquina virtual.
1. Para asegurarse de que las máquinas virtuales dispongan de los certificados raíz de confianza más recientes y de una lista de revocación de certificados (CRL) actualizada, haga lo siguiente:
    - En las máquinas virtuales Windows, instale las actualizaciones de Windows más recientes.
    - En las máquinas virtuales Linux, siga las indicaciones del distribuidor para que las máquinas dispongan de los certificados y la lista de revocación de certificados más recientes. 
1. Para permitir la conectividad saliente desde las máquinas virtuales, realice una de las siguientes acciones:
    - Si utiliza un servidor proxy de firewall basado en direcciones URL para controlar la conectividad de salida, debe [permitir el acceso a estas URL](support-matrix-move-region-azure-vm.md#url-access).
    - Si utiliza reglas de grupo de seguridad de red para determinar la conectividad saliente, cree estas [reglas de etiquetas de servicio](support-matrix-move-region-azure-vm.md#nsg-rules).

## <a name="select-the-resources-to-move"></a>Selección de los recursos que se trasladarán

- Puede seleccionar cualquier tipo de recurso admitido de cualquiera de los grupos de recursos de la región de origen seleccionada.  
- Puede trasladar los recursos a una región de destino que está en la misma suscripción que la región de origen. Si desea cambiar la suscripción, puede hacerlo después de trasladar los recursos.

Para seleccionar los recursos, haga lo siguiente:

1. En Azure Portal, busque **resource mover**. A continuación, en **Servicios**, seleccione **Azure Resource Mover**.

    :::image type="content" source="./media/tutorial-move-region-encrypted-virtual-machines/search.png" alt-text="Captura de pantalla de los resultados de la búsqueda de Azure Resource Mover en Azure Portal" :::.

1. En el panel de **información general** de Azure Resource Mover, seleccione **Mover entre regiones**.

    :::image type="content" source="./media/tutorial-move-region-encrypted-virtual-machines/move-across-regions.png" alt-text="Captura de pantalla del botón &quot;Mover entre regiones&quot; para agregar recursos para moverse a otra región" lightbox="./media/tutorial-move-region-encrypted-virtual-machines/move-across-regions.png":::.

1. En el panel **Mover recursos**, seleccione la pestaña **Origen y destino**. A continuación, en las listas desplegables, seleccione la suscripción y la región de origen.

    :::image type="content" source="./media/tutorial-move-region-encrypted-virtual-machines/source-target.png" alt-text="Página para seleccionar las regiones de origen y de destino." :::

1. En **Destino**, seleccione la región a la que desea trasladar las máquinas virtuales y, a continuación, seleccione **Siguiente**.

1. Seleccione la pestaña **Recursos que se van a mover** y, a continuación, seleccione **Seleccionar recursos**.

    :::image type="content" source="./media/tutorial-move-region-encrypted-virtual-machines/select-resources.png" alt-text="Captura de pantalla del panel &quot;Mover recursos&quot; y el botón &quot;Seleccionar recursos&quot;" :::

1. En el panel **Seleccionar recursos**, seleccione las máquinas virtuales que desea trasladar. Como se mencionó en la sección [Selección de los recursos que se moverán](#select-the-resources-to-move), solo puede agregar los recursos que se admiten para un traslado.

    :::image type="content" source="./media/tutorial-move-region-encrypted-virtual-machines/select-vm.png" alt-text="Captura de pantalla del panel &quot;Seleccionar recursos&quot; para seleccionar las máquinas virtuales que se van a trasladar." :::

    > [!NOTE]
    >  En este tutorial, vas a seleccionar una máquina virtual que usa el cifrado del lado servidor (rayne-vm) con una clave administrada por el cliente y una máquina virtual con cifrado de disco habilitado (rayne-vm-ade).

1. Seleccione **Listo**.
1. Seleccione la pestaña **Recursos que se van a mover** y, a continuación, seleccione **Siguiente**.
1. Seleccione la pestaña **Revisar** y compruebe la configuración de origen y destino. 

    :::image type="content" source="./media/tutorial-move-region-encrypted-virtual-machines/review.png" alt-text="Captura de pantalla del panel para revisar la configuración de origen y destino." :::

1. Seleccione **Continuar** para empezar a agregar recursos.
1. Seleccione el icono de notificaciones para realizar un seguimiento del progreso. Una vez que el proceso finalice correctamente, en el panel **Notificaciones**, seleccione **Added resources for move** (Se han agregado recursos para mover).

    :::image type="content" source="./media/tutorial-move-region-encrypted-virtual-machines/added-resources-notification.png" alt-text="Captura de pantalla del panel &quot;Notificaciones&quot; para confirmar que los recursos se agregaron correctamente." lightbox="./media/tutorial-move-region-encrypted-virtual-machines/added-resources-notification.png":::
    
1. Después de seleccionar la notificación, revise los recursos de la página **Entre regiones**.

    :::image type="content" source="./media/tutorial-move-region-encrypted-virtual-machines/resources-prepare-pending.png" alt-text="Captura de pantalla de los recursos agregados con el estado &quot;Preparación pendiente&quot;." :::

> [!NOTE]
> - Los recursos que agregue se colocan en el estado *Prepare pending* (Preparación pendiente).
> - El grupo de recursos de las máquinas virtuales se agrega automáticamente.
> - Si modifica las entradas de la columna **Destination configuration** (Configuración de destino) para usar un recurso que ya existe en la región de destino, el estado del recurso se establece en *Commit pending* (Pendiente de confirmación), ya que no es necesario iniciar un traslado.
> - Si quiere eliminar un recurso que se ha agregado, el método que utilizará depende del punto en el que se encuentre en el proceso de traslado. Para más información, consulte [Administración de grupos de recursos y colecciones de transferencia de recursos](remove-move-resources.md).


## <a name="resolve-dependencies"></a>Resolución de dependencias

1. Si algún recurso muestra el mensaje *Validate dependencies* (Validar dependencias) en la columna **Issues** (Problemas), seleccione el botón **Validate dependencies** (Validar dependencias).

    :::image type="content" source="./media/tutorial-move-region-encrypted-virtual-machines/check-dependencies.png" alt-text="Captura de pantalla que muestra el botón &quot;Validar dependencias&quot;" lightbox="./media/tutorial-move-region-encrypted-virtual-machines/check-dependencies.png":::.

    Se iniciará el proceso de validación.
1. Si se encuentran dependencias, seleccione **Agregar dependencias**.  

    :::image type="content" source="./media/tutorial-move-region-encrypted-virtual-machines/add-dependencies.png" alt-text="Captura de pantalla del botón &quot;Agregar dependencias&quot;" lightbox="./media/tutorial-move-region-encrypted-virtual-machines/add-dependencies.png":::.


1. En el panel **Agregar dependencias**, deje la opción predeterminada **Mostrar todas las dependencias**.

    - La opción **Show all dependencies** (Mostrar todas las dependencias) recorre en iteración todas las dependencias directas e indirectas de un recurso. Por ejemplo, para una máquina virtual, muestra la NIC, la red virtual, los grupos de seguridad de red, etc.
    - La opción **Mostrar solo las dependencias de primer nivel** muestra solo las dependencias directas. Por ejemplo, para una máquina virtual, muestra la NIC, pero no la red virtual.
 
1. Seleccione los recursos dependientes que desea agregar y, a continuación, seleccione **Agregar dependencias**.

    :::image type="content" source="./media/tutorial-move-region-encrypted-virtual-machines/select-dependencies.png" alt-text="Captura de pantalla de la lista de dependencias y el botón &quot;Agregar dependencias&quot;." lightbox="./media/tutorial-move-region-encrypted-virtual-machines/select-dependencies.png":::

1. Vuelva a validar las dependencias. 

    :::image type="content" source="./media/tutorial-move-region-encrypted-virtual-machines/validate-again.png" alt-text="Captura de pantalla del panel para volver a validar las dependencias." lightbox="./media/tutorial-move-region-encrypted-virtual-machines/validate-again.png":::

## <a name="assign-destination-resources"></a>Asignación de los recursos de destino

Debe asignar manualmente los recursos de destino que se asocian al cifrado.

- Si va a trasladar una máquina virtual que tiene Azure Disk Encryption habilitado, el almacén de claves de la región de destino aparecerá como una dependencia.
- Si va a mover una máquina virtual que tiene cifrado del lado servidor y que usa claves administradas por el cliente, el conjunto de cifrado de disco de la región de destino aparece como una dependencia. 
- Dado que en este tutorial se muestra cómo trasladar una máquina virtual que tiene Azure Disk Encryption habilitado y que usa una clave administrada por el cliente, el almacén de claves de destino y el conjunto de cifrado de disco aparecen como dependencias.

Para asignar manualmente los recursos de destino, haga lo siguiente:

1. En la entrada del conjunto de cifrado de disco, seleccione **Resource not assigned** (Recurso no asignado) en la columna **Destination configuration** (Configuración de destino).
1. En **Opciones de configuración**, seleccione el conjunto de cifrado de disco de destino y, después, seleccione **Guardar cambios**.
1. Puede guardar y validar las dependencias del recurso que está modificando, o guardar únicamente los cambios y validar todo lo que modifique de una sola vez.

    :::image type="content" source="./media/tutorial-move-region-encrypted-virtual-machines/select-destination-set.png" alt-text="Captura de pantalla del panel &quot;Configuración de destino&quot; para guardar los cambios en la región de destino." lightbox="./media/tutorial-move-region-encrypted-virtual-machines/select-destination-set.png":::

    Después de agregar el recurso de destino, el estado del conjunto de cifrado de disco se convierte en *Commit move pending* (Confirmar movimiento pendiente).

1. En la entrada del almacén de claves, seleccione **Resource not assigned** (Recurso no asignado) en la columna **Destination configuration** (Configuración de destino). En **Opciones de configuración**, seleccione el almacén de claves de destino y guarde los cambios. 

En esta fase, el estado del conjunto de cifrado de disco y del almacén de claves cambia a *Commit move pending* (Confirmar movimiento pendiente).

:::image type="content" source="./media/tutorial-move-region-encrypted-virtual-machines/prepare-other-resources.png" alt-text="Captura de pantalla del panel para preparar otros recursos." lightbox="./media/tutorial-move-region-encrypted-virtual-machines/prepare-other-resources.png":::

Para confirmar y finalizar el proceso de traslado de los recursos de cifrado, haga lo siguiente:

1. En **Entre regiones**, seleccione el recurso (conjunto de cifrado de disco o almacén de claves) y, a continuación, seleccione **Commit move** (Confirmar traslado).
1. En **Mover recursos**, seleccione **Confirmar**.

> [!NOTE]
> Después de haber confirmado el traslado, el estado del recurso cambia a *Delete source pending* (Eliminar origen pendiente).


## <a name="move-the-source-resource-group"></a>Traslado del grupo de recursos de origen 

Antes de poder preparar y trasladar las máquinas virtuales, el grupo de recursos de la máquina virtual debe estar presente en la región de destino. 

### <a name="prepare-to-move-the-source-resource-group"></a>Preparación del grupo de recursos de origen para el traslado

Durante el proceso de preparación, Resource Mover genera plantillas de Azure Resource Manager a partir de la configuración del grupo de recursos. Los recursos que pertenecen al grupo no se ven afectados.

Para prepararse para trasladar el grupo de recursos de origen, haga lo siguiente:

1. En **Entre regiones**, seleccione el grupo de recursos de origen y, a continuación, seleccione **Preparar**.

    :::image type="content" source="./media/tutorial-move-region-encrypted-virtual-machines/prepare-resource-group.png" alt-text="Captura de pantalla del botón &quot;Preparar&quot; en el panel &quot;Preparar recursos&quot;." lightbox="./media/tutorial-move-region-encrypted-virtual-machines/prepare-resource-group.png":::

1. En **Preparar recursos**, seleccione **Preparar**.

> [!NOTE]
> Después de preparar el traslado, el estado del grupo de recursos cambia a *Initiate move pending* (Iniciar movimiento pendiente). 

 
### <a name="move-the-source-resource-group"></a>Traslado del grupo de recursos de origen

Para empezar a trasladar el grupo de recursos de origen, haga lo siguiente:

1. En el panel **Entre regiones**, seleccione el grupo de recursos y, a continuación, seleccione **Iniciar movimiento**.

    :::image type="content" source="./media/tutorial-move-region-encrypted-virtual-machines/initiate-move-resource-group.png" alt-text="Captura de pantalla del botón &quot;Iniciar movimiento&quot; en el panel &quot;Entre regiones&quot;." lightbox="./media/tutorial-move-region-encrypted-virtual-machines/initiate-move-resource-group.png":::

1. En el panel **Mover recursos**, seleccione **Iniciar movimiento**. El estado del grupo de recursos cambia a *Initiate move in progress* (Iniciar movimiento en curso).   
1. Una vez iniciado el traslado, se creará el grupo de recursos de destino, basado en la plantilla de Resource Manager generada. El estado del grupo de recursos de origen cambia a *Commit move pending* (Confirmar movimiento pendiente).

    :::image type="content" source="./media/tutorial-move-region-encrypted-virtual-machines/resource-group-commit-move-pending.png" alt-text="Captura de pantalla del panel &quot;Mover recursos&quot; que muestra el estado del grupo de recursos cambiado a &quot;Commit move pending&quot; (Confirmar movimiento pendiente)" lightbox="./media/tutorial-move-region-encrypted-virtual-machines/resource-group-commit-move-pending.png":::.

Para confirmar el traslado y finalizar el proceso, haga lo siguiente:

1. En el panel **Entre regiones**, seleccione el grupo de recursos y, a continuación, seleccione **Commit move** (Confirmar traslado).
1. En el panel **Mover recursos**, seleccione **Confirmar**.

> [!NOTE]
> Después de haber confirmado el traslado, el estado del grupo de recursos de origen cambia a *Delete source pending* (Eliminar origen pendiente).

:::image type="content" source="./media/tutorial-move-region-encrypted-virtual-machines/resource-group-delete-move-pending.png" alt-text="Captura de pantalla del grupo de recursos de origen que muestra el estado cambiado a &quot;Delete source pending&quot;" lightbox="./media/tutorial-move-region-encrypted-virtual-machines/resource-group-delete-move-pending.png"::: (Eliminar origen pendiente).

## <a name="prepare-resources-to-move"></a>Preparación de los recursos que se van a trasladar

Ahora que los recursos de cifrado y el grupo de recursos de origen se han trasladado, puede preparar el traslado de otros recursos cuyo estado actual sea *Prepare pending* (Preparación pendiente).


1. En el panel **Entre regiones** valide el traslado de nuevo y resuelva los problemas.
1. Si desea editar la configuración de destino antes de trasladar los recursos, seleccione el vínculo en la columna **Destination configuration** (Configuración de destino) del recurso y edite la configuración. Si edita la configuración de la máquina virtual de destino, el tamaño de esta máquina virtual no puede ser inferior al tamaño de la máquina virtual de origen.
1. Para los recursos con el estado *Prepare pending* (Preparación pendiente) que desea trasladar, seleccione **Preparar**.
1. En el panel **Preparar recursos**, seleccione **Preparar**.

    - Durante la preparación, el agente de movilidad de Azure Site Recovery se instala en las máquinas virtuales para replicarlas.
    - Los datos de las máquinas virtuales se replican periódicamente en la región de destino. Esto no afecta a la máquina virtual de origen.
    - Resource Mover genera plantillas de Resource Manager para los demás recursos de origen.

> [!NOTE]
> Después de preparar los recursos, su estado cambia a *Initiate move pending* (Iniciar movimiento pendiente).

:::image type="content" source="./media/tutorial-move-region-encrypted-virtual-machines/resources-initiate-move-pending.png" alt-text="Captura de pantalla del panel &quot;Preparar recursos&quot;, en la que se muestran los recursos con el estado &quot;Initiate move pending&quot; (Iniciar movimiento pendiente)." lightbox="./media/tutorial-move-region-encrypted-virtual-machines/resources-initiate-move-pending.png":::



## <a name="initiate-the-move"></a>Inicio de la migración

Ahora que ha preparado los recursos, puede iniciar el traslado. 

1. En el panel **Entre regiones**, seleccione los recursos cuyo estado sea *Initiate move pending* (Iniciar movimiento pendiente) y, a continuación, seleccione **Initiate move** (Iniciar movimiento).
1. En el panel **Mover recursos**, seleccione **Iniciar movimiento**.
1. Realice un seguimiento del progreso del traslado en la barra de notificaciones.

    - En el caso de las máquinas virtuales, se crearán réplicas en la región de destino. La máquina virtual de origen se apagará y se producirá cierto tiempo de inactividad (normalmente es cuestión de minutos).
    - Resource Mover recrea otros recursos mediante las plantillas de Resource Manager preparadas. Normalmente, no se produce tiempo de inactividad.
    - Después de trasladar los recursos, su estado cambia a *Commit move pending* (Confirmar movimiento pendiente).

:::image type="content" source="./media/tutorial-move-region-encrypted-virtual-machines/resources-commit-move-pending.png" alt-text="Captura de pantalla de una lista de recursos con el estado &quot;Commit move pending&quot; (Confirmar movimiento pendiente)" lightbox="./media/tutorial-move-region-encrypted-virtual-machines/resources-commit-move-pending.png" :::.


## <a name="discard-or-commit"></a>¿Descartar o confirmar?

Tras la operación inicial de traslado, puede decidir si desea confirmarla o descartarla. 

- **Descartar**: puede descartar un traslado si está haciendo pruebas y no quiere trasladar verdaderamente el recurso de origen. Si el traslado se descarta, el recurso volverá a tener el estado *Initiate move pending* (Iniciar movimiento pendiente).
- **Confirmación**: la confirmación completa la operación de traslado a la región de destino. Después de haber confirmado un recurso de origen, su estado cambia a *Delete source pending* (Eliminar origen pendiente) y puede decidir si desea eliminarlo.


## <a name="discard-the-move"></a>Descartar la operación de traslado 

Para descartar el traslado, haga lo siguiente:

1. En el panel **Entre regiones**, seleccione los recursos cuyo estado sea *Commit move pending* (Confirmar movimiento pendiente) y, a continuación, seleccione **Discard move** (Descartar movimiento).
1. En el panel **Discard move** (Descartar movimiento) seleccione **Descartar**.
1. Realice un seguimiento del progreso del traslado en la barra de notificaciones.


> [!NOTE]
> Después de descartar los recursos, los estados de las máquinas virtuales cambian a *Initiate move pending* (Iniciar movimiento pendiente).

## <a name="commit-the-move"></a>Confirmación de la operación de traslado

Para completar el proceso de traslado, debe confirmar el traslado haciendo lo siguiente: 

1. En el panel **Entre regiones**, seleccione los recursos cuyo estado sea *Commit move pending* (Confirmar movimiento pendiente) y, a continuación, seleccione **Commit move** (Confirmar movimiento).
1. En el panel **Commit resources** (Confirmar recursos), seleccione **Confirmar**.

    :::image type="content" source="./media/tutorial-move-region-encrypted-virtual-machines/resources-commit-move.png" alt-text="Captura de pantalla de una lista de recursos para confirmar recursos y finalizar el traslado." lightbox="./media/tutorial-move-region-encrypted-virtual-machines/resources-commit-move.png" :::

1. Haga un seguimiento del progreso de la operación de confirmación en la barra de notificaciones.

> [!NOTE]
> - Después de haber confirmado el traslado, las máquinas virtuales dejan de replicarse. La confirmación no afecta a la máquina virtual de origen.
> - El proceso de confirmación no afecta a los recursos de red de origen.
> - Después de haber confirmado el traslado, los estados de los recursos cambian a *Delete source pending* (Eliminar origen pendiente).



## <a name="configure-settings-after-the-move"></a>Configuración de parámetros tras el traslado

- El servicio Mobility no se desinstala automáticamente de las máquinas virtuales. Puede desinstalarlo manualmente, o bien mantenerlo si tiene previsto trasladar el servidor de nuevo.
- Modifique las reglas de control de acceso basado en rol de Azure después del traslado.

## <a name="delete-source-resources-after-commit"></a>Eliminación de los recursos de origen después de la confirmación

Después del traslado, puede eliminar los recursos de la región de origen si lo desea. 

1. En el panel **Entre regiones** seleccione cada recurso de origen que quiera eliminar y, a continuación, seleccione **Delete source** (Eliminar origen).
1. En **Delete source** (Eliminar origen), revise lo que intenta eliminar y, en **Confirm delete** (Confirmar eliminación), escriba **yes** (Sí). La acción es irreversible, por lo que debe comprobarlo con cuidado.
1. Después de escribir **yes** (Sí), seleccione **Delete source** (Eliminar origen).

> [!NOTE]
>  En el portal de Resource Mover no se pueden eliminar grupos de recursos, almacenes de claves ni instancias de SQL Server. Debe eliminarlos de forma individual en la página de propiedades de cada recurso.


## <a name="delete-resources-that-you-created-for-the-move"></a>Eliminación de los recursos que ha creado para el traslado

Después del traslado, puede eliminar manualmente la colección de transferencia de recursos y los recursos de Site Recovery que se crearon durante este proceso.

- La colección de traslado está oculta de forma predeterminada. Para verla, debe activar los recursos ocultos.
- El almacenamiento en caché tiene un bloqueo que debe eliminarse antes de que se pueda eliminar la colección.

Para eliminar los recursos, haga lo siguiente: 
1. Identifique los recursos pertenecientes al grupo ```RegionMoveRG-<sourceregion>-<target-region>```.
1. Compruebe que todas las máquinas virtuales y los demás recursos de la región de origen se hayan trasladado o eliminado. De este modo, se garantiza que no haya recursos pendientes que los utilicen.
1. Eliminación de los recursos:

    - Nombre de la colección de transferencia de recursos: ```movecollection-<sourceregion>-<target-region>```
    - Nombre de la cuenta de almacenamiento en caché: ```resmovecache<guid>```
    - Nombre del almacén: ```ResourceMove-<sourceregion>-<target-region>-GUID```
## <a name="next-steps"></a>Pasos siguientes

En este tutorial, hizo lo siguiente:

> [!div class="checklist"]
> * Ha trasladado máquinas virtuales de Azure cifradas y sus recursos dependientes a otra región de Azure.


En el próximo paso, intentaremos trasladar grupos elásticos y bases de datos de Azure SQL a otra región.

> [!div class="nextstepaction"]
> [Traslado de recursos de Azure SQL](./tutorial-move-region-sql.md)
