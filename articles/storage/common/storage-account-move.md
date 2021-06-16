---
title: Traslado de una cuenta de Azure Storage a otra región | Microsoft Docs
description: Muestra cómo mover una cuenta de Azure Storage a otra región.
services: storage
author: normesta
ms.service: storage
ms.subservice: common
ms.topic: how-to
ms.date: 05/11/2020
ms.author: normesta
ms.reviewer: dineshm
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: d403bb00099de4a1fb5f4b9799f45386beb6a9f0
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110679259"
---
# <a name="move-an-azure-storage-account-to-another-region"></a>Traslado de una cuenta de Azure Storage a otra región

Para mover una cuenta de almacenamiento, cree una copia de esta en otra región. Luego, mueva los datos a esa cuenta mediante AzCopy u otra herramienta de su elección.

En este artículo, aprenderá a:

> [!div class="checklist"]
> 
> * Exportar una plantilla.
> * Modificar la plantilla agregando el nombre de la cuenta de almacenamiento y la región de destino.
> * Implementar la plantilla para crear la cuenta de almacenamiento nueva.
> * Configurar la cuenta de almacenamiento nueva.
> * Mover datos a la cuenta de almacenamiento nueva.
> * Eliminar los recursos en la región de origen.

## <a name="prerequisites"></a>Prerrequisitos

- Asegúrese de que los servicios y las características que usa su cuenta se admitan en la región de destino.

- En el caso de las características en versión preliminar, asegúrese de que la suscripción está en la lista de permitidos para la región de destino.

<a id="prepare"></a>

## <a name="prepare"></a>Preparación

Para empezar, exporte y luego modifique una plantilla de Resource Manager. 

### <a name="export-a-template"></a>Exportación de una plantilla

Esta plantilla contiene la configuración que describe la cuenta de almacenamiento. 

# <a name="portal"></a>[Portal](#tab/azure-portal)

Para exportar una plantilla mediante Azure Portal:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

2. Seleccione **Todos los recursos** y seleccione su cuenta de almacenamiento.

3. Seleccione > **Automation** > **Exportar plantilla**.

4. Elija **Descargar** en la hoja **Exportar plantilla**.

5. Busque el archivo ZIP que descargó desde el portal y descomprímalo en la carpeta que prefiera.

   Este archivo ZIP contiene los archivos .json que componen la plantilla y los scripts para implementar la plantilla.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Para exportar una plantilla mediante PowerShell:

1. Inicie sesión en la suscripción a Azure con el comando [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount) y siga las instrucciones de la pantalla:

   ```azurepowershell-interactive
   Connect-AzAccount
   ```
2. Si su identidad se asocia a más de una suscripción, establezca su suscripción activa en la suscripción de la cuenta de almacenamiento que quiere mover.

   ```azurepowershell-interactive
   $context = Get-AzSubscription -SubscriptionId <subscription-id>
   Set-AzContext $context
   ```

3. Exporte la plantilla de la cuenta de almacenamiento de origen. Estos comandos guardan una plantilla JSON en el directorio actual.

   ```azurepowershell-interactive
   $resource = Get-AzResource `
     -ResourceGroupName <resource-group-name> `
     -ResourceName <storage-account-name> `
     -ResourceType Microsoft.Storage/storageAccounts
   Export-AzResourceGroup `
     -ResourceGroupName <resource-group-name> `
     -Resource $resource.ResourceId
   ```

---

### <a name="modify-the-template"></a>Modificación de la plantilla 

Para modificar la plantilla, cambie el nombre y la región de la cuenta de almacenamiento.

# <a name="portal"></a>[Portal](#tab/azure-portal)

Para implementar la plantilla con Azure Portal:

1. En Azure Portal, haga clic en **Crear un recurso**.

2. En **Buscar en Marketplace**, escriba **implementación de plantillas** y, después, presione **ENTRAR**.

3. Seleccione **Implementación de plantillas**.

    ![Biblioteca de plantillas de Azure Resource Manager](./media/storage-account-move/azure-resource-manager-template-library.png)

4. Seleccione **Crear**.

5. Seleccione **Cree su propia plantilla en el editor**.

6. Seleccione **Cargar archivo** y, después, siga las instrucciones para cargar el archivo **template.json** que descargó en la última sección.

7. En el archivo **template.json**, asigne un nombre a la cuenta de almacenamiento de destino mediante el establecimiento del valor predeterminado del nombre de la cuenta de almacenamiento. En este ejemplo se establece el valor predeterminado del nombre de la cuenta de almacenamiento en `mytargetaccount`.
    
    ```json
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "storageAccounts_mysourceaccount_name": {
            "defaultValue": "mytargetaccount",
            "type": "String"
        }
    },
 
