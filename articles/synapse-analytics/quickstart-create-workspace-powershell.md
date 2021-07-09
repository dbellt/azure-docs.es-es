---
title: 'Inicio rápido: Creación de un área de trabajo de Synapse mediante Azure PowerShell'
description: Siga los pasos de esta guía para crear un área de trabajo de Azure Synapse mediante Azure PowerShell.
services: synapse-analytics
author: lenadroid
ms.service: synapse-analytics
ms.topic: quickstart
ms.subservice: workspace
ms.date: 10/19/2020
ms.author: alehall
ms.reviewer: jrasnick
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 1b602ecb096293df7071489262175ae4bb7d71b0
ms.sourcegitcommit: 98308c4b775a049a4a035ccf60c8b163f86f04ca
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/30/2021
ms.locfileid: "113113542"
---
# <a name="quickstart-create-an-azure-synapse-workspace-with-azure-powershell"></a>Inicio rápido: Creación de un área de trabajo de Azure Synapse con Azure PowerShell

Azure PowerShell es un conjunto de cmdlets para administrar los recursos de Azure directamente desde PowerShell. Puede utilizarlo en el explorador con Azure Cloud Shell. También puede instalarlo en macOS, Linux o Windows.

En este inicio rápido, aprenderá a crear un área de trabajo de Synapse mediante Azure PowerShell.

