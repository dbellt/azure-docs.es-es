---
title: Configuración de entornos de desarrollo e implementación de Bicep
description: Configuración de entornos de desarrollo e implementación de Bicep
ms.topic: conceptual
ms.date: 03/26/2021
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 08880e1f41ef3fdaacc119cd6c4c1ed191425285
ms.sourcegitcommit: 52491b361b1cd51c4785c91e6f4acb2f3c76f0d5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108316522"
---
# <a name="install-bicep-preview"></a>Instalación de Bicep (versión preliminar)

Aprenda a configurar entornos de desarrollo e implementación de Bicep.

## <a name="development-environment"></a>Entorno de desarrollo

Para disfrutar de la mejor experiencia de creación de Bicep, necesitará dos componentes:

- **Extensión Bicep para Visual Studio Code**. Para crear archivos de Bicep, necesitará un buen editor de Bicep. Le recomendamos [Visual Studio Code](https://code.visualstudio.com/) con la [extensión Bicep](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-bicep). Estas herramientas proporcionan compatibilidad con el lenguaje y la función de autocompletar de los recursos. Facilitan la creación y validación de archivos de Bicep. Para obtener más información sobre el uso de Visual Studio Code y la extensión Bicep, consulte [Inicio rápido: Creación de plantillas de archivos de Bicep con Visual Studio Code](./quickstart-create-bicep-use-visual-studio-code.md).
- **CLI de BICEP**. Use la CLI de Bicep para compilar archivos de Bicep en plantillas JSON de ARM y descompilar este tipo de plantillas en archivos de Bicep. Para obtener instrucciones sobre la instalación, consulte [Instalación de la CLI de Bicep](#install-manually).

## <a name="deployment-environment"></a>Entorno de implementación

Para implementar archivos de Bicep locales, necesita dos componentes:

- **La versión 2.20.0 o posterior de la CLI de Azure o la versión 5.6.0 o posterior de Azure PowerShell**. Para obtener instrucciones de instalación, consulte:

  - [Azure PowerShell](/powershell/azure/install-az-ps)
  - [Instalación de la CLI de Azure en Windows](/cli/azure/install-azure-cli-windows)
  - [Instalación de la CLI de Azure en Linux](/cli/azure/install-azure-cli-linux)
  - [Instalación de la CLI de Azure en macOS](/cli/azure/install-azure-cli-macos)

  > [!NOTE]
  > Actualmente, tanto CLI de Azure como Azure PowerShell solo pueden implementar archivos de Bicep locales. Para más información sobre cómo implementar archivos de Bicep mediante la CLI de Azure, consulte [Implementación: CLI](./deploy-cli.md#deploy-remote-template). Para más información sobre cómo implementar archivos de Bicep mediante Azure PowerShell, consulte [Implementación: PowerShell]( ./deploy-powershell.md#deploy-remote-template).

- **CLI de BICEP**. La CLI de Bicep es necesaria para compilar archivos de Bicep en plantillas JSON antes de la implementación. Para obtener instrucciones sobre la instalación, consulte [Instalación de la CLI de Bicep](#install-bicep-cli).

Una vez instalados los componentes, puede implementar un archivo de Bicep con:

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
New-AzResourceGroupDeployment `
  -Name ExampleDeployment `
  -ResourceGroupName ExampleGroup `
  -TemplateFile <path-to-template-or-bicep> `
  -storageAccountType Standard_GRS
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli-interactive
az deployment group create \
  --name ExampleDeployment \
  --resource-group ExampleGroup \
  --template-file <path-to-template-or-bicep> \
  --parameters storageAccountType=Standard_GRS
```

---

## <a name="install-bicep-cli"></a>Instalación de la CLI de Bicep

- Para usar la CLI de Bicep para compilar y descompilar archivos de Bicep, consulte [Instalación manual](#install-manually).
- Para usar la CLI de Azure para implementar archivos de Bicep, consulte [Uso con la CLI de Azure](#use-with-azure-cli).
- Para usar Azure PowerShell para implementar archivos de Bicep, consulte [Uso con Azure PowerShell](#use-with-azure-powershell).

### <a name="use-with-azure-cli"></a>Uso con la CLI de Azure

Con la versión 2.20.0 o posterior de la CLI de Azure instalada, la CLI de Bicep se instala automáticamente cuando se ejecuta un comando que depende de ella. Por ejemplo:

```azurecli
az deployment group create --template-file azuredeploy.bicep --resource-group myResourceGroup
```

or

```azurecli
az bicep ...
```

También puede instalar manualmente la CLI con los comandos integrados:

```bash
az bicep install
```

Para actualizar a la versión más reciente:

```bash
az bicep upgrade
```

Para instalar una versión específica:

```bash
az bicep install --version v0.3.255
```

> [!IMPORTANT]
> La CLI de Azure instala una versión independiente de la CLI de Bicep que no entra en conflicto con ninguna otra instalación de Bicep que haya realizado. Además, la CLI de Azure no agrega Bicep a la variable PATH. Para usar la CLI de Bicep para compilar o descompilar archivos de Bicep, o para usar Azure PowerShell para implementar archivos de Bicep, consulte [Instalación manual](#install-manually) o [Uso con Azure PowerShell](#use-with-azure-powershell).

Para enumerar todas las versiones disponibles de la CLI de Bicep:

```bash
az bicep list-versions
```

Para mostrar las versiones instaladas:

```bash
az bicep version
```

### <a name="use-with-azure-powershell"></a>Uso con Azure PowerShell

Azure PowerShell todavía no tiene la capacidad de instalar la CLI de Bicep. Azure PowerShell (versión 5.6.0 o posterior) espera que la CLI de Bicep ya esté instalada y disponible en la variable PATH. Siga uno de los [métodos de instalación manual](#install-manually).

Para implementar archivos de Bicep, se requiere la versión 0.3.1 o posterior de la CLI de Bicep. Para comprobar la versión de la CLI de Bicep:

```cmd
bicep --version
```

> [!IMPORTANT]
> La CLI de Azure instala su propia versión independiente de la CLI de Bicep. Aunque tenga instaladas las versiones necesarias para la CLI de Azure, se produce un error en la implementación de Azure PowerShell.

Una vez que se instala la CLI de Bicep, se la llamará cada vez que sea necesaria para un cmdlet de implementación. Por ejemplo:

```azurepowershell
New-AzResourceGroupDeployment -ResourceGroupName myResourceGroup -TemplateFile azuredeploy.bicep
```

### <a name="install-manually"></a>Instalación manual

Los métodos siguientes instalan la CLI de Bicep y la agregan a la variable PATH.

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

#### <a name="macos"></a>macOS

##### <a name="via-homebrew"></a>Mediante Homebrew

```sh
# Add the tap for bicep
brew tap azure/bicep

# Install the tool
brew install bicep
```

##### <a name="macos-manual-install"></a>Instalación manual de macOS

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

## <a name="install-the-nightly-builds"></a>Instalación de las compilaciones nocturnas

Si desea probar las compilaciones más recientes de Bicep antes de su lanzamiento, consulte el artículo sobre la [instalación de compilaciones nocturnas](https://github.com/Azure/bicep/blob/main/docs/installing-nightly.md).

> [!WARNING]
> Estas compilaciones previas al lanzamiento tienen muchas más probabilidades de incluir errores conocidos o desconocidos.

## <a name="next-steps"></a>Pasos siguientes

Introducción al [inicio rápido de Bicep](./quickstart-create-bicep-use-visual-studio-code.md).
