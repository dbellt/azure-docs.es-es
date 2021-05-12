---
title: Automatización de la instalación del entorno de ejecución de integración autohospedado mediante scripts locales de PowerShell
description: Para automatizar la instalación del entorno de ejecución de integración autohospedado en máquinas locales.
ms.service: data-factory
ms.topic: conceptual
author: lrtoyou1223
ms.author: lle
ms.custom: seo-lt-2019
ms.date: 05/09/2020
ms.openlocfilehash: 54699a5641b98cde5bbff1f55add8b8345d6a1ac
ms.sourcegitcommit: 1fbd591a67e6422edb6de8fc901ac7063172f49e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109488292"
---
# <a name="automating-self-hosted-integration-runtime-installation-using-local-powershell-scripts"></a>Automatización de la instalación del entorno de ejecución de integración autohospedado mediante scripts locales de PowerShell
Para automatizar la instalación del entorno de ejecución de integración autohospedado en máquinas locales (que no sean VM de Azure en las que podamos aprovechar la plantilla de Resource Manager en su lugar), puede usar scripts locales de PowerShell. En este artículo se presentan dos scripts que puede usar.

## <a name="prerequisites"></a>Prerrequisitos

* Inicie PowerShell en la máquina local. Para ejecutar los scripts, debe elegir **Ejecutar como administrador**.
* [Descargue](https://www.microsoft.com/download/details.aspx?id=39717) el software del entorno de ejecución de integración autohospedado. Copie la ruta de acceso donde se encuentra el archivo descargado. 
* También necesita una **clave de autenticación** para registrar el entorno de ejecución de integración autohospedado.
* Para automatizar las actualizaciones manuales, debe tener un entorno de ejecución de integración autohospedado preconfigurado.

## <a name="scripts-introduction"></a>Introducción a los scripts 

> [!NOTE]
> Estos scripts se crean mediante la [utilidad de línea de comandos documentada](./create-self-hosted-integration-runtime.md#set-up-an-existing-self-hosted-ir-via-local-powershell) en el entorno de ejecución de integración autohospedado. Si es necesario, puede personalizar estos scripts como corresponda para adaptarlos a sus necesidades de automatización.
> Los scripts deben aplicarse por nodo, por lo que debe asegurarse de ejecutarlo en todos los nodos en caso de una configuración de alta disponibilidad (2 o más nodos).

* Para automatizar la configuración: instale y registre un nuevo nodo de entorno de ejecución de integración autohospedado mediante **[InstallGatewayOnLocalMachine.ps1](https://github.com/nabhishek/SelfHosted-IntegrationRuntime_AutomationScripts/blob/master/InstallGatewayOnLocalMachine.ps1)** : el script se puede usar para instalar el nodo del entorno de ejecución de integración autohospedado y registrarlo con una clave de autenticación. El script acepta dos argumentos: el **primero** especifica la ubicación del [entorno de ejecución de integración autohospedado](https://www.microsoft.com/download/details.aspx?id=39717) en un disco local y el **segundo** especifica la **clave de autenticación** (para registrar el nodo de IR autohospedado).

* Para automatizar las actualizaciones manuales: actualice el nodo de IR autohospedado con una versión específica o a la versión más reciente **[script-update-gateway.ps1](https://github.com/nabhishek/SelfHosted-IntegrationRuntime_AutomationScripts/blob/master/script-update-gateway.ps1)** . Esto también se admite en caso de que haya desactivado la actualización automática o quiera tener más control sobre las actualizaciones. El script se puede usar para actualizar el nodo del entorno de ejecución de integración autohospedado a la versión más reciente o a una versión superior especificada (el cambio a una versión anterior no funciona). Acepta un argumento para especificar el número de versión (ejemplo: -version 3.13.6942.1). Cuando no se especifica ninguna versión, siempre actualiza la instancia de IR autohospedada a la última versión que se encuentra en las [descargas](https://www.microsoft.com/download/details.aspx?id=39717).
    > [!NOTE]
    > Solo se pueden especificar las tres últimas versiones. Idealmente, esto se usa para actualizar un nodo existente a la versión más reciente. **SE ASUME QUE TIENE UN IR AUTOHOSPEDADO REGISTRADO**. 

## <a name="usage-examples"></a>Ejemplos de uso

### <a name="for-automating-setup"></a>Para automatizar la configuración
1. Descargue el IR autohospedado desde [aquí](https://www.microsoft.com/download/details.aspx?id=39717). 
1. Especifique la ruta de acceso donde se encuentra el SHIR MSI descargado (archivo de instalación). Por ejemplo, si la ruta de acceso es *C:\Users\username\Downloads\IntegrationRuntime_4.7.7368.1.msi*, puede usar el siguiente ejemplo de línea de comandos de PowerShell para esta tarea:

   ```powershell
   PS C:\windows\system32> C:\Users\username\Desktop\InstallGatewayOnLocalMachine.ps1 -path "C:\Users\username\Downloads\IntegrationRuntime_4.7.7368.1.msi" -authKey "[key]"
   ```

    > [!NOTE]
    > Reemplace [key] por la clave de autenticación para registrar el IR.
    > Reemplace "username" por su nombre de usuario.
    > Especifique la ubicación del archivo "InstallGatewayOnLocalMachine.ps1" al ejecutar el script. En este ejemplo, se ha almacenado en el escritorio.

1. Si hay una instancia de IR autohospedado preinstalado en la máquina, el script la desinstalará automáticamente y, a continuación, configurará una nueva. Verá la ventana siguiente: ![configurar entorno de ejecución de integración](media/self-hosted-integration-runtime-automation-scripts/integration-runtime-configure.png)

1. Cuando se complete la instalación y el registro de claves, verá los resultados *Succeed to install gateway* (Puerta de enlace instalada correctamente) y *Succeed to register gateway* (Puerta de enlace registrada correctamente) en la instancia local de PowerShell.
        [![resultado de la ejecución del script 1](media/self-hosted-integration-runtime-automation-scripts/script-1-run-result.png)](media/self-hosted-integration-runtime-automation-scripts/script-1-run-result.png#lightbox)

### <a name="for-automating-manual-updates"></a>Para automatizar las actualizaciones manuales
Este script se usa para actualizar o instalar y registrar la versión más reciente del entorno de ejecución de integración autohospedado. La ejecución del script sigue los pasos que se describen a continuación:
1. Compruebe la versión actual del IR autohospedado.
2. Obtenga la versión más reciente o la versión especificada del argumento.
3. Si hay una versión más reciente que la actual:
    * descargue el archivo MSI de IR autohospedado;
    * actualícelo.

Puede seguir el siguiente ejemplo de línea de comandos para usar este script:
* Descargue e instale la versión más reciente de la puerta de enlace:

   ```powershell
   PS C:\windows\system32> C:\Users\username\Desktop\script-update-gateway.ps1
   ```    
* Descargue e instale la puerta de enlace de la versión especificada:
   ```powershell
   PS C:\windows\system32> C:\Users\username\Desktop\script-update-gateway.ps1 -version 3.13.6942.1
   ``` 
   Si la versión actual ya es la más reciente, verá el resultado siguiente, lo que sugiere que no se requiere ninguna actualización.   
    [![resultado de la ejecución del script 2](media/self-hosted-integration-runtime-automation-scripts/script-2-run-result.png)](media/self-hosted-integration-runtime-automation-scripts/script-2-run-result.png#lightbox)