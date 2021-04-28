---
title: Reinicio, inicio o detención de un servidor flexible de Azure Database for MySQL mediante Azure Portal
description: En este artículo se explica cómo reiniciar, iniciar y detener operaciones en Azure Database for MySQL mediante la CLI de Azure.
author: mksuni
ms.author: sumuth
ms.service: mysql
ms.topic: how-to
ms.date: 03/30/2021
ms.openlocfilehash: c9e646ad40c669e51f052ac9888cdd7c6883a848
ms.sourcegitcommit: 3b5cb7fb84a427aee5b15fb96b89ec213a6536c2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107508978"
---
# <a name="restartstopstart-an-azure-database-for-mysql---flexible-server-preview"></a>Reinicio, inicio o detención de un servidor flexible de Azure Database for MySQL (versión preliminar)

> [!IMPORTANT]
> Actualmente, la opción de implementación Servidor flexible de Azure Database for MySQL se encuentra en versión preliminar pública.

En este artículo se muestra cómo reiniciar, iniciar y detener un servidor flexible mediante la CLI de Azure.

## <a name="prerequisites"></a>Requisitos previos
- Si no tiene una suscripción a Azure, cree una cuenta [gratuita](https://azure.microsoft.com/free/) antes de empezar.
- Instale la CLI de Azure más reciente o actualice la que ya tiene a la versión más reciente. Consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).
-  Inicie sesión en la cuenta de Azure mediante el comando [az login](/cli/azure/reference-index#az-login). Tenga en cuenta la propiedad **id**, que hace referencia al **identificador de suscripción** para su cuenta de Azure.

    ```azurecli-interactive
    az login
    ````

- Si tiene varias suscripciones, elija la más adecuada en la que quiera crear el servidor mediante el comando ```az account set```.
`
    ```azurecli
    az account set --subscription <subscription id>
    ```

- Si aún no ha creado un servidor flexible de MySQL, hágalo ahora con el comando ```az mysql flexible-server create```.

    ```azurecli
    az mysql flexible-server create --resource-group myresourcegroup --name myservername
    ```

## <a name="stop-a-running-server"></a>Detención de un servidor en ejecución
Para detener un servidor, ejecute el comando ```az mysql flexible-server stop```. Si usa [contexto local](/cli/azure/config/param-persist), no es necesario proporcionar ningún argumento.

**Uso:**
```azurecli
az mysql flexible-server stop  [--name]
                               [--resource-group]
                               [--subscription]
```

**Ejemplo sin contexto local:**
```azurecli
az mysql flexible-server stop  --resource-group --name myservername
```

**Ejemplo con contexto local:**
```azurecli
az mysql flexible-server stop
```

## <a name="start-a-stopped-server"></a>Inicio de un servidor detenido
Para iniciar un servidor, ejecute el comando ```az mysql flexible-server start```. Si usa [contexto local](/cli/azure/config/param-persist), no es necesario proporcionar ningún argumento.

**Uso:**
```azurecli
az mysql flexible-server start [--name]
                               [--resource-group]
                               [--subscription]
```

**Ejemplo sin contexto local:**
```azurecli
az mysql flexible-server start  --resource-group --name myservername
```

**Ejemplo con contexto local:**
```azurecli
az mysql flexible-server start
```

> [!IMPORTANT]
> Una vez reiniciado el servidor correctamente, todas las operaciones de administración estarán disponibles para el servidor flexible.

## <a name="restart-a-server"></a>Reinicio de un servidor
Para reiniciar un servidor, ejecute el comando ```az mysql flexible-server restart```. Si usa [contexto local](/cli/azure/config/param-persist), no es necesario proporcionar ningún argumento.

**Uso:**
```azurecli
az mysql flexible-server restart [--name]
                                 [--resource-group]
                                 [--subscription]
```

**Ejemplo sin contexto local:**
```azurecli
az mysql flexible-server restart  --resource-group --name myservername
```

**Ejemplo con contexto local:**
```azurecli
az mysql flexible-server restart
```


> [!IMPORTANT]
> Una vez reiniciado el servidor correctamente, todas las operaciones de administración estarán disponibles para el servidor flexible.

## <a name="next-steps"></a>Pasos siguientes
- Obtenga más información sobre la [conexión de red en servidores flexibles de Azure Database for MySQL](./concepts-networking.md).
- [Creación y administración de una red virtual para un servidor flexible de Azure Database for MySQL mediante Azure Portal](./how-to-manage-virtual-network-portal.md).

