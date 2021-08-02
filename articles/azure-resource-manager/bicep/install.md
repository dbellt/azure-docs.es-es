---
title: Configuración de entornos de desarrollo e implementación de Bicep
description: Configuración de entornos de desarrollo e implementación de Bicep
ms.topic: conceptual
ms.date: 06/04/2021
ms.custom: devx-track-azurepowershell, devx-track-azurecli
ms.openlocfilehash: 89212841ca4f17802f6b7288b441a5f22ea2c9d6
ms.sourcegitcommit: bd65925eb409d0c516c48494c5b97960949aee05
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/06/2021
ms.locfileid: "111537062"
---
# <a name="install-bicep-tools"></a>Instalación de herramientas de Bicep

Vamos a asegurarnos de que el entorno esté configurado para desarrollar e implementar archivos de Bicep.

## <a name="development-environment"></a>Entorno de desarrollo

Para crear archivos de Bicep, necesitará un buen editor de Bicep. Es recomendable que:

- **Visual Studio Code**: si aún no tiene Visual Studio Code, [instálelo](https://code.visualstudio.com/).
- **Extensión Bicep para Visual Studio Code**.  Visual Studio Code con la extensión de Bicep proporciona compatibilidad con el lenguaje y la finalización automática de recursos. La extensión le ayuda a crear y validar archivos de Bicep.

  Para instalar la extensión, busque *bicep* en la pestaña **Extensiones** o en [Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-bicep).

  Seleccione **Instalar**.

  :::image type="content" source="./media/install/install-extension.png" alt-text="Instalación de la extensión de Bicep":::

Para comprobar que ha instalado la extensión, abra cualquier archivo con la extensión de archivo `.bicep`. Debería ver que el modo de lenguaje en la esquina inferior derecha cambia a **Bicep**.

:::image type="content" source="./media/install/language-mode.png" alt-text="Modo de lenguaje de Bicep":::

## <a name="deployment-environment"></a>Entorno de implementación

La manera más fácil de obtener los comandos que necesita para implementar un archivo de Bicep es instalar la versión más reciente de la CLI de Azure. También puede usar PowerShell, pero requiere una instalación adicional.

### <a name="azure-cli"></a>Azure CLI

Debe tener instalada la CLI de Azure, versión 2.20.0 o posterior. Para instalar o actualizar la CLI de Azure, consulte:

- [Instalación de la CLI de Azure en Windows](/cli/azure/install-azure-cli-windows)
- [Instalación de la CLI de Azure en Linux](/cli/azure/install-azure-cli-linux)
- [Instalación de la CLI de Azure en macOS](/cli/azure/install-azure-cli-macos)

Para comprobar su versión actual, ejecute:

```azurecli
az --version
```

Ahora tiene todo lo que necesita para [implementar](deploy-cli.md) y [descompilar](decompile.md) archivos de Bicep. Tiene todo porque la CLI de Azure 2.20.0 o posterior instala automáticamente la CLI de Bicep cuando se ejecuta un comando que la necesita.

Para iniciar manualmente la instalación de la CLI de Bicep, use:

```azurecli
az bicep install
```

Para actualizar a la versión más reciente, use:

```azurecli
az bicep upgrade
```

Para más comandos, consulte la [CLI de Bicep](bicep-cli.md).

> [!IMPORTANT]
> La CLI de Azure instala una instancia independiente de la CLI de Bicep. Esta instancia no entra en conflicto con ninguna versión que pueda haber instalado manualmente. La CLI de Azure no agrega la CLI de Bicep a PATH.

### <a name="powershell"></a>PowerShell

Debe tener instalado Azure PowerShell, versión 5.6.0 o posterior. Para actualizar o instalar, consulte [Instalación de Azure PowerShell](/powershell/azure/install-az-ps).

Azure PowerShell no instala automáticamente la CLI de Bicep. En su lugar, tiene que [instalar manualmente la CLI de Bicep](#install-manually).

> [!IMPORTANT]
> La instancia independiente de la CLI de Bicep instalada por CLI de Azure no está disponible para los comandos de PowerShell. Las implementaciones de Azure PowerShell producirán un error si no ha instalado manualmente la CLI de Bicep.

Cuando instale manualmente la CLI de Bicep, ejecute los comandos de Bicep con la sintaxis `bicep`, en lugar de la sintaxis `az bicep` para CLI de Azure.

Para implementar archivos de Bicep, use la CLI de Bicep, versión 0.3.1 o posterior. Para comprobar la versión de la CLI de Bicep, ejecute:

```cmd
bicep --version
```

### <a name="install-manually"></a>Instalación manual

Los métodos siguientes instalan la CLI de Bicep y la agregan a la variable PATH. Tiene que realizar una instalación manual para cualquier uso que no sea la CLI de Azure.

#### <a name="linux"></a>Linux

```sh
# Fetch the latest Bicep CLI binary
curl -Lo bicep https://github.com/Azure/bicep/releases/latest/download/bicep-linux-x64
# Mark it as executable
chmod +x ./bicep
# Add bicep to your PATH (requires admin)
sudo mv ./bicep /usr/local/bin/bicep
# Verify you can now access the 'bicep' command
bicep --help
# Done!
```

> [!NOTE]
> En el caso de distribuciones ligeras de Linux, como [Alpine](https://alpinelinux.org/), use **bicep-linux-musl-x64**, en lugar de **bicep-linux-x64**, en el script anterior.

#### <a name="macos"></a>macOS

##### <a name="via-homebrew"></a>Mediante Homebrew

```sh
# Add the tap for bicep
brew tap azure/bicep

# Install the tool
brew install bicep
```

##### <a name="via-bash"></a>Mediante BASH

```sh
# Fetch the latest Bicep CLI binary
curl -Lo bicep https://github.com/Azure/bicep/releases/latest/download/bicep-osx-x64
# Mark it as executable
chmod +x ./bicep
# Add Gatekeeper exception (requires admin)
sudo spctl --add ./bicep
# Add bicep to your PATH (requires admin)
sudo mv ./bicep /usr/local/bin/bicep
# Verify you can now access the 'bicep' command
bicep --help
# Done!

```

#### <a name="windows"></a>Windows

##### <a name="windows-installer"></a>Windows Installer

Descargue y ejecute la [versión más reciente de Windows Installer](https://github.com/Azure/bicep/releases/latest/download/bicep-setup-win-x64.exe). El instalador no requiere privilegios administrativos. Después de la instalación, la CLI de Bicep se agrega a la variable PATH del usuario. Cierre las ventanas del shell de comandos que estén abiertas y vuelva a abrirlas para que el cambio de la variable PATH surta efecto.

##### <a name="chocolatey"></a>Chocolatey

```powershell
choco install bicep
```

##### <a name="winget"></a>Winget

```powershell
winget install -e --id Microsoft.Bicep
```

##### <a name="manual-with-powershell"></a>Manual con PowerShell

```powershell
# Create the install folder
$installPath = "$env:USERPROFILE\.bicep"
$installDir = New-Item -ItemType Directory -Path $installPath -Force
$installDir.Attributes += 'Hidden'
# Fetch the latest Bicep CLI binary
(New-Object Net.WebClient).DownloadFile("https://github.com/Azure/bicep/releases/latest/download/bicep-win-x64.exe", "$installPath\bicep.exe")
# Add bicep to your PATH
$currentPath = (Get-Item -path "HKCU:\Environment" ).GetValue('Path', '', 'DoNotExpandEnvironmentNames')
if (-not $currentPath.Contains("%USERPROFILE%\.bicep")) { setx PATH ($currentPath + ";%USERPROFILE%\.bicep") }
if (-not $env:path.Contains($installPath)) { $env:path += ";$installPath" }
# Verify you can now access the 'bicep' command.
bicep --help
# Done!
```

### <a name="install-the-nightly-builds"></a>Instalación de las compilaciones nocturnas

Si quiere probar las compilaciones más recientes de Bicep antes de su lanzamiento, consulte [Instalación de compilaciones nocturnas](https://github.com/Azure/bicep/blob/main/docs/installing-nightly.md).

> [!WARNING]
> Estas compilaciones previas al lanzamiento tienen muchas más probabilidades de incluir errores conocidos o desconocidos.

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre el uso de Visual Studio Code y la extensión Bicep, consulte [Inicio rápido: Creación de plantillas de archivos de Bicep con Visual Studio Code](./quickstart-create-bicep-use-visual-studio-code.md).
