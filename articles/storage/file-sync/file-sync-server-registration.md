---
title: Administración de servidores registrados con Azure File Sync | Microsoft Docs
description: Aprenda a registrar y anular el registro de una instancia de Windows Server con un servicio de sincronización de almacenamiento de Azure File Sync.
author: roygara
ms.service: storage
ms.topic: how-to
ms.date: 04/13/2021
ms.author: rogarana
ms.subservice: files
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 2e0e42cb89e4fa15046f3b2de2f43637ca975bfe
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110679156"
---
# <a name="manage-registered-servers-with-azure-file-sync"></a>Administración de servidores registrados con Azure File Sync
Azure File Sync le permite centralizar los recursos compartidos de archivos de su organización en Azure Files sin renunciar a la flexibilidad, el rendimiento y la compatibilidad de un servidor de archivos local. Para ello, transforma los servidores de Windows Server en una caché rápida del recurso compartido de archivos de Azure. Puede usar cualquier protocolo disponible en Windows Server para tener acceso a los datos localmente (incluidos SMB, NFS y FTPS) y puede tener tantas cachés según sea necesario en todo el mundo.

En el artículo siguiente se ilustra cómo registrar y administrar un servidor con un servicio de sincronización de almacenamiento. Para más información sobre cómo implementar Azure File Sync de un extremo a otro, vea [How to deploy Azure File Sync](file-sync-deployment-guide.md) (Implementación de Azure Files Sync).

## <a name="registerunregister-a-server-with-storage-sync-service"></a>Registro y anulación del registro de un servidor con el servicio de sincronización de almacenamiento
Al registrar un servidor con Azure File Sync se establece una relación de confianza entre Windows Server y Azure. A continuación, se puede usar esta relación para crear *puntos de conexión de servidor* en el servidor, que representan carpetas concretas que deben sincronizarse con un recurso compartido de archivos de Azure (también conocido como *punto de conexión de nube*). 

### <a name="prerequisites"></a>Requisitos previos
Para registrar un servidor con un servicio de sincronización de almacenamiento, debe preparar el servidor con los requisitos previos necesarios:

