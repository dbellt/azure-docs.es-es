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
ms.openlocfilehash: 851557c4cdc0f913247d40f2aaea3230a2e8551f
ms.sourcegitcommit: a5dd9799fa93c175b4644c9fe1509e9f97506cc6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108204736"
---
# <a name="set-up-diagnostics-with-a-trusted-platform-module-tpm-endpoint-of-azure-attestation"></a>Configuración de diagnósticos con el punto de conexión del Módulo de plataforma segura (TPM) de Azure Attestation

Este artículo le ayuda a crear y establecer la configuración de diagnóstico para enviar las métricas y los registros de la plataforma a diferentes destinos. Los [registros de plataforma](../azure-monitor/essentials/platform-logs-overview.md) de Azure, como los registros de recursos y los registros de actividad de Azure, proporcionan información de diagnóstico y auditoría detallada sobre los recursos de Azure y la plataforma de Azure de la que dependen. Las [métricas de plataforma](../azure-monitor/essentials/data-platform-metrics.md) se recopilan de manera predeterminada y se almacenan en la base de datos de métricas de Azure Monitor.

Antes de empezar, asegúrese de que ha [configurado Azure Attestation con Azure PowerShell](quickstart-powershell.md).

El servicio de punto de conexión del Módulo de plataforma segura (TPM) está habilitado en la configuración de diagnóstico y se puede usar para supervisar la actividad. Configure la [supervisión de Azure](../azure-monitor/overview.md) para el punto de conexión del servicio de TPM mediante el código siguiente.

```powershell

 Connect-AzAccount 

 Set-AzContext -Subscription <Subscription id> 

 $attestationProviderName=<Name of the attestation provider> 

 $attestationResourceGroup=<Name of the resource Group> 

 $attestationProvider=Get-AzAttestation -Name $attestationProviderName -ResourceGroupName $attestationResourceGroup 

 $storageAccount=New-AzStorageAccount -ResourceGroupName $attestationProvider.ResourceGroupName -Name <Storage Account Name> -SkuName Standard_LRS -Location <Location> 

 Set-AzDiagnosticSetting -ResourceId $ attestationProvider.Id -StorageAccountId $ storageAccount.Id -Enabled $true 

```

Los registros de actividad se encuentran en la sección **Contenedores** de la cuenta de almacenamiento. Para más información, consulte [Tutorial: Recopilación y análisis de registros de recurso desde un recurso de Azure](../azure-monitor/essentials/tutorial-resource-logs.md).