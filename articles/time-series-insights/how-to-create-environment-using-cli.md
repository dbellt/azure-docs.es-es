---
title: Creación de un entorno de Azure Time Series Insights Gen2 con la CLI de Azure - Azure Time Series Insights Gen2 | Microsoft Docs
description: Aprenda a configurar un entorno en Azure Time Series Insights Gen2 con la CLI de Azure.
author: tedvilutis
ms.author: tvilutis
manager: cnovak
ms.reviewer: orspodek
ms.workload: big-data
ms.service: time-series-insights
services: time-series-insights
ms.topic: how-to
ms.date: 03/15/2021
ms.custom: seodec18, devx-track-azurecli
ms.openlocfilehash: 36a6da683912ac004b948b9a334da1758ab903fd
ms.sourcegitcommit: 7f59e3b79a12395d37d569c250285a15df7a1077
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/02/2021
ms.locfileid: "110791935"
---
# <a name="create-an-azure-time-series-insights-gen2-environment-using-the-azure-cli"></a>Creación de un entorno de Azure Time Series Insights Gen2 mediante la CLI de Azure

Este documento le guiará a través de la creación de un nuevo entorno de Time Series Insights Gen2.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="prerequisites"></a>Prerequisites

* Crear una cuenta de Azure Storage para el [almacenamiento en reposo](./concepts-storage.md#cold-store) del entorno. Esta cuenta está diseñada para la retención y el análisis a largo plazo de datos históricos.

> [!NOTE]
> En el código, reemplace `mytsicoldstore` por un nombre único para la cuenta de almacenamiento en frío.

Primero, cree la cuenta de almacenamiento:

```azurecli-interactive
storage=mytsicoldstore
rg=-my-resource-group-name
az storage account create -g $rg -n $storage --https-only
key=$(az storage account keys list -g $rg -n $storage --query [0].value --output tsv
```

## <a name="creating-the-environment"></a>Creación del entorno

Ahora que se ha creado la cuenta de almacenamiento y que su nombre y clave de administración están asignados a las variables, ejecute el siguiente comando para crear el entorno de Azure Time Series Insights:

> [!NOTE]
> En el código, reemplace lo siguiente por nombres únicos para su escenario:
>
> * `my-tsi-env` por el nombre del entorno.
> * `my-ts-id-prop` por el nombre de la propiedad de identificador de serie temporal.

> [!IMPORTANT]
> El identificador de serie temporal del entorno es como una clave de partición de base de datos. El identificador de serie temporal también actúa como clave principal para el modelo de serie temporal.
>
> Para más información, consulte [Procedimientos recomendados al elegir un id. de serie temporal](./how-to-select-tsid.md).

```azurecli-interactive
az tsi environment gen2 create --name "my-tsi-env" --location eastus2 --resource-group $rg --sku name="L1" capacity=1 --time-series-id-properties name=my-ts-id-prop type=String --warm-store-configuration data-retention=P7D --storage-configuration account-name=$storage management-key=$key
```

## <a name="remove-an-azure-time-series-insights-environment"></a>Eliminación de un entorno de Azure Time Series Insights

Puede usar la CLI de Azure para eliminar un recurso individual, como un entorno de Time Series Insights, o eliminar un grupo de recursos y todos sus recursos, incluidos los entornos de Time Series Insights.

Para [eliminar un entorno de Time Series Insights](/cli/azure/tsi/environment#az_tsi_environment_delete), ejecute el siguiente comando:

```azurecli-interactive
az tsi environment delete --name "my-tsi-env" --resource-group $rg
```

Para [eliminar la cuenta de almacenamiento](/cli/azure/storage/account#az_storage_account_delete), ejecute el siguiente comando:

```azurecli-interactive
az storage account delete --name $storage --resource-group $rg
```

Ejecute el comando siguiente para [eliminar un grupo de recursos](/cli/azure/group#az_group_delete) y todos sus recursos:

```azurecli-interactive
az group delete --name $rg
```

## <a name="next-steps"></a>Pasos siguientes

* Obtenga información acerca de los [orígenes de eventos de ingesta de streaming](./concepts-streaming-ingestion-event-sources.md) en el entorno de Azure Time Series Insights Gen2.
* Obtenga información acerca de cómo conectarse a una instancia de [IoT Hub](./how-to-ingest-data-iot-hub.md).
