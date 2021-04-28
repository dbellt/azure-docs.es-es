---
title: Supervisión de diagnósticos de Azure para Azure Attestation
description: Supervisión de diagnósticos de Azure para Azure Attestation
services: attestation
author: msmbaldwin
ms.service: attestation
ms.topic: overview
ms.date: 08/31/2020
ms.author: mbaldwin
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 2b0cd0402348e4aa45b291f30b677fc9e4bbdb98
ms.sourcegitcommit: 3c460886f53a84ae104d8a09d94acb3444a23cdc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107833641"
---
# <a name="set-up-diagnostics-with-a-trusted-platform-module-tpm-endpoint-of-azure-attestation"></a>Configuración de diagnósticos con el punto de conexión del Módulo de plataforma segura (TPM) de Azure Attestation

Este artículo le ayuda a crear y establecer la configuración de diagnóstico para enviar las métricas y los registros de la plataforma a diferentes destinos. Los [registros de plataforma](/azure/azure-monitor/platform/platform-logs-overview) de Azure, como los registros de recursos y los registros de actividad de Azure, proporcionan información de diagnóstico y auditoría detallada sobre los recursos de Azure y la plataforma de Azure de la que dependen. Las [métricas de plataforma](/azure/azure-monitor/platform/data-platform-metrics) se recopilan de manera predeterminada y se almacenan en la base de datos de métricas de Azure Monitor.

Antes de empezar, asegúrese de que ha [configurado Azure Attestation con Azure PowerShell](quickstart-powershell.md).

El servicio de punto de conexión del Módulo de plataforma segura (TPM) está habilitado en la configuración de diagnóstico y se puede usar para supervisar la actividad. Configure la [supervisión de Azure](/azure/azure-monitor/overview) para el punto de conexión del servicio de TPM mediante el código siguiente.

```powershell

 Connect-AzAccount 

 Set-AzContext -Subscription <Subscription id> 

 $attestationProviderName=<Name of the attestation provider> 

 $attestationResourceGroup=<Name of the resource Group> 

 $attestationProvider=Get-AzAttestation -Name $attestationProviderName -ResourceGroupName $attestationResourceGroup 

 $storageAccount=New-AzStorageAccount -ResourceGroupName $attestationProvider.ResourceGroupName -Name <Storage Account Name> -SkuName Standard_LRS -Location <Location> 

 Set-AzDiagnosticSetting -ResourceId $ attestationProvider.Id -StorageAccountId $ storageAccount.Id -Enabled $true 

```

Los registros de actividad se encuentran en la sección **Contenedores** de la cuenta de almacenamiento. Para más información, consulte [Tutorial: Recopilación y análisis de registros de recurso desde un recurso de Azure](/azure/azure-monitor/learn/tutorial-resource-logs).
