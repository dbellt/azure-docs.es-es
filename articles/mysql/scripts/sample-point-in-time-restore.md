---
title: 'Script de la CLI: Restauración de un servidor de Azure Database for MySQL'
description: En este script de la CLI de Azure de ejemplo se muestra cómo restaurar un servidor de Azure Database for MySQL y sus bases de datos a un momento anterior.
author: savjani
ms.author: pariks
ms.service: mysql
ms.devlang: azurecli
ms.topic: sample
ms.custom: mvc, devx-track-azurecli
ms.date: 12/02/2019
ms.openlocfilehash: 2a2409c94e28135ca5c1fb8fa7210cd9b8b42858
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107791698"
---
# <a name="restore-an-azure-database-for-mysql-server-using-azure-cli"></a>Restauración de un servidor de Azure Database for MySQL mediante la CLI de Azure
Este script de la CLI de ejemplo restaura un único servidor de Azure Database for MySQL a un momento anterior.

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

- En este artículo se necesita la versión 2.0 o posterior de la CLI de Azure. Si usa Azure Cloud Shell, ya está instalada la versión más reciente. 

## <a name="sample-script"></a>Script de ejemplo
En este script de ejemplo, va a modificar las líneas resaltadas para actualizar el nombre de usuario administrador y la contraseña a los suyos propios. Reemplace el identificador de suscripción que se usa en los comandos `az monitor` por su propio identificador de suscripción.
[!code-azurecli-interactive[main](../../../cli_scripts/mysql/backup-restore-pitr/backup-restore.sh?highlight=15-16 "Restore Azure Database for MySQL.")]

## <a name="clean-up-deployment"></a>Limpieza de la implementación
Use el comando siguiente para quitar el grupo de recursos y todos los recursos asociados a él después de ejecutarse el script de ejemplo. 
[!code-azurecli-interactive[main](../../../cli_scripts/mysql/backup-restore-pitr/delete-mysql.sh  "Delete the resource group.")]

## <a name="script-explanation"></a>Explicación del script
Este script usa los comandos que se describen en la tabla siguiente:

| **Comando** | **Notas** |
|---|---|
| [az group create](/cli/azure/group#az_group_create) | Crea un grupo de recursos en el que se almacenan todos los recursos. |
| [az mysql server create](/cli/azure/mysql/server#az_mysql_server_create) | Crea un servidor MySQL que hospeda las bases de datos. |
| [az mysql server restore](/cli/azure/mysql/server#az_mysql_server_restore) | Restaura un servidor desde una copia de seguridad. |
| [az group delete](/cli/azure/group#az_group_delete) | Elimina un grupo de recursos, incluidos todos los recursos anidados. |

## <a name="next-steps"></a>Pasos siguientes
- Para más información sobre la CLI de Azure: [Documentación de la CLI de Azure](/cli/azure).
- Pruebe otros scripts adicionales: [Ejemplos de la CLI de Azure para Azure Database for MySQL](../sample-scripts-azure-cli.md)
