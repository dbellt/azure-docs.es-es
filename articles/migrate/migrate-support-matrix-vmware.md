---
title: Compatibilidad para la evaluación de VMware en Azure Migrate
description: 'Aprenda cómo los servidores que se ejecutan en el entorno de VMware admiten la evaluación con Azure Migrate: Discovery and assessment.'
author: vineetvikram
ms.author: vivikram
ms.manager: abhemraj
ms.topic: conceptual
ms.date: 03/17/2021
ms.openlocfilehash: 8cf976c358d35b151e94ce42fc89cc53d361f699
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108131339"
---
# <a name="support-matrix-for-vmware-assessment"></a>Matriz de compatibilidad para la evaluación de VMware 

En este artículo se resumen los requisitos previos y los requisitos de soporte técnico al detectar y evaluar servidores que se ejecutan en el entorno de VMware para la migración a Azure con la herramienta [Azure Migrate: Discovery and assessment](migrate-services-overview.md#azure-migrate-discovery-and-assessment-tool). Para evaluar los servidores, creará un proyecto al que agregará la herramienta Azure Migrate: Discovery and assessment. Después de agregarla, implemente el dispositivo de Azure Migrate. El dispositivo detecta los servidores locales de forma continuada y envía los metadatos de configuración y rendimiento a Azure. Una vez completada la detección, recopile los servidores detectados en grupos y ejecute una evaluación de un grupo.

Si quiere migrar servidores de VMware a Azure, revise la [matriz de compatibilidad para la migración](migrate-support-matrix-vmware-migration.md).



## <a name="limitations"></a>Limitaciones

**Requisito** | **Detalles**
--- | ---
**Límites del proyecto** | Puede crear varios proyectos en una suscripción a Azure.<br/><br/> Puede detectar y evaluar hasta 50 000 servidores de VMware en un único [proyecto](migrate-support-matrix.md#project). Un proyecto también puede incluir servidores físicos y servidores del entorno de Hyper-V, hasta sus respectivos límites de evaluación.
**Detección** | El dispositivo de Azure Migrate puede detectar hasta 10 000 servidores en una instancia de vCenter Server.
**Valoración** | Puede agregar hasta 35 000 servidores en un solo grupo.<br/><br/> Puede evaluar hasta 35 000 máquinas virtuales en una única evaluación.

[Más información](concepts-assessment-calculation.md) sobre las evaluaciones.


## <a name="vmware-requirements"></a>Requisitos de VMware

**VMware** | **Detalles**
--- | ---
**vCenter Server** | Los servidores que quiera detectar y evaluar deben estar administrados por vCenter Server, versión 5.5, 6.0, 6.5, 6.7 o 7.0.<br/><br/> Actualmente no se admite la detección de servidores proporcionando detalles del host ESXi en el dispositivo.
**Permisos** | Azure Migrate: Discovery and assessment necesita una cuenta de solo lectura de vCenter Server para la detección y la evaluación.<br/><br/> Si quiere realizar la detección del inventario de software y el análisis de dependencias sin agente, la cuenta necesita tener habilitados los privilegios para **Virtual Machines** > **Guest Operations** (Operaciones de invitado).

## <a name="server-requirements"></a>Requisitos del servidor
**VMware** | **Detalles**
--- | ---
**Sistema operativo compatible** | Todos los sistemas operativos Windows y Linux se pueden evaluar para la migración.
**Storage** | Se admiten los discos conectados a controladores basados en SCSI, IDE y SATA.


## <a name="azure-migrate-appliance-requirements"></a>Requisitos del dispositivo de Azure Migrate

Azure Migrate usa el [dispositivo de Azure Migrate](migrate-appliance.md) para la detección y la evaluación. Puede implementar el dispositivo como un servidor de VMware mediante una plantilla OVA, importada a vCenter Server, o bien mediante un [script de PowerShell](deploy-appliance-script.md).

- Obtenga más información sobre los [requisitos del dispositivo](migrate-appliance.md#appliance---vmware) para VMware.
- En Azure Government, debe implementar el dispositivo [mediante el script](deploy-appliance-script-government.md).
- Revise las direcciones URL a las que el dispositivo necesita acceder en nubes [públicas](migrate-appliance.md#public-cloud-urls) y [gubernamentales](migrate-appliance.md#government-cloud-urls).


## <a name="port-access-requirements"></a>Requisitos de acceso a puertos

**Dispositivo** | **Connection**
--- | ---
**Dispositivo** | Conexiones entrantes en el puerto TCP 3389 para permitir las conexiones del Escritorio remoto al dispositivo.<br/><br/> Conexiones entrantes en el puerto 44368 para tener acceso de forma remota a la aplicación de administración del dispositivo mediante la dirección URL: ```https://<appliance-ip-or-name>:44368``` <br/><br/>Conexiones salientes en el puerto 443 (HTTPS) para enviar metadatos de detección y rendimiento a Azure Migrate.
**Servidor vCenter** | Conexiones entrantes en el puerto TCP 443 para permitir que el dispositivo recopile los metadatos de configuración y rendimiento de las evaluaciones. <br/><br/> De forma predeterminada, el dispositivo se conecta a vCenter en el puerto 443. Si el servidor vCenter escucha en un puerto diferente, puede modificar el puerto al configurar la detección.
**Hosts de ESXi** | Si quiere realizar la [detección del inventario de software](how-to-discover-applications.md) o un [análisis de dependencias sin agente](concepts-dependency-visualization.md#agentless-analysis), el dispositivo se conecta a los hosts ESXi en el puerto TCP 443 para realizar esta tarea.

## <a name="application-discovery-requirements"></a>Problemas de detección de aplicaciones

Además de detectar servidores, Azure Migrate: Discovery and assessment puede detectar el inventario de software que se ejecuta en ellos. La detección de aplicaciones le permite identificar y planificar una ruta de migración adaptada para sus cargas de trabajo locales.

**Soporte técnico** | **Detalles**
--- | ---
**Servidores admitidos** | Actualmente solo se admite en servidores de su entorno de VMware. Puede ejecutar la detección de aplicaciones en un máximo de 10 000 servidores, desde cada dispositivo de Azure Migrate.
**Sistemas operativos** | Se admiten los servidores que ejecutan todas las versiones de Windows y Linux.
**Requisitos de máquina virtual** | Para realizar la detección del inventario de software, las herramientas de VMware deben estar instaladas y en ejecución en los servidores. <br/><br/> La versión de las herramientas de VMware debe ser posterior a 10.2.0.<br/><br/> Las instancias de Windows Server deben tener instalada la versión 2.0 de PowerShell o una versión posterior.
**Detección** | La detección de aplicaciones en servidores se realiza desde vCenter Server, con las herramientas de VMware instaladas en los servidores. El dispositivo recopila la información sobre el inventario de software de vCenter Server mediante las API de vSphere. La detección de aplicaciones se realiza sin agente. No hay ningún agente instalado en el servidor y el dispositivo no se conecta directamente a los servidores. WMI y SSH deben estar habilitados y disponibles en servidores de Windows y Linux, respectivamente.
**Cuenta de usuario de vCenter Server** | La cuenta de vCenter Server de solo lectura que se usa para la evaluación, necesita tener habilitados los privilegios para **Virtual Machines** > **Guest Operations** (Operaciones de invitado), con el fin de interactuar con los servidores para la detección de aplicaciones.
**Acceso de servidor** | Puede agregar varias credenciales de dominio y no de dominio (Windows o Linux) en el administrador de configuración del dispositivo para la detección de aplicaciones.<br/><br/> Se necesita una cuenta de usuario invitado para las instancias de Windows Server y una cuenta de usuario normal (sin acceso sudo) para todas las máquinas virtuales Linux.
**Acceso a puertos** | El dispositivo de Azure Migrate debe poder conectarse al puerto TCP 443 en los hosts ESXi que ejecutan las servidores en los que quiere realizar la detección de aplicaciones. vCenter Server devuelve una conexión de host ESXi para descargar el archivo que contiene los detalles del inventario de software.

## <a name="requirements-for-discovery-of-sql-server-instances-and-databases"></a>Requisitos para la detección de instancias y bases de datos de SQL Server

La [detección de aplicaciones](how-to-discover-applications.md) identifica las instancias de SQL Server. Con esta información, el dispositivo intenta conectarse a las instancias de SQL Server correspondientes, a través de la autenticación de Windows o las credenciales de autenticación de SQL Server proporcionadas en el dispositivo. Una vez conectado, el dispositivo recopila datos de configuración y rendimiento de las instancias y de las bases de datos de SQL Server. Los datos de configuración de SQL Server se actualizan una vez cada 24 horas y los datos de rendimiento se capturan cada 30 segundos.

**Soporte técnico** | **Detalles**
--- | ---
**Servidores admitidos** | Actualmente solo se admite para instancias de SQL Server de su entorno de VMware. Puede detectar hasta 300 instancias de SQL Server o 6000 bases de datos SQL, el que sea menor.
**Servidores Windows** | Se admite Windows Server 2008 y versiones posteriores.
**Servidores Linux** | En la actualidad no se admite.
**Mecanismo de autenticación** | Se admiten tanto la autenticación de Windows como la de SQL Server. Puede proporcionar las credenciales de ambos tipos de autenticación en el administrador de configuración del dispositivo.
**Acceso de SQL Server** | Azure Migrate requiere una cuenta de usuario de Windows que sea miembro del rol de servidor sysadmin.
**Versiones de SQL Server** | Se admite SQL Server 2008 y versiones posteriores.
**Ediciones de SQL Server** | Se admiten las ediciones Enterprise, Standard, Developer y Express.
**Configuración de SQL compatible** | Actualmente se admite la detección de instancias de SQL Server independientes y las bases de datos correspondientes.<br/> No se admite la identificación de clústeres de conmutación por error y grupos de disponibilidad Always On.
**Servicios SQL compatibles** | Solo se admite el motor de base de datos de SQL Server. <br/> No se admite la detección de SQL Server Reporting Services (SSRS), SQL Server Integration Services (SSIS) y SQL Server Analysis Services (SSAS).

> [!Note]
> Azure Migrate cifrará la comunicación entre el dispositivo de Azure Migrate y las instancias de origen de SQL Server (con la propiedad Cifrar conexión establecida en TRUE). Estas conexiones se cifran con [**TrustServerCertificate**](/dotnet/api/system.data.sqlclient.sqlconnectionstringbuilder.trustservercertificate) (establecido en TRUE); la capa de transporte usará SSL para cifrar el canal y evitar la cadena de certificados para validar la confianza. El servidor del dispositivo se debe configurar para [**confiar en la entidad de certificación raíz del certificado**](/sql/database-engine/configure-windows/enable-encrypted-connections-to-the-database-engine).<br/>
Si no se ha aprovisionado ningún certificado en el servidor cuando se inicia, SQL Server genera un certificado autofirmado que se usa para cifrar los paquetes de inicio de sesión. [**Obtenga más información**](/sql/database-engine/configure-windows/enable-encrypted-connections-to-the-database-engine).

## <a name="dependency-analysis-requirements-agentless"></a>Requisitos para el análisis de dependencias (sin agentes)

El [análisis de dependencias](concepts-dependency-visualization.md) le ayuda a identificar las dependencias entre los servidores locales que quiere evaluar y migrar a Azure. En esta tabla se resumen los requisitos para configurar el análisis de dependencia sin agentes.

**Soporte técnico** | **Detalles**
--- | --- 
**Servidores admitidos** | Actualmente solo se admite en servidores de su entorno de VMware.
**Servidores Windows** | Windows Server 2016<br/> Windows Server 2012 R2<br/> Windows Server 2012<br/> Windows Server 2008 R2 (64 bits).<br/>Microsoft Windows Server 2008 (32 bits).
**Servidores Linux** | Red Hat Enterprise Linux 7, 6, 5<br/> Ubuntu Linux 14.04, 16.04<br/> Debian 7, 8<br/> Oracle Linux 6, 7<br/> CentOS 5, 6, 7<br/> SUSE Linux Enterprise Server 11 y posterior.
**Requisitos del servidor** | Las herramientas de VMware (versión posterior a 10.2.0) deben estar instaladas y en ejecución en los servidores que quiere analizar.<br/><br/> Los servidores deben tener instalada la versión 2.0 de PowerShell o una versión posterior.
**Método de detección** |  La información sobre las dependencias entre los servidores se recopila de vCenter Server mediante las herramientas de VMware instaladas en el servidor. El dispositivo recopila la información desde vCenter Server mediante las API de vSphere. No hay ningún agente instalado en el servidor y el dispositivo no se conecta directamente a los servidores. WMI y SSH deben estar habilitados y disponibles en servidores de Windows y Linux, respectivamente.
**Cuenta de vCenter** | La cuenta de solo lectura que Azure Migrate usa para la evaluación necesita privilegios habilitados para **Máquinas virtuales > Operaciones de invitado**.
**Permisos de Windows Server** |  Una cuenta de usuario (local o de dominio) con permisos administrativos en los servidores.
**Cuenta de Linux** | Cuenta de usuario raíz, o una cuenta con estos permisos en los archivos /bin/netstat y /bin/ls: CAP_DAC_READ_SEARCH y CAP_SYS_PTRACE.<br/><br/> Establezca estas funciones con los siguientes comandos: <br/><br/> sudo setcap CAP_DAC_READ_SEARCH,CAP_SYS_PTRACE=ep /bin/ls<br/><br/> sudo setcap CAP_DAC_READ_SEARCH,CAP_SYS_PTRACE=ep /bin/netstat
**Acceso a puertos** | El dispositivo de Azure Migrate debe poder conectarse al puerto TCP 443 en los hosts ESXi que ejecutan los servidores cuyas dependencias quiere detectar. vCenter Server devuelve una conexión de host ESXi para descargar el archivo que contiene los datos de las dependencias.


## <a name="dependency-analysis-requirements-agent-based"></a>Requisitos para el análisis de dependencia (con agentes)

El [análisis de dependencias](concepts-dependency-visualization.md) le ayuda a identificar las dependencias entre los servidores locales que quiere evaluar y migrar a Azure. En esta tabla se resumen los requisitos para configurar el análisis de dependencia con agentes.

**Requisito** | **Detalles** 
--- | --- 
**Antes de la implementación** | Debe tener un proyecto al que se le haya agregado la herramienta Azure Migrate: Discovery and assessment.<br/><br/>  La visualización de dependencias se implementa después de configurar un dispositivo de Azure Migrate para detectar los servidores locales.<br/><br/> [Aprenda](create-manage-projects.md) a crear un proyecto por primera vez.<br/> [Aprenda](how-to-assess.md) a agregar una herramienta de detección y evaluación a un proyecto existente.<br/> Aprenda a configurar un dispositivo de Azure Migrate para la evaluación de [Hyper-V](how-to-set-up-appliance-hyper-v.md), [VMware](how-to-set-up-appliance-vmware.md) o de servidores físicos.
**Servidores admitidos** | Compatible con todos los servidores del entorno local.
**Log Analytics** | Azure Migrate utiliza la solución [Service Map](../azure-monitor/vm/service-map.md) de los [registros de Azure Monitor](../azure-monitor/logs/log-query-overview.md) para la visualización de dependencias.<br/><br/> Puede asociar a un proyecto un área de trabajo de Log Analytics nueva o existente. El área de trabajo de un proyecto no se puede modificar una vez que se ha agregado. <br/><br/> El área de trabajo debe estar en la misma suscripción que el proyecto.<br/><br/> El área de trabajo debe residir en las regiones Este de EE. UU., Sudeste Asiático u Oeste de Europa. Las áreas de trabajo de otras regiones no se pueden asociar a un proyecto.<br/><br/> El área de trabajo debe estar en una región en la que [se admita Service Map](../azure-monitor/vm/vminsights-configure-workspace.md#supported-regions).<br/><br/> En Log Analytics, el área de trabajo asociada a Azure Migrate está etiquetada con la clave y el nombre del proyecto.
**Agentes necesarios** | En cada servidor que quiera analizar, instale los siguientes agentes:<br/><br/> [Microsoft Monitoring agent (MMA)](../azure-monitor/agents/agent-windows.md).<br/> [Dependency Agent](../azure-monitor/agents/agents-overview.md#dependency-agent).<br/><br/> Si los servidores locales no están conectados a Internet, tiene que descargar e instalar la puerta de enlace de Log Analytics en ellas.<br/><br/> Conozca más información sobre la instalación de [Dependency Agent](how-to-create-group-machine-dependencies.md#install-the-dependency-agent) y [MMA](how-to-create-group-machine-dependencies.md#install-the-mma).
**Área de trabajo de Log Analytics** | El área de trabajo debe estar en la misma suscripción que el proyecto.<br/><br/> Azure Migrate admite áreas de trabajo que residen en las regiones Este de EE. UU., Sudeste Asiático y Oeste de Europa.<br/><br/>  El área de trabajo debe estar en una región en la que [se admita Service Map](../azure-monitor/vm/vminsights-configure-workspace.md#supported-regions).<br/><br/> El área de trabajo de un proyecto no se puede modificar una vez que se ha agregado.
**Costee** | La solución Service Map no genera ningún gasto durante los primeros 180 días (a partir del día que asocie el área de trabajo de Log Analytics al proyecto).<br/><br/> Transcurridos los 180 días, se aplicarán las tarifas normales de Log Analytics.<br/><br/> Si se usa una solución que no sea Service Map en el área de trabajo de Log Analytics asociada generará los [gastos estándar](https://azure.microsoft.com/pricing/details/log-analytics/) de Log Analytics.<br/><br/> Cuando se elimina el proyecto, el área de trabajo no se elimina con él. Tras la eliminación del proyecto, el uso de Service Map no será gratuito y cada nodo se facturará conforme al nivel de pago del área de trabajo de Log Analytics.<br/><br/>Si tiene proyectos creados antes de que Azure Migrate estuviera disponible con carácter general (el 28 de febrero de 2018), puede que haya incurrido en cargos de Service Map adicionales. Para garantizar el pago solo después de 180 días, es recomendable que cree un nuevo proyecto, ya que las áreas de trabajo existentes antes de la disponibilidad general siguen siendo facturables.
**Administración** | Al registrar agentes en el área de trabajo, use el identificador y la clave proporcionados por el proyecto.<br/><br/> Puede usar el área de trabajo de Log Analytics fuera de Azure Migrate.<br/><br/> Si elimina el proyecto asociado, el área de trabajo no se elimina automáticamente. [Elimínela manualmente](../azure-monitor/logs/manage-access.md).<br/><br/> No elimine el área de trabajo creada por Azure Migrate, a menos que elimine el proyecto. Si lo hace, la característica de visualización de dependencias no funcionará según lo previsto.
**Conectividad de Internet** | Si los servidores no están conectados a Internet, debe instalar la puerta de enlace de Log Analytics en ellos.
**Azure Government** | El análisis de dependencias basado en agente no se admite.


## <a name="next-steps"></a>Pasos siguientes

- [Revise](best-practices-assessment.md) los procedimientos recomendados para crear evaluaciones.
- [Prepárese para la evaluación de VMware](./tutorial-discover-vmware.md).