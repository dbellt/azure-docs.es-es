---
title: Envío de métricas de Cloud Services clásico a la base de datos de métricas de Azure Monitor
description: Se describe el proceso de envío de métricas de rendimiento del sistema operativo invitado para Cloud Services clásico de Azure al almacén de métricas de Azure Monitor.
author: anirudhcavale
services: azure-monitor
ms.topic: conceptual
ms.date: 09/09/2019
ms.author: ancav
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 02918b4f33c01aab5a250e3228b6ad7ec5930394
ms.sourcegitcommit: 52491b361b1cd51c4785c91e6f4acb2f3c76f0d5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108322246"
---
# <a name="send-guest-os-metrics-to-the-azure-monitor-metric-store-classic-cloud-services"></a>Envío de métricas de SO invitado al almacén de métricas de Azure Monitor en Cloud Services clásico 

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

Con la [extensión Diagnostics](../agents/diagnostics-extension-overview.md) de Azure Monitor, puede recopilar métricas y registros del sistema operativo invitado (SO invitado) que se ejecuta como parte de un clúster de Service Fabric, un servicio en la nube o una máquina virtual. La extensión puede enviar datos de telemetría a [muchas ubicaciones diferentes](../data-platform.md?toc=%2fazure%2fazure-monitor%2ftoc.json).

En este artículo se describe el proceso de envío de métricas de rendimiento del SO invitado para Cloud Services clásico de Azure al almacén de métricas de Azure Monitor. A partir de Diagnostics versión 1.11, puede escribir las métricas directamente en el almacén de métricas de Azure Monitor, donde ya se recopilan métricas de la plataforma estándar. 

Almacenarlas en esta ubicación permite tener acceso a las mismas acciones disponibles para las métricas de la plataforma. Las acciones incluyen la generación de alertas casi en tiempo real, la creación de gráficos, el enrutamiento, el acceso desde una API REST y mucho más.  Anteriormente, la extensión Diagnostics se escribía en Azure Storage, pero no en el almacén de datos de Azure Monitor.  

El proceso descrito en este artículo solo funciona para los contadores de rendimiento en Azure Cloud Services. No funciona para otras métricas personalizadas. 

## <a name="prerequisites"></a>Requisitos previos

- Debe ser [administrador de servicios o administrador](../../cost-management-billing/manage/add-change-subscription-administrator.md) en su suscripción de Azure. 

- La suscripción debe estar registrada en [Microsoft.Insights](../../azure-resource-manager/management/resource-providers-and-types.md). 

- Debe tener instalado [Azure PowerShell](/powershell/azure) o [Azure Cloud Shell](../../cloud-shell/overview.md).

