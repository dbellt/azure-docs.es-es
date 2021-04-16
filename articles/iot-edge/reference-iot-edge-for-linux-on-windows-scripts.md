---
title: Script de PowerShell para Azure IoT Edge para Linux en Windows | Microsoft Docs
description: Información de referencia de scripts de PowerShell para Azure IoT Edge para Linux en Windows para implementar, aprovisionar y obtener el estado de máquinas virtuales de IoT Edge para Linux en Windows.
author: v-tcassi
manager: philmea
ms.author: v-tcassi
ms.date: 02/16/2021
ms.topic: reference
ms.service: iot-edge
services: iot-edge
monikerRange: =iotedge-2018-06
ms.openlocfilehash: a24b39107d8f78c049afa230fe678ec92852eeb0
ms.sourcegitcommit: edc7dc50c4f5550d9776a4c42167a872032a4151
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "105959694"
---
# <a name="powershell-scripts-for-iot-edge-for-linux-on-windows"></a>Scripts de PowerShell para IoT Edge para Linux en Windows

[!INCLUDE [iot-edge-version-201806](../../includes/iot-edge-version-201806.md)]

Conozca los scripts de PowerShell que implementan, aprovisionan y obtienen el estado de máquinas virtuales de IoT Edge para Linux en Windows.

Los comandos descritos en este artículo proceden del archivo `AzureEFLOW.psm1`, que puede encontrar en su sistema en el directorio `WindowsPowerShell`, en `C:\Program Files\WindowsPowerShellModules\AzureEFLOW`.

## <a name="deploy-eflow"></a>Deploy-Eflow

El comando **Deploy-Eflow** es el método de implementación principal. El comando de implementación crea la máquina virtual, aprovisiona archivos e implementa el módulo de agente de IoT Edge. Aunque ninguno de los parámetros es obligatorio, se pueden usar para aprovisionar el dispositivo IoT Edge durante la implementación y para modificar la configuración de la máquina virtual durante la creación. Para obtener una lista completa, use el comando `Get-Help Deploy-Eflow -full`.  

>[!NOTE]
>En el caso del tipo de aprovisionamiento, **X509** actualmente hace referencia exclusivamente al aprovisionamiento X509 mediante [Azure IoT Hub Device Provisioning Service](../iot-dps/about-iot-dps.md). Actualmente no se admite el método de aprovisionamiento X509 manual.

