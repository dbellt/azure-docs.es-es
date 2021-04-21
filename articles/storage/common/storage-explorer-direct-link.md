---
title: Vínculo directo del Explorador de Azure Storage | Microsoft Docs
description: Documentación de vínculo directo del Explorador de Azure Storage
services: storage
author: chuye
ms.service: storage
ms.topic: article
ms.date: 02/24/2021
ms.author: chuye
ms.openlocfilehash: 121df1e1c3ab6d0741d3e4da1144a86938da70ed
ms.sourcegitcommit: d63f15674f74d908f4017176f8eddf0283f3fac8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "106582112"
---
# <a name="azure-storage-explorer-direct-link"></a>Vínculo directo del Explorador de Azure Storage

En Windows y macOS, el Explorador de Storage admite URI con el protocolo `storageexplorer://`. Estos URI se conocen como vínculos directos. Un vínculo directo apunta a un recurso de Azure Storage en el Explorador de Storage. Al seguir un vínculo directo, se abrirá el Explorador de Storage y se navegará hasta el recurso al que apunta. En este artículo se describe cómo funcionan los vínculos directos y cómo se pueden usar.

## <a name="how-direct-links-work"></a>Funcionamiento de los vínculos directos

El Explorador de Storage utiliza la vista de árbol para visualizar recursos en Azure. Un vínculo directo contiene la información jerárquica del nodo de recursos vinculados en el árbol. Cuando se sigue un vínculo directo, el Explorador de Storage se abre y recibe los parámetros del vínculo directo. Después, el Explorador de Storage utiliza estos parámetros para navegar al recurso vinculado en la vista de árbol.

> [!IMPORTANT]
> Debe haber iniciado sesión y tener los permisos necesarios para acceder al recurso vinculado para que funcione un vínculo directo.

## <a name="parameters"></a>Parámetros

Un vínculo directo del Explorador de Storage siempre comienza con el protocolo `storageexplorer://`. En la tabla siguiente se explica cada uno de los parámetros posibles de un vínculo directo.

Parámetro | Descripción
:---------| :---------
`v`         | Versión del protocolo del vínculo directo.
`accountid` | Identificador de recursos de Azure Resource Manager de la cuenta de almacenamiento para el recurso vinculado. Si el recurso vinculado es una cuenta de almacenamiento, este identificador será el identificador de recursos de Azure Resource Manager de esa cuenta de almacenamiento. En caso contrario, este identificador será el identificador de recursos de Azure Resource Manager de la cuenta de almacenamiento a la que pertenece el recurso vinculado.
`resourcetype` | Opcional. Solo se usa cuando el recurso vinculado es un contenedor de blobs, un recurso compartido de archivos, una cola o una tabla. Debe ser "Azure.BlobContainer", "Azure.FileShare", "Azure.Queue" o "Azure.FileShare".
`resourcename` | Opcional. Solo se usa cuando el recurso vinculado es un contenedor de blobs, un recurso compartido de archivos, una cola o una tabla. Nombre del recurso vinculado.

Este es un vínculo directo de ejemplo a un contenedor de blobs. 
`storageexplorer://v=1&accountid=/subscriptions/the_subscription_id/resourceGroups/the_resource_group_name/providers/Microsoft.Storage/storageAccounts/the_storage_account_name&subscriptionid=the_subscription_id&resourcetype=Azure.BlobContainer&resourcename=the_blob_container_name`

## <a name="get-a-direct-link-from-storage-explorer"></a>Obtención de un vínculo directo desde el Explorador de Storage

Puede usar el Explorador de Storage para obtener un vínculo directo para un recurso. Abra el menú contextual del nodo para el recurso en la vista de árbol. A continuación, use la acción "Copiar vínculo directo" para copiar su vínculo directo al Portapapeles.

## <a name="direct-link-limitations"></a>Limitaciones de los vínculos directos

Los vínculos directos solo se admiten para los recursos de los nodos de suscripción. Además, solo se admiten los siguientes tipos de recursos:

- Cuentas de almacenamiento
- Contenedores de blobs
- Colas
- Recursos compartidos de archivos
- Tablas
