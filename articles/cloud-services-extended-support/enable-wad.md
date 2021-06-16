---
title: Aplicación de la extensión de Windows Azure Diagnostics en Cloud Services (soporte extendido)
description: Aplicación de la extensión de Windows Azure Diagnostics en Cloud Services (soporte extendido)
ms.topic: how-to
ms.service: cloud-services-extended-support
author: gachandw
ms.author: gachandw
ms.reviewer: mimckitt
ms.date: 10/13/2020
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 03ab6be5b05cb42951b666776942270e4b430195
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110691214"
---
# <a name="apply-the-windows-azure-diagnostics-extension-in-cloud-services-extended-support"></a>Aplicación de la extensión de Windows Azure Diagnostics en Cloud Services (soporte extendido) 
Puede supervisar las métricas de rendimiento claves de cualquier servicio en la nube. Cada rol de servicio en la nube recopila datos mínimos: uso de CPU, uso de red y uso de disco. Si el servicio en la nube tiene la extensión Microsoft.Azure.Diagnostics aplicada a un rol, este último puede recopilar puntos de datos adicionales. Para obtener más información, consulte [Introducción a las extensiones](extensions.md).

La extensión de Windows Azure Diagnostics se puede habilitar para Cloud Services (soporte extendido) a través de [PowerShell](deploy-powershell.md) o la [plantilla de ARM](deploy-template.md)

## <a name="apply-windows-azure-diagnostics-extension-using-powershell"></a>Aplicación de la extensión de Windows Azure Diagnostics mediante PowerShell

```powershell
# Create WAD extension object
$storageAccountKey = Get-AzStorageAccountKey -ResourceGroupName "ContosOrg" -Name "contosostorageaccount"
$configFilePath = "<Insert WAD public configuration file path>"
$wadExtension = New-AzCloudServiceDiagnosticsExtension -Name "WADExtension" -ResourceGroupName "ContosOrg" -CloudServiceName "ContosoCS" -StorageAccountName "contosostorageaccount" -StorageAccountKey $storageAccountKey[0].Value -DiagnosticsConfigurationPath $configFilePath -TypeHandlerVersion "1.5" -AutoUpgradeMinorVersion $true 

# Add <privateConfig> settings
$wadExtension.ProtectedSetting = "<Insert WAD Private Configuration as raw string here>"

# Get existing Cloud Service
$cloudService = Get-AzCloudService -ResourceGroup "ContosOrg" -CloudServiceName "ContosoCS"

# Add WAD extension to existing Cloud Service extension object
$cloudService.ExtensionProfile.Extension = $cloudService.ExtensionProfile.Extension + $wadExtension

# Update Cloud Service
$cloudService | Update-AzCloudService
```
Descargue la definición del esquema del archivo de configuración público ejecutando el comando de PowerShell siguiente:

```powershell
(Get-AzureServiceAvailableExtension -ExtensionName 'PaaSDiagnostics' -ProviderNamespace 'Microsoft.Azure.Diagnostics').PublicConfigurationSchema | Out-File -Encoding utf8 -FilePath 'PublicWadConfig.xsd'
```
Este es un ejemplo del archivo XML de configuración pública
```
<?xml version="1.0" encoding="utf-8"?>
<PublicConfig xmlns="http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration">
  <WadCfg>
    <DiagnosticMonitorConfiguration overallQuotaInMB="25000">
      <PerformanceCounters scheduledTransferPeriod="PT1M">
        <PerformanceCounterConfiguration counterSpecifier="\Processor(_Total)\% Processor Time" sampleRate="PT1M" unit="percent" />
        <PerformanceCounterConfiguration counterSpecifier="\Memory\Committed Bytes" sampleRate="PT1M" unit="bytes"/>
      </PerformanceCounters>
      <EtwProviders>
        <EtwEventSourceProviderConfiguration provider="SampleEventSourceWriter" scheduledTransferPeriod="PT5M">
          <Event id="1" eventDestination="EnumsTable"/>
          <DefaultEvents eventDestination="DefaultTable" />
        </EtwEventSourceProviderConfiguration>
      </EtwProviders>
    </DiagnosticMonitorConfiguration>
  </WadCfg>
</PublicConfig>
```
Descargue la definición del esquema del archivo de configuración privada ejecutando el comando de PowerShell siguiente:

```powershell
(Get-AzureServiceAvailableExtension -ExtensionName 'PaaSDiagnostics' -ProviderNamespace 'Microsoft.Azure.Diagnostics').PrivateConfigurationSchema | Out-File -Encoding utf8 -FilePath 'PrivateWadConfig.xsd'
```
Este es un ejemplo del archivo XML de configuración privada

```
<?xml version="1.0" encoding="utf-8"?>
<PrivateConfig xmlns="http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration">
  <StorageAccount name="string" key="string" />
  <AzureMonitorAccount>
    <ServicePrincipalMeta>
      <PrincipalId>string</PrincipalId>
      <Secret>string</Secret>
    </ServicePrincipalMeta>
  </AzureMonitorAccount>
  <SecondaryStorageAccounts>
    <StorageAccount name="string" />
  </SecondaryStorageAccounts>
  <SecondaryEventHubs>
    <EventHub Url="string" SharedAccessKeyName="string" SharedAccessKey="string" />
  </SecondaryEventHubs>
</PrivateConfig>
```

## <a name="apply-windows-azure-diagnostics-extension-using-arm-template"></a>Aplicación de la extensión de Windows Azure Diagnostics mediante la plantilla de ARM
```json
"extensionProfile": { 
          "extensions": [ 
            { 
              "name": "Microsoft.Insights.VMDiagnosticsSettings_WebRole1", 
              "properties": { 
                "autoUpgradeMinorVersion": true, 
                "publisher": "Microsoft.Azure.Diagnostics", 
                "type": "PaaSDiagnostics", 
                "typeHandlerVersion": "1.5", 
                "settings": "Include PublicConfig XML as a raw string", 
                "protectedSettings": "Include PrivateConfig XML as a raw string”", 
                "rolesAppliedTo": [ 
                  "WebRole1" 
                ] 
              } 
            }
          ]
        },

```


## <a name="next-steps"></a>Pasos siguientes 
- Revise los [requisitos previos de implementación](deploy-prerequisite.md) de Cloud Services (soporte extendido).
- Vea las [preguntas más frecuentes](faq.md) sobre Cloud Services (soporte extendido).
- Implemente una instancia de Cloud Services (soporte extendido) mediante [Azure Portal](deploy-portal.md), [PowerShell](deploy-powershell.md), una [plantilla](deploy-template.md) o [Visual Studio](deploy-visual-studio.md).