| Parámetro | Valores aceptados | Comentarios |
| --------- | --------------- | -------- |
| eflowVhdxDir | Ruta de acceso del directorio | Directorio donde la implementación almacena el archivo VHDX para la VM. |
| provisioningType | **manual**, **TPM**, **X509** o **symmetric** |  Define el tipo de aprovisionamiento que quiere usar para el dispositivo IoT Edge. |
| devConnString | Cadena de conexión de dispositivo de un dispositivo IoT Edge existente. | Cadena de conexión de dispositivo para el aprovisionamiento manual de un dispositivo IoT Edge (**manual**). |
| symmKey | Clave principal de una inscripción de DPS existente o clave principal de un dispositivo IoT Edge existente registrado mediante claves simétricas. | Clave simétrica para el aprovisionamiento de un dispositivo IoT Edge (**X509** o **symmetric**). |
| scopeId | Identificador de ámbito de una instancia de DPS existente. | Identificador de ámbito para el aprovisionamiento de un dispositivo IoT Edge (**X509** o **symmetric**). |
| registrationId | Identificador de registro de un dispositivo de IoT Edge existente. | Identificador de registro para el aprovisionamiento de un dispositivo IoT Edge (**X509** o **symmetric**). |
| identityCertLocVm | Ruta de acceso del directorio; debe estar en una carpeta que pueda ser propiedad del servicio `iotedge`. | Ruta de acceso de destino absoluta del certificado de identidad de la máquina virtual para el aprovisionamiento de un dispositivo IoT Edge (**X509** o **symmetric**). |
| identityCertLocWin | Ruta de acceso del directorio | Ruta de acceso de origen absoluta del certificado de identidad en Windows para el aprovisionamiento de un dispositivo IoT Edge (**X509** o **symmetric**). |
| identityPkLocVm | Ruta de acceso del directorio; debe estar en una carpeta que pueda ser propiedad del servicio `iotedge`. | Ruta de acceso de destino absoluta de la clave privada de identidad de la máquina virtual para el aprovisionamiento de un dispositivo IoT Edge (**X509** o **symmetric**). |
| identityPkLocWin | Ruta de acceso del directorio | Ruta de acceso de origen absoluta de la clave privada de identidad en Windows para el aprovisionamiento de un dispositivo IoT Edge (**X509** o **symmetric**). |
| vmSizeDefintion | No más de 30 caracteres. | Definición del número de núcleos y RAM disponible para la máquina virtual. **Valor predeterminado**: Standard_K8S_v1. |
| vmDiskSize | Entre 8 y 256 GB. | Tamaño de disco máximo del disco duro virtual de expansión dinámica. **Valor predeterminado**: 16 GB. |
| vmUser | No más de 30 caracteres. | Nombre de usuario para iniciar sesión en la máquina virtual. |
| vnetType | **Transparent** o **ICS**. | Tipo de conmutador virtual. **Valor predeterminado**: Transparent. Transparent hace referencia a un conmutador externo, mientras que ICS hace referencia a un conmutador interno. |
| vnetName | No más de 64 caracteres. | Nombre del conmutador virtual. **Valor predeterminado**: External. |
| enableVtpm | None | **Parámetro de modificador**. Cree la máquina virtual con TPM habilitado o deshabilitado. |
| mobyPackageVersion | No más de 30 caracteres. |  Versión del paquete de Moby que se verificará o instalará en la máquina virtual.  **Valor predeterminado:** 19.03.11. |
| iotedgePackageVersion | No más de 30 caracteres. | Versión del paquete de IoT Edge que se verificará o instalará en la máquina virtual. **Valor predeterminado:** 1.1.0. |
| installPackages | None | **Parámetro de modificador**. Cuando se alterna, el script intentará instalar los paquetes de Moby y IoT Edge en lugar de solo verificar que los paquetes estén presentes. |

>[!NOTE]
>De forma predeterminada, si el proceso no encuentra un conmutador externo con el nombre `External`, buscará cualquier conmutador externo existente a través del cual obtener una dirección IP. Si no hay ningún conmutador externo disponible, buscará un conmutador interno. Si en cambio no hay ningún conmutador interno disponible, intentará crear el conmutador predeterminado a través del cual se obtendrá una dirección IP.

## <a name="verify-eflowvm"></a>Verify-EflowVm

El comando **Verify-EflowVm** es una función expuesta para comprobar que se ha creado la máquina virtual de IOT Edge para Linux en Windows. Solo toma parámetros comunes y devolverá **true** si se creó la máquina virtual, y **false** en caso contrario. Para obtener información adicional, use el comando `Get-Help Verify-EflowVm -full`.

## <a name="provision-eflowvm"></a>Provision-EflowVm

El comando **Provision-EflowVm** agrega la información de aprovisionamiento del dispositivo IoT Edge al archivo `config.yaml` de IOT Edge de la máquina virtual. El aprovisionamiento también puede realizarse durante la fase de implementación si se establecen los parámetros en el comando **Deploy-Eflow**. Para obtener información adicional, use el comando `Get-Help Provision-EflowVm -full`.

