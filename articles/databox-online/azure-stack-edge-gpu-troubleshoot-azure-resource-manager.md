---
title: Solución de problemas de la configuración de Azure Resource Manager en Azure Stack Edge Pro con GPU | Microsoft Docs
description: Describe cómo solucionar problemas con la configuración de Azure Resource Manager en los dispositivos de Azure Stack Edge.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: troubleshooting
ms.date: 06/10/2021
ms.author: alkohli
ms.custom: contperf-fy21q4
ms.openlocfilehash: d8226d780c3caabb94f0780f3a4296765207f1b1
ms.sourcegitcommit: e39ad7e8db27c97c8fb0d6afa322d4d135fd2066
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111987905"
---
# <a name="troubleshoot-azure-resource-manager-issues-on-an-azure-stack-edge-device"></a>Solución de problemas de Azure Resource Manager en un dispositivo de Azure Stack Edge 

[!INCLUDE [applies-to-GPU-and-pro-r-and-mini-r-skus](../../includes/azure-stack-edge-applies-to-gpu-pro-r-mini-r-sku.md)]

En este artículo se describe cómo solucionar problemas con Azure Resource Manager que pueden interferir con la administración de recursos en el dispositivo de Azure Stack Edge. Azure Resource Manager proporciona una capa de administración que le permite crear, actualizar y eliminar recursos de su cuenta de Azure.
 
## <a name="azure-resource-manager-configuration-errors"></a>Errores de configuración de Azure Resource Manager

Los siguientes errores pueden indicar un problema con la configuración de Azure Resource Manager. 

