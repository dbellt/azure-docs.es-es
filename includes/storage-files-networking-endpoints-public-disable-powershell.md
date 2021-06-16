---
title: archivo de inclusión
description: archivo de inclusión
services: storage
author: roygara
ms.service: storage
ms.topic: include
ms.date: 6/2/2020
ms.author: rogarana
ms.custom: include file, devx-track-azurepowershell
ms.openlocfilehash: d53bfaab5e00b95be52f652b72e9d55a76c396a6
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110720450"
---
El siguiente comando de PowerShell denegará todo el tráfico que llega al punto de conexión público de la cuenta de almacenamiento. Tenga en cuenta que este comando tiene el parámetro `-Bypass` establecido en `AzureServices`. De esta forma, los servicios propios de confianza, como Azure File Sync, acceden a la cuenta de almacenamiento mediante el punto de conexión público.

```PowerShell
# This assumes $storageAccount is still defined from the beginning of this of this guide.
$storageAccount | Update-AzStorageAccountNetworkRuleSet `
        -DefaultAction Deny `
        -Bypass AzureServices `
        -WarningAction SilentlyContinue `
        -ErrorAction Stop | `
    Out-Null
```