| Parámetro | Valores aceptados | Comentarios |
| --------- | --------------- | -------- |
| vmUser | No más de 30 caracteres. | Nombre de usuario para iniciar sesión en la máquina virtual. |
| provisioningType | **manual**, **TPM**, **X509** o **symmetric** |  Define el tipo de aprovisionamiento que quiere usar para el dispositivo IoT Edge. |
| devConnString | Cadena de conexión de dispositivo de un dispositivo IoT Edge existente. | Cadena de conexión de dispositivo para el aprovisionamiento manual de un dispositivo IoT Edge (**manual**). |
| symmKey | Clave principal de una inscripción de DPS existente o clave principal de un dispositivo IoT Edge existente registrado mediante claves simétricas. | Clave simétrica para el aprovisionamiento de un dispositivo IoT Edge (**DPS** o **symmetric**). |
| scopeId | Identificador de ámbito de una instancia de DPS existente. | Identificador de ámbito para el aprovisionamiento de un dispositivo IoT Edge (**DPS**). |
| registrationId | Identificador de registro de un dispositivo de IoT Edge existente. | Identificador de registro para el aprovisionamiento de un dispositivo IoT Edge (**DPS**). |
| identityCertLocVm | Ruta de acceso del directorio; debe estar en una carpeta que pueda ser propiedad del servicio `iotedge`. | Ruta de acceso de destino absoluta del certificado de identidad de la máquina virtual para el aprovisionamiento de un dispositivo IoT Edge (**DPS** o **X509**). |
| identityCertLocWin | Ruta de acceso del directorio | Ruta de acceso de origen absoluta del certificado de identidad en Windows para el aprovisionamiento de un dispositivo IoT Edge (**dps** o **X509**). |
| identityPkLocVm  | Ruta de acceso del directorio; debe estar en una carpeta que pueda ser propiedad del servicio `iotedge`. | Ruta de acceso de destino absoluta de la clave privada de identidad de la máquina virtual para el aprovisionamiento de un dispositivo IoT Edge (**DPS** o **X509**). |
| identityPkLocWin | Ruta de acceso del directorio | Ruta de acceso de origen absoluta de la clave privada de identidad en Windows para el aprovisionamiento de un dispositivo IoT Edge (**dps** o **X509**). |

## <a name="get-eflowvmname"></a>Get-EflowVmName

El comando **Get-EflowVmName** se usa para consultar el nombre de host actual de la máquina virtual. Este comando existe para tener en cuenta el hecho de que el nombre de host de Windows puede cambiar con el tiempo. Solo toma parámetros comunes y devuelve el nombre de host actual de la máquina virtual. Para obtener información adicional, use el comando `Get-Help Get-EflowVmName -full`.

## <a name="get-eflowlogs"></a>Get-EflowLogs

El comando **Get-EflowLogs** se usa para recopilar y agrupar los registros de la implementación de IoT Edge para Linux en Windows. Genera los registros agrupados en forma de una carpeta `.zip`. Para obtener información adicional, use el comando `Get-Help Get-EflowLogs -full`.

| Parámetro | Valores aceptados | Comentarios |
| --------- | --------------- | -------- |
| vmUser | No más de 30 caracteres. | Nombre de usuario para iniciar sesión en la máquina virtual. |

## <a name="get-eflowvmtpmprovisioninginfo"></a>Get-EflowVmTpmProvisioningInfo

El comando **Get-EflowVmTpmProvisioningInfo** se usa para recopilar y mostrar la información de aprovisionamiento de vTPM de la máquina virtual. Si la máquina virtual se creó sin vTPM, el comando devolverá que no se pudo encontrar la información de aprovisionamiento de TPM. Para obtener información adicional, use el comando `Get-Help Get-EflowVmTpmProvisioningInfo -full`.

| Parámetro | Valores aceptados | Comentarios |
| --------- | --------------- | -------- |
| vmUser | No más de 30 caracteres. | Nombre de usuario para iniciar sesión en la máquina virtual. |

## <a name="get-eflowvmaddr"></a>Get-EflowVmAddr

El comando **Get-EflowVmAddr** se usa para buscar y mostrar las direcciones IP y MAC de la máquina virtual. Solo toma parámetros comunes. Para obtener información adicional, use el comando `Get-Help Get-EflowVmAddr -full`.