| **Problemas o errores** |  **Resolución** | 
|------------|-----------------|
|Problemas generales|<li>[Verifique que el dispositivo está configurado correctamente](#verify-the-device-is-configured-properly).<li> [Verifique que el cliente está configurado correctamente](#verify-the-client-is-configured-properly).|
|Add-AzureRmEnvironment: Se produjo un error al enviar la solicitud.<br>At line:1 char:1<br>+ Add-AzureRmEnvironment -Name Az3 -ARMEndpoint "https://management.dbe ...|El dispositivo no es accesible o no está configurado correctamente. Verifique que el dispositivo y el cliente estén configurados correctamente. Para obtener una guía, consulte la fila **Problemas generales** en esta tabla.|
|El servicio devolvió un error. Consulte InnerException para obtener más información: Se ha cerrado la conexión subyacente: No se ha podido establecer la relación de confianza para el canal seguro SSL/TLS. |  Se produjo un error en la creación e instalación del certificado en el dispositivo. Para obtener más información, consulte el artículo sobre cómo [crear e instalar una certificados](azure-stack-edge-gpu-connect-resource-manager.md#step-2-create-and-install-certificates). |
|La operación devolvió un código de estado no válido "ServiceUnavailable". <br> Response status code does not indicate success: 503 (Servicio no disponible). | Este error puede deberse a cualquiera de las condiciones:<li>ArmStsPool está en estado detenido.</li><li>Azure Resource Manager o el sitio web del servicio de token de seguridad están inactivos.</li><li>El recurso de clúster de Azure Resource Manager está inactivo.</li><br>Reiniciar el dispositivo puede corregir el problema. Para que podamos depurar aún más, [recopile un paquete de soporte técnico](azure-stack-edge-gpu-troubleshoot.md#collect-support-package).|
|AADSTS50126: Nombre de usuario o contraseña no válidos.<br>Id. de seguimiento: 29317da9-52fc-4ba0-9778-446ae5625e5a<br>Id. de correlación: 1b9752c4-8cbf-4304-a714-8a16527410f4<br>Marca de tiempo: 2019-11-15 09:21:57Z: Error en el servidor remoto: (400) Solicitud incorrecta.<br>At line:1 char:1 |Este error puede deberse a cualquiera de las condiciones:<li>Si el nombre de usuario y la contraseña no son válidos, asegúrese de que ha [restablecido la contraseña del administrador de Azure Storage desde Azure Portal](./azure-stack-edge-gpu-set-azure-resource-manager-password.md) y, a continuación, use la contraseña correcta.<li>Si un identificador de inquilino no es válido, asegúrese de que esté establecido en `c0257de7-538f-415c-993a-1b87a031879d`.</li>|
|connect-AzureRmAccount: AADSTS90056: El recurso está deshabilitado o no existe. Compruebe el código de la aplicación para asegurarse de que ha especificado la URL exacta del recurso al que está intentando acceder.<br>Id. de seguimiento: e19bdbc9-5dc8-4a74-85c3-ac6abdfda115<br>Id. de correlación: 75c8ef5a-830e-48b5-b039-595a96488ff9 Marca de tiempo: 2019-11-18 07:00:51Z: Error en el servidor remoto: (400) Incorrecto |Los puntos de conexión de Azure Resource Manager usados en el comando `Add-AzureRmEnvironment` no son correctos.<br>Para buscar los puntos de conexión de Azure Resource Manager, active **Puntos de conexión de dispositivo** en la página **Dispositivo** de la interfaz de usuario web local del dispositivo.<br>Para obtener instrucciones de PowerShell, vea [Establecimiento del entorno de Azure Resource Manager](azure-stack-edge-gpu-connect-resource-manager.md#step-7-set-azure-resource-manager-environment). |
|No se pueden obtener los puntos de conexión de la nube.<br>Asegúrese de tener conexión de red. Detalle del error: HTTPSConnectionPool(host='management.dbg-of4k6suvm.microsoftdatabox.com', port=30005): Se superó el máximo de reintentos en la URL: /metadata/endpoints?api-version=2015-01-01 (Caused by SSLError(SSLError("bad handshake: Error([('SSL routines', 'tls_process_server_certificate', 'certificate verify failed')],)",),)) |Este error aparece principalmente en un entorno de Mac o Linux. El error se produce porque no se agregó ningún certificado de formato PEM al almacén de certificados de Python. |


## <a name="troubleshoot-general-issues-with-azure-resource-manager"></a>Solución de problemas generales con Azure Resource Manager

Si tiene problemas generales con Azure Resource Manager, asegúrese de que el dispositivo y el cliente estén configurados correctamente. Para consultar los procedimientos completos, consulte [Conexión a Azure Resource Manager en un dispositivo de Azure Stack Edge Pro con GPU](azure-stack-edge-gpu-connect-resource-manager.md).


### <a name="verify-the-device-is-configured-properly"></a>Comprobación de que el dispositivo está configurado correctamente

1. En la interfaz de usuario local, compruebe que la red del dispositivo está configurada correctamente.

2. [Verifique que los certificados están actualizados para todos los puntos de conexión](./azure-stack-edge-gpu-connect-resource-manager.md#step-2-create-and-install-certificates).

3. Obtenga el punto de conexión de inicio de sesión y administración de Azure Resource Manager en la página **Dispositivo** en la interfaz de usuario local.

4. Compruebe que el dispositivo está activado y registrado en Azure.


### <a name="verify-the-client-is-configured-properly"></a>Comprobación de que el cliente está configurado correctamente

1. [Valide que esté instalada la versión correcta de PowerShell](./azure-stack-edge-gpu-connect-resource-manager.md#step-3-install-powershell-on-the-client).

2. [Valide que estén instalados los módulos de PowerShell correctos](./azure-stack-edge-gpu-connect-resource-manager.md#step-4-set-up-azure-powershell-on-the-client).

3. Compruebe que se puede comunicar con los puntos de conexión de Azure Resource Manager y de inicio de sesión. Puede intentar hacer ping a los puntos de conexión. Por ejemplo:

   `ping management.28bmdw2-bb9.microsoftdatabox.com`
   `ping login.28bmdw2-bb9.microsoftdatabox.com`
   
   Si no se puede acceder a ellos, [agregue entradas de archivo DNS/host](./azure-stack-edge-gpu-connect-resource-manager.md#step-5-modify-host-file-for-endpoint-name-resolution).
   
4. [Valide que los certificados de cliente estén instalados](./azure-stack-edge-gpu-connect-resource-manager.md#import-certificates-on-the-client-running-azure-powershell).

5. Si usa PowerShell, habilite la preferencia de depuración para ver los mensajes detallados mediante la ejecución de este comando de PowerShell: 

    `$debugpreference = "continue"`


## <a name="next-steps"></a>Pasos siguientes

- [Solución de problemas de activación de dispositivos](azure-stack-edge-gpu-troubleshoot-activation.md).
- [Solución de problemas de dispositivos](azure-stack-edge-gpu-troubleshoot.md).