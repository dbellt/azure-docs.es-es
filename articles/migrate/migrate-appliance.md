---
title: Dispositivo con Azure Migrate
description: Proporciona un resumen de compatibilidad con el dispositivo de Azure Migrate.
author: vineetvikram
ms.author: vivikram
ms.manager: abhemraj
ms.topic: conceptual
ms.date: 03/18/2021
ms.openlocfilehash: d7fc04e65e2b79d43c48acd5a8c621f28d5c0403
ms.sourcegitcommit: 3ed0f0b1b66a741399dc59df2285546c66d1df38
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/19/2021
ms.locfileid: "107714675"
---
# <a name="azure-migrate-appliance"></a>Dispositivo con Azure Migrate

En este artículo se resumen los requisitos previos y los requisitos de compatibilidad del dispositivo de Azure Migrate.

## <a name="deployment-scenarios"></a>Escenarios de implementación

El dispositivo Azure Migrate se usa en los escenarios siguientes.

**Escenario** | **Herramienta** | **Se usa para**
--- | --- | ---
**Detección y evaluación de servidores que se ejecutan en el entorno de VMware** | Azure Migrate: Detección y evaluación | Detectar servidores que se ejecutan en el entorno de VMware.<br/><br/> Realizar la detección del inventario de software instaladas, el análisis de dependencias sin agente y la detección de instancias y bases de datos de SQL Server.<br/><br/> Recopilar la configuración del servidor y los metadatos de rendimiento para las evaluaciones.
**Migración sin agente de servidores que se ejecutan en el entorno de VMware** | Server Migration de Azure Migrate | Detectar servidores que se ejecutan en el entorno de VMware. <br/><br/> Replicar servidores sin necesidad de instalar ningún agente en ellos.
**Detección y evaluación de servidores que se ejecutan en el entorno de Hyper-V** | Azure Migrate: Detección y evaluación | Detectar servidores que se ejecutan en el entorno de Hyper-V.<br/><br/> Recopilar la configuración del servidor y los metadatos de rendimiento para las evaluaciones.
**Detección y evaluación de servidores físicos o virtualizados locales** |  Azure Migrate: Detección y evaluación |  Detectar servidores físicos o virtualizados locales.<br/><br/> Recopilar la configuración del servidor y los metadatos de rendimiento para las evaluaciones.

## <a name="deployment-methods"></a>Métodos de implementación

Para implementar el dispositivo se pueden usar un par de métodos:

- El dispositivo se puede implementar mediante una plantilla para servidores que se ejecutan en los entornos de VMware o Hyper-V ([plantilla OVA en el caso de VMware](how-to-set-up-appliance-vmware.md) o [VHD en el de Hyper-V](how-to-set-up-appliance-hyper-v.md)).
- Si no quiere usar una plantilla, puede implementar el dispositivo para los entornos de VMware o Hyper-V mediante un [script del instalador de PowerShell](deploy-appliance-script.md).
- En Azure Government, el dispositivo se debe implementar mediante un script del instalador de PowerShell. Consulte los pasos de la implementación [aquí](deploy-appliance-script-government.md).
- En el caso de los servidores físicos o virtualizados locales o de cualquier otra nube, el dispositivo siempre se implementa mediante un script del instalador de PowerShell. Consulte los pasos de la implementación [aquí](how-to-set-up-appliance-physical.md).
- Los vínculos de descarga están disponibles en las tablas siguientes.

## <a name="appliance---vmware"></a>Dispositivo: VMware

En la tabla siguiente se resumen los requisitos del dispositivo de Azure Migrate para VMware.

