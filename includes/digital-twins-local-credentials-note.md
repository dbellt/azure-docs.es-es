---
author: baanders
description: 'archivo de inclusión para DefaultAzureCredential en ejemplos de Azure Digital Twins: nota'
ms.service: digital-twins
ms.topic: include
ms.date: 10/22/2020
ms.author: baanders
ms.openlocfilehash: 462098536ae4598c7875a1b8291e477f6b4a8d7d
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108778416"
---
>[!NOTE]
> En este ejemplo se usa [DefaultAzureCredential](/dotnet/api/azure.identity.defaultazurecredential) (parte de la biblioteca `Azure.Identity`) para autenticar a los usuarios en la instancia de Azure Digital Twins cuando la ejecuta en la máquina local. Con este tipo de autenticación, el ejemplo buscará las credenciales de Azure en el entorno local, como un inicio de sesión de una instancia local de la [CLI de Azure](/cli/azure/install-azure-cli) o en Visual Studio y Visual Studio Code.
>
> Para más información sobre el uso de `DefaultAzureCredential` y otras opciones de autenticación, consulte [Procedimiento: Escritura de código de autenticación de aplicación](../articles/digital-twins/how-to-authenticate-client.md).