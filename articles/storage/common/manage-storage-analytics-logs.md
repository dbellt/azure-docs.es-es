---
title: Habilitación y administración de los registros de Azure Storage Analytics (clásico) | Microsoft Docs
description: Aprenda a supervisar una cuenta de almacenamiento en Azure usando Azure Storage Analytics.
author: normesta
ms.service: storage
ms.topic: conceptual
ms.date: 01/29/2021
ms.author: normesta
ms.reviewer: fryu
ms.subservice: common
ms.custom: monitoring
ms.openlocfilehash: 7634a07ab3a0026a06f959c99e7b104a9dc91650
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110459728"
---
# <a name="enable-and-manage-azure-storage-analytics-logs-classic"></a>Habilitación y administración de los registros de Azure Storage Analytics (clásico)

[Azure Storage Analytics](storage-analytics.md) proporciona registros para blobs, colas y tablas. Puede usar [Azure Portal](https://portal.azure.com) para configurar los registros que se registran para su cuenta. En este artículo se muestra cómo habilitar y administrar los registros. Para obtener información sobre cómo habilitar las métricas, vea [Habilitación y administración de las métricas de Azure Storage Analytics (clásico)]().  Existen costos asociados con el análisis y el almacenamiento de los datos de supervisión en Azure Portal. Para más información, consulte [Storage Analytics](storage-analytics.md).

> [!NOTE]
> Se recomienda usar los registros de Azure Storage en Azure Monitor en lugar de los registros de Storage Analytics. Los registros de Azure Storage en Azure Monitor están en versión preliminar pública, además de estar disponibles para pruebas de versión preliminar en todas las regiones de nube pública. Esta versión preliminar habilita los registros de blobs (que incluye Azure Data Lake Storage Gen2), archivos, colas y tablas. Para más información, consulte cualquiera de los siguientes artículos:
>
> - [Supervisión de Azure Blob Storage](../blobs/monitor-blob-storage.md)
> - [Supervisión de Azure Files](../files/storage-files-monitoring.md)
> - [Supervisión de Azure Queue Storage](../queues/monitor-queue-storage.md)
> - [Supervisión de Azure Table Storage](../tables/monitor-table-storage.md)

Para obtener orientación exhaustiva sobre el uso de análisis de almacenamiento y otras herramientas para identificar, diagnosticar y solucionar problemas relacionados con Azure Storage, consulte [Supervisión, diagnóstico y solución de problemas de Microsoft Azure Storage](storage-monitoring-diagnosing-troubleshooting.md).

<a id="configure-logging"></a>

## <a name="enable-logs"></a>Habilitamiento de registros

También puede indicar a Azure Storage que guarde los registros de diagnóstico de lectura, escritura y eliminación de las solicitudes de los servicios Blob, Tabla y Cola. La directiva de retención de datos que estableció también se aplica a estos registros.

> [!NOTE]
> Azure Files admite actualmente las métricas de Storage Analytics, pero no admite los registros de Storage Analytics.

### <a name="portal"></a>[Portal](#tab/azure-portal)

1. En [Azure Portal](https://portal.azure.com), seleccione **Cuentas de almacenamiento** y, después, seleccione el nombre de la cuenta de almacenamiento para abrir la hoja de esta.

2. Seleccione **Configuración de diagnóstico (clásica)** en la sección **Supervisión (clásica)** de la hoja de menú.

    ![Elemento de menú Diagnóstico de la sección SUPERVISIÓN de Azure Portal.](./media/manage-storage-analytics-logs/storage-enable-metrics-00.png)

3. Asegúrese de que el **estado** está establecido en **Activado** y seleccione los **servicios** para los que desea habilitar el registro.

   > [!div class="mx-imgBorder"]
   > ![Configure el registro en Azure Portal.](./media/manage-storage-analytics-logs/enable-diagnostics.png)    


4. Asegúrese de que la casilla **Eliminar datos** está activada.  A continuación, establezca el número de días durante los que desea conservar los datos de registro moviendo el control deslizante debajo de la casilla, o modificando directamente el valor que aparece en el cuadro de texto situado al lado del control deslizante. El valor predeterminado para las nuevas cuentas de almacenamiento es de siete días. Si no desea configurar una directiva de retención, escriba un cero. Si no existe una directiva de retención, es posible eliminar los datos de registro.

   > [!WARNING]
   > Los registros se almacenan como datos en la cuenta. Los datos de registro se pueden acumular en la cuenta a lo largo del tiempo, lo que puede aumentar el costo de almacenamiento. Si necesita datos de registro solo durante un breve período de tiempo, puede reducir los costos mediante la modificación de la directiva de retención de datos. El sistema eliminará los datos de registro obsoletos (aquellos anteriores a la directiva de retención de datos). Es recomendable configurar una directiva de retención basada en el tiempo que desee retener los datos de registro de su cuenta. Vea [Facturación según las métricas de almacenamiento](storage-analytics-metrics.md#billing-on-storage-metrics) para más información.

4. Haga clic en **Save**(Guardar).

   Los registros de diagnóstico se guardan en un contenedor de blobs denominado *$logs* en su cuenta de almacenamiento. Para ver los datos de registro se puede usar un explorador de almacenamiento como el [Explorador de Microsoft Azure Storage](https://storageexplorer.com), o bien, mediante programación, la biblioteca de cliente de almacenamiento o PowerShell.

   Para más información sobre el acceso al contenedor $logs, vea [Registro de Storage Analytics](storage-analytics-logging.md).

### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

1. Abra una ventana de comando de Windows PowerShell.

2. Inicie sesión en la suscripción a Azure con el comando `Connect-AzAccount` y siga las instrucciones de la pantalla.

   ```powershell
   Connect-AzAccount
   ```

3. Si su identidad está asociada a más de una suscripción, establezca la suscripción activa.

   ```powershell
   $context = Get-AzSubscription -SubscriptionId <subscription-id>
   Set-AzContext $context
   ```

   Reemplace el valor de marcador de posición `<subscription-id>` por el identificador de la suscripción.

5. Obtenga el contexto de la cuenta de almacenamiento que define la cuenta de almacenamiento que desea usar.

   ```powershell
   $storageAccount = Get-AzStorageAccount -ResourceGroupName "<resource-group-name>" -AccountName "<storage-account-name>"
   $ctx = $storageAccount.Context
   ```

   * Reemplace el marcador de posición `<resource-group-name>` por el nombre del grupo de recursos.

   * Reemplace el valor de marcador de posición `<storage-account-name>` por el nombre de la cuenta de almacenamiento. 

6. Use **Set-AzStorageServiceLoggingProperty** para cambiar la configuración de registro actual. Los cmdlets que controlan el registro de almacenamiento usan un parámetro **LoggingOperations**, que es una cadena que contiene una lista separada por comas de los tipos de solicitud que se van a registrar. Los tres tipos de solicitud posibles son **read**, **write** y **delete**. Para desactivar el registro, use el valor **none** en el parámetro **LoggingOperations**.  

   Con el siguiente comando se activa el registro de solicitudes de lectura, escritura y eliminación en el servicio Queue service de la cuenta de almacenamiento predeterminada, con un período de retención establecido en cinco días:  

   ```powershell
   Set-AzStorageServiceLoggingProperty -ServiceType Queue -LoggingOperations read,write,delete -RetentionDays 5 -Context $ctx
   ```  

   > [!WARNING]
   > Los registros se almacenan como datos en la cuenta. Los datos de registro se pueden acumular en la cuenta a lo largo del tiempo, lo que puede aumentar el costo de almacenamiento. Si necesita datos de registro solo durante un breve período de tiempo, puede reducir los costos mediante la modificación de la directiva de retención de datos. El sistema eliminará los datos de registro obsoletos (aquellos anteriores a la directiva de retención de datos). Es recomendable configurar una directiva de retención basada en el tiempo que desee retener los datos de registro de su cuenta. Vea [Facturación según las métricas de almacenamiento](storage-analytics-metrics.md#billing-on-storage-metrics) para más información.
   
   Con el siguiente comando se desactiva el registro en el servicio Table service de la cuenta de almacenamiento predeterminada:  

   ```powershell
   Set-AzStorageServiceLoggingProperty -ServiceType Table -LoggingOperations none -Context $ctx 
   ```  

   Para más información sobre cómo configurar los cmdlets de Azure PowerShell para que funcionen con su suscripción de Azure y cómo seleccionar la cuenta de almacenamiento predeterminada que quiere usar, vea: [Cómo instalar y configurar Azure PowerShell](/powershell/azure/).  

### <a name="net-v12-sdk"></a>[SDK de .NET, versión 12](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/queues/howto/dotnet/dotnet-v12/Monitoring.cs" id="snippet_EnableDiagnosticLogs":::

### <a name="net-v11-sdk"></a>[SDK de .NET, versión 11](#tab/dotnet11)

```csharp
var storageAccount = CloudStorageAccount.Parse(connStr);  
var queueClient = storageAccount.CreateCloudQueueClient();  
var serviceProperties = queueClient.GetServiceProperties();  

serviceProperties.Logging.LoggingOperations = LoggingOperations.All;  
serviceProperties.Logging.RetentionDays = 2;  

queueClient.SetServiceProperties(serviceProperties);  
``` 

---

<a id="modify-retention-policy"></a>

## <a name="modify-log-data-retention-period"></a>Modificación del período de retención de los datos de registro

Los datos de registro se pueden acumular en la cuenta a lo largo del tiempo, lo que puede aumentar el costo de almacenamiento. Si necesita datos de registro solo durante un breve período de tiempo, puede reducir los costos mediante la modificación del período de retención de los datos de registro. Por ejemplo, si necesita registros solo durante 3 días, establezca el período de retención de los datos de registro en un valor de `3`. De este modo, los registros se eliminarán automáticamente de la cuenta después de 3 días. En esta sección se muestra cómo ver el período de retención de los datos de registro actual y, después, actualizar ese período si es lo que desea hacer.

> [!NOTE]
> Estos pasos solo se aplican a las cuentas que no tienen el **Espacio de nombres jerárquico** habilitado. Si ha habilitado esa configuración en la cuenta, aún no se admite el valor de los días de retención. En su lugar, tendrá que eliminar los registros manualmente mediante cualquier herramienta compatible, como el Explorador de Azure Storage, REST o un SDK. Para encontrar los registros en la cuenta de almacenamiento, vea [Cómo se almacenan los registros](storage-analytics-logging.md#how-logs-are-stored).

### <a name="portal"></a>[Portal](#tab/azure-portal)

1. En [Azure Portal](https://portal.azure.com), seleccione **Cuentas de almacenamiento** y, después, seleccione el nombre de la cuenta de almacenamiento para abrir la hoja de esta.
2. Seleccione **Configuración de diagnóstico (clásica)** en la sección **Supervisión (clásica)** de la hoja de menú.

    ![Elemento de menú Diagnóstico de la sección SUPERVISIÓN de Azure Portal](./media/manage-storage-analytics-logs/storage-enable-metrics-00.png)

3. Asegúrese de que la casilla **Eliminar datos** está activada.  A continuación, establezca el número de días durante los que desea conservar los datos de registro moviendo el control deslizante debajo de la casilla, o modificando directamente el valor que aparece en el cuadro de texto situado al lado del control deslizante.

   > [!div class="mx-imgBorder"]
   > ![Modificar el período de retención en Azure Portal](./media/manage-storage-analytics-logs/modify-retention-period.png)

   El número predeterminado de días para las nuevas cuentas de almacenamiento es de 7 días. Si no desea configurar una directiva de retención, escriba un cero. Si no existe una directiva de retención, es posible eliminar los datos de supervisión.
   
4. Haga clic en **Save**(Guardar).

   Los registros de diagnóstico se guardan en un contenedor de blobs denominado *$logs* en su cuenta de almacenamiento. Para ver los datos de registro se puede usar un explorador de almacenamiento como el [Explorador de Microsoft Azure Storage](https://storageexplorer.com), o bien, mediante programación, la biblioteca de cliente de almacenamiento o PowerShell.

   Para más información sobre el acceso al contenedor $logs, vea [Registro de Storage Analytics](storage-analytics-logging.md).

### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

1. Abra una ventana de comando de Windows PowerShell.

2. Inicie sesión en la suscripción a Azure con el comando `Connect-AzAccount` y siga las instrucciones de la pantalla.

   ```powershell
   Connect-AzAccount
   ```

3. Si su identidad está asociada a más de una suscripción, establezca la suscripción activa.

   ```powershell
   $context = Get-AzSubscription -SubscriptionId <subscription-id>
   Set-AzContext $context
   ```

   Reemplace el valor de marcador de posición `<subscription-id>` por el identificador de la suscripción.

5. Obtenga el contexto de la cuenta de almacenamiento que define la cuenta de almacenamiento.

   ```powershell
   $storageAccount = Get-AzStorageAccount -ResourceGroupName "<resource-group-name>" -AccountName "<storage-account-name>"
   $ctx = $storageAccount.Context
   ```

   * Reemplace el marcador de posición `<resource-group-name>` por el nombre del grupo de recursos.

   * Reemplace el valor de marcador de posición `<storage-account-name>` por el nombre de la cuenta de almacenamiento. 

6. Use [Get-AzStorageServiceLoggingProperty](/powershell/module/az.storage/get-azstorageserviceloggingproperty) para ver la directiva de retención de registros actual. En el ejemplo siguiente, se imprime en la consola el período de retención para los servicios de almacenamiento de blobs y colas.

   ```powershell
   Get-AzStorageServiceLoggingProperty -ServiceType Blob, Queue -Context $ctx
   ```  

   En el resultado de la consola, el período de retención aparece bajo el encabezado de columna `RetentionDays`.

   > [!div class="mx-imgBorder"]
   > ![Directiva de retención en la salida de PowerShell](./media/manage-storage-analytics-logs/retention-period-powershell.png)

7. Use [Set-AzStorageServiceLoggingProperty](/powershell/module/az.storage/set-azstorageserviceloggingproperty) para cambiar el período de retención. En el ejemplo siguiente se cambia el período de retención a 4 días.  

   ```powershell
   Set-AzStorageServiceLoggingProperty -ServiceType Blob, Queue -RetentionDays 4 -Context $ctx
   ```  

   Para más información sobre cómo configurar los cmdlets de Azure PowerShell para que funcionen con su suscripción de Azure y cómo seleccionar la cuenta de almacenamiento predeterminada que quiere usar, vea: [Cómo instalar y configurar Azure PowerShell](/powershell/azure/).  

### <a name="net-v12-sdk"></a>[SDK de .NET, versión 12](#tab/dotnet)

En el ejemplo siguiente, se imprime en la consola el período de retención para los servicios de almacenamiento de blobs y colas.

:::code language="csharp" source="~/azure-storage-snippets/queues/howto/dotnet/dotnet-v12/Monitoring.cs" id="snippet_ViewRetentionPeriod":::

En el ejemplo siguiente se cambia el período de retención a 4 días. 

:::code language="csharp" source="~/azure-storage-snippets/queues/howto/dotnet/dotnet-v12/Monitoring.cs" id="snippet_ModifyRetentionPeriod":::

### <a name="net-v11-sdk"></a>[SDK de .NET, versión 11](#tab/dotnet11)

En el ejemplo siguiente, se imprime en la consola el período de retención para los servicios de almacenamiento de blobs y colas.

```csharp
var storageAccount = CloudStorageAccount.Parse(connectionString);

var blobClient = storageAccount.CreateCloudBlobClient();
var queueClient = storageAccount.CreateCloudQueueClient();

var blobserviceProperties = blobClient.GetServiceProperties();
var queueserviceProperties = queueClient.GetServiceProperties();

Console.WriteLine("Retention period for logs from the blob service is: " +
   blobserviceProperties.Logging.RetentionDays.ToString());

Console.WriteLine("Retention period for logs from the queue service is: " +
   queueserviceProperties.Logging.RetentionDays.ToString());
```

En el ejemplo siguiente, se cambia el período de retención de los registros para los servicios de almacenamiento de blobs y colas a 4 días. 

```csharp

blobserviceProperties.Logging.RetentionDays = 4;
queueserviceProperties.Logging.RetentionDays = 4;

blobClient.SetServiceProperties(blobserviceProperties);
queueClient.SetServiceProperties(queueserviceProperties);  
``` 

---

### <a name="verify-that-log-data-is-being-deleted"></a>Comprobación de la eliminación de los datos de registro

Puede comprobar la eliminación de los registros; para ello, consulte el contenido del contenedor `$logs` de la cuenta de almacenamiento. En la imagen siguiente se muestra el contenido de una carpeta en el contenedor `$logs`. La carpeta corresponde a enero de 2021, y cada carpeta contiene los registros durante un día. Si el día actual era el 29 de enero de 2021 y la directiva de retención se establece solo en un día, esta carpeta solo debe contener registros durante un día.

> [!div class="mx-imgBorder"]
> ![Lista de carpetas de registros en Azure Portal](./media/manage-storage-analytics-logs/verify-and-delete-logs.png)

<a id="download-storage-logging-log-data"></a>

## <a name="view-log-data"></a>Visualización de los datos de registro

 Para ver y analizar los datos de registro, conviene descargar en un equipo local los blobs que contengan los datos de registro de su interés. Muchas herramientas de exploración del almacenamiento permiten descargar blobs de la cuenta de almacenamiento; también puede usar la herramienta de la línea de comandos para copiar de Azure [AzCopy](storage-use-azcopy-v10.md), suministrada por el equipo de Azure Storage, para descargar los datos de registro.  
 
>[!NOTE]
> El contenedor `$logs` no está integrado con Event Grid, por lo que no recibirá notificaciones cuando se escriban en los archivos de registro. 

 Para estar seguro de que descarga los datos de registro de su interés y no descarga los mismos datos de registro más de una vez:  

-   Use la convención de nomenclatura de fecha y hora en los blobs que contienen datos de registro para realizar un seguimiento de los blobs que ya ha descargado para analizarlos y, así, evitar volver a descargar los mismos datos más de una vez.  

-   Use los metadatos de los blobs que contienen datos de registro para identificar el período específico en el que el blob contiene los datos de registro y, así, detectar el blob exacto que necesita descargar.  

Para empezar a trabajar con AzCopy, consulte [Introducción a AzCopy](storage-use-azcopy-v10.md). 

En el siguiente ejemplo se muestra cómo descargar los datos de registro de Queue service correspondientes a las horas 9 A.M., 10 A.M. y 11 A.M. del 20 de mayo de 2014.

```
azcopy copy 'https://mystorageaccount.blob.core.windows.net/$logs/queue' 'C:\Logs\Storage' --include-path '2014/05/20/09;2014/05/20/10;2014/05/20/11' --recursive
```

Para obtener más información sobre cómo descargar archivos específicos, vea [Descarga de blobs de Azure Blob Storage mediante AzCopy v10](./storage-use-azcopy-blobs-download.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json).

Cuando haya descargado los datos de registro, puede ver las entradas de registro en los archivos. Estos archivos de registro usan un formato de texto delimitado que muchas herramientas de lectura de registros son capaces de analizar; para más información, consulte la guía [Supervisión, diagnóstico y solución de problemas de Microsoft Azure Storage](storage-monitoring-diagnosing-troubleshooting.md). Cada herramienta tiene diferentes recursos para aplicar formato, filtrar, ordenar y buscar contenido en los archivos de registro. Para obtener más información sobre el contenido y el formato de los archivos de registro del registro de almacenamiento, vea [Formato de registros de Storage Analytics](/rest/api/storageservices/storage-analytics-log-format) y [Operaciones registradas y mensajes de estado de Storage Analytics](/rest/api/storageservices/storage-analytics-logged-operations-and-status-messages).

## <a name="next-steps"></a>Pasos siguientes

* Para obtener más información sobre Storage Analytics, vea [Storage Analytics](storage-analytics.md) para Storage Analytics.
* Para obtener más información sobre cómo usar un lenguaje .NET para configurar el registro de almacenamiento, vea [Referencia de biblioteca de cliente de almacenamiento](/previous-versions/azure/dn261237(v=azure.100)). 
* Para obtener información general sobre cómo configurar el registro de almacenamiento con la API de REST, vea [Habilitar y configurar Storage Analytics](/rest/api/storageservices/Enabling-and-Configuring-Storage-Analytics).
* Obtenga más información sobre el formato de los registros de Storage Analytics. Vea [Formato del registro de Storage Analytics](/rest/api/storageservices/storage-analytics-log-format).