## <a name="get-eflowvmsysteminformation"></a>Get-EflowVmSystemInformation

El comando **Get-EflowVmSystemInformation** se usa para recopilar y mostrar información del sistema de la máquina virtual, como el uso de memoria y almacenamiento. Para obtener información adicional, use el comando `Get-Help Get-EflowVmSystemInformation -full`.

| Parámetro | Valores aceptados | Comentarios |
| --------- | --------------- | -------- |
| vmUser | No más de 30 caracteres. | Nombre de usuario para iniciar sesión en la máquina virtual. |

## <a name="get-eflowvmedgeinformation"></a>Get-EflowVmEdgeInformation

El comando **Get-EflowVmEdgeInformation** se usa para recopilar y mostrar información de IoT Edge a partir de la máquina virtual, como la versión de IoT Edge que ejecuta la máquina virtual. Para obtener información adicional, use el comando `Get-Help Get-EflowVmEdgeInformation -full`.

| Parámetro | Valores aceptados | Comentarios |
| --------- | --------------- | -------- |
| vmUser | No más de 30 caracteres. | Nombre de usuario para iniciar sesión en la máquina virtual. |

## <a name="get-eflowvmedgemodulelist"></a>Get-EflowVmEdgeModuleList

El comando **Get-EflowVmEdgeModuleList** se usa para consultar y mostrar la lista de módulos de IoT Edge que se ejecutan en la máquina virtual. Para obtener información adicional, use el comando `Get-Help Get-EflowVmEdgeModuleList -full`.

| Parámetro | Valores aceptados | Comentarios |
| --------- | --------------- | -------- |
| vmUser | No más de 30 caracteres. | Nombre de usuario para iniciar sesión en la máquina virtual. |

## <a name="get-eflowvmedgestatus"></a>Get-EflowVmEdgeStatus

El comando **Get-EflowVmEdgeStatus** se usa para consultar y mostrar el estado del entorno de ejecución de Azure IoT Edge en la máquina virtual. Para obtener información adicional, use el comando `Get-Help Get-EflowVmEdgeStatus -full`.

| Parámetro | Valores aceptados | Comentarios |
| --------- | --------------- | -------- |
| vmUser | No más de 30 caracteres. | Nombre de usuario para iniciar sesión en la máquina virtual. |

## <a name="get-eflowvmusername"></a>Get-EflowVmUserName

El comando **Get-EflowVmUserName** se usa para consultar y mostrar el nombre de usuario de la máquina virtual, que se usó para crear la máquina virtual a partir del registro. Solo toma parámetros comunes. Para obtener información adicional, use el comando `Get-Help Get-EflowVmUserName -full`.

## <a name="get-eflowvmsshkey"></a>Get-EflowVmSshKey

El comando **Get-EflowVmSshKey** se usa para consultar y mostrar la clave SSH usada por la máquina virtual. Solo toma parámetros comunes. Para obtener información adicional, use el comando `Get-Help Get-EflowVmSshKey -full`.

## <a name="ssh-eflowvm"></a>Ssh-EflowVm

El comando **Ssh-EflowVm** se usa para aplicar SSH en la máquina virtual. La única cuenta que se permite a SSH en la máquina virtual es la del usuario que la creó. Para obtener información adicional, use el comando `Get-Help Ssh-EflowVm -full`.

| Parámetro | Valores aceptados | Comentarios |
| --------- | --------------- | -------- |
| vmUser | No más de 30 caracteres. | Nombre de usuario para iniciar sesión en la máquina virtual. |

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre cómo usar estos comandos en el artículo siguiente:

* [Instalación de Azure IoT Edge para Linux en Windows](./how-to-install-iot-edge-windows-on-windows.md)

* Consulte la [referencia de scripts de PowerShell para IoT Edge para Linux en Windows](reference-iot-edge-for-linux-on-windows-scripts.md#deploy-eflow) para ver todos los comandos disponibles a través de PowerShell.