> [!Note]
> La detección y evaluación de las instancias y bases de datos de SQL Server que se ejecutan en el entorno de VMware se encuentran ahora en versión preliminar. Para probar esta característica, use [**este vínculo**](https://aka.ms/AzureMigrate/SQL) para crear un proyecto en la región **Este de Australia**. Si ya tiene un proyecto en la región Este de Australia y desea probar esta característica, asegúrese de que ha completado estos [**requisitos previos**](how-to-discover-sql-existing-project.md) en el portal.

**Requisito** | **VMware**
--- | ---
**Permisos** | Para acceder al administrador de configuración del dispositivo de manera local o remota, debe tener una cuenta de usuario local o de dominio con privilegios administrativos en el servidor del dispositivo.
**Servicios del dispositivo** | El dispositivo tiene los servicios siguientes:<br/><br/> - **Administrador de configuración del dispositivo**: se trata de una aplicación web que se puede configurar con detalles de origen para iniciar la detección y la evaluación de los servidores.<br/> - **Agente de detección de VMware**: el agente recopila los metadatos de configuración del servidor que se pueden usar para crear evaluaciones locales.<br/>- **Agente de evaluación de VMware**: el agente recopila los metadatos de rendimiento del servidor que se pueden usar para crear evaluaciones basadas en el rendimiento.<br/>- **Servicio de actualización automática**: el servicio mantiene actualizados todos los agentes que se ejecutan en el dispositivo. Se ejecuta automáticamente una vez cada 24 horas.<br/>- **Agente DRA**: orquesta la replicación del servidor y coordina la comunicación entre los servidores replicados y Azure. Solo se usa al replicar servidores en Azure mediante la migración sin agente.<br/>- **Puerta de enlace**: Envía los datos replicados a Azure. Solo se usa al replicar servidores en Azure mediante la migración sin agente.<br/>- **Agente de detección y evaluación de SQL**: envía los metadatos de configuración y rendimiento de las instancias y bases de datos de SQL Server a Azure.
**Límites del proyecto** |  Un dispositivo solo puede registrarse en un único proyecto.<br/> Un único proyecto puede tener varios dispositivos registrados.
**Límites de detección** | Un dispositivo puede detectar hasta 10 000 servidores en una instancia de vCenter Server.<br/> Un dispositivo solo puede conectarse a una instancia de vCenter Server.
**Implementación admitida** | Se implementa como un nuevo servidor que se ejecuta en vCenter Server mediante la plantilla OVA.<br/><br/> Se implementa en un servidor existente que se ejecuta en Windows Server 2016 mediante el script del instalador de PowerShell.
**Plantilla de OVA** | Se descarga desde el proyecto o desde [aquí](https://go.microsoft.com/fwlink/?linkid=2140333).<br/><br/> El tamaño de la descarga es 11,9 GB.<br/><br/> La plantilla del dispositivo descargada incluye una licencia de evaluación de Windows Server 2016 que es válida durante 180 días.<br/>Si el período de evaluación está a punto de expirar, se recomienda descargar e implementar un nuevo dispositivo mediante OVA, o bien activar la licencia del sistema operativo del servidor del dispositivo.
**Comprobación de OVA** | [Compruebe](tutorial-discover-vmware.md#verify-security) la plantilla OVA descargada del proyecto comprobando los valores hash.
**Script de PowerShell** | Consulte este [artículo](./deploy-appliance-script.md#set-up-the-appliance-for-vmware) sobre cómo implementar un dispositivo mediante el script del instalador de PowerShell.<br/><br/> 
**Requisitos de hardware y de red** |  El dispositivo debe ejecutarse en un servidor con Windows Server 2016, 32 GB de RAM, 8 vCPU, alrededor de 80 GB de almacenamiento en disco y un conmutador virtual externo.<br/> El dispositivo requiere acceso a Internet, ya sea directamente o a través de un proxy.<br/><br/> Si implementa el dispositivo mediante la plantilla OVA, necesitará suficientes recursos en vCenter Server para crear un servidor que cumpla los requisitos de hardware.<br/><br/> Si ejecuta el dispositivo en un servidor existente, asegúrese de que está ejecutando Windows Server 2016 y que cumple los requisitos de hardware.<br/>_(Actualmente, la implementación del dispositivo solo se admite en Windows Server 2016)._
**Requisitos de VMware** | Si implementa el dispositivo como servidor en vCenter Server, debe implementarse en una instancia de vCenter Server que ejecute la versión 5.5, 6.0, 6.5 o 6.7 y un host ESXi que ejecute la versión 5.5 o posterior.<br/><br/> 
**VDDK (migración sin agentes)** | Si quiere utilizar el dispositivo para la migración sin agente de servidores, VMware vSphere VDDK debe estar instalado en el servidor del dispositivo.

## <a name="appliance---hyper-v"></a>Dispositivo: Hyper-V

**Requisito** | **Hyper-V**
--- | ---
**Permisos** | Para acceder al administrador de configuración del dispositivo de manera local o remota, debe tener una cuenta de usuario local o de dominio con privilegios administrativos en el servidor del dispositivo.
**Servicios del dispositivo** | El dispositivo tiene los servicios siguientes:<br/><br/> - **Administrador de configuración del dispositivo**: se trata de una aplicación web que se puede configurar con detalles de origen para iniciar la detección y la evaluación de los servidores.<br/> - **Agente de detección**: el agente recopila los metadatos de configuración del servidor que se pueden usar para crear evaluaciones locales.<br/>- **Agente de evaluación**: el agente recopila los metadatos de rendimiento del servidor que se pueden usar para crear evaluaciones basadas en el rendimiento.<br/>- **Servicio de actualización automática**: el servicio mantiene actualizados todos los agentes que se ejecutan en el dispositivo. Se ejecuta automáticamente una vez cada 24 horas.
**Límites del proyecto** |  Un dispositivo solo puede registrarse en un único proyecto.<br/> Un único proyecto puede tener varios dispositivos registrados.
**Límites de detección** | Un dispositivo puede detectar hasta 5000 servidores que se ejecutan en el entorno de Hyper-V.<br/> Un dispositivo puede conectarse hasta a 300 hosts de Hyper-V.
**Implementación admitida** | Implementar como servidor que se ejecuta en un host de Hyper-V mediante una plantilla VHD.<br/><br/> Se implementa en un servidor existente que se ejecuta en Windows Server 2016 mediante el script del instalador de PowerShell.
**Plantilla de VHD** | Archivo ZIP que incluye una plantilla VHD. Se descarga desde el proyecto o desde [aquí](https://go.microsoft.com/fwlink/?linkid=2140422).<br/><br/> El tamaño de la descarga es 8,91 GB.<br/><br/> La plantilla del dispositivo descargada incluye una licencia de evaluación de Windows Server 2016 que es válida durante 180 días.<br/> Si el período de evaluación está a punto de expirar, se recomienda descargar e implementar un nuevo dispositivo, o bien activar la licencia del sistema operativo del servidor del dispositivo.
**Comprobación de VHD** | [Compruebe](tutorial-discover-hyper-v.md#verify-security) la plantilla VHD descargada del proyecto comprobando los valores hash.
**Script de PowerShell** | Consulte este [artículo](./deploy-appliance-script.md#set-up-the-appliance-for-hyper-v) sobre cómo implementar un dispositivo mediante el script del instalador de PowerShell.<br/>
**Requisitos de hardware y de red**  |  El dispositivo debe ejecutarse en un servidor con Windows Server 2016, 16 GB de RAM, 8 vCPU, alrededor de 80 GB de almacenamiento en disco y un conmutador virtual externo.<br/> El dispositivo necesita una dirección IP estática o dinámica y requiere acceso a Internet, ya sea directamente o a través de un proxy.<br/><br/> Si ejecuta el dispositivo como servidor que se ejecuta en un host de Hyper-V, necesita suficientes recursos en el host para crear un servidor que cumpla los requisitos de hardware.<br/><br/> Si ejecuta el dispositivo en un servidor existente, asegúrese de que está ejecutando Windows Server 2016 y que cumple los requisitos de hardware.<br/>_(Actualmente, la implementación del dispositivo solo se admite en Windows Server 2016)._
**Requisitos de Hyper-V** | Si se implementa el dispositivo con la plantilla VHD, el dispositivo que proporciona Azure Migrate es la versión 5.0 de la máquina virtual de Hyper-V.<br/><br/> El host de Hyper-V debe ejecutarse en Windows Server 2012 R2 o posterior.

## <a name="appliance---physical"></a>Dispositivo: físico

**Requisito** | **Físico**
--- | ---
**Permisos** | Para acceder al administrador de configuración del dispositivo de manera local o remota, debe tener una cuenta de usuario local o de dominio con privilegios administrativos en el servidor del dispositivo.
**Servicios del dispositivo** | El dispositivo tiene los servicios siguientes:<br/><br/> - **Administrador de configuración del dispositivo**: se trata de una aplicación web que se puede configurar con detalles de origen para iniciar la detección y la evaluación de los servidores.<br/> - **Agente de detección**: el agente recopila los metadatos de configuración del servidor que se pueden usar para crear evaluaciones locales.<br/>- **Agente de evaluación**: el agente recopila los metadatos de rendimiento del servidor que se pueden usar para crear evaluaciones basadas en el rendimiento.<br/>- **Servicio de actualización automática**: el servicio mantiene actualizados todos los agentes que se ejecutan en el dispositivo. Se ejecuta automáticamente una vez cada 24 horas.
**Límites del proyecto** |  Un dispositivo solo puede registrarse en un único proyecto.<br/> Un único proyecto puede tener varios dispositivos registrados.<br/>
**Límites de detección** | Un dispositivo puede detectar hasta 1000 servidores físicos.
**Implementación admitida** | Se implementa en un servidor existente que se ejecuta en Windows Server 2016 mediante el script del instalador de PowerShell.
**Script de PowerShell** | Descargue el script (AzureMigrateInstaller.ps1) en un archivo ZIP desde el proyecto o desde [aquí](https://go.microsoft.com/fwlink/?linkid=2140334). [Más información](tutorial-discover-physical.md).<br/><br/> El tamaño de la descarga es de 85,8 MB.
**Comprobación del script** | [Compruebe](tutorial-discover-physical.md#verify-security) el script del instalador de PowerShell descargado del proyecto comprobando los valores hash.
**Requisitos de hardware y de red** |  El dispositivo debe ejecutarse en un servidor con Windows Server 2016, 16 GB de RAM, 8 vCPU, alrededor de 80 GB de almacenamiento en disco.<br/> El dispositivo necesita una dirección IP estática o dinámica y requiere acceso a Internet, ya sea directamente o a través de un proxy.<br/><br/> Si ejecuta el dispositivo en un servidor existente, asegúrese de que está ejecutando Windows Server 2016 y que cumple los requisitos de hardware.<br/>_(Actualmente, la implementación del dispositivo solo se admite en Windows Server 2016)._

## <a name="url-access"></a>acceso URL

El dispositivo de Azure Migrate necesita conectividad a Internet.

- Al implementar el dispositivo, Azure Migrate realiza una comprobación de conectividad con las direcciones URL necesarias.
- Debe permitir el acceso a todas las direcciones URL de la lista. Si solo está realizando la evaluación, puede omitir las direcciones URL marcadas como necesarias para la migración sin agente de VMware.
- Si usa un proxy basado en URL para conectarse a Internet, asegúrese de que el proxy resuelve los registros CNAME recibidos al buscar estas direcciones.

### <a name="public-cloud-urls"></a>Direcciones URL de la nube pública

**URL** | **Detalles**  
--- | --- |
*.portal.azure.com  | Acceda a Azure Portal.
\* .windows.net <br/> *.msftauth.net <br/> *.msauth.net <br/> *.microsoft.com <br/> *.live.com <br/> *.office.com | Inicie sesión en la suscripción de Azure.
*.microsoftonline.com <br/> *.microsoftonline-p.com | Cree aplicaciones de Azure Active Directory (AD) para que el dispositivo se comunique con Azure Migrate.
management.azure.com | Cree aplicaciones de Azure AD para que el dispositivo se comunique con Azure Migrate.
*.services.visualstudio.com | Cargue los registros del dispositivo que se usan para la supervisión interna.
*.vault.azure.net | Administre secretos en Azure Key Vault.<br/> Nota: Asegúrese de que los servidores que se van a replicar tengan acceso a esto.
aka.ms/* | Permita el acceso a los vínculos aka; se usa para descargar e instalar las actualizaciones más recientes de los servicios del dispositivo.
download.microsoft.com/download | Permita descargas del centro de descargas de Microsoft.
*.servicebus.windows.net | Comunicación entre el dispositivo y el servicio Azure Migrate.
*.discoverysrv.windowsazure.com <br/> *.migration.windowsazure.com | Conéctese a las direcciones URL del servicio Azure Migrate.
*.hypervrecoverymanager.windowsazure.com | **Se usa para la migración sin agentes de VMware.**<br/><br/> Conéctese a las direcciones URL del servicio Azure Migrate.
*.blob.core.windows.net |  **Se usa para la migración sin agentes de VMware.**<br/><br/>Cargue los datos al almacenamiento para la migración.

### <a name="government-cloud-urls"></a>Direcciones URL de la nube del gobierno

**URL** | **Detalles**  
--- | --- |
*.portal.azure.us  | Acceda a Azure Portal.
graph.windows.net | Inicie sesión en la suscripción de Azure.
login.microsoftonline.us  | Cree aplicaciones de Azure Active Directory (AD) para que el dispositivo se comunique con Azure Migrate.
management.usgovcloudapi.net | Cree aplicaciones de Azure AD para que el dispositivo se comunique con el servicio Azure Migrate.
*.services.visualstudio.com | Cargue los registros del dispositivo que se usan para la supervisión interna.
*.vault.usgovcloudapi.net | Administre secretos en Azure Key Vault.
aka.ms/* | Permita el acceso a los vínculos aka; se usa para descargar e instalar las actualizaciones más recientes de los servicios del dispositivo.
download.microsoft.com/download | Permita descargas del centro de descargas de Microsoft.
*.servicebus.usgovcloudapi.net  | Comunicación entre el dispositivo y el servicio Azure Migrate.
*.discoverysrv.windowsazure.us <br/> *.migration.windowsazure.us | Conéctese a las direcciones URL del servicio Azure Migrate.
*.hypervrecoverymanager.windowsazure.us | **Se usa para la migración sin agentes de VMware.**<br/><br/> Conéctese a las direcciones URL del servicio Azure Migrate.
*.blob.core.usgovcloudapi.net  |  **Se usa para la migración sin agentes de VMware.**<br/><br/>Cargue los datos al almacenamiento para la migración.
*.applicationinsights.us | Cargue los registros del dispositivo que se usan para la supervisión interna.  

### <a name="public-cloud-urls-for-private-link-connectivity"></a>Direcciones URL de nube pública para la conectividad de vínculo privado

El dispositivo necesita acceso a las siguientes direcciones URL (directamente o a través de proxy) a través del acceso de vínculo privado y superior. 

**URL** | **Detalles**  
--- | --- | 
*.portal.azure.com  | Acceda a Azure Portal.
\* .windows.net <br/> *.msftauth.net <br/> *.msauth.net <br/> *.microsoft.com <br/> *.live.com <br/> *.office.com | Inicie sesión en la suscripción de Azure.
*.microsoftonline.com <br/> *.microsoftonline-p.com | Cree aplicaciones de Azure Active Directory (AD) para que el dispositivo se comunique con Azure Migrate.
management.azure.com | Cree aplicaciones de Azure AD para que el dispositivo se comunique con Azure Migrate.
*.services.visualstudio.com (opcional) | Cargue los registros del dispositivo que se usan para la supervisión interna.
aka.ms/* (opcional) | Permita el acceso a los vínculos aka; se usa para descargar e instalar las actualizaciones más recientes de los servicios del dispositivo.
download.microsoft.com/download | Permita descargas del centro de descargas de Microsoft.
*.servicebus.windows.net | **Se usa para la migración sin agentes de VMware.**<br/><br/> Comunicación entre el dispositivo y el servicio Azure Migrate.
*.vault.azure.net | **Se usa para la migración sin agentes de VMware.**<br/><br/>  Asegúrese de que los servidores que se van a replicar tengan acceso a esto.
*.hypervrecoverymanager.windowsazure.com | **Se usa para la migración sin agentes de VMware.**<br/><br/> Conéctese a las direcciones URL del servicio Azure Migrate.
*.blob.core.windows.net |  **Se usa para la migración sin agentes de VMware.**<br/><br/>Cargue los datos al almacenamiento para la migración.

### <a name="government-cloud-urls-for-private-link-connectivity"></a>Direcciones URL de nube gubernamental para la conectividad de vínculo privado   

El dispositivo necesita acceso a las siguientes direcciones URL (directamente o a través de proxy) a través del acceso de vínculo privado y superior. 

**URL** | **Detalles**  
--- | --- |
*.portal.azure.us  | Acceda a Azure Portal.
graph.windows.net | Inicie sesión en la suscripción de Azure.
login.microsoftonline.us  | Cree aplicaciones de Azure Active Directory (AD) para que el dispositivo se comunique con Azure Migrate.
management.usgovcloudapi.net | Cree aplicaciones de Azure AD para que el dispositivo se comunique con el servicio Azure Migrate.
*.services.visualstudio.com (opcional) | Cargue los registros del dispositivo que se usan para la supervisión interna.
aka.ms/* (opcional) | Permita el acceso a los vínculos aka; se usa para descargar e instalar las actualizaciones más recientes de los servicios del dispositivo.
download.microsoft.com/download | Permita descargas del centro de descargas de Microsoft.
*.servicebus.usgovcloudapi.net  | **Se usa para la migración sin agentes de VMware.**<br/><br/> Comunicación entre el dispositivo y el servicio Azure Migrate. 
*.vault.usgovcloudapi.net | **Se usa para la migración sin agentes de VMware.**<br/><br/> Administre secretos en Azure Key Vault.
*.hypervrecoverymanager.windowsazure.us | **Se usa para la migración sin agentes de VMware.**<br/><br/> Conéctese a las direcciones URL del servicio Azure Migrate.
*.blob.core.usgovcloudapi.net  |  **Se usa para la migración sin agentes de VMware.**<br/><br/>Cargue los datos al almacenamiento para la migración.
*.applicationinsights.us (opcional) | Cargue los registros del dispositivo que se usan para la supervisión interna.  

## <a name="collected-data---vmware"></a>Datos recopilados: VMware

El dispositivo recopila los metadatos de configuración, los metadatos de rendimiento y los datos de dependencias del servidor (si se usa el [análisis de dependencias](concepts-dependency-visualization.md) sin agente).

### <a name="metadata"></a>Metadatos

Los metadatos que el dispositivo de Azure Migrate detecta ayudan a averiguar si los servidores están listos para la migración a Azure, el tamaño correcto de los servidores, los planes de costes y el análisis de las dependencias de la aplicación. Microsoft no usa estos datos en ninguna auditoría de cumplimiento de licencias.

Esta es la lista completa de metadatos del servidor que el dispositivo recopila y envía a Azure.

**DATOS** | **CONTADOR**
--- | ---
**Detalles del servidor** |
Id. de servidor | vm.Config.InstanceUuid
Nombre de servidor | vm.Config.Name
Id. de vCenter Server | VMwareClient.Instance.Uuid
Descripción del servidor | vm.Summary.Config.Annotation
Nombre del producto de licencia | vm.Client.ServiceContent.About.LicenseProductName
Tipo de sistema operativo | vm.SummaryConfig.GuestFullName
Tipo de arranque | vm.Config.Firmware
Número de núcleos | vm.Config.Hardware.NumCPU
Memoria (MB) | vm.Config.Hardware.MemoryMB
Número de discos | vm.Config.Hardware.Device.ToList().FindAll(x => es VirtualDisk).count
Lista de tamaños de los discos | vm.Config.Hardware.Device.ToList().FindAll(x => es VirtualDisk)
Lista de adaptadores de red | vm.Config.Hardware.Device.ToList().FindAll(x => es VirtualEthernet).count
Uso de CPU | cpu.usage.average
Uso de memoria |mem.usage.average
**Detalles por disco** |
Valor de clave del disco | disk.Key
Número de unidades de disco | disk.UnitNumber
Valor de clave de controladora de disco | disk.ControllerKey.Value
Gigabytes aprovisionados | virtualDisk.DeviceInfo.Summary
Nombre del disco | Valor generado mediante disk.UnitNumber, disk.Key y disk.ControllerKey.Value
Operaciones de lectura por segundo | virtualDisk.numberReadAveraged.average
Operaciones de escritura por segundo | virtualDisk.numberWriteAveraged.average
Rendimiento de lectura (MB por segundo) | virtualDisk.read.average
Rendimiento de escritura (MB por segundo) | virtualDisk.write.average
**Detalles por NIC** |
Nombre del adaptador de red | nic.Key
Dirección MAC | ((VirtualEthernetCard)nic).MacAddress
Direcciones IPv4 | vm.Guest.Net
Direcciones IPv6 | vm.Guest.Net
Rendimiento de lectura (MB por segundo) | net.received.average
Rendimiento de escritura (MB por segundo) | net.transmitted.average
**Detalles de la ruta de acceso de inventario** |
Nombre | container.GetType().Name
Tipo de objeto secundario | container.ChildType
Información de referencia | container.MoRef
Detalles del objeto primario | Container.Parent
Detalles de la carpeta por servidor | ((Folder)container).ChildEntity.Type
Detalles del centro de datos por servidor | ((Datacenter)container).VmFolder
Detalles del centro de datos por carpeta de host | ((Datacenter)container).HostFolder
Detalles del clúster por host | ((ClusterComputeResource)container).Host
Detalles del host por servidor | ((HostSystem)container).VM

### <a name="performance-data"></a>Datos de rendimiento

Estos son los datos de rendimiento que un dispositivo recopila para un servidor que se ejecuta en VMware y lo envía a Azure.

**Data** | **Contador** | **Impacto en la evaluación**
--- | --- | ---
Uso de CPU | cpu.usage.average | Tamaño de servidor recomendado/costo
Uso de memoria | mem.usage.average | Tamaño o costo del servidor recomendado
Rendimiento de lectura de disco (MB por segundo) | virtualDisk.read.average | Cálculo del tamaño del disco, el costo de almacenamiento y el tamaño de servidor
Rendimiento de escrituras de discos (MB por segundo) | virtualDisk.write.average | Cálculo del tamaño del disco, el costo de almacenamiento y el tamaño de servidor
Operaciones de lectura de disco por segundo | virtualDisk.numberReadAveraged.average | Cálculo del tamaño del disco, el costo de almacenamiento y el tamaño de servidor
Operaciones de escritura de discos por segundo | virtualDisk.numberWriteAveraged.average  | Cálculo del tamaño del disco, el costo de almacenamiento y el tamaño de servidor
Rendimiento de lectura de NIC (MB por segundo) | net.received.average | Cálculo de tamaño del servidor
Rendimiento de escrituras de adaptadores de red (MB por segundo) | net.transmitted.average  |Cálculo de tamaño del servidor

### <a name="installed-software-inventory"></a>Inventario del software instalado

El dispositivo recoge datos sobre el inventario de software instalado en los servidores.

#### <a name="windows-server-software-inventory-data"></a>Datos de inventario del software de Windows Server

Estos son los datos de inventario del software que el dispositivo recopila de cada instancia de Windows Server detectada en el entorno de VMware.

**Data** | **Ubicación del registro** | **Clave**
--- | --- | ---
Nombre de la aplicación  | HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall\* <br/> HKLM:\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*  | DisplayName
Versión  | HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall\*  <br/> HKLM:\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*  | DisplayVersion
Proveedor  | HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall\*  <br/> HKLM:\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*  | Publicador

#### <a name="windows-server-features-data"></a>Datos de las características de Windows Server

Estos son los datos de las características que el dispositivo recopila de cada instancia de Windows Server detectada en el entorno de VMware.

**Data**  | **Cmdlet de PowerShell** | **Propiedad**
--- | --- | ---
Nombre  | Get-WindowsFeature  | Nombre
Tipo de característica | Get-WindowsFeature  | FeatureType
Parent  | Get-WindowsFeature  | Parent

#### <a name="sql-server-metadata"></a>Metadatos de SQL Server

Estos son los datos de SQL Server que el dispositivo recopila de cada instancia de Windows Server detectada en el entorno de VMware.

**Data**  | **Ubicación del registro**  | **Clave**
--- | --- | ---
Nombre  | HKLM:\SOFTWARE\Microsoft\Microsoft SQL Server\Instance Names\SQL  | installedInstance
Edición  | HKLM:\SOFTWARE\Microsoft\Microsoft SQL Server\\\<InstanceName>\Setup  | Edición
Service Pack  | HKLM:\SOFTWARE\Microsoft\Microsoft SQL Server\\\<InstanceName>\Setup  | SP
Versión  | HKLM:\SOFTWARE\Microsoft\Microsoft SQL Server\\\<InstanceName>\Setup  | Versión

#### <a name="windows-server-operating-system-data"></a>Datos del sistema operativo de Windows Server

Estos son los datos del sistema operativo que el dispositivo recopila de cada instancia de Windows Server detectada en el entorno de VMware.

**Data**  | **Clase WMI**  | **Propiedad de clase WMI**
--- | --- | ---
Nombre  | Win32_operatingsystem  | Caption
Versión  | Win32_operatingsystem  | Versión
Architecture  | Win32_operatingsystem  | OSArchitecture

#### <a name="linux-server-software-inventory-data"></a>Datos de inventario del software del servidor de Linux

Estos son los datos de inventario del software que el dispositivo recopila de cada servidor de Linux detectado en el entorno de VMware. En función del sistema operativo del servidor, se ejecutan uno o varios comandos.

**Data**  | **Comandos**
--- | ---
Nombre | rpm, dpkg-query, snap
Versión | rpm, dpkg-query, snap
Proveedor | rpm, dpkg-query, snap

#### <a name="linux-server-operating-system-data"></a>Datos del sistema operativo del servidor Linux

Estos son los datos del sistema operativo que el dispositivo recopila de cada servidor Linux detectado en el entorno de VMware.

**Data**  | **Comandos**
--- | ---
Nombre <br/> version | Recopilado de uno o varios de los archivos siguientes:<br/> <br/>/etc/os-release  <br> /usr/lib/os-release  <br> /etc/enterprise-release  <br> /etc/redhat-release  <br> /etc/oracle-release  <br> /etc/SuSE-release  <br> /etc/lsb-release  <br> /etc/debian_version
Architecture | uname

### <a name="sql-server-instances-and-databases-data"></a>Instancias y bases de datos de SQL Server

El dispositivo recopila datos en las instancias y bases de datos de SQL Server.

> [!Note]
> La detección y evaluación de las instancias y bases de datos de SQL Server que se ejecutan en el entorno de VMware se encuentran ahora en versión preliminar. Para probar esta característica, use [**este vínculo**](https://aka.ms/AzureMigrate/SQL) para crear un proyecto en la región **Este de Australia**. Si ya tiene un proyecto en la región Este de Australia y desea probar esta característica, asegúrese de que ha completado estos [**requisitos previos**](how-to-discover-sql-existing-project.md) en el portal.

#### <a name="sql-database-metadata"></a>Metadatos de base de datos SQL

**Metadatos de base de datos** | **Propiedades de vistas o SQL Server**
--- | ---
Identificador único de la base de datos | sys.databases
Id. de base de datos definido por el usuario | sys.databases
Nombre de la base de datos | sys.databases
Nivel de compatibilidad de base de datos | sys.databases
Nombre de intercalación de base de datos | sys.databases
Estado de la base de datos | sys.databases
Tamaño de la base de datos (en MB) | sys.master_files
Letra de unidad de la ubicación que contiene los archivos de datos | SERVERPROPERTY y Software\Microsoft\MSSQLServer\MSSQLServer
Lista de archivos de base de datos | sys.databases, sys.master_files
Service Broker está habilitado o no | sys.databases
La base de datos está habilitada para la captura de datos modificados o no | sys.databases

#### <a name="sql-server-metadata"></a>Metadatos de SQL Server

**Metadatos del servidor** | **Propiedades de vistas o SQL Server**
--- | ---
Nombre de servidor |SERVERPROPERTY
FQDN | Cadena de conexión derivada de la detección de aplicaciones instaladas
Id. de instalación | sys.dm_server_registry
Versión del servidor | SERVERPROPERTY
Edición del servidor | SERVERPROPERTY
Plataforma de host de servidor (Windows o Linux) | SERVERPROPERTY
Nivel de producto del servidor (RTM SP CTP) | SERVERPROPERTY
Ruta de acceso de la copia de seguridad predeterminada | SERVERPROPERTY
Ruta de acceso predeterminada de los archivos de datos | SERVERPROPERTY y Software\Microsoft\MSSQLServer\MSSQLServer
Ruta de acceso predeterminada de los archivos de registro | SERVERPROPERTY y Software\Microsoft\MSSQLServer\MSSQLServer
No. de núcleos en el servidor | sys.dm_os_schedulers, sys.dm_os_sys_info
Nombre de intercalación de SQL Server | SERVERPROPERTY
No. de núcleos en el servidor con el estado VISIBLE ONLINE | sys.dm_os_schedulers
Id. de servidor único | sys.dm_server_registry
Alta disponibilidad habilitada o no | SERVERPROPERTY
Extensión del grupo de búferes habilitada o no | sys.dm_os_buffer_pool_extension_configuration
Clúster de conmutación por error configurado o no | SERVERPROPERTY
Servidor que usa solo el modo de autenticación de Windows | SERVERPROPERTY
El servidor instala PolyBase | SERVERPROPERTY
No. de CPU lógicas en el sistema | sys.dm_server_registry, sys.dm_os_sys_info
Proporción del número de núcleos lógicos o físicos expuestos por un paquete de procesadores físicos | sys.dm_os_schedulers, sys.dm_os_sys_info
Número de CPU físicas en el sistema | sys.dm_os_schedulers, sys.dm_os_sys_info
Fecha y hora en que se inició por última vez el servidor | sys.dm_server_registry
Uso máximo de memoria del servidor (en MB) | sys.dm_os_process_memory
Número total de usuarios en todas las bases de datos | sys.databases, sys.logins
Tamaño total de todas las bases de datos de usuario | sys.databases
Tamaño de la base de datos temporal | sys.master_files, sys.configurations, sys.dm_os_sys_info
No. de inicios de sesión | sys.logins
Lista de servidores vinculados | sys.servers
Lista de trabajos del agente | [msdb].[dbo].[sysjobs], [sys].[syslogins], [msdb].[dbo].[syscategories]

### <a name="performance-metadata"></a>Metadatos de rendimiento

**Rendimiento** | **Propiedades de vistas o SQL Server** | **Impacto en la evaluación**
--- | --- | ---
Uso de CPU de SQL Server| sys.dm_os_ring_buffers| Tamaño de SKU recomendado (dimensión de CPU)
Recuento de CPU lógicas de SQL| sys.dm_os_sys_info| Tamaño de SKU recomendado (dimensión de CPU)
Memoria física de SQL en uso| sys.dm_os_process_memory| No utilizado
Porcentaje de uso de la memoria de SQL| sys.dm_os_process_memory | No utilizado
Uso de CPU de base de datos| sys.dm_exec_query_stats, sys.dm_exec_plan_attributes| Tamaño de SKU recomendado (dimensión de CPU)
Memoria de base de datos en uso (grupo de búferes)| sys.dm_os_buffer_descriptors| Tamaño de SKU recomendado (dimensión de memoria)
E/S de lectura o escritura de archivos| sys.dm_io_virtual_file_stats, sys.master_files| Tamaño de SKU recomendado (dimensión de E/S)
Número de archivos de lecturas o escrituras| sys.dm_io_virtual_file_stats, sys.master_files| Tamaño de SKU recomendado (dimensión de rendimiento)
Detención de lectura o escritura de E/S de archivo (ms)| sys.dm_io_virtual_file_stats, sys.master_files| Tamaño de SKU recomendado (dimensión de latencia de E/S)
Tamaño de archivo| sys.master_files| Tamaño de SKU recomendado (dimensión de almacenamiento)


### <a name="application-dependency-data"></a>Datos de dependencia de aplicación

El análisis de dependencias sin agente recopila datos de la conexión y el proceso.

#### <a name="windows-server-dependencies-data"></a>Datos de dependencias de Windows Server

Estos son los datos de conexión que el dispositivo recopila de cada Windows Server habilitado para el análisis de dependencias sin agente.

**Data** | **Comandos**
--- | ---
Puerto local | netstat
Dirección IP local | netstat
Puerto remoto | netstat
Dirección IP remota | netstat
Estado de conexión de TCP | netstat
Identificador del proceso | netstat
Número de conexiones activas | netstat

Estos son los datos de conexión que el dispositivo recopila de cada Windows Server habilitado para el análisis de dependencias sin agente.

**Data** | **Clase WMI** | **Propiedad de clase WMI**
--- | --- | ---
Nombre del proceso | Win32_Process | ExecutablePath _s
Argumentos de procesos | Win32_Process | CommandLine
Nombre de la aplicación | Win32_Process | Parámetro VersionInfo.ProductName de la propiedad ExecutablePath

#### <a name="linux-server-dependencies-data"></a>Datos de dependencias de servidor Linux

Estos son los datos de conexión que el dispositivo recopila de cada servidor Linux habilitado para el análisis de dependencias sin agente.

**Data** | **Comandos**
--- | ---
Puerto local | netstat
Dirección IP local | netstat
Puerto remoto | netstat
Dirección IP remota | netstat
Estado de conexión de TCP | netstat
Número de conexiones activas | netstat
Identificador del proceso  | netstat
Nombre del proceso | ps
Argumentos de procesos | ps
Nombre de la aplicación | dpkg o rpm

## <a name="collected-data---hyper-v"></a>Datos recopilados: Hyper-V

El dispositivo recopila los metadatos de configuración y rendimiento de los servidores que se ejecutan en el entorno de Hyper-V.

### <a name="metadata"></a>Metadatos
Los metadatos que el dispositivo de Azure Migrate detecta ayudan a averiguar si los servidores están listos para la migración a Azure, el tamaño correcto de los servidores y los planes de costes. Microsoft no usa estos datos en ninguna auditoría de cumplimiento de licencias.

Esta es la lista completa de metadatos del servidor que el dispositivo recopila y envía a Azure.

**Data** | **Clase WMI** | **Propiedad de clase WMI**
--- | --- | ---
**Detalles del servidor** | 
Número de serie del BIOS | Msvm_BIOSElement | BIOSSerialNumber
Tipo de servidor (Gen. 1 o 2) | Msvm_VirtualSystemSettingData | VirtualSystemSubType
Nombre para mostrar del servidor | Msvm_VirtualSystemSettingData | ElementName
Versión del servidor | Msvm_ProcessorSettingData | VirtualQuantity
Memoria (bytes) | Msvm_MemorySettingData | VirtualQuantity
Memoria máxima que puede consumir servidor | Msvm_MemorySettingData | Límite
Memoria dinámica habilitada | Msvm_MemorySettingData | DynamicMemoryEnabled
Nombre/versión/FQDN del sistema operativo | Msvm_KvpExchangeComponent | Datos de nombre de GuestIntrinsicExchangeItems
Estado de energía del servidor | Msvm_ComputerSystem | EnabledState
**Detalles por disco** |
Identificador de disco | Msvm_VirtualHardDiskSettingData | VirtualDiskId
Tipo de disco duro virtual | Msvm_VirtualHardDiskSettingData | Tipo
Tamaño de disco duro virtual | Msvm_VirtualHardDiskSettingData | MaxInternalSize
Primario de disco duro virtual | Msvm_VirtualHardDiskSettingData | ParentPath
**Detalles por NIC** |
Direcciones IP (NIC sintéticas) | Msvm_GuestNetworkAdapterConfiguration | IPAddresses
DHCP habilitado (NIC sintéticas) | Msvm_GuestNetworkAdapterConfiguration | DHCPEnabled
Id. de NIC (NIC sintéticas) | Msvm_SyntheticEthernetPortSettingData | InstanceID
Dirección MAC de NIC (NIC sintéticas) | Msvm_SyntheticEthernetPortSettingData | Dirección
Id. de NIC (NIC heredadas) | Datos de MsvmEmulatedEthernetPortSetting | InstanceID
Id. de MAC de NIC (NIC heredadas) | Datos de MsvmEmulatedEthernetPortSetting | Dirección

### <a name="performance-data"></a>Datos de rendimiento

Estos son los datos de rendimiento del servidor que el dispositivo recopila y envía a Azure.

**Clase de contador de rendimiento** | **Contador** | **Impacto en la evaluación**
--- | --- | ---
Procesador virtual del hipervisor de Hyper-V | Tiempo de ejecución de invitado (%) | Tamaño o costo del servidor recomendado
Servidor de Memoria dinámica de Hyper-V | Presión actual (%)<br/> Memoria física visible de invitado (MB) | Tamaño o costo del servidor recomendado
Dispositivo de almacenamiento virtual de Hyper-V | Bytes de lectura/segundo | Cálculo del tamaño del disco, el costo de almacenamiento y el tamaño de servidor
Dispositivo de almacenamiento virtual de Hyper-V | Bytes de escritura/segundo | Cálculo del tamaño del disco, el costo de almacenamiento y el tamaño de servidor
Adaptador de red virtual de Hyper-V | Bytes recibidos/segundo | Cálculo de tamaño del servidor
Adaptador de red virtual de Hyper-V | Bytes enviados/segundo | Cálculo de tamaño del servidor

- El uso de CPU es la suma de todo el uso, para todos los procesadores virtuales conectados a un servidor.
- El uso de memoria es (presión actual * memoria física visible del invitado)/100.
- Los valores de uso de disco y red se recopilan de los contadores de rendimiento de Hyper-V enumerados.

## <a name="collected-data---physical"></a>Datos recopilados: físicos

El dispositivo recopila los metadatos de configuración y rendimiento de los servidores físicos o virtuales que se ejecutan de forma local.

### <a name="metadata"></a>Metadatos

Los metadatos que el dispositivo de Azure Migrate detecta ayudan a averiguar si los servidores están listos para la migración a Azure, el tamaño correcto de los servidores y los planes de costes. Microsoft no usa estos datos en ninguna auditoría de cumplimiento de licencias.

### <a name="windows-server-metadata"></a>Metadatos de Windows Server

Esta es la lista completa de metadatos de Windows Server que el dispositivo recopila y envía a Azure.

**Data** | **Clase WMI** | **Propiedad de clase WMI**
--- | --- | ---
FQDN | Win32_ComputerSystem | Dominio, nombre, PartOfDomain
Número de núcleos del procesador | Win32_PRocessor | NumberOfCores
Memoria asignada | Win32_ComputerSystem | TotalPhysicalMemory
Número de serie del BIOS | Win32_ComputerSystemProduct | IdentifyingNumber
GUID del BIOS | Win32_ComputerSystemProduct | UUID
Tipo de arranque | Win32_DiskPartition | Busque la partición con el tipo = **GPT:System** para EFI/BIOS
Nombre del SO | Win32_OperatingSystem | Caption
Versión del SO |Win32_OperatingSystem | Versión
Arquitectura del SO | Win32_OperatingSystem | OSArchitecture
Número de discos | Win32_DiskDrive | Modelo, tamaño, DeviceID, MediaType, nombre
Tamaño del disco | Win32_DiskDrive | Size
Lista de NIC | Win32_NetworkAdapterConfiguration | Descripción, índice
Dirección IP de NIC | Win32_NetworkAdapterConfiguration | IPAddress
Dirección MAC de NIC | Win32_NetworkAdapterConfiguration | MACAddress

### <a name="linux-server-metadata"></a>Metadatos de servidor Linux

Esta es la lista completa de metadatos del servidor de Linux que el dispositivo recopila y envía a Azure.

**Data** | **Comandos**
--- | ---
FQDN | cat /proc/sys/kernel/hostname, hostname -f
Número de núcleos del procesador |  /proc/cpuinfo \| awk "/^processor/{print $3}" \| wc -l
Memoria asignada | cat /proc/meminfo \| grep MemTotal \| awk "{printf "%.0f", $2/1024}"
Número de serie del BIOS | lshw \| grep "serial:" \| head -n1 \| awk "{print $2}" <br/> /usr/sbin/dmidecode -t 1 \| grep 'Serial' \| awk '{ $1="" ; $2=""; print}'
GUID del BIOS | cat /sys/class/dmi/id/product_uuid
Tipo de arranque | [ -d /sys/firmware/efi ] && echo EFI \|\| echo BIOS
Versión o nombre del SO | Obtenemos acceso a estos archivos para el nombre y la versión del SO:<br/><br/> /etc/os-release<br/> /usr/lib/os-release <br/> /etc/enterprise-release <br/> /etc/redhat-release<br/> /etc/oracle-release<br/>  /etc/SuSE-release<br/>  /etc/lsb-release  <br/> /etc/debian_version
Arquitectura del SO | Uname -m
Número de discos | fdisk -l \| egrep "Disk.*bytes" \| awk "{print $2}" \| cut -f1 -d ":"
Disco de arranque | df /boot \| sed -n 2p \| awk "{print $1}"
Tamaño del disco | fdisk -l \| egrep "Disk.*bytes" \| egrep $disk: \| awk "{print $5}"
Lista de NIC | ip -o -4 addr show \| awk "{print $2}"
Dirección IP de NIC | ip addr show $nic \| grep inet \| awk "{print $2}" \| cut -f1 -d "/" 
Dirección MAC de NIC | ip addr show $nic \| grep ether  \| awk "{print $2}"

### <a name="windows-performance-data"></a>Datos de rendimiento de Windows

Estos son los datos de rendimiento de Windows Sever que el dispositivo recopila y envía a Azure.

**Data** | **Clase WMI** | **Propiedad de clase WMI**
--- | --- | ---
Uso de CPU | Win32_PerfFormattedData_PerfOS_Processor | PercentIdleTime
Uso de la memoria | Win32_PerfFormattedData_PerfOS_Memory | AvailableMBytes
NIC (recuento) | Win32_PerfFormattedData_Tcpip_NetworkInterface | Obtiene el número de dispositivos de red.
Datos recibidos por NIC | Win32_PerfFormattedData_Tcpip_NetworkInterface  | BytesReceivedPerSec
Datos transmitidos por NIC | BWin32_PerfFormattedData_Tcpip_NetworkInterface | BytesSentPersec
Número de discos | BWin32_PerfFormattedData_PerfDisk_PhysicalDisk | Número de discos
Detalles del disco | Win32_PerfFormattedData_PerfDisk_PhysicalDisk | DiskWritesPerSec, DiskWriteBytesPerSec, DiskReadsPerSec, DiskReadBytesPerSec.

### <a name="linux-performance-data"></a>Datos de rendimiento de Linux

Estos son los datos de rendimiento de Linux que el dispositivo recopila y envía a Azure.

| **Data** | **Comandos** |
| --- | --- |
| Uso de CPU | cat /proc/stat/ \| grep 'cpu' /proc/stat |
| Uso de la memoria | free \| grep Mem \| awk "{print $3/$2 * 100.0}" |
| NIC (recuento) | lshw -class network \| eth[0-60] \| wc -l |
| Datos recibidos por NIC | cat /sys/class/net/eth$nic/statistics/rx_bytes |
| Datos transmitidos por NIC | cat /sys/class/net/eth$nic/statistics/tx_bytes |
| Número de discos | fdisk -l \| egrep 'Disk.\*bytes' \| awk '{print $2}' \| cut -f1 -d ':' |
| Detalles del disco | cat /proc/diskstats |

## <a name="appliance-upgrades"></a>Actualizaciones del dispositivo

El dispositivo se actualiza a medida que se actualizan los servicios de Azure Migrate que se ejecutan en el dispositivo. Esto sucede automáticamente porque la actualización automática está habilitada en el dispositivo de forma predeterminada. Puede cambiar esta configuración predeterminada para actualizar los servicios de dispositivo manualmente.

### <a name="turn-off-auto-update"></a>Desactivación de la actualización automática

1. En el servidor que ejecuta el dispositivo, abra el Editor del Registro.
2. Vaya a **HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\AzureAppliance**.
3. Para desactivar la actualización automática, cree una clave **AutoUpdate** de clave del Registro con un valor de DWORD de 0.

    ![Definición de clave del Registro](./media/migrate-appliance/registry-key.png)


### <a name="turn-on-auto-update"></a>Activación de la actualización automática

Puede activar la actualización automática mediante cualquiera de estos métodos:

- Eliminando la clave del Registro de AutoUpdate de HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\AzureAppliance.
- Haga clic en **Ver servicios del dispositivo** desde las comprobaciones de actualizaciones más recientes en el panel **Configurar los requisitos previos** para activar la actualización automática.

Para eliminar la clave del Registro:

1. En el servidor que ejecuta el dispositivo, abra el Editor del Registro.
2. Vaya a **HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\AzureAppliance**.
3. Elimine la clave del registro **AutoUpdate** que se creó anteriormente para desactivar la actualización automática.

Para realizar la activación desde Appliance Configuration Manager, una vez completada la detección:

1. En el administrador de configuración del dispositivo, vaya al panel **Configurar los requisitos previos**.
2. En la comprobación de actualizaciones más recientes, haga clic en **Ver servicios del dispositivo** y haga clic en el vínculo para activar la actualización automática.

    ![Activar las actualizaciones automáticas](./media/migrate-appliance/autoupdate-off.png)

### <a name="check-the-appliance-services-version"></a>Comprobación de la versión de servicios del dispositivo

Puede comprobar la versión de los servicios del dispositivo con cualquiera de estos métodos:

- En el administrador de configuración del dispositivo, vaya al panel **Configurar los requisitos previos**.
- En el dispositivo, en el **Panel de control** > **Programas y características**.

Para registrar el administrador de configuración del dispositivo:

1. En el administrador de configuración del dispositivo, vaya al panel **Configurar los requisitos previos**.
2. En la comprobación de actualizaciones más recientes, haga clic en **Ver servicios del dispositivo**.

    ![Comprobación de la versión](./media/migrate-appliance/versions.png)

Para consultarlo en el Panel de control.

1. En el dispositivo, haga clic en **Inicio** > **Panel de control** > **Programas y características**
2. Compruebe las versiones de los servicios de dispositivo en la lista.

    ![Comprobación de la versión en el Panel de control](./media/migrate-appliance/programs-features.png)

### <a name="manually-update-an-older-version"></a>Actualización manual de una versión anterior

Si está ejecutando una versión anterior para cualquiera de los servicios, debe desinstalar el servicio y actualizar manualmente a la versión más reciente.

1. Para buscar las versiones de servicio de dispositivo más recientes, [descargue](https://aka.ms/latestapplianceservices) el archivo LatestComponents.json.
2. Después de la descarga, abra el archivo LatestComponents.json en el Bloc de notas.
3. Busque la última versión del servicio en el archivo y el vínculo de descarga correspondiente. Por ejemplo:

    "Name": "ASRMigrationWebApp", "DownloadLink": "https://download.microsoft.com/download/f/3/4/f34b2eb9-cc8d-4978-9ffb-17321ad9b7ed/MicrosoftAzureApplianceConfigurationManager.msi", "Version": "6.0.211.2", "Md5Hash": "e00a742acc35e78a64a6a81e75469b84"

4. Descargue la versión más reciente de un servicio obsoleto mediante el vínculo de descarga del archivo.
5. Después de la descarga, ejecute el siguiente comando en una ventana de comandos de administrador para comprobar la integridad del MSI descargado.

    Por ejemplo: ``` C:\>Get-FileHash -Path <file_location> -Algorithm [Hashing Algorithm] ```  C:\>CertUtil -HashFile C:\Users\public\downloads\MicrosoftAzureApplianceConfigurationManager.MSI MD5

5. Compruebe que la salida del comando coincide con la entrada de valor hash del servicio en el archivo (por ejemplo, el valor hash MD5 anterior).
6. Ahora, ejecute el archivo MSI para instalar el servicio. Se trata de una instalación silenciosa y la ventana de instalación se cierra una vez finalizada.
7. Una vez finalizada la instalación, compruebe la versión del servicio en **Panel de control** > **Programas y características**. Ahora, la versión del servicio debe actualizarse a la última que se muestra en el archivo json.

## <a name="next-steps"></a>Pasos siguientes

- [Obtenga información](how-to-set-up-appliance-vmware.md) sobre cómo configurar el dispositivo para VMware.
- [Obtenga información](how-to-set-up-appliance-hyper-v.md) sobre cómo configurar el dispositivo para Hyper-V.
- [Obtenga información](how-to-set-up-appliance-physical.md) sobre cómo configurar el dispositivo para servidores físicos.
