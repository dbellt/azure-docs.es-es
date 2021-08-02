---
title: 'Inicio y detención de Azure Database for PostgreSQL con la CLI de Azure: servidor flexible'
description: En este artículo se explica cómo iniciar y detener operaciones en Azure Database for PostgreSQL mediante la CLI de Azure.
author: mksuni
ms.author: sumuth
ms.service: postgresql
ms.topic: how-to
ms.date: 05/29/2021
ms.openlocfilehash: bebcc9e29620c8e739735553dd64e791c5bcdb00
ms.sourcegitcommit: 7f59e3b79a12395d37d569c250285a15df7a1077
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/02/2021
ms.locfileid: "111027301"
---
# <a name="stopstart-azure-database-for-postgresql---flexible-server-preview-using-azure-cli"></a>Inicio y detención de Azure Database for PostgreSQL, servidor flexible (versión preliminar) mediante la CLI de Azure

> [!IMPORTANT]
> Actualmente Servidor flexible de Azure Database for PostgreSQL se encuentra en versión preliminar.

En este artículo se muestra cómo reiniciar, iniciar y detener un servidor flexible mediante la CLI de Azure.

## <a name="prerequisites"></a>Requisitos previos

- Si no tiene una suscripción a Azure, cree una cuenta [gratuita](https://azure.microsoft.com/free/) antes de empezar.
- Instale la CLI de Azure más reciente o actualice la que ya tiene a la versión más reciente. Consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).
-  Inicie sesión en la cuenta de Azure mediante el comando [az login](/cli/azure/reference-index#az_login). Tenga en cuenta la propiedad **id**, que hace referencia al **identificador de suscripción** para su cuenta de Azure.

    ```azurecli-interactive
    az login
    ````

- Si tiene varias suscripciones, elija la más adecuada en la que quiera crear el servidor mediante el comando ```az account set```.
`
    ```azurecli
    az account set --subscription <subscription id>
    ```

- Si aún no ha creado un servidor flexible de PostgreSQL, hágalo ahora con el comando ```az postgres flexible-server create```.

    ```azurecli
    az postgres flexible-server create --resource-group myresourcegroup --name myservername
    ```

## <a name="stop-a-running-server"></a>Detención de un servidor en ejecución
Para detener un servidor, ejecute el comando ```az postgres flexible-server stop```. Si usa [contexto local](/cli/azure/config/param-persist), no es necesario proporcionar ningún argumento.

**Uso:**
```azurecli
az postgres flexible-server stop  [--name]
                               [--resource-group]
                               [--subscription]
```

**Ejemplo sin contexto local:**
```azurecli
az postgres flexible-server stop  --resource-group --name myservername
```

**Ejemplo con contexto local:**
```azurecli
az postgres flexible-server stop
```

## <a name="start-a-stopped-server"></a>Inicio de un servidor detenido
Para iniciar un servidor, ejecute el comando ```az postgres flexible-server start```. Si usa [contexto local](/cli/azure/config/param-persist), no es necesario proporcionar ningún argumento.

**Uso:**
```azurecli
az postgres flexible-server start [--name]
                               [--resource-group]
                               [--subscription]
```

**Ejemplo sin contexto local:**
```azurecli
az postgres flexible-server start  --resource-group --name myservername
```

**Ejemplo con contexto local:**
```azurecli
az postgres flexible-server start
```

> [!IMPORTANT]
> Una vez reiniciado el servidor correctamente, todas las operaciones de administración estarán disponibles para el servidor flexible.

## <a name="next-steps"></a>Pasos siguientes
- Obtener más información sobre el [reinicio de Azure Database for PostgreSQL: servidor flexible](./how-to-restart-server-cli.md)