* El servidor debe ejecutar una versión compatible de Windows Server. Para más información, consulte [Requisitos del sistema de Azure File Sync e interoperabilidad](file-sync-planning.md#windows-file-server-considerations).
* Asegúrese de que se ha implementado un servicio de sincronización de almacenamiento. Para más información sobre cómo implementar un servicio de sincronización de almacenamiento, vea [How to deploy Azure File Sync](file-sync-deployment-guide.md) (Implementación de Azure Files Sync).
* Asegúrese de que el servidor está conectado a Internet y que se puede acceder a Azure.
* Deshabilite la configuración de seguridad mejorada de IE para administradores con la interfaz de usuario del Administrador del servidor.
    
    ![Interfaz de usuario del Administrador del servidor con la configuración de seguridad mejorada de IE resaltada](media/storage-sync-files-server-registration/server-manager-ie-config.png)

* Asegúrese de que el módulo Azure PowerShell esté instalado en el servidor. Si el servidor es miembro de un clúster de conmutación por error, todos los nodos del clúster necesitarán el módulo Az. Se pueden encontrar más detalles sobre cómo instalar el módulo Az en [Instalación del módulo de Azure PowerShell](/powershell/azure/install-Az-ps).

    > [!Note]  
    > Se recomienda usar la versión más reciente del módulo Azure PowerShell para registrar o anular el registro de un servidor. Si el paquete Az se ha instalado anteriormente en este servidor (y la versión de PowerShell en este servidor es 5.* o superior), puede usar el cmdlet `Update-Module` para actualizar este paquete. 
* Si utiliza un servidor proxy de red en su entorno, configure el proxy en el servidor para que lo utilice el agente de sincronización.
    1. Determine la dirección IP y el número de puerto del proxy.
    2. Edite estos dos archivos:
        * C:\Windows\Microsoft.NET\Framework64\v4.0.30319\Config\machine.config
        * C:\Windows\Microsoft.NET\Framework\v4.0.30319\Config\machine.config
    3. Agregue las líneas de la figura 1 (debajo de esta sección) a /System.ServiceModel en los dos archivos anteriores cambiando 127.0.0.1:8888 por la dirección IP correcta (reemplace 127.0.0.1) y el número de puerto correcto (reemplace 8888):
    4. Establezca la configuración del proxy WinHTTP mediante la línea de comandos:
        * Mostrar el proxy: netsh winhttp show proxy
        * Establecer el proxy: netsh winhttp set proxy 127.0.0.1:8888
        * Restablecer el proxy: netsh winhttp reset proxy
        * Si se configura esto después de instalar el agente, reinicie el agente de sincronización: net stop filesyncsvc
    
```XML
    Figure 1:
    <system.net>
        <defaultProxy enabled="true" useDefaultCredentials="true">
            <proxy autoDetect="false" bypassonlocal="false" proxyaddress="http://127.0.0.1:8888" usesystemdefault="false" />
        </defaultProxy>
    </system.net>
```    

### <a name="register-a-server-with-storage-sync-service"></a>Registro de un servidor con el servicio de sincronización de almacenamiento
Antes de usar un servidor como *punto de conexión del servidor* en un *grupo de sincronización* de Azure File Sync, se debe registrar con un *servicio de sincronización de almacenamiento*. Un servidor solo se puede registrar con un único servicio de sincronización de almacenamiento al mismo tiempo.

#### <a name="install-the-azure-file-sync-agent"></a>Instalación del agente de Azure File Sync
1. [Descargue el agente de Azure File Sync](https://go.microsoft.com/fwlink/?linkid=858257).
2. Inicie el instalador del agente de Azure File Sync.
    
    ![El primer panel del instalador del agente de Azure File Sync](media/storage-sync-files-server-registration/install-afs-agent-1.png)

3. Asegúrese de habilitar las actualizaciones del agente de Azure File Sync mediante Microsoft Update. Esto es importante porque las correcciones críticas de seguridad y las mejoras de características del paquete de servidor se envían a través de Microsoft Update.

    ![Asegúrese de que Microsoft Update está habilitado en el panel de Microsoft Update del instalador del agente de Azure File Sync.](media/storage-sync-files-server-registration/install-afs-agent-2.png)

4. Si el servidor no se ha registrado anteriormente, se abre inmediatamente la interfaz de usuario de registro del servidor tras finalizar la instalación.

> [!Important]  
> Si el servidor es miembro de un clúster de conmutación por error, el agente de Azure File Sync se deberá instalar en cada nodo del clúster.

#### <a name="register-the-server-using-the-server-registration-ui"></a>Registro del servidor mediante la interfaz de usuario de registro del servidor
1. Si la interfaz de usuario de registro del servidor no se inició inmediatamente después de finalizar la instalación del agente de Azure File Sync, se puede iniciar manualmente ejecutando `C:\Program Files\Azure\StorageSyncAgent\ServerRegistration.exe`.
2. Haga clic en *Iniciar sesión* para acceder a la suscripción de Azure. 

    ![Apertura del cuadro de diálogo de la interfaz de usuario de registro de servidor](media/storage-sync-files-server-registration/server-registration-ui-1.png)

3. En el cuadro de diálogo, seleccione la suscripción correcta, el grupo de recursos y el servicio de sincronización de almacenamiento.

    ![Información del servicio de sincronización de almacenamiento](media/storage-sync-files-server-registration/server-registration-ui-2.png)

4. En la versión preliminar, se necesita otro inicio de sesión más para completar el proceso. 

    ![Cuadro de diálogo de inicio de sesión](media/storage-sync-files-server-registration/server-registration-ui-3.png)

> [!Important]  
> Si el servidor es miembro de un clúster de conmutación por error, cada servidor debe ejecutar el registro de servidor. Cuando ve los servidores registrados en Azure Portal, Azure File Sync reconoce automáticamente cada nodo como miembro del mismo clúster de conmutación por error y los agrupa de la manera adecuada.

#### <a name="register-the-server-with-powershell"></a>Registro del servidor con PowerShell
También puede realizar el registro del servidor a través de PowerShell. 

```powershell
Register-AzStorageSyncServer -ResourceGroupName "<your-resource-group-name>" -StorageSyncServiceName "<your-storage-sync-service-name>"
```

### <a name="unregister-the-server-with-storage-sync-service"></a>Cancelación del registro del servidor del servicio de sincronización de almacenamiento
Hay varios pasos que son necesarios para anular el registro de un servidor del servicio de sincronización de almacenamiento. A continuación se indica cómo anular el registro correctamente de un servidor.

> [!Warning]  
> No intente solucionar los problemas de sincronización, niveles de la nube ni otros aspectos de Azure File Sync mediante la anulación del registro y posterior registro de un servidor o mediante la eliminación y nueva creación de los puntos de conexión de un servidor, salvo que lo indique explícitamente un ingeniero de Microsoft. La anulación del registro de un servidor y eliminación de sus puntos de conexión es una operación destructiva y los archivos en capas de los volúmenes con puntos de conexión de servidor no se "volverán a conectar" a sus ubicaciones del recurso compartido de archivos de Azure después de que el servidor registrado y los punto de conexión del servidor se creen de nuevo, lo que provocará errores de sincronización. Tenga también en cuenta que los archivos en capas que haya fuera de un espacio de nombres de un punto de conexión de servidor pueden perderse de forma permanente. Puede haber archivos en capas en los puntos de conexión de un servidor aunque nunca se hayan habilitado los niveles de la nube.

#### <a name="optional-recall-all-tiered-data"></a>(Opcional) Recuperación de todos los datos con niveles
Si desea que los archivos que actualmente están organizados en niveles estén disponibles después de quitar Azure File Sync (este es un entorno de producción, no de prueba), recupere todos los archivos de todos los volúmenes que contengan puntos de conexión de servidor. Deshabilite los niveles de la nube en todos los puntos de conexión del servidor y ejecute el siguiente cmdlet de PowerShell:

```powershell
Import-Module "C:\Program Files\Azure\StorageSyncAgent\StorageSync.Management.ServerCmdlets.dll"
Invoke-StorageSyncFileRecall -Path <a-volume-with-server-endpoints-on-it>
```

> [!Warning]  
> Si el volumen local que hospeda el punto de conexión de servidor no tiene suficiente espacio disponible para recuperar todos los datos con niveles, el cmdlet `Invoke-StorageSyncFileRecall` dará error.  

#### <a name="remove-the-server-from-all-sync-groups"></a>Eliminación del servidor de todos los grupos de sincronización
Antes de anular el registro del servidor en el servicio de sincronización de almacenamiento, se deben quitar todos los puntos de conexión de servidor de ese servidor. Esta acción también se puede hacer mediante Azure Portal:

1. Vaya al servicio de sincronización de almacenamiento donde está registrado el servidor.
2. Quite todos los puntos de conexión de servidor de este servidor en cada grupo de sincronización del servicio de sincronización de almacenamiento. Para ello, haga clic con el botón derecho en el punto de conexión de servidor que le interese en el panel Grupo de sincronización.

    ![Eliminación de un punto de conexión de servidor de un grupo de sincronización](media/storage-sync-files-server-registration/sync-group-server-endpoint-remove-1.png)

Esta acción también se puede realizar con un script simple de PowerShell:

```powershell
Connect-AzAccount

$storageSyncServiceName = "<your-storage-sync-service>"
$resourceGroup = "<your-resource-group>"

Get-AzStorageSyncGroup -ResourceGroupName $resourceGroup -StorageSyncServiceName $storageSyncServiceName | ForEach-Object { 
    $syncGroup = $_; 
    Get-AzStorageSyncServerEndpoint -ParentObject $syncGroup | Where-Object { $_.ServerEndpointName -eq $env:ComputerName } | ForEach-Object { 
        Remove-AzStorageSyncServerEndpoint -InputObject $_ 
    } 
}
```

#### <a name="unregister-the-server"></a>Anulación del registro del servidor
Ahora que todos los datos se han recuperado y que el servidor se ha quitado de todos los grupos de sincronización, se puede anular el registro del servidor. 

1. En Azure Portal, vaya a la sección *Servidores registrados* del servicio de sincronización de almacenamiento.
2. Haga clic con el botón derecho en el servidor cuyo registro desea anular y haga clic en "Cancelar registro de servidor".

    ![Anulación del registro del servidor](media/storage-sync-files-server-registration/unregister-server-1.png)

## <a name="ensuring-azure-file-sync-is-a-good-neighbor-in-your-datacenter"></a>Configuración de Azure File Sync para que sea un buen vecino en el centro de datos 
Puesto que Azure File Sync rara vez será el único servicio en ejecución en el centro de datos, es posible que le interese limitar el uso de red y de almacenamiento de Azure File Sync.

> [!Important]  
> Si establece límites demasiado bajos, el rendimiento de la sincronización y la recuperación de Azure File Sync se verá afectado.

### <a name="set-azure-file-sync-network-limits"></a>Establecimiento de límites de red de Azure File Sync
Puede limitar el uso que hace Azure File Sync de la red mediante los cmdlets `StorageSyncNetworkLimit`.

> [!Note]  
> Los límites de red no se aplican cuando se tiene acceso a un archivo en capas.

Por ejemplo, puede crear un nuevo límite del regulador de carga de trabajo para asegurarse de que Azure File Sync no utiliza más de 10 Mbps entre las 9 a. m. y las 5 p. m. (17:00 h) durante la semana laboral: 

```powershell
Import-Module "C:\Program Files\Azure\StorageSyncAgent\StorageSync.Management.ServerCmdlets.dll"
New-StorageSyncNetworkLimit -Day Monday, Tuesday, Wednesday, Thursday, Friday -StartHour 9 -EndHour 17 -LimitKbps 10000
```

Puede ver el límite mediante el siguiente cmdlet:

```powershell
Get-StorageSyncNetworkLimit # assumes StorageSync.Management.ServerCmdlets.dll is imported
```

Para quitar límites de red, use `Remove-StorageSyncNetworkLimit`. Por ejemplo, el comando siguiente quita todos los límites de red:

```powershell
Get-StorageSyncNetworkLimit | ForEach-Object { Remove-StorageSyncNetworkLimit -Id $_.Id } # assumes StorageSync.Management.ServerCmdlets.dll is imported
```

### <a name="use-windows-server-storage-qos"></a>Uso de la calidad de servicio de almacenamiento de Windows Server 
Cuando Azure File Sync se hospeda en una máquina virtual que se ejecuta en un host de virtualización de Windows Server, puede usar la calidad de servicio de almacenamiento (QoS de almacenamiento) para regular el consumo de E/S de almacenamiento. La directiva de QoS de almacenamiento puede establecerse como un valor máximo (o límite, como el límite StorageSyncNetwork aplicado anteriormente) o como un valor mínimo (o reserva). Al establecer un mínimo en lugar de un máximo, Azure File Sync puede usar el ancho de banda de almacenamiento disponible si otras cargas de trabajo no lo están utilizando. Para más información, consulte [Calidad de servicio de almacenamiento](/windows-server/storage/storage-qos/storage-qos-overview).

## <a name="see-also"></a>Consulte también
- [Planeamiento de una implementación de Azure File Sync](file-sync-planning.md)
- [Implementación de Azure File Sync](file-sync-deployment-guide.md)
- [Supervisión de Azure File Sync](file-sync-monitoring.md)
- [Solución de problemas de Azure File Sync](file-sync-troubleshoot.md)
