---
title: Uso de DFS-N con Azure Files
description: Casos de uso habituales de DFS-N con Azure Files
author: roygara
ms.service: storage
ms.topic: how-to
ms.date: 3/02/2021
ms.author: rogarana
ms.subservice: files
ms.openlocfilehash: f571fa6e04d19412db856a42232cb2d151ceb6ab
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108141494"
---
# <a name="how-to-use-dfs-namespaces-with-azure-files"></a>Uso de Espacios de nombres DFS con Azure Files
[Espacios de nombres del Sistema de archivos distribuido](/windows-server/storage/dfs-namespaces/dfs-overview), normalmente conocido como Espacios de nombres DFS o DFS-N, es un rol de servidor de Windows Server muy utilizado para simplificar la implementación y el mantenimiento de los recursos compartidos de archivos SMB en la producción. Espacios de nombres DFS es una tecnología de virtualización de espacios de nombres de almacenamiento, lo que significa que le permite proporcionar una capa de direccionamiento indirecto entre la ruta de acceso UNC de los recursos compartidos de archivos y los recursos compartidos de archivos en sí. Los espacios de nombres DFS funcionan con recursos compartidos de archivos SMB y se hospedan con independencia de estos: se pueden usar con recursos compartidos de SMB hospedados en un servidor de archivos de Windows local con o sin Azure File Sync, recursos compartidos de archivos de Azure directamente, recursos compartidos de archivos SMB hospedados en Azure NetApp Files u otras ofertas de terceros e incluso con recursos compartidos de archivos hospedados en otras nubes. 

Básicamente, Espacios de nombres DFS proporciona una asignación entre una ruta de acceso UNC descriptiva, como `\\contoso\shares\ProjectX`, y la ruta de acceso UNC subyacente del recurso compartido SMB, como `\\Server01-Prod\ProjectX` o `\\storageaccount.file.core.windows.net\projectx`. Cuando el usuario final quiere navegar al recurso compartido de archivos, escribe la ruta de acceso UNC descriptiva, pero el cliente SMB accede a la ruta de acceso SMB subyacente de la asignación. También puede ampliar este concepto básico para que asuma un nombre de servidor de archivos existente, como `\\MyServer\ProjectX`. Puede usar esta funcionalidad para conseguir lo siguiente:

- Proporcionar un nombre a prueba de migraciones para un conjunto lógico de datos. En este ejemplo, tiene una asignación como `\\contoso\shares\Engineering` que se asigna a `\\OldServer\Engineering`. Cuando complete la migración a Azure Files, puede cambiar la asignación para que la ruta de acceso UNC descriptiva apunte a `\\storageaccount.file.core.windows.net\engineering`. Cuando un usuario final acceda a la ruta de acceso UNC descriptiva, se le redirigirá sin problemas a la ruta de acceso del recurso compartido de archivos de Azure.

- Establecer un nombre común para un conjunto lógico de datos que se distribuye a varios servidores en distintos sitios físicos, por ejemplo, a través de Azure File Sync. En este ejemplo, un nombre como `\\contoso\shares\FileSyncExample` se asigna a varias rutas de acceso UNC, como `\\FileSyncServer1\ExampleShare`, `\\FileSyncServer2\DifferentShareName` y `\\FileSyncServer3\ExampleShare`. Cuando el usuario accede a la UNC descriptiva, se le proporciona una lista de rutas de acceso UNC posibles y este elige la más cercana en función de las definiciones de sitio de Windows Server Active Directory (AD).

- Ampliar un conjunto lógico de datos más allá de los umbrales de tamaño, E/S u otras escalas. Esto es habitual cuando se trabaja con directorios de usuario, en los que cada usuario obtiene su carpeta propia en un recurso compartido, o con los recursos compartidos temporales, donde los usuarios obtienen un espacio arbitrario para administrar las necesidades de datos temporales. Con Espacios de nombres DFS, se unen varias carpetas en un espacio de nombres cohesionado. Por ejemplo, `\\contoso\shares\UserShares\user1` se asigna a `\\storageaccount.file.core.windows.net\user1`, `\\contoso\shares\UserShares\user2` se asigna a `\\storageaccount.file.core.windows.net\user2` y así sucesivamente.  

