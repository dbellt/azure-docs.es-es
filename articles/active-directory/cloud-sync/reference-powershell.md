---
title: Módulo AADCloudSyncTools de PowerShell para la sincronización en la nube de Azure AD Connect
description: En este artículo se describe cómo instalar el agente de aprovisionamiento en la nube de Azure AD Connect.
services: active-directory
author: billmath
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 11/30/2020
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 12eebf9c43dff9b3a83458051f5a3980b5afcbe0
ms.sourcegitcommit: 34feb2a5bdba1351d9fc375c46e62aa40bbd5a1f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111888993"
---
# <a name="aadcloudsynctools-powershell-module-for-azure-ad-connect-cloud-sync"></a>Módulo AADCloudSyncTools de PowerShell para la sincronización en la nube de Azure AD Connect

El módulo AADCloudSyncTools proporciona un conjunto de herramientas útiles que puede usar para administrar las implementaciones de sincronización en la nube de Azure AD Connect.

## <a name="pre-requisites"></a>Requisitos previos
Los siguientes requisitos previos son obligatorios:

- Todos los requisitos previos de este módulo se pueden instalar automáticamente con `Install-AADCloudSyncToolsPrerequisites`.
- Este módulo usa la autenticación MSAL, por lo que es necesario que el módulo MSAL.PS esté instalado. Para comprobarlo, en una ventana de PowerShell, ejecute `Get-module MSAL.PS -ListAvailable`. Si el módulo está instalado correctamente, obtendrá una respuesta. Puede usar `Install-AADCloudSyncToolsPrerequisites` para instalar la versión más reciente de MSAL.PS.
- Aunque el módulo AzureAD de PowerShell no es un requisito previo para acceder a las funcionalidades de este módulo, es útil tenerlo, por lo que también se instala automáticamente con el uso de `Install-AADCloudSyncToolsPrerequisites`.
- La instalación manual de módulos de PowerShell requiere el cumplimiento de TLS 1.2. Para asegurarse de que puede instalar los módulos, configure lo siguiente en la sesión de PowerShell mediante:
  ```
   Install-Module:
  [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12 
  ```


## <a name="install-the-aadcloudsynctools-powershell-module"></a>Instalación del módulo AADCloudSyncTools de PowerShell
Para instalar y usar el módulo AADCloudSyncTools, siga estos pasos:

1. Abra Windows PowerShell con privilegios de administrador.
2. Escriba o copie y pegue lo siguiente: `Import-module -Name "C:\Program Files\Microsoft Azure AD Connect Provisioning Agent\Utility\AADCloudSyncTools"`
3. Presione Entrar.
4. Para comprobar que se ha importado el módulo, escriba o copie y pegue lo siguiente: `Get-module AADCloudSyncTools`
5. Ahora debería ver información sobre el módulo.
6. A continuación, para instalar los requisitos previos del módulo AADCloudSyncTools, ejecute `Install-AADCloudSyncToolsPrerequisites`.
7. En la primera ejecución, el módulo PoweShellGet se instalará si no existe. Para cargar el nuevo módulo PowershellGet, cierre la ventana de PowerShell y abra una nueva sesión de PowerShell con privilegios de administrador. 
8. Vuelva a importar el módulo siguiendo el paso 2.
9. Ejecute `Install-AADCloudSyncToolsPrerequisites` para instalar los módulos MSAL y AzureAD.
11. Todos los requisitos previos deben estar instalados correctamente. ![Módulo de instalación](media/reference-powershell/install-1.png)


## <a name="aadcloudsynctools--cmdlets"></a>Cmdlets de AADCloudSyncTools
### <a name="connect-aadcloudsynctools"></a>Connect-AADCloudSyncTools
Usa el módulo MSAL.PS para solicitar un token para que el administrador de Azure AD tenga acceso a Microsoft Graph. 


### <a name="export-aadcloudsynctoolslogs"></a>Export-AADCloudSyncToolsLogs
Exporta y empaqueta todos los datos de solución de problemas en un archivo comprimido, como se indica a continuación:
 1. Inicia un seguimiento detallado con Start-AADCloudSyncToolsVerboseLogs.  Puede encontrar estos registros de seguimiento en la carpeta C:\ProgramData\Microsoft\Azure AD Connect Provisioning Agent\Trace.
 2. Recopila un registro de seguimiento durante 3 minutos.
   Puede especificar otro tiempo con -TracingDurationMins u omitir el seguimiento detallado con -SkipVerboseTrace.
 3. Detiene el seguimiento detallado con Stop-AADCloudSyncToolsVerboseLogs.
 4. Recopila registros del Visor de eventos durante las últimas 24 horas.
 5. Comprime todos los registros del agente, los registros detallados y los registros del visor de eventos en un archivo ZIP comprimido en la carpeta Documentos del usuario. 
 </br>Puede especificar otra carpeta de salida con -OutputPath \<folder path\>.