8. Edit the **location** property in the **template.json** file to the target region. This example sets the target region to `centralus`.

    ```json
    "resources": [{
         "type": "Microsoft.Storage/storageAccounts",
         "apiVersion": "2019-04-01",
         "name": "[parameters('storageAccounts_mysourceaccount_name')]",
         "location": "centralus"
         }]          
    ```
    Para obtener los códigos de ubicación de la región, consulte [Ubicaciones de Azure](https://azure.microsoft.com/global-infrastructure/locations/).  El código de una región es el nombre de la región sin espacios, **Centro de EE. UU.**  = **centralus**.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Para implementar la plantilla con PowerShell:

1. En el archivo **template.json**, asigne un nombre a la cuenta de almacenamiento de destino mediante el establecimiento del valor predeterminado del nombre de la cuenta de almacenamiento. En este ejemplo se establece el valor predeterminado del nombre de la cuenta de almacenamiento en `mytargetaccount`.
    
    ```json
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "storageAccounts_mysourceaccount_name": {
            "defaultValue": "mytargetaccount",
            "type": "String"
        }
    },
    ``` 

2. Edite la propiedad **location** del archivo **template.json** en la región de destino. En este ejemplo, la región de destino se establece en `eastus`.

    ```json
    "resources": [{
         "type": "Microsoft.Storage/storageAccounts",
         "apiVersion": "2019-04-01",
         "name": "[parameters('storageAccounts_mysourceaccount_name')]",
         "location": "eastus"
         }]          
    ```

    Para obtener los códigos de las regiones, ejecute el comando [Get-AzLocation](/powershell/module/az.resources/get-azlocation).

    ```azurepowershell-interactive
    Get-AzLocation | format-table 
    ```
---

<a id="move"></a>

## <a name="move"></a>Move

Implemente la plantilla para crear una nueva cuenta de almacenamiento en la región de destino. 

# <a name="portal"></a>[Portal](#tab/azure-portal)

1. Guarde el archivo **template.json**.

2. Escriba o seleccione los valores de propiedad:

- **Suscripción**: Seleccione una suscripción de Azure.

- **Grupo de recursos**: Seleccione **Crear nuevo** y asígnele un nombre al grupo de recursos.

- **Ubicación**: Seleccione una ubicación de Azure.

3. Active la casilla **Acepto los términos y condiciones indicados anteriormente** y haga clic en el botón **Select Purchase** (Seleccionar compra).

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

1. Obtenga el identificador de suscripción donde quiere implementar la IP pública de destino con [Get-AzSubscription](/powershell/module/az.accounts/get-azsubscription):

   ```azurepowershell-interactive
   Get-AzSubscription
   ```

2. Use estos comandos para implementar la plantilla:

   ```azurepowershell-interactive
   $resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"
   $location = Read-Host -Prompt "Enter the location (i.e. centralus)"

   New-AzResourceGroup -Name $resourceGroupName -Location "$location"
   New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName -TemplateUri "<name of your local template file>"  
   ```
---

### <a name="configure-the-new-storage-account"></a>Configuración de la cuenta de almacenamiento nueva

Algunas características no se exportarán a una plantilla, por lo que tendrá que agregarlas a la cuenta de almacenamiento nueva. 

En la tabla siguiente se enumeran estas características junto con instrucciones para agregarlas a la cuenta de almacenamiento nueva.

| Característica    | Guía    |
|--------|-----------|
| **Directivas de administración del ciclo de vida** | [Administración del ciclo de vida de Azure Blob Storage](../blobs/storage-lifecycle-management-concepts.md) |
| **Sitios web estáticos** | [Hospedaje de sitios web estáticos en Azure Storage](../blobs/storage-blob-static-website-how-to.md) |
| **Suscripciones a eventos** | [Reacción a eventos de Blob Storage](../blobs/storage-blob-event-overview.md) |
| **Alertas** | [Crear, ver y administrar las alertas del registro de actividad mediante Azure Monitor](../../azure-monitor/alerts/alerts-activity-log.md) |
| **Content Delivery Network (CDN)** | [Uso de Azure CDN para obtener acceso a blobs con dominios personalizados mediante HTTPS](../blobs/storage-https-custom-domain-cdn.md) |

> [!NOTE] 
> Si configura una red CDN para la cuenta de almacenamiento de origen, solo tiene que cambiar el origen de la red CDN existente por el punto de conexión de servicio de blob principal (o el punto de conexión del sitio web estático principal) de la cuenta nueva. 

### <a name="move-data-to-the-new-storage-account"></a>Traslado de datos a la cuenta de almacenamiento nueva

AzCopy es la herramienta preferida para trasladar los datos. Está optimizado para el rendimiento.  Una manera más rápida de hacerlo es que los datos se copien directamente entre los servidores de almacenamiento, para que AzCopy no use el ancho de banda de red del equipo. Use AzCopy en la línea de comandos o como parte de un script personalizado. Consulte [Introducción a AzCopy](/azure/storage/common/storage-use-azcopy-v10?toc=%2fazure%2fstorage%2fblobs%2ftoc.json).

También puede usar Azure Data Factory para trasladar los datos. Esta proporciona una interfaz de usuario intuitiva. Para usar Azure Data Factory, consulte cualquiera de estos vínculos: 

  - [Copia de datos con Azure Blob Storage como origen o destino mediante Azure Data Factory](/azure/data-factory/connector-azure-blob-storage)
  - [Copia de datos con Azure Data Lake Storage Gen2 como origen o destino mediante Azure Data Factory](/azure/data-factory/connector-azure-data-lake-storage)
  - [Copia de datos con Azure File Storage como origen o destino mediante Azure Data Factory](/azure/data-factory/connector-azure-file-storage)
  - [Copia de datos con Azure Table Storage como origen o destino mediante Azure Data Factory](/azure/data-factory/connector-azure-table-storage)

---

## <a name="discard-or-clean-up"></a>Descarte o limpieza

Después de la implementación, si quiere empezar de nuevo, puede eliminar la cuenta de almacenamiento de destino y repetir los pasos descritos en las secciones [Preparación](#prepare) y [Traslado](#move) de este artículo.

Para confirmar los cambios y completar el traslado de una cuenta de almacenamiento, elimine la cuenta de almacenamiento de origen.

# <a name="portal"></a>[Portal](#tab/azure-portal)

Para quitar una cuenta de almacenamiento mediante Azure Portal:

1. En Azure Portal, expanda el menú de la izquierda para abrir el menú de servicios y elija **Cuentas de almacenamiento** para mostrar la lista de las cuentas de almacenamiento.

2. Busque la cuenta de almacenamiento de destino que se va a eliminar y haga clic con el botón derecho en el botón **Más** ( **…** ) situado en la parte derecha de la lista.

3. Seleccione **Eliminar** y confirme.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Para quitar el grupo de recursos y sus recursos asociados, incluida la nueva cuenta de almacenamiento, use el comando [Remove-AzStorageAccount](/powershell/module/az.storage/remove-azstorageaccount):

```powershell
Remove-AzStorageAccount -ResourceGroupName  $resourceGroup -AccountName $storageAccount
```
---

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, trasladó una cuenta de almacenamiento de Azure de una región a otra y ha limpiado los recursos de origen.  Para obtener más información sobre cómo trasladar recursos entre regiones y la recuperación ante desastres en Azure, consulte:


- [Traslado de los recursos a un nuevo grupo de recursos o a una nueva suscripción](../../azure-resource-manager/management/move-resource-group-and-subscription.md)
- [Traslado de máquinas virtuales de Azure a otra región](../../site-recovery/azure-to-azure-tutorial-migrate.md)
