---
title: Introducción a Update Management en Azure Automation
description: En este artículo se ofrece información general de la característica Update Management que implementa las actualizaciones de las máquinas Windows y Linux.
services: automation
ms.subservice: update-management
ms.date: 05/04/2021
ms.topic: conceptual
ms.openlocfilehash: f04334b4b9704d1048d03ed82aaef400806b219f
ms.sourcegitcommit: 32ee8da1440a2d81c49ff25c5922f786e85109b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109787037"
---
# <a name="update-management-overview"></a>Introducción a Update Management

Puede usar Update Management en Azure Automation para administrar las actualizaciones del sistema operativo de las máquinas virtuales Windows y Linux en Azure, en entornos locales y en otros entornos en la nube. Puede evaluar rápidamente el estado de las actualizaciones disponibles en todas las máquinas agente y administrar el proceso de instalación de las actualizaciones necesarias para los servidores.

Como proveedor de servicios, es posible que haya incorporado varios inquilinos de cliente para [Azure Lighthouse](../../lighthouse/overview.md). Azure Lighthouse permite realizar operaciones a gran escala en varios inquilinos de Azure Active Directory (Azure AD) a la vez, lo que hace que las tareas de administración, como la administración de actualizaciones, sean más eficaces en esos inquilinos de los que es responsable.

> [!NOTE]
> No se puede usar una máquina configurada con Update Management para ejecutar scripts personalizados desde Azure Automation. Esta máquina solo puede ejecutar el script de actualización con firma de Microsoft.

> [!NOTE]
> En este momento, no se admite la habilitación de Update Management directamente desde un servidor habilitado para Arc. Consulte [Habilitación de Update Management desde una cuenta de Automation](../../automation/update-management/enable-from-automation-account.md) para saber más sobre los requisitos y cómo habilitarlo para el servidor.

Para descargar e instalar automáticamente las revisiones *críticas* y de *seguridad* disponibles en la máquina virtual de Azure, consulte [Revisiones de invitado de máquina virtual automática](../../virtual-machines/automatic-vm-guest-patching.md) para máquinas virtuales Windows.

Antes de implementar Update Management y habilitar las máquinas para su administración, asegúrese de que comprende la información de las secciones siguientes.

## <a name="about-update-management"></a>Acerca de Update Management

Las máquinas administradas por Update Management se basan en lo siguiente para realizar valoraciones y para implementar actualizaciones:

* [Agente de Log Analytics](../../azure-monitor/agents/log-analytics-agent.md) para Windows o Linux
* Extensión DSC (configuración de estado deseado) de PowerShell para Linux
* Hybrid Runbook Worker de Automation (se instala automáticamente al habilitar Update Management en la máquina)
* Microsoft Update o [Windows Server Update Services](/windows-server/administration/windows-server-update-services/get-started/windows-server-update-services-wsus) (WSUS) para máquinas Windows
* Un repositorio de actualización privado o público para máquinas Linux

El siguiente diagrama muestra cómo Update Management evalúa y aplica las actualizaciones de seguridad a todos los servidores Windows Server y Linux conectados de un área de trabajo:

![Flujo de trabajo de Update Management](./media/overview/update-mgmt-updateworkflow.png)

Update Management puede usarse para implementar de forma nativa máquinas en varias suscripciones del mismo inquilino o en varios inquilinos mediante [Azure Lighthouse](../../lighthouse/overview.md).

Después de publicarse un paquete, la revisión tarda de 2 a 3 horas en aparecer en las máquinas Linux para su evaluación. Para las máquinas Windows, la revisión tarda de 12 a 15 horas en aparecer para su evaluación tras su publicación. Después de que una máquina finalice un examen de cumplimiento de actualizaciones, el agente reenvía la información de forma masiva a los registros de Azure Monitor. En una máquina Windows, el examen de cumplimiento se ejecuta cada 12 horas de forma predeterminada. En una máquina Linux, el examen de cumplimiento se realiza cada hora de manera predeterminada. Si se reinicia el agente de Log Analytics, se inicia un examen de cumplimiento al cabo de 15 minutos.

