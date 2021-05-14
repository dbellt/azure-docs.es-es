---
title: Creación, visualización y administración de temas del sistema en Azure Event Grid con la CLI
description: En este artículo se muestra cómo usar la CLI de Azure para crear, ver y eliminar temas del sistema.
ms.topic: conceptual
ms.date: 07/07/2020
ms.openlocfilehash: 34a098406762fd57dc9dc4b58fc375286f5d5b13
ms.sourcegitcommit: 2aeb2c41fd22a02552ff871479124b567fa4463c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107874304"
---
# <a name="create-view-and-manage-event-grid-system-topics-using-azure-cli"></a>Creación, visualización y administración de temas del sistema de Event Grid en la CLI de Azure
En este artículo se muestra cómo crear y administrar temas del sistema con la CLI de Azure. Para una introducción a los temas del sistema, consulte [Temas del sistema](system-topics.md).

## <a name="install-extension-for-azure-cli"></a>Instalación de la extensión para la CLI de Azure
Para la CLI de Azure, necesita la [extensión de Event Grid](/cli/azure/azure-cli-extensions-list).

En Cloud Shell:

- Si ha instalado la extensión anteriormente, actualícela: `az extension update -n eventgrid`
- Si no ha instalado la extensión anteriormente, instálela: `az extension add -n eventgrid`

Para una instalación local:

1. [Instalación de la CLI de Azure](/cli/azure/install-azure-cli). Asegúrese de que tiene la última versión. Para ello, debe comprobarlo con `az --version`.
2. Desinstale las versiones anteriores de la extensión: `az extension remove -n eventgrid`
3. Instale la extensión eventgrid con `az extension add -n eventgrid`

## <a name="create-a-system-topic"></a>Creación de un tema del sistema

- Para crear un tema del sistema en un origen de Azure en primer lugar y después una suscripción de eventos para ese tema, consulte los temas de referencia siguientes:
    - [az eventgrid system-topic create](/cli/azure/eventgrid/system-topic#az_eventgrid_system_topic_create)

        ```azurecli-interactive
        # Get the ID of the Azure source (for example: Azure Storage account)
        storageid=$(az storage account show \
                --name <AZURE STORAGE ACCOUNT NAME> \
                --resource-group <AZURE RESOURCE GROUP NAME> \
                    --query id --output tsv)
    
        # Create the system topic on the Azure source (example: Azure Storage account)
        az eventgrid system-topic create \
            -g <AZURE RESOURCE GROUP NAME> \
            --name <SPECIFY SYSTEM TOPIC NAME> \
            --location <LOCATION> \
            --topic-type microsoft.storage.storageaccounts \
            --source $storageid
        ```           

        Para obtener una lista de los valores de `topic-type` que puede usar para crear un tema del sistema, ejecute el comando siguiente. Estos valores de tipo de tema representan los orígenes de eventos que admiten la creación de temas del sistema. Pase por alto `Microsoft.EventGrid.Topics` y `Microsoft.EventGrid.Domains` de la lista. 

        ```azurecli-interactive
        az eventgrid topic-type  list --output json | grep -w id
        ```
    - [az eventgrid system-topic event-subscription create](/cli/azure/eventgrid/system-topic/event-subscription#az_eventgrid_system_topic_event-subscription-create)

        ```azurecli-interactive
        az eventgrid system-topic event-subscription create --name <SPECIFY EVENT SUBSCRIPTION NAME> \
            -g rg1 --system-topic-name <SYSTEM TOPIC NAME> \
            --endpoint <ENDPOINT URL>         
        ```
- Para crear un tema del sistema (implícitamente) al crear una suscripción de eventos para un origen de Azure, use el método [az eventgrid event-subscription create](/cli/azure/eventgrid/event-subscription#az_eventgrid_event_subscription_create). Este es un ejemplo:
    
    ```azurecli-interactive
    storageid=$(az storage account show --name <AZURE STORAGE ACCOUNT NAME> --resource-group <AZURE RESOURCE GROUP NAME> --query id --output tsv)
    endpoint=<ENDPOINT URL>

    az eventgrid event-subscription create \
      --source-resource-id $storageid \
      --name <EVENT SUBSCRIPTION NAME> \
      --endpoint $endpoint
    ```
    Para ver un tutorial con instrucciones paso a paso, consulte [Suscripción a la cuenta de almacenamiento](../storage/blobs/storage-blob-event-quickstart.md?toc=%2Fazure%2Fevent-grid%2Ftoc.json#subscribe-to-your-storage-account).

## <a name="view-all-system-topics"></a>Visualización de todos los temas del sistema
Para ver todos los temas del sistema y los detalles de uno seleccionado, use los comandos siguientes:

- [az eventgrid system-topic list](/cli/azure/eventgrid/system-topic#az_eventgrid_system_topic_list)

    ```azurecli-interactive
    az eventgrid system-topic list   
     ```
- [az eventgrid system-topic show](/cli/azure/eventgrid/system-topic#az_eventgrid_system_topic_show)

    ```azurecli-interactive
    az eventgrid system-topic show -g <AZURE RESOURCE GROUP NAME> -n <SYSTEM TOPIC NAME>     
     ```

## <a name="delete-a-system-topic"></a>Elimina un tema del sistema.
Para eliminar un tema del sistema, use el comando siguiente: 

- [az eventgrid system-topic delete](/cli/azure/eventgrid/system-topic#az_eventgrid_system_topic_delete)

    ```azurecli-interactive
    az eventgrid system-topic delete -g <AZURE RESOURCE GROUP NAME> --name <SYSTEM TOPIC NAME>   
     ```

## <a name="next-steps"></a>Pasos siguientes
Consulte los [temas del sistema de Azure Event Grid](system-topics.md) para más información sobre los temas del sistema y los tipos de temas admitidos en Azure Event Grid. 