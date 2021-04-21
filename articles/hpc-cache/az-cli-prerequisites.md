---
title: Requisitos previos de la CLI de Azure para Azure HPC Cache
description: Pasos de configuración para usar la CLI de Azure para crear o modificar Azure HPC Cache
author: ekpgh
ms.service: hpc-cache
ms.topic: how-to
ms.date: 07/08/2020
ms.author: v-erkel
ms.openlocfilehash: 30621eceefd69cd3e08de137bb34f1079a17a406
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107780500"
---
# <a name="set-up-azure-cli-for-azure-hpc-cache"></a>Configuración de la CLI de Azure para Azure HPC Cache

Siga estos pasos para preparar el entorno antes de usar la CLI de Azure para crear o administrar la extensión Azure HPC Cache.

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

 - Azure HPC Cache requiere la versión 2.7 o posterior de la CLI de Azure. Si usa Azure Cloud Shell, ya está instalada la versión más reciente.

## <a name="set-default-resource-group-optional"></a>Configuración del grupo de recursos predeterminado (opcional)

La mayoría de los comandos hpc-cache requieren que se pase el grupo de recursos de la memoria caché. Puede establecer el grupo de recursos predeterminado mediante [az configure](/cli/azure/reference-index#az_configure).

## <a name="next-steps"></a>Pasos siguientes

Después de instalar la extensión de la CLI de Azure e iniciar sesión, puede usar la CLI de Azure para crear y administrar sistemas de Azure HPC Cache.

* [Creación de una instancia de Azure HPC Cache](hpc-cache-create.md)
* [Documentación de hpc-cache de la CLI de Azure](/cli/azure/ext/hpc-cache/hpc-cache)
