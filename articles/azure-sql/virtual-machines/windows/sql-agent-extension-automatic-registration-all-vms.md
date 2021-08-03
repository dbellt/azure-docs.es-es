---
title: Registro automático con la extensión Agente de IaaS de SQL
description: Obtenga información sobre cómo habilitar la característica de registro automático para registrar automáticamente todas las VM con SQL Server pasadas y futuras con la extensión Agente de IaaS de SQL mediante Azure Portal.
author: MashaMSFT
ms.author: mathoma
tags: azure-service-management
ms.service: virtual-machines-sql
ms.subservice: management
ms.topic: conceptual
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 11/07/2020
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 187e51d049d965d0fb9cef1af93a5036819e957f
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110665817"
---
# <a name="automatic-registration-with-sql-iaas-agent-extension"></a>Registro automático con la extensión Agente de IaaS de SQL
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

Habilite la característica de registro automático en Azure Portal para registrar automáticamente todas las máquinas de Azure Virtual Machines con SQL Server pasadas y futuras con la [extensión Agente de IaaS de SQL](sql-server-iaas-agent-extension-automate-management.md) en modo ligero. 

En este artículo se enseña a habilitar la característica de registro automático. Como alternativa, puede [registrar una única VM](sql-agent-extension-manually-register-single-vm.md) o [registrar las VM de forma masiva](sql-agent-extension-manually-register-vms-bulk.md) con la extensión Agente de IaaS de SQL. 

## <a name="overview"></a>Información general

Registro de la VM con SQL Server con la [extensión Agente de IaaS de SQL](sql-server-iaas-agent-extension-automate-management.md) para desbloquear todo el conjunto de ventajosas características. 

Cuando el registro automático está habilitado, un trabajo se ejecuta diariamente para detectar si SQL Server está instalado en todas las máquinas virtuales no registradas de la suscripción. Para ello, copie los archivos binarios de extensión del agente de IaaS de SQL en la máquina virtual y, a continuación, ejecute una utilidad puntual que compruebe el subárbol del Registro de SQL Server. Si se detecta el subárbol de SQL Server, la máquina virtual se registra con la extensión en modo ligero. Si no existe ningún subárbol de SQL Server en el Registro, se quitan los archivos binarios. El registro automático puede tardar hasta 4 días en detectar máquinas virtuales con SQL Server recién creadas.

Una vez que el registro automático esté habilitado para una suscripción, todas las máquinas virtuales actuales y futuras que tengan SQL Server instalado se registrarán con la extensión de agente de IaaS de SQL **en modo ligero sin tiempo de inactividad y sin reiniciar el servicio de SQL Server**. Aún debe [actualizar manualmente al modo de administración completo](sql-agent-extension-manually-register-single-vm.md#upgrade-to-full) para aprovechar todo el conjunto de características. El tipo de licencia predeterminado es el de la imagen de máquina virtual. Si usa una imagen de pago por uso para la máquina virtual, el tipo de licencia será `PAYG`; de lo contrario, el tipo de licencia será `AHUB` de forma predeterminada. 

> [!IMPORTANT]
> La extensión del Agente de IaaS de SQL recopila datos con el fin de ofrecer a los clientes ventajas opcionales al usar SQL Server en Azure Virtual Machines. Microsoft no usará estos datos para auditorías de licencias sin el consentimiento previo del cliente. Para obtener más información, consulte [Complemento de privacidad de SQL Server](/sql/sql-server/sql-server-privacy#non-personal-data).

## <a name="prerequisites"></a>Requisitos previos

Para registrar una máquina virtual con SQL Server con la extensión, necesita lo siguiente: 

- Un [suscripción de Azure](https://azure.microsoft.com/free/) y, como mínimo, permisos del [rol colaborador](../../../role-based-access-control/built-in-roles.md#all).
- Una [máquina virtual Windows Server 2008 R2 (o posterior)](../../../virtual-machines/windows/quick-create-portal.md) del modelo de recursos de Azure con [SQL Server](https://www.microsoft.com/sql-server/sql-server-downloads) implementado en la nube pública o de Azure Government. No se admite Windows Server 2008. 


## <a name="enable"></a>Habilitar

Para habilitar el registro automático de las VM con SQL Server en Azure Portal, siga los pasos que se indican a continuación:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Vaya a la página del recurso [**Máquinas virtuales SQL**](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResource/resourceType/Microsoft.SqlVirtualMachine%2FSqlVirtualMachines). 
1. Seleccione **Automatic SQL Server VM registration** (Registro automático de VM con SQL Server) para abrir la página **Automatic registration** (Registro automático). 

   :::image type="content" source="media/sql-agent-extension-automatic-registration-all-vms/automatic-registration.png" alt-text="Seleccione Registro automático de VM con SQL Server para abrir la página Registro automático.":::

1. Elija la suscripción en el menú desplegable. 
1. Lea los términos y, si está de acuerdo, seleccione **Acepto**. 
1. Seleccione **Registrarse** para habilitar la característica y registrar automáticamente todas las VM con SQL Server actuales y futuras con la extensión Agente de IaaS de SQL. Esto no reiniciará el servicio SQL Server en ninguna de las VM. 

## <a name="disable"></a>Deshabilitar

Use la [CLI de Azure](/cli/azure/install-azure-cli) o [Azure PowerShell](/powershell/azure/install-az-ps) para deshabilitar la característica de registro automático. Cuando la característica de registro automático está deshabilitada, las VM con SQL Server agregadas a la suscripción deben registrarse manualmente en la extensión Agente de IaaS de SQL. No se anulará el registro de las VM con SQL Server existentes que ya se hayan registrado.



# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para deshabilitar el registro automático mediante la CLI de Azure, ejecute el siguiente comando: 

```azurecli-interactive
az feature unregister --namespace Microsoft.SqlVirtualMachine --name BulkRegistration
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

Para deshabilitar el registro automático mediante Azure PowerShell, ejecute el siguiente comando: 

```powershell-interactive
Unregister-AzProviderFeature -FeatureName BulkRegistration -ProviderNamespace Microsoft.SqlVirtualMachine
```

---

## <a name="enable-for-multiple-subscriptions"></a>Habilitación de varias suscripciones

Puede habilitar la característica de registro automático para varias suscripciones de Azure mediante PowerShell. 

Para ello, siga estos pasos:

1. Guarde [este script](https://github.com/microsoft/tigertoolbox/blob/master/AzureSQLVM/AutoRegTools.psm1).
1. Navegue hasta la ubicación donde guardó el script mediante un símbolo del sistema administrativo o una ventana de PowerShell. 
1. Conéctese a Azure (`az login`).
1. Ejecute el script, pasando SubscriptionIds como parámetros, como:   
   `.\EnableBySubscription.ps1 -SubscriptionList SubscriptionId1,SubscriptionId2`

   Por ejemplo: 

   ```console
   .\EnableBySubscription.ps1 -SubscriptionList a1a1a-aa11-11aa-a1a1-a11a111a1,b2b2b2-bb22-22bb-b2b2-b2b2b2bb
   ```

Los errores de registro incorrecto se almacenan en `RegistrationErrors.csv`, que se encuentra en el mismo directorio donde guardó y dese el que ejecutó el script `.ps1`. 

## <a name="next-steps"></a>Pasos siguientes

Actualice el modo de administración a [completo](sql-agent-extension-manually-register-single-vm.md#upgrade-to-full) para aprovechar todo el conjunto de características que proporciona la extensión Agente de IaaS de SQL. 
