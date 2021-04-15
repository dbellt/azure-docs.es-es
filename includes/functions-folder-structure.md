---
title: archivo de inclusión
description: archivo de inclusión
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 09/12/2018
ms.author: glenga
ms.custom: include file
ms.openlocfilehash: 5f5f682c8c31c17f1a645bcdacdc78f32bd5c001
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "96018458"
---
El código de todas las funciones de una aplicación de función está ubicado en una carpeta de proyecto raíz que contiene un archivo de configuración de host y una o varias subcarpetas. Cada subcarpeta contiene el código de una función diferente. La estructura de carpetas se muestra en la siguiente representación:

```
FunctionApp
 | - host.json
 | - MyFirstFunction
 | | - function.json
 | | - ...  
 | - MySecondFunction
 | | - function.json
 | | - ...  
 | - SharedCode
 | - bin
```

Tanto en la versión 2.x como en las superiores del runtime de Functions, todas las funciones de la aplicación de función deben compartir la misma pila de lenguaje.  

El archivo [host.json](../articles/azure-functions/functions-host-json.md) contiene configuraciones específicas del entorno de ejecución y está en la carpeta raíz de la aplicación de funciones. Un carpeta *bin* contiene paquetes y otros archivos de biblioteca que requiere la aplicación de funciones. Consulte los requisitos específicos de idioma para un proyecto de aplicación de función:

* [Biblioteca de clases C# (.csproj)](../articles/azure-functions/functions-dotnet-class-library.md#functions-class-library-project)
* [Script de C# (.csx)](../articles/azure-functions/functions-reference-csharp.md#folder-structure)
* [Script de F#](../articles/azure-functions/functions-reference-fsharp.md#folder-structure)
* [Java](../articles/azure-functions/functions-reference-java.md#folder-structure)
* [JavaScript](../articles/azure-functions/functions-reference-node.md#folder-structure)
* [Python](../articles/azure-functions/functions-reference-python.md#folder-structure)
