---
title: 'Recurso compartido de archivos de Azure: no se pudieron eliminar archivos del recurso compartido de archivos de Azure'
description: Identifique y solucione el error al eliminar archivos del recurso compartido de archivos de Azure.
author: v-miegge
ms.topic: troubleshooting
ms.author: kartup
manager: dcscontentpm
ms.date: 10/25/2019
ms.service: storage
ms.subservice: files
services: storage
tags: ''
ms.openlocfilehash: d8cc0cb7df4bb7bfff5a6b9d2f159cb674532927
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107789760"
---
# <a name="azure-file-share--failed-to-delete-files-from-azure-file-share"></a>Recurso compartido de archivos de Azure: no se pudieron eliminar archivos del recurso compartido de archivos de Azure

El error al eliminar archivos del recurso compartido de archivos de Azure puede mostrar varios síntomas:

**Síntoma 1:**

no se pudo eliminar un archivo en el recurso compartido de archivos de Azure debido a uno de los dos problemas siguientes:

* El archivo que está marcado para eliminarse.
* Un cliente SMB puede estar usando el recurso especificado.

**Síntoma 2:**

cuota insuficiente para procesar este comando.

## <a name="cause"></a>Causa

El error 1816 se muestra cuando se alcanza el límite superior de identificadores abiertos simultáneos que se permiten para un archivo en el equipo donde se está montando el recurso compartido de archivos. Para obtener más información, consulte [Lista de comprobación de rendimiento y escalabilidad de Azure Storage](../blobs/storage-performance-checklist.md).

## <a name="resolution"></a>Solución

Reduzca el número de identificadores abiertos al cerrar algunos.

## <a name="prerequisite"></a>Requisito previo

### <a name="install-the-latest-azure-powershell-module"></a>Instalación del módulo de Azure PowerShell más reciente.

* [Instalación del módulo de Azure PowerShell](/powershell/azure/install-az-ps)

### <a name="connect-to-azure"></a>Conexión a Azure:

```
# Connect-AzAccount
```

### <a name="select-the-subscription-of-the-target-storage-account"></a>Selección de la suscripción de la cuenta de almacenamiento de destino:

```
# Select-AzSubscription -subscriptionid "SubscriptionID"
```

### <a name="create-context-for-the-target-storage-account"></a>Creación del contexto para la cuenta de almacenamiento de destino:

```
$Context = New-AzStorageContext -StorageAccountName "StorageAccountName" -StorageAccountKey "StorageAccessKey"
```

### <a name="get-the-current-open-handles-of-the-file-share"></a>Obtención de los identificadores abiertos actuales del recurso compartido de archivos:

```
# Get-AzStorageFileHandle -Context $Context -ShareName "FileShareName" -Recursive
```

## <a name="example-result"></a>Resultado de ejemplo:

|HandleId|Path|ClientIp|ClientPort|OpenTime|LastReconnectTime|FileId|ParentId|SessionId|
|---|---|---|---|---|---|---|---|---|
|259101229083|---|10.222.10.123|62758|2019-10-05|12:16:50Z|0|0|9507758546259807489|
|259101229131|---|10.222.10.123|62758|2019-10-05|12:36:20Z|0|0|9507758546259807489|
|259101229137|---|10.222.10.123|62758|2019-10-05|12:36:53Z|0|0|9507758546259807489|
|259101229136|Nueva carpeta/test.zip|10.222.10.123|62758|2019-10-05|12:36:29Z|13835132822072852480|9223446803645464576|9507758546259807489|
|259101229135|test.zip|37.222.22.143|62758|2019-10-05|12:36:24Z|11529250230440558592|0|9507758546259807489|

### <a name="close-an-open-handle"></a>Cierre de un identificador abierto:

Para cerrar un identificador abierto, use el siguiente comando:

```
# Close-AzStorageFileHandle -Context $Context -ShareName "FileShareName" -Path 'New folder/test.zip' -CloseAll
```

## <a name="next-steps"></a>Pasos siguientes

* [Solución de problemas de Azure Files en Windows](storage-troubleshoot-windows-file-connection-problems.md)
* [Solución de problemas de Azure Files en Linux](storage-troubleshoot-linux-file-connection-problems.md)
* [Solución de problemas de Azure File Sync](../file-sync/file-sync-troubleshoot.md)