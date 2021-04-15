---
title: archivo de inclusión
description: archivo de inclusión
services: functions
author: craigshoemaker
manager: gwallace
ms.service: azure-functions
ms.topic: include
ms.date: 01/28/2020
ms.author: cshoe
ms.custom: include file
ms.openlocfilehash: 6a37850eb6536c5399d63144e60ea210fbc194d8
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "77198432"
---
#### <a name="azure-storage-sdk-version-in-functions-1x"></a>Versión de SDK de Azure Storage en Functions 1.x

En Functions 1.x, los desencadenadores y enlaces de Storage usan la versión 7.2.1 del SDK de Azure Storage (paquete NuGet [WindowsAzure.Storage](https://www.nuget.org/packages/WindowsAzure.Storage/7.2.1)) Si hace referencia a una versión diferente del SDK de Storage y enlaza a un tipo de SDK de Storage en la signatura de función, el runtime de Functions puede notificar que no se puede enlazar con ese tipo. La solución es asegurarse de que el proyecto hace referencia a [WindowsAzure.Storage 7.2.1](https://www.nuget.org/packages/WindowsAzure.Storage/7.2.1).
