---
title: 'Inicio rápido: Creación de un grupo de Synapse SQL con la CLI de Azure'
description: Cree rápidamente un grupo de Synapse SQL con una regla de firewall de nivel de servidor mediante la CLI de Azure.
services: synapse-analytics
author: julieMSFT
ms.service: synapse-analytics
ms.topic: quickstart
ms.subservice: sql-dw
ms.date: 11/20/2020
ms.author: jrasnick
ms.custom: azure-synapse, devx-track-azurecli
ms.openlocfilehash: 3903aa0be5ffa63bc4292371c59002846ec9363c
ms.sourcegitcommit: 5ce88326f2b02fda54dad05df94cf0b440da284b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107892101"
---
# <a name="quickstart-create-a-synapse-sql-pool-with-azure-cli"></a>Inicio rápido: Creación de un grupo de Synapse SQL con la CLI de Azure

Cree un grupo de Synapse SQL (almacenamiento de datos) en Azure Synapse Analytics mediante la CLI de Azure.

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

## <a name="getting-started"></a>Introducción

Use estos comandos para iniciar sesión en Azure y configurar un grupo de recursos.

1. Si usa una instalación local, ejecute el comando [az login](/cli/azure/reference-index#az_login) para iniciar sesión en Azure:

   ```azurecli
   az login
   ```

1. Si es necesario, use el comando [az account set](/cli/azure/account#az_account_set) para seleccionar la suscripción:

   ```azurecli
   az account set --subscription 00000000-0000-0000-0000-000000000000
   ```

1. Ejecute el comando [az group create](/cli/azure/group#az_group_create) para crear un grupo de recursos:

   ```azurecli
   az group create --name myResourceGroup --location WestEurope
   ```

1. Cree un [servidor SQL lógico](../../azure-sql/database/logical-servers.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json) con el comando [az sql server create](/cli/azure/sql/server#az_sql_server_create):

   ```azurecli
   az sql server create --resource-group myResourceGroup --name mysqlserver \
      --admin-user ServerAdmin --admin-password ChangeYourAdminPassword1
   ```

   Un servidor contiene un conjunto de bases de datos administradas como un grupo.

## <a name="configure-a-server-level-firewall-rule"></a>Configuración de una regla de firewall de nivel de servidor

Cree una [regla de firewall de nivel de servidor](../../azure-sql/database/firewall-configure.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json). Una regla de firewall de nivel de servidor permite a una aplicación externa, como SQL Server Management Studio, o la utilidad SQLCMD, conectarse a un grupo de SQL mediante el firewall de servicio del grupo de SQL.

Ejecute el comando [az sql server firewall-rule create](/cli/azure/sql/server/firewall-rule#az_sql_server_firewall_rule_create) para crear una regla de firewall:

```azurecli
az sql server firewall-rule create --resource-group myResourceGroup --name AllowSome \
   --server mysqlserver --start-ip-address 0.0.0.0 --end-ip-address 0.0.0.0
```

En este ejemplo el firewall está abierto solo para otros recursos de Azure. Para habilitar la conectividad externa, cambie la dirección IP a una dirección apropiada para su entorno. Para abrir todas las direcciones IP, utilice 0.0.0.0 como la dirección IP inicial y 255.255.255.255 como la dirección final.

> [!NOTE]
> Los puntos de conexión de SQL se comunican a través del puerto 1433. Si intenta conectarse desde dentro de una red corporativa, es posible que el firewall de la red no permita el tráfico de salida a través del puerto 1433. Si es así, no podrá conectarse al servidor a menos que el departamento de TI abra el puerto 1433.
>

## <a name="create-and-manage-your-sql-pool"></a>Creación y administración de un grupo de SQL

Cree el grupo de SQL. En este ejemplo se usa DW100c como objetivo del servicio, que es un punto de partida de menor costo para el grupo de SQL.

> [!NOTE]
> Debe haber creado antes un área de trabajo. Para más información, consulte [Inicio rápido: Creación de un área de trabajo de Azure Synapse con la CLI de Azure](../quickstart-create-workspace-cli.md).

Use el comando [az synapse sql pool create](/cli/azure/synapse/sql/pool#az_synapse_sql_pool_create) para crear el grupo de SQL:

```azurecli
az synapse sql pool create --resource-group myResourceGroup --name mySampleDataWarehouse \
   --performance-level "DW1000c" --workspace-name testsynapseworkspace
```

Para más información sobre las opciones de parámetro, consulte [az synapse sql pool](/cli/azure/synapse/sql/pool).

Puede ver los grupos de SQL mediante el comando [az synapse sql pool list](/cli/azure/synapse/sql/pool#az_synapse_sql_pool_list):

```azurecli
az synapse sql pool list --resource-group myResourceGroup --workspace-name testsynapseworkspace
```

Use el comando [az synapse sql pool update](/cli/azure/synapse/sql/pool#az_synapse_sql_pool_update) para actualizar un grupo existente:

```azurecli
az synapse sql pool update --resource-group myResourceGroup --name mySampleDataWarehouse \
   --workspace-name testsynapseworkspace
```

Use el comando [az synapse sql pool pause](/cli/azure/synapse/sql/pool#az_synapse_sql_pool_pause) para pausar un grupo:

```azurecli
az synapse sql pool pause --resource-group myResourceGroup --name mySampleDataWarehouse \
   --workspace-name testsynapseworkspace
```

Use el comando [az synapse sql pool resume](/cli/azure/synapse/sql/pool#az_synapse_sql_pool_resume) para reanudar un grupo pausado:

```azurecli
az synapse sql pool resume --resource-group myResourceGroup --name mySampleDataWarehouse \
   --workspace-name testsynapseworkspace
```

Para quitar un grupo de SQL existente, use el comando [az synapse sql pool delete](/cli/azure/synapse/sql/pool#az_synapse_sql_pool_delete):

```azurecli
az synapse sql pool delete --resource-group myResourceGroup --name mySampleDataWarehouse \
   --workspace-name testsynapseworkspace
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Otros tutoriales de inicio rápido de esta colección se basan en esta guía.

> [!TIP]
> Si tiene previsto seguir trabajando con otros tutoriales de inicio rápido, no elimine los recursos creados en esta guía de inicio rápido. Si no va a continuar, use el comando [az ggroup delete](/cli/azure/group#az_group_delete) para eliminar todos los recursos creados mediante este inicio rápido.
>

```azurecli
az group delete --ResourceGroupName MyResourceGroup
```

## <a name="next-steps"></a>Pasos siguientes

Ya ha creado un grupo de SQL, una regla de firewall y la ha conectado al grupo de SQL. Para más información, diríjase al artículo [Carga de datos en un grupo de SQL](./load-data-from-azure-blob-storage-using-copy.md).