### <a name="get-aadcloudsynctoolsinfo"></a>Get-AADCloudSyncToolsInfo
Muestra los detalles del inquilino de Azure AD y el estado de las variables internas.

### <a name="get-aadcloudsynctoolsjob"></a>Get-AADCloudSyncToolsJob
Usa Graph para obtener entidades de servicio de AD2AAD y devuelve la información del trabajo de sincronización.
También se le puede llamar con el identificador del trabajo de sincronización específico como parámetro.

### <a name="get-aadcloudsynctoolsjobschedule"></a>Get-AADCloudSyncToolsJobSchedule
Usa Graph para obtener entidades de servicio de AD2AAD y devuelve la programación del trabajo de sincronización.
También se le puede llamar con el identificador del trabajo de sincronización específico como parámetro.

### <a name="get-aadcloudsynctoolsjobschema"></a>Get-AADCloudSyncToolsJobSchema
Usa Graph para obtener entidades de servicio de AD2AAD y devuelve el esquema del trabajo de sincronización.

### <a name="get-aadcloudsynctoolsjobscope"></a>Get-AADCloudSyncToolsJobScope
Usa Graph para obtener el esquema del trabajo de sincronización para el identificador del trabajo de sincronización proporcionado y muestra todos los ámbitos del grupo de filtros.

### <a name="get-aadcloudsynctoolsjobsettings"></a>Get-AADCloudSyncToolsJobSettings
Usa Graph para obtener entidades de servicio de AD2AAD y devuelve la configuración del trabajo de sincronización.
También se le puede llamar con el identificador del trabajo de sincronización específico como parámetro.

### <a name="get-aadcloudsynctoolsjobstatus"></a>Get-AADCloudSyncToolsJobStatus
Usa Graph para obtener entidades de servicio de AD2AAD y devuelve el estado del trabajo de sincronización.
También se le puede llamar con el identificador del trabajo de sincronización específico como parámetro.

### <a name="get-aadcloudsynctoolsserviceprincipal"></a>Get-AADCloudSyncToolsServicePrincipal
Usa Graph para obtener las entidades de servicio de AD2AAD o SyncFabric.
Sin parámetros, solo devolverá las entidades de servicio de AD2AAD.

### <a name="install-aadcloudsynctoolsprerequisites"></a>Install-AADCloudSyncToolsPrerequisites
Comprueba la presencia de los módulos PowerShellGet v2.2.4.1 o posterior, Azure AD y MSAL.PS y los instala si faltan.

### <a name="invoke-aadcloudsynctoolsgraphquery"></a>Invoke-AADCloudSyncToolsGraphQuery
Invoca una solicitud web para el URI, el método y el cuerpo especificados como parámetros.

### <a name="repair-aadcloudsynctoolsaccount"></a>Repair-AADCloudSyncToolsAccount
Usa Azure AD PowerShell para eliminar la cuenta actual (si está presente) y restablece la autenticación de la cuenta de sincronización con una nueva cuenta de sincronización en Azure AD.

### <a name="restart-aadcloudsynctoolsjob"></a>Restart-AADCloudSyncToolsJob
Reinicia una sincronización completa.

### <a name="resume-aadcloudsynctoolsjob"></a>Resume-AADCloudSyncToolsJob
Continúa la sincronización desde la marca de agua anterior.

### <a name="start-aadcloudsynctoolsverboselogs"></a>Start-AADCloudSyncToolsVerboseLogs
Modifica AADConnectProvisioningAgent.exe.config para habilitar el seguimiento detallado y reinicia el servicio AADConnectProvisioningAgent. Puede usar -SkipServiceRestart para evitar el reinicio del servicio, pero los cambios de configuración no surtirán efecto.  Puede encontrar estos registros de seguimiento en la carpeta C:\ProgramData\Microsoft\Azure AD Connect Provisioning Agent\Trace.

### <a name="stop-aadcloudsynctoolsverboselogs"></a>Stop-AADCloudSyncToolsVerboseLogs
Modifica AADConnectProvisioningAgent.exe.config para deshabilitar el seguimiento detallado y reinicia el servicio AADConnectProvisioningAgent. Puede usar -SkipServiceRestart para evitar el reinicio del servicio, pero los cambios de configuración no surtirán efecto.

### <a name="suspend-aadcloudsynctoolsjob"></a>Suspend-AADCloudSyncToolsJob
Pausa la sincronización.

## <a name="next-steps"></a>Pasos siguientes 

- [¿Qué es el aprovisionamiento?](what-is-provisioning.md)
- [¿Qué es la sincronización en la nube de Azure AD Connect?](what-is-cloud-sync.md)