Además del examen programado, el examen de cumplimiento de las actualizaciones se inicia a los 15 minutos del reinicio del agente de Log Analytics, así como antes y después de la instalación de actualizaciones.

Update Management informa del grado de actualización de la máquina en función del origen configurado para la sincronización. Si la máquina Windows está configurada para informar a [Windows Server Update Services](/windows-server/administration/windows-server-update-services/get-started/windows-server-update-services-wsus) (WSUS), en función de cuándo WSUS se sincronizó por última vez con Microsoft Update, los resultados pueden diferir de lo que se muestra en Microsoft Update. Este comportamiento es el mismo para las máquinas Linux que están configuradas para informar a un repositorio local en lugar de a uno público.

> [!NOTE]
> Para informar correctamente al servicio, Update Management requiere la habilitación de ciertas direcciones URL y puertos. Para más información sobre estos requisitos, consulte [Configuración de red](../automation-hybrid-runbook-worker.md#network-planning).

Puede implementar e instalar las actualizaciones de software en las máquinas que requieren las actualizaciones mediante la creación de una implementación programada. Las actualizaciones clasificadas como opcionales no se incluyen en el ámbito de implementación en máquinas Windows. Solo se incluyen las actualizaciones necesarias.

La implementación programada define qué máquina de destino reciben las actualizaciones aplicables. Lo hace mediante la especificación explícita de determinadas máquinas o por medio de la selección de un [grupo de equipos](../../azure-monitor/logs/computer-groups.md) que se basa en las búsquedas de registros de un conjunto determinado de máquinas (o en una [consulta de Azure](query-logs.md) que selecciona de forma dinámica las máquinas virtuales de Azure en función de los criterios especificados). Estos grupos difieren de la [configuración de ámbito](../../azure-monitor/insights/solution-targeting.md), que se usa para controlar los destinos de las máquinas que reciben la configuración para habilitar Update Management. Esto evita que realicen la comprobación de actualizaciones e informen sobre la misma, además de instalar las actualizaciones necesarias aprobadas.

Al definir una implementación, también se especifica una programación para aprobar y establecer un período de tiempo durante el que se pueden instalar actualizaciones. Este período se denomina ventana de mantenimiento. Un intervalo de 20 minutos de la ventana de mantenimiento se reserva para los reinicios, suponiendo que sea necesario reiniciar y que haya seleccionado la opción de reinicio adecuada. Si la aplicación de revisiones tarda más de lo esperado y la ventana de mantenimiento dura menos de 20 minutos, no se producirá un reinicio.

Los Runbooks instalan las actualizaciones en Azure Automation. No puede ver estos runbooks, que no requieren ninguna configuración. Cuando se crea una implementación de actualizaciones, esta crea una programación que inicia un runbook de actualización maestro a la hora especificada para las máquinas incluidas. El runbook maestro inicia un runbook secundario en cada agente para instalar las actualizaciones necesarias.

En la fecha y hora especificadas en la implementación de actualizaciones, las máquinas de destino ejecutan la implementación en paralelo. Antes de la instalación, se ejecuta un examen para comprobar que las actualizaciones siguen siendo necesarias. En las máquinas cliente de WSUS, si no se aprueban las actualizaciones en WSUS, se produce un error en la implementación de actualizaciones.

No se permite tener registrada una máquina para Update Management en más de un área de trabajo de Log Analytics (también conocido como hospedaje múltiple).

## <a name="limits"></a>Límites

Para ver los límites que se aplican a Update Management, consulte los [límites del servicio Azure Automation](../../azure-resource-manager/management/azure-subscription-service-limits.md#update-management).

## <a name="clients"></a>Clientes

### <a name="supported-operating-systems"></a>Sistemas operativos admitidos

En la tabla siguiente se muestra una lista de sistemas operativos compatibles para evaluaciones de actualizaciones y parches. La aplicación de revisiones requiere un sistema Hybrid Runbook Worker, que se instala automáticamente al habilitar la máquina virtual o el servidor para la administración mediante Update Management. Para obtener información sobre los requisitos del sistema para Hybrid Runbook Worker, consulte [Implementación de Hybrid Runbook Worker en Windows](../automation-windows-hrw-install.md) e [Implementación de Hybrid Runbook Worker en Linux](../automation-linux-hrw-install.md).

> [!NOTE]
> La evaluación de la actualización de máquinas Linux solo se admite en determinadas regiones tal como se muestra en la [tabla de asignaciones](../how-to/region-mappings.md#supported-mappings) del área de trabajo de Log Analytics y la cuenta de Automation.

|Sistema operativo  |Notas  |
|---------|---------|
|Windows Server 2019 (Datacenter o Standard incluido Server Core)<br><br>Windows Server 2016 (Datacenter o Standard sin incluir Server Core)<br><br>Windows Server 2012 R2 (Datacenter/Standard)<br><br>Windows Server 2012 | |
|Windows Server 2008 R2 (RTM and SP1 Standard)| Update Management admite evaluaciones y parches para este sistema operativo. [Hybrid Runbook Worker](../automation-windows-hrw-install.md) es compatible con Windows Server 2008 R2. |
|CentOS 6, 7 y 8 (x64)      | Los agentes de Linux requieren acceso a un repositorio de actualización. La aplicación de revisiones basada en la clasificación requiere `yum` para devolver los datos de seguridad que CentOS no tiene en sus versiones RTM. Para más información sobre la aplicación de revisiones basadas en clasificaciones en CentOS, consulte [Actualización de clasificaciones en Linux](view-update-assessments.md#linux).          |
|Red Hat Enterprise 6, 7 y 8 (x64)     | Los agentes de Linux requieren acceso a un repositorio de actualización.        |
|SUSE Linux Enterprise Server 12, 15 y 15.1 (x64)     | Los agentes de Linux requieren acceso a un repositorio de actualización. Para SUSE 15.x, se requiere Python 3 en el equipo.      |
|Ubuntu 14.04 LTS, 16.04 LTS y 18.04 LTS (x64)      |Los agentes de Linux requieren acceso a un repositorio de actualización.         |

> [!NOTE]
> Update Management no admite la automatización de la administración de actualizaciones de forma segura en todas las instancias de un conjunto de escalado de máquinas virtuales de Azure. La [actualización automática de imágenes de sistema operativo](../../virtual-machine-scale-sets/virtual-machine-scale-sets-automatic-upgrade.md) es el método recomendado para administrar las actualizaciones de este tipo de imágenes en el conjunto de escalado.

### <a name="unsupported-operating-systems"></a>Sistemas operativos no admitidos

En la tabla siguiente se enumeran los sistemas operativos no admitidos por Update Management:

|Sistema operativo  |Notas  |
|---------|---------|
|Cliente Windows     | No se admiten sistemas operativos cliente (por ejemplo, Windows 7 y Windows 10).<br> En el caso de Windows Virtual Desktop(WVD) de Azure, el método recomendado<br> para administrar las actualizaciones es usar [Microsoft Endpoint Configuration Manager](../../virtual-desktop/configure-automatic-updates.md) para la administración de revisiones de máquinas cliente de Windows 10. |
|Windows Server 2016 Nano Server     | No compatible.       |
|Nodos de Azure Kubernetes Service | No compatible. Use el proceso de aplicación de revisiones que se describe en [Aplicación de actualizaciones de kernel y seguridad en los nodos de Linux en Azure Kubernetes Service (AKS)](../../aks/node-updates-kured.md)|

### <a name="system-requirements"></a>Requisitos del sistema

La información siguiente describe los requisitos específicos del sistema operativo. Para obtener más instrucciones, vea [Planeamiento de red](#ports). Para comprender los requisitos para TLS 1.2, consulte el artículo sobre el [cumplimiento de TLS 1.2 para Azure Automation](../automation-managing-data.md#tls-12-enforcement-for-azure-automation).

#### <a name="windows"></a>Windows

Requisitos de software:

- Se requiere .NET Framework 4.6 o posterior. ([Descargue .NET Framework](/dotnet/framework/install/guide-for-developers)).
- Se requiere Windows PowerShell 5.1. ([Descargue Windows Management Framework 5.1](https://www.microsoft.com/download/details.aspx?id=54616)).

Los agentes de Windows deben estar configurados para comunicarse con un servidor de WSUS o requieren acceso a Microsoft Update. Para las máquinas híbridas, se recomienda instalar el agente de Log Analytics para Windows; para ello, primero debe conectar la máquina a los [servidores habilitados para Azure Arc](../../azure-arc/servers/overview.md) y, después, use Azure Policy para asignar la directiva integrada [Implementar el agente de Log Analytics en máquinas de Azure Arc con Windows](../../governance/policy/samples/built-in-policies.md#monitoring). Como alternativa, si planea supervisar las máquinas con Azure Monitor para VM, en su lugar, use la iniciativa [Habilitar Azure Monitor para VM](../../governance/policy/samples/built-in-initiatives.md#monitoring).

Puede usar Update Management con Microsoft Endpoint Configuration Manager. Para más información sobre escenarios de integración, consulte este artículo sobre la [integración de Update Management con Endpoint Configuration Manager de Windows](mecmintegration.md). El [agente de Log Analytics para Windows](../../azure-monitor/agents/agent-windows.md) es necesario para los servidores Windows administrados por sitios en su entorno de Configuration Manager.

De forma predeterminada, las máquinas virtuales Windows implementadas desde Azure Marketplace se establecen para recibir actualizaciones automáticas del servicio de Windows Update. Este comportamiento no cambia cuando se agregan VM Windows al área de trabajo. Si no administra activamente las actualizaciones mediante Update Management, se aplica el comportamiento predeterminado (las actualizaciones se aplican automáticamente).

> [!NOTE]
> Puede modificar la directiva de grupo para que los reinicios de la máquina solo los pueda realizar el usuario, no el sistema. Es posible que las máquinas administradas se bloqueen si Update Management no tiene derechos para reiniciar la máquina sin la intervención manual del usuario. Para obtener más información, consulte el tema sobre la [configuración de la directiva de grupo para actualizaciones automáticas](/windows-server/administration/windows-server-update-services/deploy/4-configure-group-policy-settings-for-automatic-updates).

#### <a name="linux"></a>Linux

Requisitos de software:

- La máquina requiere acceso a un repositorio de actualización, privado o público.
- Se requiere TLS 1.1 o TLS 1.2 para interactuar con Update Management.
- Python 2.x instalado.

> [!NOTE]
> La evaluación de la actualización de máquinas Linux solo se admite en determinadas regiones. Consulte la [tabla de asignaciones](../how-to/region-mappings.md#supported-mappings) del área de trabajo de Log Analytics y la cuenta de Automation.

Para las máquinas híbridas, se recomienda instalar el agente de Log Analytics para Linux; para ello, primero debe conectar la máquina a los [servidores habilitados para Azure Arc](../../azure-arc/servers/overview.md) y, después, use Azure Policy para asignar la directiva integrada [Implementar el agente de Log Analytics en máquinas de Azure Arc con Linux](../../governance/policy/samples/built-in-policies.md#monitoring). Como alternativa, si planea supervisar las máquinas con Azure Monitor para VM, en su lugar, use la iniciativa [Habilitar Azure Monitor para VM](../../governance/policy/samples/built-in-initiatives.md#monitoring).

Las máquinas virtuales que se crearon a partir de las imágenes a petición de Red Hat Enterprise Linux (RHEL) que están disponibles en Azure Marketplace están registradas para acceder a la instancia de [Red Hat Update Infrastructure (RHUI)](../../virtual-machines/workloads/redhat/redhat-rhui.md) implementada en Azure. Cualquier otra distribución de Linux se debe actualizar desde el repositorio de archivos en línea de la distribución mediante los métodos que admite esta.

## <a name="permissions"></a>Permisos

Para crear y administrar implementaciones de actualizaciones, necesita permisos concretos. Para más información sobre estos permisos, consulte [Acceso basado en rol: Update Management](../automation-role-based-access-control.md#update-management-permissions).

## <a name="update-management-components"></a>Componentes de Update Management

Update Management usa los recursos descritos en esta sección. Estos recursos se agregan automáticamente a la cuenta de Automation al habilitar Update Management.

### <a name="hybrid-runbook-worker-groups"></a>Grupos de Trabajos híbridos de runbook

Después de habilitar Update Management, las máquinas Windows conectadas directamente al área de trabajo de Log Analytics se configuran de modo automático como un sistema Hybrid Runbook Worker para admitir los runbooks que admiten a su vez Update Management.

Cada máquina Windows administrada por Update Management se muestra en el panel Grupos de Hybrid Worker como un grupo Hybrid Worker del sistema para la cuenta de Automation. Los grupos usan la convención de nomenclatura `Hostname FQDN_GUID`. No puede usar estos grupos como destino con runbooks de su cuenta. Si lo intenta, se producirá un error. Estos grupos están diseñados únicamente para admitir Update Management. Para más información sobre la visualización de la lista de máquinas Windows configuradas como Hybrid Runbook Worker, consulte [Hybrid Runbook Worker](../automation-hybrid-runbook-worker.md#view-system-hybrid-runbook-workers).

Puede agregar la máquina Windows a un grupo de Hybrid Runbook Worker de usuario en la cuenta de Automation para admitir runbooks de Automation si usa la misma cuenta para Update Management y la pertenencia a grupos de Hybrid Runbook Worker. Esta funcionalidad se agregó en la versión 7.2.12024.0 de Hybrid Runbook Worker.

### <a name="management-packs"></a>Módulos de administración

Si el grupo de administración de Operations Manager está [conectado a un área de trabajo de Log Analytics](../../azure-monitor/agents/om-agents.md), se instalarán los siguientes módulos de administración en Operations Manager. Estos módulos de administración también se instalan para Update Management en las máquinas Windows conectadas directamente. No es necesario configurar ni administrar dichos módulos.

* Intelligence Pack Update Assessment de Microsoft System Center Advisor (Microsoft.IntelligencePacks.UpdateAssessment)
* Microsoft.IntelligencePack.UpdateAssessment.Configuration (Microsoft.IntelligencePack.UpdateAssessment.Configuration)
* Módulo de administración de Update Deployment

> [!NOTE]
> Si tiene un grupo de administración de Operations Manager 1807 o 2019 conectado a un área de trabajo de Log Analytics con agentes configurados en el grupo de administración para recopilar los datos de registro, debe invalidar el parámetro `IsAutoRegistrationEnabled` y establecerlo en True en la regla **Microsoft.IntelligencePacks.AzureAutomation.HybridAgent.Init**.

Para más información sobre las actualizaciones de los módulos de administración, consulte [Conexión de Operations Manager con registros de Azure Monitor](../../azure-monitor/agents/om-agents.md).

> [!NOTE]
> Para que Update Management administre totalmente las máquinas que tengan el agente de Log Analytics, debe actualizar al agente de Log Analytics para Windows o Linux. Para aprender a actualizar el agente, consulte [Actualización de Operations Manager Agent](/system-center/scom/deploy-upgrade-agents). En entornos que usan Operations Manager, debe ejecutar System Center Operations Manager 2012 R2 UR 14 o posterior.

## <a name="data-collection"></a>datos, recopilación

### <a name="supported-sources"></a>Orígenes compatibles

En la tabla siguiente se describen los orígenes conectados compatibles con Update Management:

| Origen conectado | Compatible | Descripción |
| --- | --- | --- |
| Agentes de Windows |Sí |Update Management recopila información sobre las actualizaciones del sistema de los agentes de Windows e inicia la instalación de las actualizaciones necesarias. |
| Agentes de Linux |Sí |Update Management recopila información sobre las actualizaciones del sistema de los agentes de Linux e inicia la instalación de las actualizaciones necesarias en las distribuciones admitidas. |
| Grupo de administración de Operations Manager |Sí |Update Management recopila información sobre las actualizaciones del sistema de los agentes de un grupo de administración conectado.<br/><br/>No se requiere ninguna conexión directa entre el agente de Operations Manager y los registros de Azure Monitor. Los datos se reenvían desde el grupo de administración al área de trabajo de Log Analytics. |

### <a name="collection-frequency"></a>Frecuencia de recopilación

Update Management examina los datos de las máquinas administradas con las siguientes reglas. Puede que transcurran entre 30 minutos y 6 horas antes de que se muestren los datos actualizados de las máquinas administradas.

* Cada máquina Windows: Update Management realiza un examen dos veces al día de cada máquina.

* Cada máquina Linux: Update Management realiza un examen cada hora.

El uso medio de datos de los registros de Azure Monitor para una máquina que utiliza Update Management es aproximadamente de 25 MB al mes. Este valor es solo una aproximación y está sujeto a cambios en función de su entorno. Se recomienda supervisar el entorno para realizar un seguimiento del uso exacto. Para obtener más información sobre cómo analizar el uso de los datos de los registros de Azure Monitor, consulte [Administrar el uso y los costos](../../azure-monitor/logs/manage-cost-storage.md).

## <a name="network-planning"></a><a name="ports"></a>Planeamiento de red

Vea [Configuración de red de Azure Automation](../automation-network-configuration.md#hybrid-runbook-worker-and-state-configuration) para obtener información detallada sobre los puertos, las direcciones URL y otros detalles de red necesarios para Update Management.

En el caso de máquinas Windows, también debe permitir el tráfico a los puntos de conexión que necesita Windows Update. Puede encontrar una lista actualizada de los puntos de conexión necesarios en [Problemas relacionados con HTTP/Proxy](/windows/deployment/update/windows-update-troubleshooting#issues-related-to-httpproxy). Si tiene un [servidor de Windows Update](/windows-server/administration/windows-server-update-services/plan/plan-your-wsus-deployment) local, también debe permitir el tráfico al servidor especificado en la [clave de WSUS](/windows/deployment/update/waas-wu-settings#configuring-automatic-updates-by-editing-the-registry).

En el caso de máquinas de Red Hat Linux, consulte las [direcciones IP de los servidores de entrega de contenido de RHUI](../../virtual-machines/workloads/redhat/redhat-rhui.md#the-ips-for-the-rhui-content-delivery-servers) de los puntos de conexión necesarios. Para otras distribuciones de Linux, consulte la documentación del proveedor.

Para más información sobre los puertos que Hybrid Runbook Worker requiere, consulte [Direcciones de Update Management para Hybrid Runbook Worker](../automation-hybrid-runbook-worker.md#update-management-addresses-for-hybrid-runbook-worker).

Si las directivas de seguridad de TI no permiten que las máquinas de la red se conecten a Internet, puede configurar una [puerta de enlace de Log Analytics](../../azure-monitor/agents/gateway.md) y, luego, configurar la máquina para conectarse a Azure Monitor y a Azure Automation mediante la puerta de enlace.

## <a name="update-classifications"></a>Clasificaciones de actualizaciones

En la tabla siguiente se definen las clasificaciones que Update Management admite para las actualizaciones de Windows.

|clasificación  |Descripción  |
|---------|---------|
|Actualizaciones críticas     | Actualización para un problema específico que resuelve un error crítico no relacionado con la seguridad.        |
|Actualizaciones de seguridad     | Actualización para un problema específico del producto relacionado con la seguridad.        |
|Paquetes acumulativos de actualizaciones     | Conjunto acumulativo de revisiones que se empaquetan para facilitar la implementación.        |
|Feature Packs     | Nuevas características del producto que se distribuyen fuera de una versión del producto.        |
|Service Packs     | Un conjunto acumulativo de revisiones que se aplican a una aplicación.        |
|Actualizaciones de definiciones     | Una actualización de archivos de definiciones de virus o de otra índole.        |
|Herramientas     | Utilidad o característica que ayuda a realizar una o más tareas.        |
|Actualizaciones     | Actualización de una aplicación o archivo que están instalados actualmente.        |

En la tabla siguiente se definen las clasificaciones admitidas para las actualizaciones de Linux.

|clasificación  |Descripción  |
|---------|---------|
|Actualizaciones críticas y de seguridad     | Actualizaciones para un problema específico o un problema de un producto específico relacionado con la seguridad.         |
|Otras actualizaciones     | Todas las demás actualizaciones que ni son críticas por naturaleza ni son actualizaciones de seguridad.        |

>[!NOTE]
>La clasificación de actualizaciones para máquinas Linux solo está disponible en las regiones de nube pública de Azure admitidas. No hay clasificación de actualizaciones de Linux si se usa Update Management en las siguientes regiones de nube nacional:
>
>* Azure US Government
>* 21Vianet en China
>
> En lugar de clasificarse, las actualizaciones se muestran en la categoría **Otras actualizaciones**.
>
> Update Management usa los datos publicados por las distribuciones admitidas, en concreto, sus archivos de [OVAL](https://oval.mitre.org/) (Open Vulnerability and Assessment Language). Dado que el acceso a Internet está restringido desde estas nubes nacionales, Update Management no puede acceder a los archivos.

En Linux, Update Management puede distinguir entre actualizaciones críticas y de seguridad en la nube en la clasificación **Seguridad** y en **Otras**, y mostrar al mismo tiempo datos de evaluación debido al enriquecimiento de datos en la nube. Para aplicar revisiones, Update Management se basa en los datos de clasificación disponibles en la máquina. A diferencia de otras distribuciones, CentOS no dispone de esta información en la versión RTM. Si tiene máquinas de CentOS configuradas para devolver datos de seguridad para el siguiente comando, Update Management puede aplicar revisiones basadas en clasificaciones.

```bash
sudo yum -q --security check-update
```

Actualmente no hay ningún método compatible para habilitar la disponibilidad de datos de clasificación nativos en CentOS. En este momento, se proporciona soporte técnico limitado a clientes que puedan haber habilitado esta característica por su cuenta.

Para clasificar las actualizaciones de la versión 6 de Red Hat Enterprise, debe instalar el complemento de seguridad de yum. En Red Hat Enterprise Linux 7, el complemento ya forma parte de yum y no es necesario instalar nada. Para más información, consulte el siguiente [artículo de conocimientos](https://access.redhat.com/solutions/10021) de Red Hat.

Cuando se programa una actualización para que se ejecute en una máquina Linux que, por ejemplo, está configurada para instalar solo las actualizaciones que coincidan con la clasificación **Seguridad**, las actualizaciones instaladas podrían ser diferentes de las actualizaciones que coinciden con esa clasificación o constituir un subconjunto de estas. Cuando se realiza una evaluación de las actualizaciones del sistema operativo pendientes para la máquina Linux, Update Management utiliza los archivos [Open Vulnerability and Assessment Language ](https://oval.mitre.org/) (OVAL) proporcionados por el proveedor de distribución de Linux para la clasificación.

La categorización se realiza para las actualizaciones de Linux en función de las clasificaciones **Seguridad** u **Otras**, según los archivos OVAL, y estas clasificaciones incluyen las actualizaciones que tratan problemas de seguridad o vulnerabilidades. Sin embargo, cuando se ejecuta la programación de actualización, lo hace en la máquina Linux mediante el administrador de paquetes adecuado, como YUM, APT o ZYPPER, para instalarlas. El administrador de paquetes para la distribución de Linux puede tener un mecanismo diferente para clasificar las actualizaciones, donde los resultados pueden diferir de los que se obtienen de los archivos OVAL mediante Update Management. Para comprobar manualmente la máquina y saber qué actualizaciones son de seguridad relevantes para el administrador de paquetes, consulte [Solución de problemas de implementación de actualizaciones de Linux](../troubleshoot/update-management.md#updates-linux-installed-different).

## <a name="integrate-update-management-with-configuration-manager"></a>Integración de Update Management con Configuration Manager.

Los clientes que han invertido en Microsoft Endpoint Configuration Manager para administrar equipos, servidores y dispositivos móviles también dependen de la fuerza y la madurez de Configuration Manager para que les ayude a administrar las actualizaciones de software. Para aprender a integrar Update Management con Configuration Manager, consulte este artículo sobre la [integración de Update Management con Endpoint Configuration Manager de Windows](mecmintegration.md).

## <a name="third-party-updates-on-windows"></a>Actualizaciones de terceros en Windows

Update Management se basa en el repositorio de actualización configurado localmente para actualizar los sistemas Windows compatibles, ya sea WSUS o Windows Update. Herramientas como [System Center Updates Publisher](/configmgr/sum/tools/updates-publisher) le permiten importar y publicar actualizaciones personalizadas con WSUS. Este escenario permite que Update Management actualice las máquinas que usan Configuration Manager como repositorio de actualizaciones con software de terceros. Para obtener información sobre cómo configurar Updates Publisher, consulte [Instalar Updates Publisher](/configmgr/sum/tools/install-updates-publisher).

## <a name="enable-update-management"></a>Habilitación de la Administración de actualizaciones

Estas son las formas en que puede habilitar Update Management y seleccionar las máquinas que se van a administrar:

- Use una [plantilla de Azure Resource Manager](enable-from-template.md) para implementar Update Management en una cuenta de Automation nueva o existente, y el área de trabajo de Log Analytics de Azure Monitor en la suscripción. No configura el ámbito de las máquinas que deben administrarse; esto se realiza como un paso independiente después de usar la plantilla.

- Desde la [cuenta de Automation](enable-from-automation-account.md) para una o varias máquinas de Azure o que no sean de Azure, incluidos los servidores habilitados para Arc.

- Use el método del [runbook](enable-from-runbook.md) **Enable-AutomationSolution**.

- Para una [VM de Azure concreta](enable-from-vm.md), desde la página **Máquinas virtuales** en Azure Portal. Este escenario está disponible para las VM Linux y Windows.

- Para [varias máquinas virtuales de Azure](enable-from-portal.md), selecciónelas en la página **Máquinas virtuales** de Azure Portal.

> [!NOTE]
> Update Management requiere vincular un área de trabajo de Log Analytics a la cuenta de Automation. Para ver una lista definitiva de regiones admitidas, consulte [Asignaciones de áreas de trabajo](../how-to/region-mappings.md). Las asignaciones de regiones no afectan a la capacidad de administrar VM en una región independiente de la cuenta de Automation.

## <a name="next-steps"></a>Pasos siguientes

* Para más información sobre cómo trabajar con Update Management, consulte [Administración de actualizaciones para las máquinas virtuales](manage-updates-for-vm.md).

* Revise las preguntas más frecuentes sobre Update Management en [Preguntas más frecuentes sobre Azure Automation](../automation-faq.md).