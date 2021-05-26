---
title: 'Creación de un miembro de Azure Blockchain Service: Azure PowerShell'
description: Cree un miembro de Azure Blockchain Service para un consorcio de cadena de bloques mediante Azure PowerShell.
ms.reviewer: ravastra
ms.date: 9/22/2020
ms.topic: quickstart
ms.custom:
- references_regions
- devx-track-azurepowershell
- mode-api
ms.openlocfilehash: 82451b94c83c0883beed66fac1f8e1b6b7132d3a
ms.sourcegitcommit: 32ee8da1440a2d81c49ff25c5922f786e85109b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "110094211"
---
# <a name="quickstart-create-an-azure-blockchain-service-blockchain-member-using-azure-powershell"></a>Inicio rápido: Creación de un miembro de cadena de bloques de Azure Blockchain Service mediante Azure PowerShell

En este inicio rápido, implementará un miembro y un consorcio de cadena de bloques nuevos en Azure Blockchain Service mediante Azure PowerShell.

[!INCLUDE [Retirement note](./includes/retirement.md)]

## <a name="prerequisites"></a>Requisitos previos

Si no tiene una suscripción a Azure, cree una cuenta [gratuita](https://azure.microsoft.com/free/) antes de empezar.

Si decide usar PowerShell de forma local, para este artículo es preciso que instale el módulo Az PowerShell y que se conecte a su cuenta de Azure con el cmdlet [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount). Para más información sobre cómo instalar el módulo Az PowerShell, consulte [Instalación de Azure PowerShell](/powershell/azure/install-az-ps).

> [!IMPORTANT]
> Aunque el módulo **Az.Blockchain** de PowerShell se encuentra en versión preliminar, se debe instalar aparte del módulo Az de PowerShell mediante el cmdlet `Install-Module`. Una vez que este módulo de PowerShell esté disponible con carácter general, formará parte de las futuras versiones del módulo Az PowerShell y estará disponible de forma nativa en Azure Cloud Shell.

```azurepowershell-interactive
Install-Module -Name Az.Blockchain
```

[!INCLUDE [cloud-shell-try-it](../../../includes/cloud-shell-try-it.md)]

## <a name="register-resource-provider"></a>Registro del proveedor de recursos

Si esta es la primera vez que usa el servicio Azure Blockchain, debe registrar el proveedor de recursos **Microsoft.Blockchain**.

```azurepowershell-interactive
Register-AzResourceProvider -ProviderNamespace Microsoft.Blockchain
```

## <a name="choose-a-specific-azure-subscription"></a>Elección de una suscripción de Azure específica

Si tiene varias suscripciones a Azure, elija la suscripción adecuada en la que se debe facturar el recurso. Seleccione una suscripción específica con el cmdlet [Set-AzContext](/powershell/module/az.accounts/set-azcontext).

```azurepowershell-interactive
Set-AzContext -SubscriptionId 00000000-0000-0000-0000-000000000000
```

## <a name="define-variables"></a>Definición de variables

Usará varias partes de la información de forma repetida. Cree variables para almacenar la información.

```azurepowershell-interactive
# Name of resource group used throughout this article
$resourceGroupName = 'myResourceGroup'

# Azure region
$location = 'eastus'
```

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Cree un [grupo de recursos de Azure](../../azure-resource-manager/management/overview.md) con el cmdlet [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). Un grupo de recursos es un contenedor lógico en el que se implementan y se administran recursos de Azure como un grupo.

En el ejemplo siguiente se crea un grupo de recursos basado en el nombre de la variable `$resourceGroupName` en la región especificada en la variable `$location`.

```azurepowershell-interactive
New-AzResourceGroup -Name $resourceGroupName -Location $location
```

## <a name="create-a-blockchain-member"></a>Creación de un miembro de cadena de bloques

Un miembro de Azure Blockchain Service es un nodo de la cadena de bloques en una red de cadenas de bloques del consorcio privado.
Al aprovisionar un miembro, puede crear una red de consorcio o unirse a ella. Necesita al menos un miembro para una red de consorcio. El número de miembros de la cadena de bloques que necesitan los participantes depende de su escenario. Los participantes del consorcio pueden tener uno o varios miembros de la cadena de bloques o pueden compartir miembros con otros participantes. Para más información sobre los consorcios, consulte [Consorcio de Azure Blockchain Service](consortium.md).

Hay varios parámetros y propiedades que se deben pasar. Reemplace los parámetros de ejemplo por los suyos propios.

```azurepowershell-interactive
$passwd = Read-Host -Prompt 'Enter the members default transaction node password' -AsSecureString
$csPasswd = Read-Host -Prompt 'Enter  the consortium account password' -AsSecureString

$memberParams = @{
  Name = 'myblockchainmember'
  ResourceGroupName = $resourceGroupName
  Consortium = 'myconsortium'
  ConsortiumManagementAccountPassword = $csPasswd
  Location = $location
  Password = $passwd
  Protocol = 'Quorum'
  Sku = 'S0'
}
New-AzBlockchainMember @memberParams
```

| Parámetro | Descripción |
|---------|-------------|
| **ResourceGroupName** | Nombre del grupo de recursos donde se crean los recursos de Azure Blockchain Service. Use el grupo de recursos que creó en la sección anterior.
| **Nombre** | Un nombre único que identifica al miembro de cadena de bloques de Azure Blockchain Service. El nombre se usa como dirección de punto de conexión público. Por ejemplo, `myblockchainmember.blockchain.azure.com`.
| **Ubicación** | Región de Azure donde se crea el miembro de cadena de bloques. Por ejemplo, `westus2`. Elija la ubicación más cercana a los usuarios o a sus otras aplicaciones de Azure. Es posible que las características no estén disponibles en algunas regiones. Azure Blockchain Data Manager está disponible en las siguientes regiones de Azure: Este de EE. UU. y Oeste de Europa.
| **Contraseña** | La contraseña del nodo de transacción predeterminado del miembro. Use la contraseña para la autenticación básica al conectarse al punto de conexión público del nodo de transacción predeterminado del miembro de la cadena de bloques.
| **Protocolo** | Protocolo de la cadena de bloques. Actualmente, se admite el protocolo _Quorum_.
| **Consorcio** | Nombre del consorcio al que unirse o que se va a crear. Para más información sobre los consorcios, consulte [Consorcio de Azure Blockchain Service](consortium.md).
| **ConsortiumManagementAccountPassword** | Este valor también se conoce como la contraseña de la cuenta del miembro, y se usa para cifrar la clave privada de la cuenta de Ethereum que se crea para el miembro. Usará la cuenta del miembro y la contraseña de la cuenta del miembro para la administración del consorcio.
| **Sku** | Tipo de nivel. **S0** para estándar o **B0** para básico. Use el nivel _Básico_ para desarrollo, pruebas y prueba de concepto. Use el nivel _Estándar_ para las implementaciones de nivel de producción. También debe usar el nivel _Estándar_ si va a utilizar Blockchain Data Manager o va a enviar un gran volumen de transacciones privadas. Después de la creación de un miembro, no se puede cambiar el plan de tarifa de Básico a Estándar.

La creación del miembro de cadena de bloques y de los recursos auxiliares tarda unos 10 minutos.

## <a name="clean-up-resources"></a>Limpieza de recursos

Puede usar el miembro de cadena de bloques que ha creado en el siguiente inicio rápido o tutorial. Cuando ya no necesite los recursos, puede eliminarlos; para ello, elimine el grupo de recursos `myResourceGroup` que creó para el inicio rápido.

> [!CAUTION]
> En el ejemplo siguiente se elimina el grupo de recursos especificado y todos los recursos que contiene.
> Si los recursos que están fuera del ámbito de este artículo existen en el grupo de recursos especificado, también se eliminarán.

```azurepowershell-interactive
Remove-AzResourceGroup -Name $resourceGroupName
```

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido ha implementado un miembro de Azure Blockchain Service y un nuevo consorcio. Pruebe el siguiente inicio rápido para usar Azure Blockchain Development Kit para Ethereum para adjuntar a un miembro de Azure Blockchain Service.

> [!div class="nextstepaction"]
> [Uso de Visual Studio Code para conectarse a Azure Blockchain Service](connect-vscode.md)
