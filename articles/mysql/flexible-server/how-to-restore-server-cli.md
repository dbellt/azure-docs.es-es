---
title: 'Restauración de Azure Database for MySQL: servidor flexible con la CLI de Azure'
description: En este artículo se explica cómo realizar operaciones de restauración en Azure Database for MySQL mediante la CLI de Azure.
author: mksuni
ms.author: sumuth
ms.service: mysql
ms.topic: how-to
ms.date: 04/01/2021
ms.openlocfilehash: be7eaa7ce5f32aa441f6b44cc430de9a74be17d4
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108738690"
---
# <a name="point-in-time-restore-of-a-azure-database-for-mysql---flexible-server-with-azure-cli"></a>Recuperación a un momento dado de una instancia de Azure Database for MySQL: servidor flexible con la CLI de Azure


> [!IMPORTANT]
> Azure Database for MySQL: servidor flexible está actualmente en versión preliminar pública.

En este artículo se proporciona un procedimiento detallado para llevar a cabo recuperaciones a un momento dado en servidores flexibles mediante copias de seguridad.

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

- Si aún no ha creado un servidor flexible de MySQL, hágalo ahora con el comando ```az mysql flexible-server create```.

    ```azurecli
    az mysql flexible-server create --resource-group myresourcegroup --name myservername
    ```

## <a name="restore-a-server-from-backup-to-a-new-server"></a>Restauración de un servidor de la copia de seguridad en un nuevo servidor

Puede ejecutar el siguiente comando para restaurar un servidor de la copia de seguridad existente más temprana.

**Uso**
```azurecli
az mysql flexible-server restore --restore-time
                                 --source-server
                                 [--ids]
                                 [--location]
                                 [--name]
                                 [--no-wait]
                                 [--resource-group]
                                 [--subscription]
```

**Ejemplo:** Restauración de un servidor a partir de esta instantánea de copia de seguridad ```2021-03-03T13:10:00Z```.

```azurecli
az mysql server restore \
--name mydemoserver-restored \
--resource-group myresourcegroup \
--restore-point-in-time "2021-03-03T13:10:00Z" \
--source-server mydemoserver
```
El tiempo que lleve la restauración dependerá del tamaño de los datos almacenados en el servidor.

## <a name="perform-post-restore-tasks"></a>Tareas posteriores a la restauración
Cuando finalice la restauración, debe realizar las siguientes tareas para que los usuarios y las aplicaciones vuelvan a estar en funcionamiento:

- Si el nuevo servidor está destinado a reemplazar al servidor original, redirija los clientes y las aplicaciones de cliente al nuevo servidor.
- Asegúrese de que haya vigentes reglas de red virtual adecuadas para que los usuarios se conecten. Estas reglas no se copian desde el servidor original.
- No se olvide de emplear los permisos de nivel de base de datos y los inicios de sesión apropiados.
- Configuración de alertas según corresponda para el servidor recién restaurado

## <a name="next-steps"></a>Pasos siguientes
Más información sobre la [continuidad del negocio](concepts-business-continuity.md).

