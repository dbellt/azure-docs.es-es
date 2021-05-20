---
title: Información general de Change Tracking e Inventario en Azure Automation
description: En este artículo se describe la característica Change Tracking e Inventario, que ayuda a identificar los cambios en el software y el servicio de Microsoft que se producen en su entorno.
services: automation
ms.subservice: change-inventory-management
ms.date: 05/06/2021
ms.topic: conceptual
ms.openlocfilehash: 062e4cdeae9560bc5be58d8245390d4a538f5722
ms.sourcegitcommit: 32ee8da1440a2d81c49ff25c5922f786e85109b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109783916"
---
# <a name="change-tracking-and-inventory-overview"></a>Información general de Change Tracking e Inventario

En este artículo se presenta la solución Change Tracking e Inventario de Azure Automation. Esta característica realiza un seguimiento de los cambios en las máquinas virtuales hospedadas en Azure, en el entorno local y en otros entornos de nube para ayudarle a identificar problemas operativos y del entorno con el software administrado por el administrador de paquetes de distribución. Los elementos cuyo seguimiento realiza Change Tracking e Inventario incluyen:

- Software de Windows
- Software de Linux (paquetes)
- Archivos de Windows y Linux
- Clave del Registro de Windows
- Servicios de Microsoft
- Demonios de Linux

> [!NOTE]
> Para hacer el seguimiento de los cambios en las propiedades de Azure Resource Manager, consulte [Historial de cambios](../../governance/resource-graph/how-to/get-resource-changes.md) de Azure Resource Graph.

Seguimiento de cambios e inventario hace uso de [Supervisión de la integridad de los archivos (FIM) de Azure Security Center](../../security-center/security-center-file-integrity-monitoring.md) para examinar los archivos de aplicación y del sistema operativo, y el Registro de Windows. Mientras FIM supervisa esas entidades, Seguimiento de cambios e inventario realiza un seguimiento de forma nativa de:

- Cambios de software
- Servicios de Microsoft
- Demonios de Linux

