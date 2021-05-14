---
title: Aprenda a administrar cuentas de base de datos en Azure Cosmos DB
description: Aprenda a administrar recursos de Azure Cosmos DB mediante Azure Portal, PowerShell, la CLI y plantillas de Azure Resource Manager.
author: markjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 04/25/2021
ms.author: mjbrown
ms.openlocfilehash: 6ce01bf2f285513e6422f16de927fd532f148f9d
ms.sourcegitcommit: 49bd8e68bd1aff789766c24b91f957f6b4bf5a9b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/29/2021
ms.locfileid: "108228755"
---
# <a name="manage-an-azure-cosmos-account"></a>Administración de una cuenta de Azure Cosmos
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

En este artículo se describe cómo administrar varias tareas en una cuenta de Azure Cosmos mediante Azure Portal, Azure PowerShell, la CLI de Azure y plantillas de Azure Resource Manager.

## <a name="create-an-account"></a>Crear una cuenta

### <a name="azure-portal"></a><a id="create-database-account-via-portal"></a>Azure Portal

[!INCLUDE [cosmos-db-create-dbaccount](../../includes/cosmos-db-create-dbaccount.md)]

### <a name="azure-cli"></a><a id="create-database-account-via-cli"></a>Azure CLI

Consulte [Creación de una cuenta de Azure Cosmos DB mediante la CLI de Azure](manage-with-cli.md#create-an-azure-cosmos-db-account).

### <a name="azure-powershell"></a><a id="create-database-account-via-ps"></a>Azure PowerShell.

Consulte [Creación de una cuenta de Azure Cosmos DB mediante PowerShell](manage-with-powershell.md#create-account).

### <a name="azure-resource-manager-template"></a><a id="create-database-account-via-arm-template"></a>Plantilla de Azure Resource Manager

Consulte [Creación y administración de Azure Cosmos DB con plantillas de Azure Resource Manager](./manage-with-templates.md).

## <a name="addremove-regions-from-your-database-account"></a>Incorporación o eliminación de regiones de una cuenta de base de datos

> [!TIP]
> Cuando se agrega una nueva región, todos los datos deben replicarse por completo y estar confirmados en la nueva región antes de que la región se marque como disponible. La cantidad de tiempo que tarde esta operación dependerá de la cantidad de datos almacenados en la cuenta.

### <a name="azure-portal"></a><a id="add-remove-regions-via-portal"></a>Azure Portal

1. Inicie sesión en el [portal de Azure](https://portal.azure.com).

1. Vaya a la cuenta de Azure Cosmos y abra el menú **Replicar datos globalmente**.

1. Para agregar regiones, seleccione los hexágonos del mapa con la etiqueta **+** correspondiente a la región (o regiones) deseada. Otra forma de agregar una región es seleccionar la opción **+ Agregar región** y elegir una región en el menú desplegable.

1. Para quitar regiones, desactive una o varias regiones del mapa; para ello, seleccione los hexágonos azules con marcas de verificación. O seleccione el icono de "Papelera" (🗑) junto a la región en el lado derecho.

1. Seleccione **Aceptar** para guardar los cambios.

   :::image type="content" source="./media/how-to-manage-database-account/add-region.png" alt-text="Menú Agregar o eliminar regiones":::

En el modo de escritura de región individual, no puede quitar la región de escritura. Debe realizar la conmutación por error a otra región antes de poder eliminar la región de escritura actual.

En el modo de escritura de varias regiones, puede agregar o quitar cualquier región, siempre que tenga al menos una región.

### <a name="azure-cli"></a><a id="add-remove-regions-via-cli"></a>Azure CLI

Consulte [Agregar o quitar regiones con la CLI de Azure](manage-with-cli.md#add-or-remove-regions).

### <a name="azure-powershell"></a><a id="add-remove-regions-via-ps"></a>Azure PowerShell.

Consulte [Agregar o quitar regiones con PowerShell](manage-with-powershell.md#update-account).

## <a name="configure-multiple-write-regions"></a><a id="configure-multiple-write-regions"></a>Configuración de varias regiones de escritura

### <a name="azure-portal"></a><a id="configure-multiple-write-regions-portal"></a>Azure Portal

Abra la pestaña **Replicar datos globalmente** y seleccione **Habilitar** para habilitar las escrituras en varias regiones. Después de habilitar las escrituras en varias regiones, todas las regiones de lectura que actualmente tiene en la cuenta se convertirán en regiones de lectura y escritura.

:::image type="content" source="./media/how-to-manage-database-account/single-to-multi-master.png" alt-text="Captura de pantalla de la cuenta de Azure Cosmos que configura las escrituras en varias regiones":::

### <a name="azure-cli"></a><a id="configure-multiple-write-regions-cli"></a>Azure CLI

Consulte [Habilitar regiones de varias escrituras con la CLI de Azure](manage-with-cli.md#enable-multiple-write-regions).

### <a name="azure-powershell"></a><a id="configure-multiple-write-regions-ps"></a>Azure PowerShell.

Consulte [Habilitar regiones de varias escrituras con PowerShell](manage-with-powershell.md#multi-region-writes).

### <a name="resource-manager-template"></a><a id="configure-multiple-write-regions-arm"></a>plantilla de Resource Manager

Es posible migrar una cuenta desde una única región de escritura a varias regiones de escritura mediante la implementación de la plantilla de Resource Manager utilizada para crear la cuenta y estableciendo `enableMultipleWriteLocations: true`. La plantilla de Azure Resource Manager siguiente es una plantilla mínima que implementará una cuenta de Azure Cosmos para SQL API con dos regiones y varias ubicaciones de escritura habilitadas.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "name": {
            "type": "String"
        },
        "location": {
            "type": "String",
            "defaultValue": "[resourceGroup().location]"
        },
        "primaryRegion":{
            "type":"string",
            "metadata": {
                "description": "The primary replica region for the Cosmos DB account."
            }
        },
        "secondaryRegion":{
            "type":"string",
            "metadata": {
              "description": "The secondary replica region for the Cosmos DB account."
          }
        }
    },
    "resources": [
        {
            "type": "Microsoft.DocumentDb/databaseAccounts",
            "kind": "GlobalDocumentDB",
            "name": "[parameters('name')]",
            "apiVersion": "2021-03-15",
            "location": "[parameters('location')]",
            "tags": {},
            "properties": {
                "databaseAccountOfferType": "Standard",
                "consistencyPolicy": { "defaultConsistencyLevel": "Session" },
                "locations":
                [
                    {
                        "locationName": "[parameters('primaryRegion')]",
                        "failoverPriority": 0,
                        "isZoneRedundant": false
                    },
                    {
                        "locationName": "[parameters('secondaryRegion')]",
                        "failoverPriority": 1,
                        "isZoneRedundant": false
                    }
                ],
                "enableMultipleWriteLocations": true
            }
        }
    ]
}
```

## <a name="enable-automatic-failover-for-your-azure-cosmos-account"></a><a id="automatic-failover"></a>Habilitación de la conmutación por error automática en una cuenta de Azure Cosmos

La opción de conmutación automática por error permite que Azure Cosmos DB conmute por error en la región con la prioridad de conmutación por error más alta sin acción del usuario si una región deja de estar disponible. Cuando la conmutación automática por error está habilitada, es posible modificar la prioridad de región. La cuenta debe tener dos o más regiones para habilitar la conmutación automática por error.

### <a name="azure-portal"></a><a id="enable-automatic-failover-via-portal"></a>Azure Portal

1. En la cuenta de Azure Cosmos, abra el panel **Replicar datos globalmente**.

2. En la parte superior del panel, seleccione **Conmutación automática por error**.

   :::image type="content" source="./media/how-to-manage-database-account/replicate-data-globally.png" alt-text="Menú Replicar datos globalmente":::

3. En el panel **Conmutación automática por error**, asegúrese de que **Habilitar la conmutación automática por error** está establecido en **Activado**. 

4. Seleccione **Guardar**.

   :::image type="content" source="./media/how-to-manage-database-account/automatic-failover.png" alt-text="Menú Conmutación por error automática del portal":::

### <a name="azure-cli"></a><a id="enable-automatic-failover-via-cli"></a>Azure CLI

Consulte [Habilitar la conmutación automática por error con la CLI de Azure](manage-with-cli.md#enable-automatic-failover).

### <a name="azure-powershell"></a><a id="enable-automatic-failover-via-ps"></a>Azure PowerShell.

Consulte [Habilitar la conmutación automática por error con PowerShell](manage-with-powershell.md#enable-automatic-failover).

## <a name="set-failover-priorities-for-your-azure-cosmos-account"></a>Establecimiento de las prioridades de conmutación por error para la cuenta de Azure Cosmos

Una vez que una cuenta de Cosmos se configura para la conmutación automática por error, es posible modificar la prioridad de la conmutación por error para las regiones.

> [!IMPORTANT]
> No se puede modificar la región de escritura (prioridad de conmutación por error de cero) cuando la cuenta está configurada para la conmutación automática por error. Para cambiar la región de escritura, debe deshabilitar la conmutación automática por error y realizar una conmutación por error manual.

### <a name="azure-portal"></a><a id="set-failover-priorities-via-portal"></a>Azure Portal

1. En la cuenta de Azure Cosmos, abra el panel **Replicar datos globalmente**.

2. En la parte superior del panel, seleccione **Conmutación automática por error**.

   :::image type="content" source="./media/how-to-manage-database-account/replicate-data-globally.png" alt-text="Menú Replicar datos globalmente":::

3. En el panel **Conmutación automática por error**, asegúrese de que **Habilitar la conmutación automática por error** está establecido en **Activado**.

4. Para modificar la prioridad de la conmutación por error, arrastre las regiones de lectura mediante los tres puntos del lado izquierdo de la fila que aparecen al mantener el puntero sobre ellos.

5. Seleccione **Guardar**.

   :::image type="content" source="./media/how-to-manage-database-account/automatic-failover.png" alt-text="Menú Conmutación por error automática del portal":::

### <a name="azure-cli"></a><a id="set-failover-priorities-via-cli"></a>Azure CLI

Consulte [Establecimiento de la prioridad de conmutación por error con la CLI de Azure](manage-with-cli.md#set-failover-priority).

### <a name="azure-powershell"></a><a id="set-failover-priorities-via-ps"></a>Azure PowerShell.

Consulte [Establecimiento de la prioridad de conmutación por error con PowerShell](manage-with-powershell.md#modify-failover-priority).

## <a name="perform-manual-failover-on-an-azure-cosmos-account"></a><a id="manual-failover"></a>Realización de una conmutación por error manual en una cuenta de Azure Cosmos

> [!IMPORTANT]
> La cuenta de Azure Cosmos debe estar configurada para la conmutación por error manual para que esta operación se realice correctamente.

El proceso para realizar una conmutación por error manual implica cambiar la región de escritura de la cuenta (prioridad de conmutación por error = 0) a otra región configurada para la cuenta.

> [!NOTE]
> No se puede realizar la conmutación por error manual de cuentas con varias regiones de escritura. Para las aplicaciones que usan el SDK de Azure Cosmos, el SDK detecta que una región deja de estar disponible y luego redirige automáticamente a la región más cercana siguiente.

### <a name="azure-portal"></a><a id="enable-manual-failover-via-portal"></a>Azure Portal

1. Vaya a la cuenta de Azure Cosmos y abra el menú **Replicar datos globalmente**.

2. En la parte superior del menú, seleccione **Conmutación por error manual**.

   :::image type="content" source="./media/how-to-manage-database-account/replicate-data-globally.png" alt-text="Menú Replicar datos globalmente":::

3. En el menú **Conmutación por error manual**, seleccione la nueva región de escritura. Active la casilla de verificación para indicar que comprende que esta opción cambia la región de escritura.

4. Para desencadenar la conmutación por error, seleccione **Aceptar**.

   :::image type="content" source="./media/how-to-manage-database-account/manual-failover.png" alt-text="Menú Conmutación por error del portal":::

### <a name="azure-cli"></a><a id="enable-manual-failover-via-cli"></a>Azure CLI

Consulte [Desencadenamiento de una conmutación por error manual con la CLI de Azure](manage-with-cli.md#trigger-manual-failover).

### <a name="azure-powershell"></a><a id="enable-manual-failover-via-ps"></a>Azure PowerShell.

Consulte [Desencadenamiento de una conmutación por error manual con PowerShell](manage-with-powershell.md#trigger-manual-failover).

## <a name="next-steps"></a>Pasos siguientes

Para más información y ejemplos sobre cómo administrar la cuenta de Azure Cosmos así como la base de datos y los contenedores, lea estos artículos:

* [Manage Azure Cosmos DB using Azure PowerShell](manage-with-powershell.md) (Administración de Azure Cosmos DB con Azure PowerShell)
* [Manage Azure Cosmos DB using Azure CLI](manage-with-cli.md) (Administración de Azure Cosmos DB con la CLI de Azure)