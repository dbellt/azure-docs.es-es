---
title: Conexión y administración de un dispositivo Microsoft Azure Stack Edge Pro FPGA mediante la interfaz de Windows PowerShell
description: Describe la conexión y administración de Azure Stack Edge Pro FPGA a través de la interfaz de Windows PowerShell.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 09/30/2020
ms.author: alkohli
ms.openlocfilehash: 09271ddeac59446d134feaa0f2acb6c3e3d9d89f
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110461280"
---
# <a name="manage-an-azure-stack-edge-pro-fpga-device-via-windows-powershell"></a>Administración de un dispositivo Azure Stack Edge Pro con FPGA mediante Windows PowerShell

La solución Azure Stack Edge Pro FPGA permite procesar datos y enviarlos a través de la red a Azure. En este artículo se describen algunas de las tareas de configuración y administración del dispositivo Azure Stack Edge Pro FPGA. Puede usar Azure Portal, la interfaz de usuario web local o la interfaz de Windows PowerShell para administrar su dispositivo.

Este artículo se centra en las tareas que realizará mediante la interfaz de PowerShell. 

Asimismo, en este artículo se incluyen las secciones siguientes:

- Conectarse a la interfaz de PowerShell
- Crear un paquete de soporte
- Carga del certificado
- Restablecer el dispositivo
- Ver la información del dispositivo
- Obtener los registros de proceso
- Supervisar y solucionar problemas de los módulos de proceso

## <a name="connect-to-the-powershell-interface"></a>Conectarse a la interfaz de PowerShell

[!INCLUDE [Connect to admin runspace](../../includes/data-box-edge-gateway-connect-minishell.md)]

## <a name="create-a-support-package"></a>Crear un paquete de soporte

[!INCLUDE [Create a support package](../../includes/data-box-edge-gateway-create-support-package.md)]

## <a name="upload-certificate"></a>Carga del certificado

[!INCLUDE [Upload certificate](../../includes/data-box-edge-gateway-upload-certificate.md)]

También puede cargar certificados de IoT Edge para habilitar una conexión segura entre el dispositivo IoT Edge y los dispositivos de nivel inferior que pueden conectarse a él. Hay tres archivos (formato *.pem*) que debe instalar:

- Certificado de entidad de certificación raíz o el propietario de la entidad de certificación
- Certificado de entidad de certificación de dispositivo
- Clave privada de dispositivo 

En el siguiente ejemplo se muestra el uso de este cmdlet para instalar certificados de IoT Edge:

```
Set-HcsCertificate -Scope IotEdge -RootCACertificateFilePath "\\hcfs\root-ca-cert.pem" -DeviceCertificateFilePath "\\hcfs\device-ca-cert.pem\" -DeviceKeyFilePath "\\hcfs\device-private-key.pem" -Credential "username"
```
Cuando ejecute este cmdlet, se le solicitará que proporcione la contraseña para el recurso compartido de red.

Para obtener más información sobre los certificados, vaya a [Certificados de Azure IoT Edge](../iot-edge/iot-edge-certs.md) o a [Instalación de los certificados en la puerta de enlace](../iot-edge/how-to-create-transparent-gateway.md).

## <a name="view-device-information"></a>Ver la información del dispositivo
 
[!INCLUDE [View device information](../../includes/data-box-edge-gateway-view-device-info.md)]

## <a name="reset-your-device"></a>Restablecer el dispositivo

[!INCLUDE [Reset your device](../../includes/data-box-edge-gateway-deactivate-device.md)]

## <a name="get-compute-logs"></a>Obtener los registros de proceso

Si el rol de proceso está configurado en su dispositivo, también puede obtener los registros del proceso gracias a la interfaz de PowerShell.

1. [Conéctese a la interfaz de PowerShell](#connect-to-the-powershell-interface).
2. Use `Get-AzureDataBoxEdgeComputeRoleLogs` para obtener los registros del proceso del dispositivo.

    En el ejemplo siguiente se muestra el uso de este cmdlet:

    ```powershell
    Get-AzureDataBoxEdgeComputeRoleLogs -Path "\\hcsfs\logs\myacct" -Credential "username" -FullLogCollection
    ```

    Aquí tiene hay una descripción de los parámetros que se usan en el cmdlet:
    - `Path`: proporcione una ruta de red al recurso compartido en el que quiere crear el paquete de registros del proceso.
    - `Credential`: proporcione el nombre de usuario del recurso compartido de red. Cuando ejecute este cmdlet, deberá proporcionar la contraseña del recurso compartido.
    - `FullLogCollection`: este parámetro garantiza que el paquete de registros contendrá todos los registros del proceso. De forma predeterminada, el paquete de registros contiene solo un subconjunto de registros.

## <a name="monitor-and-troubleshoot-compute-modules"></a>Supervisar y solucionar problemas de los módulos de proceso

[!INCLUDE [Monitor and troubleshoot compute modules](../../includes/azure-stack-edge-monitor-troubleshoot-compute.md)]

## <a name="exit-the-remote-session"></a>Salir de la sesión remota

Para salir de la sesión remota de PowerShell, cierre la ventana de PowerShell.

## <a name="next-steps"></a>Pasos siguientes

- Implemente [Azure Stack Edge Pro FPGA](azure-stack-edge-deploy-prep.md) en Azure Portal.