- Su servicio en la nube debe estar en una [región que admita métricas personalizadas](./metrics-custom-overview.md#supported-regions).

## <a name="provision-a-cloud-service-and-storage-account"></a>Aprovisionamiento de un servicio en la nube y una cuenta de almacenamiento 

1. Cree e implemente un servicio en la nube clásico. Una aplicación de ejemplo de Cloud Services clásico y su implementación pueden encontrarse en [Introducción a Azure Cloud Services y ASP.NET](../../cloud-services/cloud-services-dotnet-get-started.md). 

2. Puede usar una cuenta de almacenamiento existente o implementar una nueva. Es mejor si la cuenta de almacenamiento se encuentra en la misma región que el servicio en la nube clásico que ha creado. En Azure Portal, vaya a la hoja de recursos **Cuentas de almacenamiento** y, después, seleccione **Claves**. Anote el nombre y la clave de la cuenta de almacenamiento. Necesitará esta información en pasos posteriores.

   ![Claves de cuenta de almacenamiento](./media/collect-custom-metrics-guestos-vm-cloud-service-classic/storage-keys.png)

## <a name="create-a-service-principal"></a>Creación de una entidad de servicio 

Cree una entidad de servicio en su inquilino de Azure Active Directory siguiendo las instrucciones de [Uso del portal para crear una aplicación de Azure Active Directory y una entidad de servicio con acceso a los recursos](../../active-directory/develop/howto-create-service-principal-portal.md). Tenga en cuenta lo siguiente al realizar este proceso: 

- Puede colocar cualquier dirección URL como dirección URL de inicio de sesión.  
- Cree un nuevo secreto de cliente para esta aplicación.  
- Guarde la clave y el identificador de cliente para usarlos en pasos posteriores.  

Asigne los permisos *Supervisión del publicador de métricas* a la aplicación que creó en el paso anterior para el recurso frente al cual quiere emitir métricas. Si va a usar la aplicación para emitir métricas personalizadas frente a muchos recursos, puede conceder estos permisos en el nivel de suscripción o de grupo de recursos.  

> [!NOTE]
> La extensión Diagnostics usa la entidad de servicio para autenticarse en Azure Monitor y emitir métricas para su servicio en la nube.

## <a name="author-diagnostics-extension-configuration"></a>Creación de la configuración de la extensión Diagnostics 

Prepare el archivo de configuración de la extensión Diagnostics. Este archivo determina qué registros y contadores de rendimiento debe recopilar la extensión Diagnostics para el servicio en la nube. A continuación encontrará un ejemplo de archivo de configuración de Diagnostics:  

```XML
<?xml version="1.0" encoding="utf-8"?> 
<DiagnosticsConfiguration xmlns="http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration"> 
  <PublicConfig xmlns="http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration"> 
    <WadCfg> 
      <DiagnosticMonitorConfiguration overallQuotaInMB="4096"> 
        <DiagnosticInfrastructureLogs scheduledTransferLogLevelFilter="Error" /> 
        <Directories scheduledTransferPeriod="PT1M"> 
          <IISLogs containerName="wad-iis-logfiles" /> 
          <FailedRequestLogs containerName="wad-failedrequestlogs" /> 
        </Directories> 
        <PerformanceCounters scheduledTransferPeriod="PT1M"> 
          <PerformanceCounterConfiguration counterSpecifier="\Processor(_Total)\% Processor Time" sampleRate="PT15S" /> 
          <PerformanceCounterConfiguration counterSpecifier="\Memory\Available MBytes" sampleRate="PT15S" /> 
          <PerformanceCounterConfiguration counterSpecifier="\Memory\Committed Bytes" sampleRate="PT15S" /> 
          <PerformanceCounterConfiguration counterSpecifier="\Memory\Page Faults/sec" sampleRate="PT15S" /> 
        </PerformanceCounters> 
        <WindowsEventLog scheduledTransferPeriod="PT1M"> 
          <DataSource name="Application!*[System[(Level=1 or Level=2 or Level=3)]]" /> 
          <DataSource name="Windows Azure!*[System[(Level=1 or Level=2 or Level=3 or Level=4)]]" /> 
        </WindowsEventLog> 
        <CrashDumps> 
          <CrashDumpConfiguration processName="WaIISHost.exe" /> 
          <CrashDumpConfiguration processName="WaWorkerHost.exe" /> 
          <CrashDumpConfiguration processName="w3wp.exe" /> 
        </CrashDumps> 
        <Logs scheduledTransferPeriod="PT1M" scheduledTransferLogLevelFilter="Error" /> 
      </DiagnosticMonitorConfiguration> 
      <SinksConfig> 
      </SinksConfig> 
    </WadCfg> 
    <StorageAccount /> 
  </PublicConfig> 
  <PrivateConfig xmlns="http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration"> 
    <StorageAccount name="" endpoint="" /> 
</PrivateConfig> 
  <IsEnabled>true</IsEnabled> 
</DiagnosticsConfiguration> 
```

En la sección "SinksConfig" del archivo de diagnóstico, defina un nuevo receptor de Azure Monitor: 

```XML
  <SinksConfig> 
    <Sink name="AzMonSink"> 
    <AzureMonitor> 
      <ResourceId>-Provide ClassicCloudService’s Resource ID-</ResourceId> 
      <Region>-Azure Region your Cloud Service is deployed in-</Region> 
    </AzureMonitor> 
    </Sink> 
  </SinksConfig> 
```

En la sección del archivo de configuración donde se enumeran los contadores de rendimiento que se van a recopilar, agregue el receptor de Azure Monitor. Esta entrada garantiza que todos los contadores de rendimiento especificados se enruten a Azure Monitor como métricas. Puede agregar o quitar contadores de rendimiento según sus necesidades. 

```xml
    <PerformanceCounters scheduledTransferPeriod="PT1M" sinks="AzMonSink">
        <PerformanceCounterConfiguration counterSpecifier="\Processor(_Total)\% Processor Time" sampleRate="PT15S" />
    ...
    </PerformanceCounters>
```

Por último, en la configuración privada, agregue la sección *Azure Monitor Account* (Cuenta de Azure Monitor). Escriba el identificador de cliente y el secreto de la entidad de servicio que creó antes. 

```XML
<PrivateConfig xmlns="http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration"> 
  <StorageAccount name="" endpoint="" /> 
    <AzureMonitorAccount> 
      <ServicePrincipalMeta> 
        <PrincipalId>clientId from step 3</PrincipalId> 
        <Secret>client secret from step 3</Secret> 
      </ServicePrincipalMeta> 
    </AzureMonitorAccount> 
</PrivateConfig> 
```

Guarde este archivo de diagnóstico localmente.  

## <a name="deploy-the-diagnostics-extension-to-your-cloud-service"></a>Implementación de la extensión Diagnostics en el servicio en la nube 

Inicie PowerShell e inicie sesión en Azure. 

```powershell
Login-AzAccount 
```

Use los siguientes comandos para almacenar los detalles de la cuenta de almacenamiento que creó anteriormente. 

```powershell
$storage_account = <name of your storage account from step 3> 
$storage_keys = <storage account key from step 3> 
```

De igual modo, establezca la ruta de acceso del archivo de diagnóstico en una variable mediante el comando siguiente:

```powershell
$diagconfig = “<path of the Diagnostics configuration file with the Azure Monitor sink configured>” 
```

Implemente la extensión Diagnostics en el servicio en la nube con el archivo de diagnóstico y el receptor de Azure Monitor configurado mediante el siguiente comando:  

```powershell
Set-AzureServiceDiagnosticsExtension -ServiceName <classicCloudServiceName> -StorageAccountName $storage_account -StorageAccountKey $storage_keys -DiagnosticsConfigurationPath $diagconfig 
```

> [!NOTE] 
> Sigue siendo obligatorio proporcionar una cuenta de almacenamiento como parte de la instalación de la extensión Diagnostics. Todos los registros o contadores de rendimiento especificados en el archivo de configuración de diagnóstico se escribirán en la cuenta de almacenamiento especificada.  

## <a name="plot-metrics-in-the-azure-portal"></a>Trazado de métricas en Azure Portal 

1. Vaya a Azure Portal. 

   ![La captura de pantalla muestra Azure Portal, con las opciones Monitor y Métricas seleccionadas.](./media/collect-custom-metrics-guestos-vm-cloud-service-classic/navigate-metrics.png)

2. En el menú de la izquierda, seleccione **Monitor**.

3. En la hoja **Monitor**, seleccione la pestaña **Métricas (versión preliminar)** .

4. En la lista desplegable de recursos, seleccione el servicio en la nube clásico.

5. En el menú desplegable de espacios de nombres, seleccione **azure.vm.windows.guest**. 

6. En el menú desplegable de métricas, seleccione **Memory\Committed Bytes in Use** (Memoria\bytes confirmados en uso). 

Puede usar las funcionalidades de división y de filtrado de dimensiones para ver la memoria total utilizada por un rol específico y cada instancia de rol. 

 ![La captura de pantalla muestra datos de métricas.](./media/collect-custom-metrics-guestos-vm-cloud-service-classic/metrics-graph.png)

## <a name="next-steps"></a>Pasos siguientes

- Más información acerca de las [métricas personalizadas](./metrics-custom-overview.md).
