---
title: Introducción y comandos de la CLI de Bicep
description: Describe los comandos que puede usar en la CLI de Bicep. Estos comandos incluyen la creación de plantillas de Azure Resource Manager desde Bicep.
ms.topic: conceptual
ms.date: 06/01/2021
ms.openlocfilehash: dd1f292d4ce60353d2f8cecaaa83e38b26bdfaa3
ms.sourcegitcommit: bd65925eb409d0c516c48494c5b97960949aee05
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/06/2021
ms.locfileid: "111540819"
---
# <a name="bicep-cli-commands"></a>Comandos de la CLI de Bicep

En este artículo se describen los comandos que puede usar en la CLI de Bicep. Para ejecutar los comandos, debe tener [instalada la CLI de Bicep](./install.md).

En este artículo se muestra cómo ejecutar los comandos en la CLI de Azure. Si no usa la CLI de Azure, ejecute los comandos sin `az` al principio del comando. Por ejemplo, `az bicep version` se convierte en ``bicep version``.

## <a name="build"></a>build

El comando `build` convierte un archivo de Bicep en una plantilla de Azure Resource Manager (plantilla de ARM). Normalmente, no es necesario ejecutar este comando, ya que se ejecuta automáticamente al implementar un archivo de Bicep. Ejecútelo manualmente cuando quiera ver el JSON de la plantilla de ARM que se crea a partir del archivo de Bicep.

En el ejemplo siguiente se convierte un archivo de Bicep denominado _main.bicep_ en una plantilla de ARM denominada _main.json_. El nuevo archivo se crea en el mismo directorio que el archivo de Bicep.

```azurecli
az bicep build --file main.bicep
```

En el ejemplo siguiente se guarda _main.json_ en otro directorio.

```azurecli
 az bicep build --file main.bicep --outdir c:\jsontemplates
```

En el ejemplo siguiente se especifica el nombre y la ubicación del archivo que se va a crear.

```azurecli
az bicep build --file main.bicep --outfile c:\jsontemplates\azuredeploy.json
```

Para imprimir el archivo en `stdout`, use:

```azurecli
az bicep build --file main.bicep --stdout
```

## <a name="decompile"></a>decompile

El comando `decompile` convierte el JSON de la plantilla de ARM en un archivo de Bicep.

```azurecli
az bicep decompile --file main.json
```

Para más información sobre el uso de este comando, consulte [Descompilación de JSON de plantilla de ARM en Bicep](decompile.md).

## <a name="install"></a>instalar

El comando `install` agrega la CLI de Bicep al entorno local. Para más información, consulte [Instalación de herramientas de Bicep](install.md).

Para instalar la versión más reciente, use:

```azurecli
az bicep install
```

Para instalar una versión específica:

```azurecli
az bicep install --version v0.3.255
```

## <a name="list-versions"></a>list-versions

El comando `list-versions` devuelve todas las versiones disponibles de la CLI de Bicep. Use este comando para ver si quiere [actualizar](#upgrade) o [instalar](#install) una nueva versión.

```azurecli
az bicep list-versions
```

El comando devuelve una matriz de versiones disponibles.

```azurecli
[
  "v0.4.1",
  "v0.3.539",
  "v0.3.255",
  "v0.3.126",
  "v0.3.1",
  "v0.2.328",
  "v0.2.317",
  "v0.2.212",
  "v0.2.59",
  "v0.2.14",
  "v0.2.3",
  "v0.1.226-alpha",
  "v0.1.223-alpha",
  "v0.1.37-alpha",
  "v0.1.1-alpha"
]
```

## <a name="upgrade"></a>upgrade

El comando `upgrade` actualiza la versión instalada a la versión más reciente.

```azurecli
az bicep upgrade
```

## <a name="version"></a>version

El comando `version` devuelve la versión instalada.

```azurecli
az bicep version
```

El comando muestra el número de versión.

```azurecli
Bicep CLI version 0.4.1 (e2387595c9)
```

Si no ha instalado la CLI de Bicep, verá un error que indica que no se encontró la CLI de Bicep.

## <a name="next-steps"></a>Pasos siguientes

Para información sobre la implementación de un archivo de Bicep, consulte:

* [CLI de Azure](deploy-cli.md)
* [Cloud Shell](deploy-cloud-shell.md)
* [PowerShell](deploy-powershell.md)
