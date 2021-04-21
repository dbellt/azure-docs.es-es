---
title: 'Crecimiento automático del almacenamiento de Azure Database for PostgreSQL: servidor único mediante la CLI de Azure'
description: 'En este artículo se describe cómo puede configurar el crecimiento automático del almacenamiento de Azure en Azure Database for PostgreSQL: servidor único mediante la CLI de Azure.'
author: sunilagarwal
ms.author: sunila
ms.service: postgresql
ms.topic: how-to
ms.date: 8/7/2019
ms.custom: devx-track-azurecli
ms.openlocfilehash: d16fe5ef6654ee29c3e345ff0532ed91206d86d3
ms.sourcegitcommit: dddd1596fa368f68861856849fbbbb9ea55cb4c7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107366177"
---
# <a name="auto-grow-azure-database-for-postgresql-storage---single-server-using-the-azure-cli"></a>Crecimiento automático del almacenamiento de Azure Database for PostgreSQL: servidor único con la CLI de Azure
En este artículo se describe cómo configurar el almacenamiento en el servidor Azure Database for PostgreSQL para que aumente sin que ello afecte a la carga de trabajo.

El servidor [que alcanza el límite de almacenamiento](./concepts-pricing-tiers.md#reaching-the-storage-limit) se establece en solo lectura. Si el crecimiento automático del almacenamiento está habilitado, para servidores con menos de 100 GB de almacenamiento aprovisionado, el tamaño del almacenamiento aprovisionado se incrementa en 5 GB tan pronto como el almacenamiento disponible se encuentre por debajo de 1 GB o el 10 % del almacenamiento aprovisionado. En los servidores con más de 100 GB de almacenamiento aprovisionado, el tamaño del almacenamiento aprovisionado se incrementa en un 5 % cuando el espacio de almacenamiento disponible está por debajo de 10 GB del tamaño del almacenamiento aprovisionado. Se aplican los límites máximos de almacenamiento según lo especificado [aquí](./concepts-pricing-tiers.md#storage).

## <a name="prerequisites"></a>Requisitos previos

- Necesita un [servidor de Azure Database for PostgreSQL](quickstart-create-server-database-azure-cli.md).

[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

- En este artículo se necesita la versión 2.0 o posterior de la CLI de Azure. Si usa Azure Cloud Shell, ya está instalada la versión más reciente.

## <a name="enable-postgresql-server-storage-auto-grow"></a>Habilitar el crecimiento automático del almacenamiento de servidores PostgreSQL

Habilite el almacenamiento de crecimiento automático de servidor en un servidor existente con el siguiente comando:

```azurecli-interactive
az postgres server update --name mydemoserver --resource-group myresourcegroup --auto-grow Enabled
```

Habilite el almacenamiento de crecimiento automático de servidor mientras crea un nuevo servidor con el siguiente comando:

```azurecli-interactive
az postgres server create --resource-group myresourcegroup --name mydemoserver  --auto-grow Enabled --location westus --admin-user myadmin --admin-password <server_admin_password> --sku-name GP_Gen5_2 --version 9.6
```

## <a name="next-steps"></a>Pasos siguientes

Aprenda sobre la [creación de alertas de métricas](howto-alert-on-metric.md).