Puede ver un ejemplo de cómo usar Espacios de nombres DFS con su implementación de Azure Files en el vídeo de introducción siguiente.

[![Demostración sobre cómo configurar DFS-N con Azure Files: hacer clic para reproducirla](./media/files-manage-namespaces/video-snapshot-dfsn.png)](https://www.youtube.com/watch?v=jd49W33DxkQ)
> [!NOTE]  
> Vaya al minuto 10:10 del vídeo para ver cómo configurar Espacios de nombres DFS.

Si ya tiene un espacio de nombres DFS, no es necesario realizar ningún paso especial para usarlo con Azure Files y File Sync. Si accede al recurso compartido de archivos de Azure desde el entorno local, se aplican las consideraciones habituales para las redes. Consulte [Consideraciones de redes para Azure Files](./storage-files-networking-overview.md) para obtener más información.

## <a name="namespace-types"></a>Tipos de espacios de nombres
Espacios de nombres DFS proporciona dos tipos de espacio de nombres principales:

- **Espacio de nombres basado en dominio**: un espacio de nombres hospedado como parte del dominio de Windows Server AD. Los espacios de nombres hospedados como parte de AD tendrán una ruta de acceso UNC que contiene el nombre del dominio, por ejemplo, `\\contoso.com\shares\myshare` si el dominio es `contoso.com`. Los espacios de nombres basados en dominio admiten límites de escalado más altos y redundancia integrada a través de AD. Asimismo, no pueden ser un recurso en clúster de conmutación por error. 
- **Espacio de nombres independiente**: un espacio de nombres hospedado en un servidor individual, no como parte de Windows Server AD. El nombre de los espacios de nombres independientes se basarán en el nombre del servidor independiente, como `\\MyStandaloneServer\shares\myshare`, donde el servidor independiente se llama `MyStandaloneServer`. Los espacios de nombres independientes admiten destinos de escala inferiores a los espacios de nombres basados en dominio, pero pueden hospedarse como un recurso en clúster de conmutación por error.

## <a name="requirements"></a>Requisitos
Para usar Espacios de nombres DFS con Azure Files y File Sync, debe tener los recursos siguientes:

- Un dominio de Active Directory. Se puede hospedar en cualquier lugar que quiera, como en el entorno local, en una máquina virtual (VM) de Azure o incluso en otra nube.
- Una instancia de Windows Server que puede hospedar el espacio de nombres. Un patrón de implementación común para los espacios de nombres DFS consiste en usar el controlador de dominio de Active Directory para hospedarlos, si bien los espacios de nombres pueden configurarse desde cualquier servidor que tenga el rol de servidor Espacios de nombres DFS instalado. Espacios de nombres DFS está disponible en todas las versiones compatibles de Windows Server.
- Un recurso compartido de archivos SMB hospedado en un entorno unido a un dominio, como un recurso compartido de archivos de Azure hospedado en una cuenta de almacenamiento unida a un dominio o un recurso compartido de archivos hospedado en un servidor de archivos de Windows unido a un dominio con Azure File Sync. Para obtener más información sobre cómo unir a un dominio la cuenta de almacenamiento, consulte el artículo sobre la [autenticación basada en la identidad](storage-files-active-directory-overview.md). Los servidores de archivos de Windows se unen a un dominio de la misma manera, independientemente de que use Azure File Sync o no.
- Los recursos compartidos de archivos SMB que quiere usar con Espacios de nombres DFS deben ser accesibles desde las redes locales. Esto es un problema sobre todo para los recursos compartidos de archivos de Azure; sin embargo, técnicamente, se aplica a cualquier recurso compartido de archivos hospedado en Azure o en cualquier otra nube. Para obtener más información sobre las redes, consulte el documento sobre [consideraciones de redes para el acceso directo](storage-files-networking-overview.md).

## <a name="install-the-dfs-namespaces-server-role"></a>Instalación del rol de servidor Espacios de nombres DFS
Si ya está usando Espacios de nombres DFS o quiere configurar este rol en el controlador de dominio, puede omitir estos pasos sin riesgo alguno.

# <a name="portal"></a>[Portal](#tab/azure-portal)
Para instalar el rol de servidor Espacios de nombres DFS, abra el Administrador del servidor en el servidor. Seleccione **Administrar** y, a continuación, elija **Agregar roles y características**. El asistente que aparece le guía a través del proceso de instalación de los componentes de Windows necesarios para ejecutar y administrar los espacios de nombres DFS. 

En la sección **Tipo de instalación** del asistente para instalación, seleccione el botón de radio **Instalación basada en características o en roles** y elija **Siguiente**. En la sección **Selección del servidor**, seleccione los servidores en los que quiera instalar el rol de servidor Espacios de nombres DFS y elija **Siguiente.** 

En la sección **Roles de servidor,** seleccione el rol **Espacios de nombres DFS** en la lista de roles. Puede encontrarla en **Servicios de archivos y almacenamiento** > **Servicios de iSCSI y archivo**. Al seleccionar el rol de servidor Espacios de nombres DFS, también puede agregar cualquier característica o rol de servidor compatible necesario que aún no tenga instalados.

![Captura de pantalla del asistente **Agregar roles y características** con el rol **Espacios de nombres DFS** seleccionado.](./media/files-manage-namespaces/dfs-namespaces-install.png)

Después de haber seleccionado el rol **Espacios de nombres DFS**, puede seleccionar **Siguiente** en todas las pantallas posteriores y elegir **Instalar** tan pronto como se habilite el botón en el asistente. Una vez completada la instalación, puede configurar el espacio de nombres.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
Ejecute los comandos siguientes desde una sesión de PowerShell con privilegios elevados (o mediante la comunicación remota de PowerShell).

```PowerShell
Install-WindowsFeature -Name "FS-DFS-Namespace", "RSAT-DFS-Mgmt-Con"
```
---

## <a name="take-over-existing-server-names-with-root-consolidation"></a>Uso de los nombres de servidor existentes con la consolidación raíz
Un uso importante de los espacios de nombres DFS es asumir un nombre de servidor existente para refactorizar el diseño físico de los recursos compartidos de archivos. Por ejemplo, puede que quiera consolidar los recursos compartidos de archivos de varios servidores de archivos antiguos en un único servidor de archivos durante una migración de modernización. Normalmente, la familiaridad del usuario final y la vinculación de documentos limitan la capacidad de consolidar los recursos compartidos de archivos de distintos servidores de archivos en un solo host, pero la característica de consolidación raíz de Espacios de nombres DFS permite establecer un único servidor para varios nombres de servidor y enrutar al nombre de recurso compartido adecuado.

Si bien resulta útil para diversos escenarios de migración de centros de datos, la consolidación raíz es especialmente útil para adoptar recursos compartidos de archivos Azure nativos de nube debido a lo siguiente:

- Los recursos compartidos de archivos de Azure no permiten mantener los nombres de servidores locales existentes.
- El acceso a los recursos compartidos de archivos de Azure debe realizarse a través del nombre de dominio completo (FQDN) de la cuenta de almacenamiento. Por ejemplo, el acceso a un recurso compartido de archivos de Azure llamado `share` en la cuenta de almacenamiento `storageaccount` siempre se realiza a través de la ruta de acceso UNC `\\storageaccount.file.core.windows.net\share`. Esto puede resultar confuso para los usuarios finales que esperan un nombre corto (por ejemplo, `\\MyServer\share`) o un nombre que sea un subdominio del nombre de dominio de la organización (por ejemplo, `\\MyServer.contoso.com\share`).

La consolidación raíz solo se puede usar con los espacios de nombres independientes. Si ya tiene un espacio de nombres basado en dominio para los recursos compartidos de archivos, no es necesario crear un espacio de nombres consolidado raíz.

### <a name="enabling-root-consolidation"></a>Habilitación de la consolidación raíz
Para habilitar la consolidación raíz, establezca las claves del Registro siguientes desde una sesión de PowerShell con privilegios elevados (o mediante la comunicación remota de PowerShell).

```PowerShell
New-Item `
    -Path "HKLM:SYSTEM\CurrentControlSet\Services\Dfs" `
    -Type Registry `
    -ErrorAction SilentlyContinue
New-Item `
    -Path "HKLM:SYSTEM\CurrentControlSet\Services\Dfs\Parameters" `
    -Type Registry `
    -ErrorAction SilentlyContinue
New-Item `
    -Path "HKLM:SYSTEM\CurrentControlSet\Services\Dfs\Parameters\Replicated" `
    -Type Registry `
    -ErrorAction SilentlyContinue
Set-ItemProperty `
    -Path "HKLM:SYSTEM\CurrentControlSet\Services\Dfs\Parameters\Replicated" `
    -Name "ServerConsolidationRetry" `
    -Value 1
```

### <a name="creating-dns-entries-for-existing-file-server-names"></a>Creación de entradas DNS para los nombres de servidores de archivos existentes
Para que los espacios de nombres DFS respondan a nombres de servidores de archivos existentes, cree registros de alias (CNAME) para los servidores de archivos existentes que apunten al nombre del servidor de Espacios de nombres DFS. El procedimiento exacto para actualizar los registros DNS puede depender de los servidores que use la organización y de si en esta se utilizan herramientas personalizadas para automatizar la administración de DNS. Los pasos siguientes se muestran para el servidor DNS que se incluye con Windows Server y que Windows AD usa de forma automática.

# <a name="portal"></a>[Portal](#tab/azure-portal)
En un servidor DNS de Windows, abra la consola de administración de DNS. Para encontrarla, seleccione el botón **Inicio** y escriba **DNS**. Vaya a la zona de búsqueda directa del dominio. Por ejemplo, si el dominio es `contoso.com`, la zona de búsqueda directa puede encontrarse en **Zonas de búsqueda directa** >  **`contoso.com`** en la consola de administración. La jerarquía exacta que se muestre en este cuadro de diálogo dependerá de la configuración de DNS para la red.

Haga clic con el botón derecho en la zona de búsqueda directa y seleccione **Nuevo alias (CNAME)** . En el cuadro de diálogo que aparece, escriba el nombre corto del servidor de archivos que va a reemplazar (el nombre de dominio completo se rellenará de forma automática en el cuadro de texto con la etiqueta **Nombre de dominio completo**). En el cuadro de texto con la etiqueta **Nombre de dominio completo (FQDN) para el host de destino**, escriba el nombre del servidor DFS-N que ha configurado. Si quiere, puede usar el botón **Examinar** para ayudarle a seleccionar el servidor. Seleccione **Aceptar** a fin de crear el registro CNAME para el servidor.

![Captura de pantalla que muestra el cuadro de diálogo **Nuevo registro de recursos** para una entrada DNS de CNAME.](./media/files-manage-namespaces/root-consolidation-cname.png)

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
En un servidor DNS de Windows, abra una sesión de PowerShell (o use la comunicación remota de PowerShell) para ejecutar los comandos siguientes y rellene `$oldServer` y `$dfsnServer` con los valores pertinentes para su entorno (`$domain` se rellenará de forma automática con el nombre de dominio, pero también puede escribirlo manualmente).

```PowerShell
# Variables
$oldServer = "MyServer"
$domain = Get-CimInstance -ClassName "Win32_ComputerSystem" | `
    Select-Object -ExpandProperty Domain
$dfsnServer = "CloudDFSN.$domain"

# Create CNAME record
Import-Module -Name DnsServer
Add-DnsServerResourceRecordCName `
    -Name $oldServer `
    -HostNameAlias $dfsnServer `
    -ZoneName $domain
```

---

## <a name="create-a-namespace"></a>Creación de un espacio de nombres
La unidad básica de administración de Espacios de nombres DFS es el espacio de nombres. La raíz del espacio de nombres, o el nombre, es el punto inicial de este, de forma que en la ruta de acceso UNC `\\contoso.com\Public\`, la raíz de espacio de nombres es `Public`. 

Si usa los espacios de nombres DFS para asumir un nombre de servidor existente con consolidación raíz, el nombre del espacio de nombres debe ser el nombre del servidor que quiere asumir, con el carácter `#` antepuesto. Por ejemplo, si quiere asumir un servidor existente llamado `MyServer`, debe crear un espacio de nombres DFS-N denominado `#MyServer`. La sección de PowerShell siguiente se encarga de anteponer `#`, pero si realiza las tareas de creación mediante la consola de Administración de DFS, deberá anteponer según corresponda. 

# <a name="portal"></a>[Portal](#tab/azure-portal)
Para crear un espacio de nombres, abra la consola de **Administración de DFS**. Para encontrarla, seleccione el botón **Inicio** y escriba **Administración de DFS**. La consola de administración que aparece tiene dos secciones: **Espacios de nombres** y **Replicación**, que hacen referencia a los espacios de nombres DFS y a la replicación DFS (DFS-R), respectivamente. Azure File Sync proporciona un mecanismo moderno de replicación y sincronización que se puede usar en lugar de DFS-R si también se quiere replicar.

Seleccione la sección **Espacios de nombres** y el botón **Nuevo espacio de nombres** (también puede hacer clic con el botón derecho en la sección **Espacios de nombres**). El **Asistente para crear nuevo espacio de nombres** que aparece le guiará a través del proceso de creación de un espacio de nombres. 

La primera sección del asistente requiere que elija el servidor de espacio de nombres DFS para hospedar el espacio de nombres. Varios servidores pueden hospedar un espacio de nombres, pero tendrá que configurar Espacios de nombres DFS con un servidor cada vez. Escriba el nombre del servidor de espacio de nombres DFS que quiera usar y seleccione **Siguiente**. En la sección **Nombre y configuración de espacio de nombres**, escriba el nombre que quiera usar para el espacio de nombres y seleccione **Siguiente**. 

La sección **Tipo de espacio de nombres** permite elegir entre un **Espacio de nombres basado en dominio** y un **Espacio de nombres independiente**. Si piensa usar Espacios de nombres DFS para conservar un nombre de dispositivo NAS o servidor de archivos existente, debe seleccionar la opción de espacio de nombres independiente. Para cualquier otro escenario, debe seleccionar un espacio de nombres basado en dominio. Consulte los [tipos de espacio de nombres](#namespace-types) anteriores para obtener más información sobre cómo elegir entre ellos.

![Captura de pantalla de la selección entre un espacio de nombres basado en dominio y un espacio de nombres independiente en el **Asistente para crear nuevo espacio de nombres**.](./media/files-manage-namespaces/dfs-namespace-type.png)

Seleccione el tipo de espacio de nombres que quiera usar para el entorno y elija **Siguiente**. A continuación, el asistente resumirá el espacio de nombres que se va a crear. Seleccione **Crear** para crear el espacio de nombres y **Cerrar** una vez que se complete el cuadro de diálogo.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
Desde una sesión de PowerShell en el servidor de espacio de nombres DFS, ejecute los comandos de PowerShell siguientes y rellene `$namespace`, `$type` y `$takeOverName` con los valores relevantes para su entorno.

```PowerShell
# Variables
$namespace = "Public"
$type = "DomainV2" # "Standalone"
$takeOverName = $false # $true

$namespace = if ($takeOverName -and $type -eq "Standalone" -and $namespace[0] -ne "#") { 
    "#$namespace" 
} else { $namespace }
$dfsnServer = $env:ComputerName
$namespaceServer = if ($type -eq "DomainV2") { 
    Get-CimInstance -ClassName "Win32_ComputerSystem" | `
    Select-Object -ExpandProperty Domain
} else { $dfsnServer }

# Create share for DFS-N namespace
$smbShare = "C:\DFSRoots\$namespace"
if (!(Test-Path -Path $smbShare)) { New-Item -Path $smbShare -ItemType Directory }
New-SmbShare -Name $namespace -Path $smbShare -FullAccess Everyone

# Create DFS-N namespace
Import-Module -Name DFSN
$namespacePath = "\\$namespaceServer\$namespace"
$targetPath = "\\$dfsnServer\$namespace"
New-DfsnRoot -Path $namespacePath -TargetPath $targetPath -Type $type
```
---

## <a name="configure-folders-and-folder-targets"></a>Configuración de carpetas y de destinos de carpeta
Para que un espacio de nombres sea útil, debe tener carpetas y destinos de carpeta. Cada carpeta puede tener uno o más destinos de carpeta, que son punteros a los recursos compartidos de archivos SMB que hospedan ese contenido. Cuando los usuarios examinan una carpeta con destinos de carpeta, el equipo cliente recibe una referencia que lo redirige de forma transparente a uno de los destinos de carpeta. También puede tener carpetas sin destinos de carpeta para agregar la estructura y la jerarquía al espacio de nombres.

Considere que las carpetas de Espacios de nombres DFS son análogas a los recursos compartidos de archivos. 

# <a name="portal"></a>[Portal](#tab/azure-portal)
En la consola de Administración de DFS, seleccione el espacio de nombres que acaba de crear y elija **Nueva carpeta**. Aparece el cuadro de diálogo **Nueva carpeta**, que le permite crear tanto la carpeta como sus destinos.

![Captura de pantalla del cuadro de diálogo **Nueva carpeta**.](./media/files-manage-namespaces/dfs-folder-targets.png)

Proporcione el nombre de la carpeta en el cuadro de texto con la etiqueta **Nombre**. Seleccione **Agregar...** a fin de agregar destinos de carpeta para esta carpeta. Se muestra el cuadro de diálogo **Agregar destino de carpeta**, que proporciona un cuadro de texto con la etiqueta **Ruta de acceso de destino de carpeta**, en el que puede proporcionar la ruta de acceso UNC a la carpeta que quiera. Seleccione **Aceptar** en el cuadro de diálogo **Agregar destino de carpeta**. Si va a agregar una ruta de acceso UNC a un recurso compartido de archivos de Azure, puede recibir un mensaje que informa de que no se puede contactar con el servidor `storageaccount.file.core.windows.net`. Este es el comportamiento esperado; seleccione **Sí** para continuar. Por último, seleccione **Aceptar** en el cuadro de diálogo **Nueva carpeta** para crear la carpeta y los destinos de carpeta.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
```PowerShell
# Variables
$shareName = "MyShare"
$targetUNC = "\\storageaccount.file.core.windows.net\myshare"

# Create folder and folder targets
$sharePath = "$namespacePath\$shareName"
New-DfsnFolder -Path $sharePath -TargetPath $targetUNC
```

---

Ahora que ha creado un espacio de nombres, una carpeta y un destino de carpeta, debe poder montar el recurso compartido de archivos en los espacios de nombres DFS. Si usa un espacio de nombres basado en dominio, la ruta de acceso completa del recurso compartido debe ser `\\<domain-name>\<namespace>\<share>`. Si usa un espacio de nombres independiente, la ruta de acceso completa del recurso compartido debe ser `\\<DFS-server>\<namespace>\<share>`. Si usa un espacio de nombres independiente con consolidación raíz, puede acceder directamente a través del nombre del servidor anterior, como `\\<old-server>\<share>`.

## <a name="see-also"></a>Consulte también
- Implementación de un recurso compartido de archivos de Azure: [Planeamiento de una implementación de Azure Files](storage-files-planning.md) y [Creación de un recurso compartido de archivos de Azure](storage-how-to-create-file-share.md).
- Configuración del acceso a recursos compartidos de archivos: documentos sobre [autenticación basada en la identidad](storage-files-active-directory-overview.md) y [consideraciones de red para el acceso directo](storage-files-networking-overview.md).
- [Espacios de nombres del Sistema de archivos distribuido de Windows Server](/windows-server/storage/dfs-namespaces/dfs-overview)