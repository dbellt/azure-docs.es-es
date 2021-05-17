---
title: Instalación del agente de aprovisionamiento en la nube de Azure AD Connect mediante una interfaz de la línea de comandos (CLI) y PowerShell
description: Aprenda a instalar el agente de aprovisionamiento en la nube de Azure AD Connect con cmdlets de PowerShell.
services: active-directory
author: billmath
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 11/16/2020
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: d5f560321b13e4c7a47aed262a1d312093e9be56
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108758832"
---
# <a name="install-the-azure-ad-connect-provisioning-agent-using-a-command-line-interface-cli-and-powershell"></a>Instalación del agente de aprovisionamiento de Azure AD Connect mediante una interfaz de la línea de comandos (CLI) y PowerShell
En el siguiente documento se le mostrará cómo instalar el agente de aprovisionamiento de Azure AD Connect con cmdlets de PowerShell.
 
>[!NOTE]
>En este documento se habla de la instalación del agente de aprovisionamiento mediante la interfaz de la línea de comandos.  Para obtener información sobre la instalación del agente de aprovisionamiento de Azure AD Connect mediante el asistente, vea [Instalación del agente de aprovisionamiento de Azure AD Connect](how-to-install.md).

## <a name="prerequisite"></a>Requisito previo: 


>[!IMPORTANT]
>En las siguientes instrucciones de instalación se supone que se han cumplido todos los [requisitos previos](how-to-prerequisites.md).
>
> Windows Server debe tener TLS 1.2 habilitado para poder instalar el agente de aprovisionamiento de Azure AD Connect con cmdlets de PowerShell. Para habilitar TLS 1.2, puede usar los pasos que se encuentran [aquí](how-to-prerequisites.md#tls-requirements).

 

## <a name="install-the-azure-ad-connect-provisioning-agent-using-powershell-cmdlets"></a>Instalación del agente de aprovisionamiento de Azure AD Connect con cmdlets de PowerShell 


 1. Inicie sesión en Azure Portal y, a continuación, vaya a **Azure Active Directory**.
 2. En el menú de la izquierda, seleccione **Azure AD Connect**.
 3. Seleccione **Administrar el aprovisionamiento (versión preliminar)**  > **Revisar todos los agentes**.
 4. Descargue el agente de aprovisionamiento de Azure AD Connect desde Azure Portal en una ubicación local.  

   ![Descargar el agente local](media/how-to-install/install-9.png)</br>
 5. A efectos de estas instrucciones, el agente se descargó en la siguiente carpeta:   Carpeta "C:\ProvisioningSetup". 
 6. Instalación de ProvisioningAgent en modo silencioso

   ```
   $installerProcess = Start-Process c:\temp\AADConnectProvisioningAgent.Installer.exe /quiet -NoNewWindow -PassThru 
   $installerProcess.WaitForExit()  
   ```
 7. Importación del módulo PS del agente de aprovisionamiento 

   ```
   Import-Module "C:\Program Files\Microsoft Azure AD Connect Provisioning Agent\Microsoft.CloudSync.PowerShell.dll" 
   ```
 8. Conéctese a Azure AD con las credenciales de administrador global; puede personalizar esta sección para capturar la contraseña desde un almacén seguro. 

   ```
   $globalAdminPassword = ConvertTo-SecureString -String "Global admin password" -AsPlainText -Force 

   $globalAdminCreds = New-Object System.Management.Automation.PSCredential -ArgumentList ("GlobalAdmin@contoso.onmicrosoft.com", $globalAdminPassword) 
   ```

   Connect-AADCloudSyncAzureAD -Credential $globalAdminCreds 

 9. Agregue la cuenta de gMSA y proporcione las credenciales del administrador de dominio para crear una cuenta de gMSA predeterminada 
 
   ```
   $domainAdminPassword = ConvertTo-SecureString -String "Domain admin password" -AsPlainText -Force 

   $domainAdminCreds = New-Object System.Management.Automation.PSCredential -ArgumentList ("DomainName\DomainAdminAccountName", $domainAdminPassword) 

   Add-AADCloudSyncGMSA -Credential $domainAdminCreds 
   ```
 10. O bien, use el cmdlet anterior como se indica a continuación para proporcionar una cuenta de gMSA creada previamente. 

 
   ```
   Add-AADCloudSyncGMSA -CustomGMSAName preCreatedGMSAName$ 
   ```
 11. Adición del dominio 

   ```
   $contosoDomainAdminPassword = ConvertTo-SecureString -String "Domain admin password" -AsPlainText -Force 

   $contosoDomainAdminCreds = New-Object System.Management.Automation.PSCredential -ArgumentList ("DomainName\DomainAdminAccountName", $contosoDomainAdminPassword) 

   Add-AADCloudSyncADDomain -DomainName contoso.com -Credential $contosoDomainAdminCreds 
   ```
 12. También puede usar el cmdlet anterior como se indica a continuación para configurar los controladores de dominio preferidos. 

   ```
   $preferredDCs = @("PreferredDC1", "PreferredDC2", "PreferredDC3") 

   Add-AADCloudSyncADDomain -DomainName contoso.com -Credential $contosoDomainAdminCreds -PreferredDomainControllers $preferredDCs 
   ```
 13. Repita el paso anterior para agregar más dominios y proporcione los nombres de cuenta y de dominio de los dominios respectivos. 
 14. Reinicie el servicio. 
   ```
   Restart-Service -Name AADConnectProvisioningAgent  
   ```
 15.  Vaya a Azure Portal para crear la configuración de sincronización en la nube.

## <a name="provisioning-agent-gmsa-powershell-cmdlets"></a>Cmdlets de PowerShell de gMSA del agente de aprovisionamiento
Ahora que ha instalado el agente, puede aplicar permisos más granulares a gMSA.  Para información e instrucciones paso a paso sobre cómo configurar los permisos, consulte [Cmdlets de PowerShell de gMSA del agente de aprovisionamiento en la nube](how-to-gmsa-cmdlets.md).

## <a name="next-steps"></a>Pasos siguientes 

- [¿Qué es el aprovisionamiento?](what-is-provisioning.md)
- [Cmdlets de PowerShell de gMSA del agente de aprovisionamiento en la nube de Azure AD](how-to-gmsa-cmdlets.md)
- [¿Qué es la sincronización en la nube de Azure AD Connect?](what-is-cloud-sync.md)