La habilitación de todas las características incluidas en Seguimiento de cambios e inventario podría generar cargos adicionales. Antes de continuar, revise la información que aparece en [Precios de Automation](https://azure.microsoft.com/pricing/details/automation/) y [Precios de Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/).

Seguimiento de cambios e inventario reenvía los datos a los registros de Azure Monitor y estos datos recopilados se almacenan en un área de trabajo de Log Analytics. La característica Supervisión de la integridad de los archivos (FIM) solo está disponible cuando **Azure Defender para servidores** está habilitado. Consulte [Precios de Azure Security Center](../../security-center/security-center-pricing.md) para más información. FIM carga los datos en la misma área de trabajo de Log Analytics que la creada para almacenar datos de Seguimiento de cambios e inventario. Se recomienda supervisar el área de trabajo de Log Analytics vinculada para realizar un seguimiento de su uso exacto. Para obtener más información sobre cómo analizar el uso de los datos de los registros de Azure Monitor, consulte [Administrar el uso y los costos](../../azure-monitor/logs/manage-cost-storage.md).

Las máquinas conectadas al área de trabajo de Log Analytics usan el [agente de Log Analytics](../../azure-monitor/agents/log-analytics-agent.md) para recopilar datos sobre los cambios en el software instalado, los servicios de Microsoft, los archivos y el Registro de Windows y los demonios de Linux de los servidores supervisados. Cuando los datos están disponibles, el agente los envía a Azure Monitor para su procesamiento. Los registros de Azure Monitor aplican la lógica a los datos recibidos, los anotan y hacen que estén disponibles para el análisis.

> [!NOTE]
> Change Tracking e Inventario requiere vincular un área de trabajo de Log Analytics a la cuenta de Automation. Para ver una lista definitiva de regiones admitidas, consulte [Asignaciones de áreas de trabajo](../how-to/region-mappings.md). Las asignaciones de regiones no afectan a la capacidad de administrar VM en una región independiente de la cuenta de Automation.

Como proveedor de servicios, es posible que haya incorporado varios inquilinos de cliente para [Azure Lighthouse](../../lighthouse/overview.md). Azure Lighthouse permite realizar operaciones a gran escala en varios inquilinos de Azure Active Directory (Azure AD) a la vez, lo que hace que las tareas de administración, como Seguimiento de cambios e inventario, sean más eficaces en aquellos inquilinos de los que es responsable. Seguimiento de cambios e inventario puede administrar máquinas de varias suscripciones del mismo inquilino o en varios inquilinos mediante la [Administración de recursos delegados de Azure](../../lighthouse/concepts/architecture.md).

## <a name="current-limitations"></a>Limitaciones actuales

Seguimiento de cambios e inventario no admite o tiene las siguientes limitaciones:

- Recursión para el seguimiento del registro de Windows
- Sistemas de archivos de red
- Otros métodos de instalación
- Archivos * **.exe** almacenados en Windows
- Los valores y la columna **Tamaño máximo de archivos** no se usan en la implementación actual.
- Si está haciendo un seguimiento de los cambios de archivo, se limita a un tamaño de archivo de 5 MB o menos. 
- Si intenta recopilar más de 2500 archivos en un ciclo de recopilación de 30 minutos, el rendimiento de Seguimiento de cambios e inventario podría degradarse.
- Cuando el tráfico de red es elevado, los registros de cambios pueden tardar hasta seis horas en aparecer.
- Si modifica una configuración mientras una máquina o un servidor están apagados, podrían publicarse los cambios que pertenecían a la configuración anterior.
- Recopilación de actualizaciones de revisiones en máquinas Windows Server 2016 Core RS3.
- Los demonios de Linux pueden mostrar un estado modificado aunque no se haya producido ningún cambio. Este problema se debe a la manera en que se escriben los datos de `SvcRunLevels` en la tabla [ConfigurationChange](/azure/azure-monitor/reference/tables/configurationchange) de Azure Monitor.

## <a name="limits"></a>Límites

Para ver los límites que se aplican a Seguimiento de cambios e inventario, consulte los [límites del servicio Azure Automation](../../azure-resource-manager/management/azure-subscription-service-limits.md#change-tracking-and-inventory).

## <a name="supported-operating-systems"></a>Sistemas operativos admitidos

Change Tracking e Inventario se admite en todos los sistemas operativos que cumplen los requisitos del agente de Log Analytics. Vea [Sistemas operativos admitidos](../../azure-monitor/agents/agents-overview.md#supported-operating-systems) para obtener una lista de las versiones del sistema operativo Windows y Linux compatibles con el agente de Log Analytics.

Para comprender los requisitos de cliente para TLS 1.2, consulte el artículo sobre el [cumplimiento de TLS 1.2 para Azure Automation](../automation-managing-data.md#tls-12-enforcement-for-azure-automation).

### <a name="python-requirement"></a>Requisito de Python

Seguimiento de cambios e inventario solo admite Python 2. Si la máquina usa una distribución que no incluye Python 2 de forma predeterminada, debe instalarla. Los siguientes comandos de ejemplo instalarán Python 2 en diferentes distribuciones.

- Red Hat, CentOS, Oracle: `yum install -y python2`
- Ubuntu, Debian: `apt-get install -y python2`
- SUSE: `zypper install -y python2`

El ejecutable python2 debe tener un alias para *python*.

## <a name="network-requirements"></a>Requisitos de red

Para información detallada sobre los puertos, las direcciones URL y otros detalles de red necesarios para Change Tracking and Inventory, consulte [Configuración de red de Azure Automation](../automation-network-configuration.md#update-management-and-change-tracking-and-inventory).

## <a name="enable-change-tracking-and-inventory"></a>Habilitación de Change Tracking e Inventario

Puede habilitar Seguimiento de cambios e inventario de las siguientes maneras:

- Desde la [cuenta de Automation](enable-from-automation-account.md) para una o varias máquinas de Azure y que no sean de Azure.

- De forma manual, para máquinas que no sean de Azure, incluidos servidores o máquinas registrados con [servidores habilitados para Azure Arc](../../azure-arc/servers/overview.md). Para las máquinas híbridas, se recomienda instalar el agente de Log Analytics para Windows; para ello, primero debe conectar la máquina a los [servidores habilitados para Azure Arc](../../azure-arc/servers/overview.md) y, después, usar Azure Policy para asignar la directiva integrada [Implementar el agente de Log Analytics en máquinas de Azure Arc con Windows con *Linux* o *Windows*](../../governance/policy/samples/built-in-policies.md#monitoring). Si también planea supervisar las máquinas con Azure Monitor para VM, en su lugar, use la iniciativa [Habilitar Azure Monitor para VM](../../governance/policy/samples/built-in-initiatives.md#monitoring).

- Para una sola máquina virtual de Azure, desde la [página de la máquina virtual](enable-from-vm.md) en Azure Portal. Este escenario está disponible para las VM Linux y Windows.

- Para [varias máquinas virtuales de Azure](enable-from-portal.md), selecciónelas en la página Máquinas virtuales de Azure Portal.

## <a name="tracking-file-changes"></a>Seguimiento de cambios en los archivos

Para realizar el seguimiento de los cambios en los archivos de Windows y Linux, Change Tracking e Inventario utiliza los valores hash MD5 de los archivos. La característica usa los valores hash para detectar si se han realizado cambios desde el último inventario.

## <a name="tracking-file-content-changes"></a>Seguimiento de cambios en el contenido de los archivos

Change Tracking e Inventario le permite ver el contenido de un archivo de Windows o Linux. Para cada cambio en un archivo, Change Tracking e Inventario almacena el contenido del archivo en una [cuenta de Azure Storage](../../storage/common/storage-account-create.md). Cuando realiza el seguimiento de un archivo, puede ver su contenido antes o después de un cambio. El contenido del archivo se puede ver en línea o en paralelo.

![Visualización de los cambios en un archivo](./media/overview/view-file-changes.png)

## <a name="tracking-of-registry-keys"></a>Seguimiento de las claves del Registro

Change Tracking e Inventario permite la supervisión de los cambios en las claves del Registro de Windows. La supervisión permite identificar los puntos de extensibilidad en los que se puede activar malware y código de terceros. En la tabla siguiente se enumeran las claves del Registro preconfiguradas (pero no habilitadas). Para realizar el seguimiento de estas claves, debe habilitar cada una de ellas.

> [!div class="mx-tdBreakAll"]
> |Clave del Registro | Propósito |
> | --- | --- |
> |`HKEY\LOCAL\MACHINE\Software\Microsoft\Windows\CurrentVersion\Group Policy\Scripts\Startup` | Supervisa los scripts que se ejecutan al arrancar.
> |`HKEY\LOCAL\MACHINE\Software\Microsoft\Windows\CurrentVersion\Group Policy\Scripts\Shutdown` | Supervisa los scripts que se ejecutan al apagar el equipo.
> |`HKEY\LOCAL\MACHINE\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Run` | Supervisa las claves que se cargan antes de que el usuario inicie sesión en la cuenta de Windows. La clave se usa para aplicaciones de 32 bits que se ejecutan en equipos de 64 bits.
> |`HKEY\LOCAL\MACHINE\SOFTWARE\Microsoft\Active Setup\Installed Components` | Supervisa los cambios en la configuración de la aplicación.
> |`HKEY\LOCAL\MACHINE\Software\Classes\Directory\ShellEx\ContextMenuHandlers` | Supervisa los controladores de los menús contextuales que se enlazan directamente con el Explorador de Windows y se suelen ejecutar en el proceso con **explorer.exe**.
> |`HKEY\LOCAL\MACHINE\Software\Classes\Directory\Shellex\CopyHookHandlers` | Supervisa los controladores de enlace de copia que se enlazan directamente con el Explorador de Windows y se suelen ejecutar en el proceso con **explorer.exe**.
> |`HKEY\LOCAL\MACHINE\Software\Microsoft\Windows\CurrentVersion\Explorer\ShellIconOverlayIdentifiers` | Supervisa el registro del controlador de superposición de iconos.
> |`HKEY\LOCAL\MACHINE\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Explorer\ShellIconOverlayIdentifiers` | Supervisa el registro del controlador de superposición de iconos para aplicaciones de 32 bits que se ejecutan en equipos de 64 bits.
> |`HKEY\LOCAL\MACHINE\Software\Microsoft\Windows\CurrentVersion\Explorer\Browser Helper Objects` | Supervisa si hay nuevos complementos de objetos auxiliares de explorador para Internet Explorer. Se utiliza para acceder a Document Object Model (DOM) de la página actual y controlar la navegación.
> |`HKEY\LOCAL\MACHINE\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Explorer\Browser Helper Objects` | Supervisa si hay nuevos complementos de objetos auxiliares de explorador para Internet Explorer. Se usa para acceder a Document Object Model (DOM) de la página actual y controlar la navegación de aplicaciones de 32 bits que se ejecutan en equipos de 64 bits.
> |`HKEY\LOCAL\MACHINE\Software\Microsoft\Internet Explorer\Extensions` | Supervisa si hay nuevas extensiones de Internet Explorer, como menús de la herramienta personalizada y botones de la barra de herramientas personalizada.
> |`HKEY\LOCAL\MACHINE\Software\Wow6432Node\Microsoft\Internet Explorer\Extensions` | Supervisa si hay nuevas extensiones de Internet Explorer, como menús de la herramienta personalizada y botones de la barra de herramientas personalizada para aplicaciones de 32 bits que se ejecutan en equipos de 64 bits.
> |`HKEY\LOCAL\MACHINE\Software\Microsoft\Windows NT\CurrentVersion\Drivers32` | Supervisa los controladores de 32 bits asociados a wavemapper, wave1 y wave2, msacm.imaadpcm, .msadpcm, .msgsm610 y vidc. Es similar a la sección [drivers] del archivo **system.ini**.
> |`HKEY\LOCAL\MACHINE\Software\Wow6432Node\Microsoft\Windows NT\CurrentVersion\Drivers32` | Supervisa los controladores de 32 bits asociados a wavemapper, wave1 y wave2, msacm.imaadpcm, .msadpcm, .msgsm610 y vidc para aplicaciones de 32 bits que se ejecutan en equipos de 64 bits. Es similar a la sección [drivers] del archivo **system.ini**.
> |`HKEY\LOCAL\MACHINE\System\CurrentControlSet\Control\Session Manager\KnownDlls` | Supervisa la lista de DDL del sistema conocidas o utilizadas habitualmente. La supervisión impide que las personas se aprovechen de permisos de directorio de aplicaciones débiles colocando versiones de troyanos en archivos DLL del sistema.
> |`HKEY\LOCAL\MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon\Notify` | Supervisa la lista de paquetes que pueden recibir notificaciones de eventos de **winlogon.exe**, el modelo de compatibilidad de inicio de sesión interactivo para Windows.

## <a name="recursion-support"></a>Compatibilidad con la recursión

Change Tracking e Inventory es compatible con la recursión, lo que permite especificar comodines para simplificar el seguimiento entre directorios. La recursión también proporciona variables de entorno que permiten realizar un seguimiento de los archivos en entornos con nombres de unidad varios o dinámicos. En la lista siguiente se incluye información común que debe conocer al configurar la recursión:

- Los caracteres comodín son necesarios para realizar el seguimiento de varios archivos.

- Puede usar caracteres comodín solo en el último segmento de la ruta de acceso de un archivo, como **c:\carpeta\\archivo** _ o _ */etc/* .conf**.

- Si una variable de entorno tiene una ruta de acceso no válida, la validación será correcta pero se producirá un error en dicha ruta durante la ejecución.

- Al establecer la ruta de acceso, debe evitar los nombres de ruta de acceso generales, ya que daría lugar a que se recorrieran demasiadas carpetas.

## <a name="change-tracking-and-inventory-data-collection"></a>Recopilación de datos de Change Tracking e Inventory

En la tabla siguiente se muestra la frecuencia de recopilación de datos para los tipos de cambios que admite Change Tracking e Inventario. Para cada tipo, la instantánea de datos del estado actual también se actualiza al menos cada 24 horas.

| **Tipo de cambio** | **Frecuencia** |
| --- | --- |
| Registro de Windows | 50 minutos |
| Archivo de Windows | 30 minutos |
| Archivo de Linux | 15 minutos |
| Servicios de Microsoft | 10 segundos hasta 30 minutos</br> Valor predeterminado: 30 minutos |
| Demonios de Linux | 5 minutos |
| Software de Windows | 30 minutos |
| Software Linux | 5 minutos |

En la siguiente tabla se muestran los límites de elementos sometidos a seguimiento por máquina para Change Tracking e Inventario.

| **Recurso** | **Límite** |
|---|---|---|
|Archivo|500|
|Registro|250|
|Software de Windows (sin incluir revisiones) |250|
|Paquetes Linux|1250|
|Servicios|250|
|Demonios|250|

El uso medio de datos de Log Analytics para una máquina con Change Tracking e Inventario es aproximadamente de 40 MB al mes, según su entorno. Con la característica Uso y costos estimados del área de trabajo de Log Analytics, puede ver los datos ingeridos por Change Tracking e Inventario en un gráfico de uso. Use esta vista de datos para evaluar el uso de los datos y determinar cómo afecta a su factura. Consulte [Información útil del uso y los costos estimados](../../azure-monitor/logs/manage-cost-storage.md#understand-your-usage-and-estimate-costs).

### <a name="microsoft-service-data"></a>Datos de servicio de Microsoft

La frecuencia de recopilación predeterminada para los servicios de Microsoft es de 30 minutos. Puede configurar la frecuencia con un control deslizante en la pestaña **Servicios de Microsoft** bajo **Editar configuración**.

![Control deslizante de Servicios de Microsoft](./media/overview/windowservices.png)

Para optimizar el rendimiento, el agente de Log Analytics solo realiza el seguimiento de los cambios. Al establecer un umbral alto, se pueden perder los cambios si el servicio vuelve a su estado original. Si la frecuencia se establece en un valor menor, es posible detectar los cambios que de otra forma podrían pasar desapercibidos.

> [!NOTE]
> Aunque el agente puede seguir los cambios hasta en intervalos de 10 segundos, los datos todavía tardan unos minutos en mostrarse en Azure Portal. Durante el tiempo que lleva la visualización en el portal, se continúa realizando el seguimiento y registro de los cambios que se producen.

## <a name="support-for-alerts-on-configuration-state"></a>Compatibilidad con alertas sobre el estado de configuración

Una funcionalidad clave de Change Tracking e Inventario son las alertas sobre los cambios en el estado de configuración de su entorno híbrido. Hay muchas acciones útiles que se pueden desencadenar en respuesta a las alertas. Por ejemplo, acciones de Azure Functions, runbooks de Automation, webhooks y similares. Las alertas sobre los cambios en el archivo **c:\windows\system32\drivers\etc\hosts** de una máquina es una buena aplicación de alertas para los datos de Change Tracking e Inventario. También hay muchos más escenarios para las alertas, incluidos los escenarios de consulta que se definen en la tabla siguiente.

|Consultar  |Descripción  |
|---------|---------|
|ConfigurationChange <br>&#124; where ConfigChangeType == "Files" and FileSystemPath contains " c:\\windows\\system32\\drivers\\"|Útil para el seguimiento de cambios en archivos críticos del sistema.|
|ConfigurationChange <br>&#124; where FieldsChanged contains "FileContentChecksum" and FileSystemPath == "c:\\windows\\system32\\drivers\\etc\\hosts"|Útil para el seguimiento de las modificaciones de los archivos de configuración de claves.|
|ConfigurationChange <br>&#124; where ConfigChangeType == "WindowsServices" and SvcName contains "w3svc" and SvcState == "Stopped"|Útil para el seguimiento de cambios en servicios críticos del sistema.|
|ConfigurationChange <br>&#124; where ConfigChangeType == "Daemons" and SvcName contains "ssh" and SvcState!= "Running"|Útil para el seguimiento de cambios en servicios críticos del sistema.|
|ConfigurationChange <br>&#124; where ConfigChangeType == "Software" and ChangeCategory == "Added"|Útil para entornos que necesitan configuraciones de software bloqueadas.|
|ConfigurationData <br>&#124; where SoftwareName contains "Monitoring Agent" and CurrentVersion!= "8.0.11081.0"|Útil para ver qué máquinas tienen instalada una versión de software obsoleta o incompatible. Esta consulta notifica el último estado de configuración notificado, no los cambios.|
|ConfigurationChange <br>&#124; where RegistryKey == @"HKEY_LOCAL_MACHINE\\SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\QualityCompat"| Útil para el seguimiento de los cambios en claves de antivirus fundamentales.|
|ConfigurationChange <br>&#124; where RegistryKey contains @"HKEY_LOCAL_MACHINE\\SYSTEM\\CurrentControlSet\\Services\\SharedAccess\\Parameters\\FirewallPolicy"| Útil para el seguimiento de los cambios en la configuración del firewall.|

## <a name="next-steps"></a>Pasos siguientes

- Para habilitar la característica desde una cuenta de Automation, consulte [Habilitación de Seguimiento de cambios e inventario desde una cuenta de Automation](enable-from-automation-account.md).

- Para habilitar la característica desde Azure Portal, consulte [Habilitación de Seguimiento de cambios e inventario desde Azure Portal](enable-from-portal.md).

- Para habilitar la característica desde un runbook, consulte [Habilitación de Seguimiento de cambios e inventario desde un runbook](enable-from-runbook.md).

- Para habilitar la característica desde una máquina virtual de Azure, consulte [Habilitación de Seguimiento de cambios e inventario desde una máquina virtual de Azure](enable-from-vm.md).
