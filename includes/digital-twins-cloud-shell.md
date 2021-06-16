---
author: baanders
description: 'Inclusión de archivos en Azure Digital Twins: Configuración de Cloud Shell y la extensión de IoT'
ms.service: digital-twins
ms.topic: include
ms.date: 7/17/2020
ms.author: baanders
ms.openlocfilehash: 5f11c78e6619c2581d04ebc8e97c683c43cf2e73
ms.sourcegitcommit: 7f59e3b79a12395d37d569c250285a15df7a1077
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/02/2021
ms.locfileid: "110793010"
---
Para empezar a trabajar con Azure Digital Twins en una ventana abierta de [Azure Cloud Shell](https://shell.azure.com), primero es necesario iniciar sesión y establecer el contexto de Shell en su suscripción para esta sesión. En Cloud Shell, ejecute estos comandos:

```azurecli-interactive
az login
az account set --subscription "<your-Azure-subscription-ID>"
```

> [!TIP]
> En el comando anterior, también puede utilizar el nombre de la suscripción en lugar del identificador. 

Si esta es la primera vez que usa esta suscripción con Azure Digital Twins, ejecute este comando para registrarse con el espacio de nombres de Azure Digital Twins. (Si no está seguro, es correcto volver a ejecutarlo aunque lo haya hecho en algún momento del pasado).

```azurecli-interactive
az provider register --namespace 'Microsoft.DigitalTwins'
```

A continuación, agregará la [Extensión de Microsoft Azure IoT para la CLI de Azure](/cli/azure/service-page/azure%20iot?view=azure-cli-latest&preserve-view=true) a la instancia de Cloud Shell, para habilitar los comandos para interactuar con Azure Digital Twins y otros servicios IoT. Ejecute este comando para asegurarse de que tiene la versión más reciente de la extensión:

```azurecli-interactive
az extension add --upgrade --name azure-iot
```

Ahora ya está listo para trabajar con Azure Digital Twins en Cloud Shell.

Para comprobarlo, puede ejecutar `az dt --help` en cualquier momento para ver una lista de los comandos de nivel superior de Azure Digital Twins que están disponibles.