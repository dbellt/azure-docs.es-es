---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 03/25/2020
ms.author: glenga
ms.openlocfilehash: 7a390c0d19a37ea18f9eac8636683ec35ecbc844
ms.sourcegitcommit: 2aeb2c41fd22a02552ff871479124b567fa4463c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107880311"
---
## <a name="configure-your-local-environment"></a>Configuración del entorno local

Antes de empezar, debe disponer de lo siguiente:

+ Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).

::: zone pivot="programming-language-csharp,programming-language-javascript,programming-language-typescript,programming-language-powershell,programming-language-java,programming-language-other"  
+ [Azure Functions Core Tools](../articles/azure-functions/functions-run-local.md#v2), versión 2.7.1846 u otra posterior.
::: zone-end  
::: zone pivot="programming-language-python"
+ La versión de Azure Functions Core Tools correspondiente a la versión instalada de Python:

   | Versión de Python | Versión de Core Tools |
   | -------------- | ------------------ |
   | Python 3.8     | [versión 3.x](../articles/azure-functions/functions-run-local.md#v2) |
   | Python 3.6<br/>Python 3.7 | [Versión 2.7.1846 o posterior](../articles/azure-functions/functions-run-local.md#v2) |
  
::: zone-end

+ La [CLI de Azure](/cli/azure/install-azure-cli), versión 2.4 o cualquier versión posterior. 
::: zone pivot="programming-language-javascript,programming-language-typescript"
+ Versiones de [Node.js](https://nodejs.org/), Active LTS y Maintenance LTS (se recomiendan 8.11.1 y 10.14.1).
::: zone-end

::: zone pivot="programming-language-python"
+ [Python 3.8 (64 bits)](https://www.python.org/downloads/release/python-382/), [Python 3.7 (64 bits)](https://www.python.org/downloads/release/python-375/), [Python 3.6 (64 bits)](https://www.python.org/downloads/release/python-368/), que son compatibles con Azure Functions. 
::: zone-end
::: zone pivot="programming-language-powershell"
+ [SDK de .NET Core 3.1](https://dotnet.microsoft.com/download).
::: zone-end
::: zone pivot="programming-language-java"  
+ [Kit para desarrolladores de Java](/azure/developer/java/fundamentals/java-jdk-long-term-support), versión 8 u 11. 

+ [Apache Maven](https://maven.apache.org), versión 3.0 o posterior.

::: zone-end
::: zone pivot="programming-language-other"
+ Herramientas de desarrollo para el lenguaje que utilice. En este tutorial se usa el [lenguaje de programación R](https://www.r-project.org/) como ejemplo.
::: zone-end