Si no tiene una suscripción a Azure, cree una cuenta [gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerrequisitos

- Una [cuenta de almacenamiento de Azure Data Lake Storage Gen2](../storage/common/storage-account-create.md).

    > [!IMPORTANT]
    > El área de trabajo de Azure Synapse debe poder leer y escribir en la cuenta de ADLS Gen2 seleccionada. Para cualquier cuenta de almacenamiento que vincule como cuenta de almacenamiento principal, debe haber habilitado el **espacio de nombres jerárquico** al crearla, como se describe en la sección [Creación de una cuenta de almacenamiento](../storage/common/storage-account-create.md?tabs=azure-powershell#create-a-storage-account).

Si decide usar Cloud Shell, consulte [Introducción a Azure Cloud Shell](../cloud-shell/overview.md) para más información.

### <a name="install-the-azure-powershell-module-locally"></a>Instalación del módulo de Azure PowerShell de forma local

Si decide usar PowerShell de forma local, para este artículo es preciso que instale el módulo Az PowerShell y que se conecte a su cuenta de Azure con el cmdlet [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount). Para más información sobre cómo instalar el módulo Az PowerShell, consulte [Instalación de Azure PowerShell](/powershell/azure/install-az-ps).

Para más información sobre la autenticación con Azure PowerShell, consulte [Inicio de sesión con Azure PowerShell](/powershell/azure/authenticate-azureps).

### <a name="install-the-azure-synapse-powershell-module"></a>Instalación del módulo de PowerShell de Azure Synapse

> [!IMPORTANT]
> Aunque el módulo **Az.HPCCache** de PowerShell está en versión preliminar, se debe instalar por separado mediante el cmdlet `Install-Module`. Una vez que este módulo de PowerShell esté disponible con carácter general, formará parte de las futuras versiones del módulo Az de PowerShell y estará disponible de forma predeterminada en Azure Cloud Shell.

```azurepowershell-interactive
Install-Module -Name Az.Synapse
```

## <a name="create-an-azure-synapse-workspace-using-azure-powershell"></a>Creación de un área de trabajo de Azure Synapse mediante Azure PowerShell

1. Defina las variables de entorno necesarias para crear recursos para el área de trabajo de Azure Synapse.

   |        Nombre de la variable        |                                                 Descripción                                                 |
   | --------------------------- | ----------------------------------------------------------------------------------------------------------- |
   | StorageAccountName          | Asigne un nombre a la cuenta de almacenamiento de ADLS Gen2 existente.                                                           |
   | StorageAccountResourceGroup | Nombre del grupo de recursos de la cuenta de almacenamiento de ADLS Gen2 existente.                                             |
   | FileShareName               | Nombre del sistema de archivos de almacenamiento existente.                                                                  |
   | SynapseResourceGroup        | Elija un nuevo nombre para el grupo de recursos de Azure Synapse.                                                    |
   | Región                      | Elija una de las [regiones de Azure](https://azure.microsoft.com/global-infrastructure/geographies/#overview). |
   | SynapseWorkspaceName        | Elija un nombre único para la nueva área de trabajo de Azure Synapse.                                                  |
   | SqlUser                     | Elija un valor para un nombre de usuario nuevo.                                                                          |
   | SqlPassword                 | Elija una contraseña segura.                                                                                   |
   | ClientIP                    | Dirección IP pública del sistema desde el que se ejecuta PowerShell.                                             |
   |                             |                                                                                                             |

1. Cree un grupo de recursos como contenedor para el área de trabajo de Azure Synapse:

   ```azurepowershell-interactive
   New-AzResourceGroup -Name $SynapseResourceGroup -Location $Region
   ```

1. Cree un área de trabajo de Azure Synapse:

   ```azurepowershell-interactive
   $Cred = New-Object -TypeName System.Management.Automation.PSCredential ($SqlUser, (ConvertTo-SecureString $SqlPassword -AsPlainText -Force))

   $WorkspaceParams = @{
     Name = $SynapseWorkspaceName
     ResourceGroupName = $SynapseResourceGroup
     DefaultDataLakeStorageAccountName = $StorageAccountName
     DefaultDataLakeStorageFilesystem = $FileShareName
     SqlAdministratorLoginCredential = $Cred
     Location = $Region
   }
   New-AzSynapseWorkspace @WorkspaceParams
   ```

1. Obtenga la dirección URL web y de desarrollo del área de trabajo de Azure Synapse:

   ```azurepowershell-interactive
   $WorkspaceWeb = (Get-AzSynapseWorkspace -Name $SynapseWorkspaceName -ResourceGroupName $StorageAccountResourceGroup).ConnectivityEndpoints.web
   $WorkspaceDev = (Get-AzSynapseWorkspace -Name $SynapseWorkspaceName -ResourceGroupName $StorageAccountResourceGroup).ConnectivityEndpoints.dev
   ```

1. Cree una regla de firewall que le permita acceder al área de trabajo de Azure Synapse desde su máquina:

   ```azurepowershell-interactive
   $FirewallParams = @{
     WorkspaceName = $SynapseWorkspaceName
     Name = 'Allow Client IP'
     ResourceGroupName = $StorageAccountResourceGroup
     StartIpAddress = $ClientIP
     EndIpAddress = $ClientIP
   }
   New-AzSynapseFirewallRule @FirewallParams
   ```

1. Abra la dirección URL web del área de trabajo de Azure Synapse almacenada en la variable de entorno `WorkspaceWeb` para acceder al área de trabajo:

   ```azurepowershell-interactive
   Start-Process $WorkspaceWeb
   ```

   ![Web del área de trabajo de Azure Synapse](media/quickstart-create-synapse-workspace-powershell/create-workspace-powershell-1.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

Siga los pasos que se indican a continuación para eliminar el área de trabajo de Azure Synapse.

> [!WARNING]
> Al eliminar un área de trabajo de Azure Synapse, se quitarán también los motores de análisis y los datos almacenados en la base de datos de los grupos de SQL y metadatos de área de trabajo incluidos. Ya no será posible conectarse a los puntos de conexión de SQL o de Apache Spark. Se eliminarán todos los artefactos de código (consultas, cuadernos, definiciones de trabajos y canalizaciones). La eliminación del área de trabajo **no** afectará a los datos de Data Lake Store Gen2 vinculados al área de trabajo.

Si el área de trabajo de Azure Synapse que se creó en este artículo no es necesaria, puede eliminarla ejecutando el siguiente ejemplo.

```azurepowershell-interactive
Remove-AzSynapseWorkspace -Name $SynapseWorkspaceNam -ResourceGroupName $SynapseResourceGroup
```

## <a name="next-steps"></a>Pasos siguientes

A continuación, puede [crear grupos de SQL](quickstart-create-sql-pool-studio.md) o [crear grupos de Apache Spark](quickstart-create-apache-spark-pool-studio.md) para empezar a analizar y explorar los datos.
