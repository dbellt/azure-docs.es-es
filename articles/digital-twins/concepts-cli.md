---
title: Conjunto de comandos de la CLI de Azure Digital Twins
titleSuffix: Azure Digital Twins
description: Conozca el conjunto de comandos de la CLI de Azure Digital Twins.
author: baanders
ms.author: baanders
ms.date: 04/30/2021
ms.topic: conceptual
ms.service: digital-twins
ms.openlocfilehash: d3e081d4a60b5e4f26e5163d55f2df682b042bca
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110066528"
---
# <a name="azure-digital-twins-cli-command-set"></a>Conjunto de comandos de la CLI de Azure Digital Twins

Además de administrar la instancia de Azure Digital Twins en Azure Portal, Azure Digital Twins tiene un conjunto de comandos para la [CLI de Azure](/cli/azure/what-is-azure-cli) que puede usar para realizar la mayoría de las acciones principales con el servicio, entre las que se incluyen:
* Administración de una instancia de Azure Digital Twins
* Administración de modelos
* Administración de gemelos digitales
* Administración de relaciones gemelas
* Configuración de puntos de conexión
* Administración de [rutas](concepts-route-events.md)
* Configuración de la [seguridad](concepts-security.md) mediante el control de acceso basado en rol de Azure (RBAC de Azure)

El conjunto de comandos se llama **az dt** y forma parte de la [extensión de Azure IoT para la CLI de Azure](https://github.com/Azure/azure-iot-cli-extension). Puede ver la lista completa de comandos y su uso como parte de la documentación de referencia del conjunto de comandos `az iot`: referencia del comando [az dt](/cli/azure/dt?view=azure-cli-latest&preserve-view=true).

## <a name="uses-deploy-and-validate"></a>Usos (implementación y validación)

Además de administrar la instancia, generalmente, la CLI es también una herramienta útil para la implementación y la validación.
* Los comandos de plano de control se pueden usar para hacer que la implementación de una nueva instancia sea repetible o automatizada.
* Los comandos de plano de datos se pueden usar para comprobar rápidamente los valores de la instancia y que las operaciones se completaron según lo previsto.

## <a name="get-the-command-set"></a>Obtención del conjunto de comandos

Los comandos de Azure Digital Twins forman parte de la [extensión de Azure IoT para la CLI de Azure (azure-iot)](https://github.com/Azure/azure-iot-cli-extension), por lo que debe seguir estos pasos para asegurarse de que tiene la extensión `azure-iot` más reciente con los comandos **az dt**.

### <a name="cli-version-requirements"></a>Requisitos de versión de la CLI

Si usa la CLI de Azure con PowerShell, el paquete de extensión requiere que la versión de la CLI de Azure sea la **2.3.1** o superior.

Puede comprobar la versión de la CLI de Azure con este comando de la CLI:
```azurecli
az --version
```

Para obtener instrucciones sobre cómo instalar o actualizar la CLI de Azure a una versión más reciente, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

### <a name="get-the-extension"></a>Obtener la extensión

La CLI de Azure le solicitará automáticamente que instale la extensión la primera vez que use algún comando que la necesite.

Como alternativa, puede usar el siguiente comando para instalar la extensión usted mismo en cualquier momento (o actualizarla si tiene una versión anterior). El comando se puede ejecutar en [Azure Cloud Shell](../cloud-shell/overview.md) o en una [CLI de Azure local](/cli/azure/install-azure-cli).

```azurecli-interactive
az extension add --upgrade --name azure-iot
```

## <a name="next-steps"></a>Pasos siguientes

Explore la CLI y su conjunto completo de comandos a través de los documentos de referencia:
* [Referencia del comando az dt](/cli/azure/dt?view=azure-cli-latest&preserve-view=true)