---
title: Compatibilidad con la evaluación de servidores VMware en Azure Migrate
description: Obtenga información sobre la compatibilidad de la detección y evaluación de Azure Migrate con los servidores de un entorno de VMware.
author: vineetvikram
ms.author: vivikram
ms.manager: abhemraj
ms.topic: conceptual
ms.date: 03/17/2021
ms.openlocfilehash: f4f534aeb915fb79bbc52f16615edd549aefe78c
ms.sourcegitcommit: 2cb7772f60599e065fff13fdecd795cce6500630
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108804239"
---
# <a name="support-matrix-for-vmware-assessment"></a>Matriz de compatibilidad para la evaluación de VMware 

En este artículo se resumen los requisitos previos y los requisitos de soporte técnico para usar la herramienta [Azure Migrate: Discovery and assessment](migrate-services-overview.md#azure-migrate-discovery-and-assessment-tool) para detectar y evaluar servidores en un entorno de VMware para la migración a Azure.

Para evaluar los servidores, primero cree un proyecto de Azure Migrate. La herramienta Azure Migrate: Discovery and assessment se agrega automáticamente al proyecto. A continuación, implemente el dispositivo de Azure Migrate. El dispositivo detecta los servidores locales de forma continuada y envía los metadatos de configuración y rendimiento a Azure. Una vez finalizada la detección, recopile los servidores detectados en grupos y ejecute evaluaciones por grupo.

A medida que planee la migración de los servidores de VMware a Azure, revise la [matriz de compatibilidad de migración](migrate-support-matrix-vmware-migration.md).

## <a name="limitations"></a>Limitaciones

Requisito | Detalles
--- | ---
**Límites del proyecto** | Puede crear varios proyectos de Azure Migrate en una suscripción a Azure.<br /><br /> Puede detectar y evaluar hasta 50 000 servidores de un entorno de VMware en un único [proyecto](migrate-support-matrix.md#project). Un proyecto puede incluir servidores físicos y servidores de un entorno de Hyper-V, hasta sus respectivos límites de evaluación.
**Detección** | El dispositivo de Azure Migrate puede detectar hasta 10 000 servidores en un servidor que ejecuta vCenter Server.
**Valoración** | Puede agregar hasta 35 000 servidores en un solo grupo.<br /><br /> Puede evaluar hasta 35 000 máquinas virtuales en una única evaluación.

Más información sobre las [valoraciones](concepts-assessment-calculation.md).

## <a name="vmware-requirements"></a>Requisitos de VMware

VMware | Detalles
--- | ---
**vCenter Server** | Los servidores que quiera detectar y evaluar deben estar administrados por vCenter Server, versión 7.0, 6.7, 6.5, 6.0 o 5.5.<br /><br /> Actualmente no se admite la detección de servidores proporcionando detalles del host ESXi en el dispositivo.
**Permisos** | La herramienta Azure Migrate: Discovery and assessment requiere una cuenta de solo lectura de vCenter Server.<br /><br /> Si desea usar la herramienta para el inventario de software y el análisis de dependencias sin agente, la cuenta debe tener privilegios para las operaciones de invitado en máquinas virtuales de VMware.

## <a name="server-requirements"></a>Requisitos del servidor

VMware | Detalles
--- | ---
**Sistemas operativos** | Todos los sistemas operativos Windows y Linux se pueden evaluar para la migración.
**Storage** | Se admiten los discos conectados a controladores basados en SCSI, IDE y SATA.

## <a name="azure-migrate-appliance-requirements"></a>Requisitos del dispositivo de Azure Migrate

Azure Migrate usa el [dispositivo de Azure Migrate](migrate-appliance.md) para la detección y la evaluación. Puede implementar el dispositivo como un servidor en un entorno de VMware mediante una plantilla Open Virtualization Appliance (OVA) de VMware que se importa a vCenter Server, o bien mediante un [script de PowerShell](deploy-appliance-script.md). Obtenga más información sobre los [requisitos del dispositivo para VMware](migrate-appliance.md#appliance---vmware).

A continuación, se detallan más requisitos para el dispositivo:

- En Azure Government, debe implementar el dispositivo mediante un [script](deploy-appliance-script-government.md).
- El dispositivo debe poder acceder a direcciones URL específicas en [nubes públicas](migrate-appliance.md#public-cloud-urls) y [nubes gubernamentales](migrate-appliance.md#government-cloud-urls).

## <a name="port-access-requirements"></a>Requisitos de acceso a puertos

Dispositivo | Conexión
--- | ---
**Dispositivo de Azure Migrate** | Conexiones entrantes en el puerto TCP 3389 para permitir las conexiones del Escritorio remoto al dispositivo.<br /><br /> Conexiones entrantes en el puerto 44368 para tener acceso de forma remota a la aplicación de administración del dispositivo mediante la dirección URL `https://<appliance-ip-or-name>:44368`. <br /><br />Conexiones salientes en el puerto 443 (HTTPS) para enviar metadatos de detección y rendimiento a Azure Migrate.
**vCenter Server** | Conexiones entrantes en el puerto TCP 443 para permitir que el dispositivo recopile los metadatos de configuración y rendimiento de las evaluaciones. <br /><br /> De forma predeterminada, el dispositivo se conecta a vCenter en el puerto 443. Si vCenter Server escucha en un puerto diferente, puede modificar el puerto al configurar la detección.
**Hosts de ESXi** | Para una [detección del inventario de software](how-to-discover-applications.md) o un [análisis de dependencias sin agente](concepts-dependency-visualization.md#agentless-analysis), el dispositivo se conecta a los hosts ESXi en el puerto TCP 443 para detectar el inventario de software y las dependencias en los servidores.

## <a name="application-discovery-requirements"></a>Problemas de detección de aplicaciones

Además de detectar servidores, Azure Migrate: Discovery and assessment puede completar un inventario del software que se ejecuta en ellos. La detección de aplicaciones le permite identificar y planificar una ruta de migración adaptada para sus cargas de trabajo locales.

Soporte técnico | Detalles
--- | ---
**Servidores admitidos** | Actualmente solo se admite en servidores de su entorno de VMware. Puede completar la detección de aplicaciones en un máximo de 10 000 servidores desde cada dispositivo de Azure Migrate.
**Sistemas operativos** | Se admiten los servidores que ejecutan todas las versiones de Windows y Linux.
**Requisitos de máquina virtual** | Para la detección del inventario de software, las herramientas de VMware deben estar instaladas y en ejecución en los servidores. <br /><br /> La versión de las herramientas de VMware debe ser la versión 10.2.1 o posterior.<br /><br /> Las instancias de Windows Server deben tener instalada la versión 2.0 de PowerShell o una versión posterior.
**Detección** | La detección de aplicaciones en servidores se realiza desde vCenter Server con las herramientas de VMware instaladas en los servidores. El dispositivo recopila la información sobre el inventario de software del servidor que ejecuta vCenter Server mediante las API de vSphere. La detección de aplicaciones se realiza sin agente. No hay ningún agente instalado en el servidor y el dispositivo no se conecta directamente a los servidores. WMI y SSH deben estar habilitados y disponibles en los servidores de Windows y Linux, respectivamente.
**Cuenta de usuario de vCenter Server** | Para interactuar con los servidores para la detección de aplicaciones, la cuenta de solo lectura de vCenter Server que se usa para la evaluación debe tener privilegios para las operaciones de invitado en las máquinas virtuales de VMware.
**Acceso de servidor** | Puede agregar varias credenciales de dominio y no de dominio (Windows o Linux) en el administrador de configuración del dispositivo para la detección de aplicaciones.<br /><br /> Debe tener una cuenta de usuario invitado para las instancias de Windows Server y una cuenta de usuario estándar (sin acceso `sudo`) para todos los servidores Linux.
**Acceso a puertos** | El dispositivo de Azure Migrate debe poder conectarse al puerto TCP 443 en los hosts ESXi que ejecutan las servidores en los que quiere realizar la detección de aplicaciones. El servidor que ejecuta vCenter Server devuelve una conexión de host ESXi para descargar el archivo que contiene los detalles del inventario de software.

## <a name="sql-server-instance-and-database-discovery-requirements"></a>Requisitos para la detección de bases de datos e instancias de SQL Server

La [detección de aplicaciones](how-to-discover-applications.md) identifica las instancias de SQL Server. Con esta información, el dispositivo intenta conectarse a las instancias de SQL Server correspondientes mediante la autenticación de Windows o las credenciales de autenticación de SQL Server proporcionadas en el administrador de configuración del dispositivo. Una vez conectado el dispositivo, este recopila datos de configuración y rendimiento de las instancias y de las bases de datos de SQL Server. Los datos de configuración de SQL Server se actualizan una vez cada 24 horas. Los datos de rendimiento se capturan cada 30 segundos.

Soporte técnico | Detalles
--- | ---
**Servidores admitidos** | Actualmente solo es compatible con los servidores que ejecutan SQL Server en su entorno de VMware. Puede detectar hasta 300 instancias de SQL Server o 6000 bases de datos SQL, lo que sea menor.
**Servidores Windows** | Se admiten Windows Server 2008 y versiones posteriores.
**Servidores Linux** | Actualmente no se admite.
**Mecanismo de autenticación** | Se admiten tanto la autenticación de Windows como la de SQL Server. Puede proporcionar las credenciales de ambos tipos de autenticación en el administrador de configuración del dispositivo.
**Acceso de SQL Server** | Azure Migrate requiere una cuenta de usuario de Windows que sea un miembro del rol del servidor sysadmin.
**Versiones de SQL Server** | Se admiten SQL Server 2008 y versiones posteriores.
**Ediciones de SQL Server** | Se admiten las ediciones Enterprise, Standard, Developer y Express.
**Configuración de SQL compatible** | Actualmente solo se admite la detección de instancias de SQL Server independientes y de las bases de datos correspondientes.<br /><br /> No se admite la identificación de clústeres de conmutación por error y grupos de disponibilidad Always On.
**Servicios SQL compatibles** | Solo se admite el motor de base de datos de SQL Server. <br /><br /> No se admite la detección de SQL Server Reporting Services (SSRS), SQL Server Integration Services (SSIS) y SQL Server Analysis Services (SSAS).

> [!NOTE]
> Azure Migrate cifra la comunicación entre el dispositivo de Azure Migrate y las instancias de SQL Server de origen cuando la propiedad [TrustServerCertificate](/dotnet/api/system.data.sqlclient.sqlconnectionstringbuilder.trustservercertificate) está establecida en `true`. La capa de transporte usa SSL para cifrar el canal y omitir la cadena de certificados para validar la confianza. El servidor del dispositivo se debe configurar para [confiar en la entidad de certificación raíz del certificado](/sql/database-engine/configure-windows/enable-encrypted-connections-to-the-database-engine).
>
> Si no se ha aprovisionado ningún certificado en el servidor cuando se inicia, SQL Server genera un certificado autofirmado que se utiliza para cifrar los paquetes de inicio de sesión. [Más información](/sql/database-engine/configure-windows/enable-encrypted-connections-to-the-database-engine).
>

## <a name="dependency-analysis-requirements-agentless"></a>Requisitos para el análisis de dependencias (sin agentes)

El [análisis de dependencias](concepts-dependency-visualization.md) le ayuda a identificar las dependencias entre los servidores locales que quiere evaluar y migrar a Azure. En la tabla siguiente se resumen los requisitos para configurar el análisis de dependencias sin agente:

Soporte técnico | Detalles
--- | --- 
**Servidores admitidos** | Actualmente solo se admite en servidores de su entorno de VMware.
**Servidores Windows** | Windows Server 2016<br /> Windows Server 2012 R2<br /> Windows Server 2012<br /> Windows Server 2008 R2 (64 bits)<br />Microsoft Windows Server 2008 (32 bits)
**Servidores Linux** | Red Hat Enterprise Linux 7, 6, 5<br /> Ubuntu Linux 16.04, 14.04<br /> Debian 8, 7<br /> Oracle Linux 7, 6<br /> CentOS 7, 6, 5<br /> SUSE Linux Enterprise Server 11 y posterior
**Requisitos del servidor** | Las herramientas de VMware (10.2.1 y versiones posteriores) deben estar instaladas y en ejecución en los servidores que quiere analizar.<br /><br /> Los servidores deben tener instalada la versión 2.0 de PowerShell o una versión posterior.
**Método de detección** |  La información sobre las dependencias entre los servidores se recopila mediante las herramientas de VMware instaladas en el servidor que ejecuta vCenter Server. El dispositivo recopila la información del servidor mediante las API de vSphere. No hay ningún agente instalado en el servidor y el dispositivo no se conecta directamente a los servidores. WMI y SSH deben estar habilitados y disponibles en servidores de Windows y Linux, respectivamente.
**Cuenta de vCenter** | La cuenta de solo lectura que usa Azure Migrate para la evaluación debe tener privilegios para las operaciones de invitado en las máquinas virtuales de VMware.
**Permisos de Windows Server** |  Una cuenta de usuario (local o de dominio) con permisos de administrador en los servidores.
**Cuenta de Linux** | Una cuenta de usuario raíz o una cuenta que tiene estos permisos en los archivos /bin/netstat y /bin/ls: <br />CAP_DAC_READ_SEARCH<br /> CAP_SYS_PTRACE<br /><br /> Establezca estas funcionalidades con los siguientes comandos:<br /><code>sudo setcap CAP_DAC_READ_SEARCH,CAP_SYS_PTRACE=ep /bin/ls<br /> sudo setcap CAP_DAC_READ_SEARCH,CAP_SYS_PTRACE=ep /bin/netstat</code>
**Acceso a puertos** | El dispositivo de Azure Migrate debe poder conectarse al puerto TCP 443 en los hosts ESXi que ejecutan los servidores que tienen dependencias que quiere detectar. El servidor que ejecuta vCenter Server devuelve una conexión de host ESXi para descargar el archivo que contiene los datos de las dependencias.

## <a name="dependency-analysis-requirements-agent-based"></a>Requisitos para el análisis de dependencia (con agentes)

El [análisis de dependencias](concepts-dependency-visualization.md) le ayuda a identificar las dependencias entre los servidores locales que quiere evaluar y migrar a Azure. En la tabla siguiente se resumen los requisitos para configurar el análisis de dependencias basadas en agentes:

Requisito | Detalles
--- | ---
**Antes de la implementación** | Debe tener un proyecto al que se le haya agregado la herramienta Azure Migrate: Discovery and assessment.<br /><br />Implemente la visualización de dependencias después de configurar un dispositivo de Azure Migrate para detectar los servidores locales.<br /><br />Aprenda a [crear un proyecto por primera vez](create-manage-projects.md).<br /> Aprenda a [agregar una herramienta de detección y evaluación a un proyecto existente](how-to-assess.md).<br /> Aprenda a configurar un dispositivo de Azure Migrate para la evaluación de [Hyper-V](how-to-set-up-appliance-hyper-v.md), [VMware](how-to-set-up-appliance-vmware.md) o de servidores físicos.
**Servidores admitidos** | Compatible con todos los servidores del entorno local.
**Log Analytics** | Azure Migrate utiliza la solución [Service Map](../azure-monitor/insights/service-map.md) de los [registros de Azure Monitor](../azure-monitor/log-query/log-query-overview.md) para la visualización de dependencias.<br /><br /> Puede asociar a un proyecto un área de trabajo de Log Analytics nueva o existente. El área de trabajo de un proyecto no se puede modificar una vez que se ha agregado el área de trabajo. <br /><br /> El área de trabajo debe estar en la misma suscripción que el proyecto.<br /><br /> El área de trabajo debe encontrarse en las regiones Este de EE. UU., Sudeste Asiático u Oeste de Europa. Las áreas de trabajo de otras regiones no se pueden asociar a un proyecto.<br /><br /> El área de trabajo debe estar en una [región en la que se admita Service Map](../azure-monitor/insights/vminsights-configure-workspace.md#supported-regions).<br /><br /> En Log Analytics, el área de trabajo asociada a Azure Migrate está etiquetada con la clave y el nombre del proyecto.
**Agentes necesarios** | En cada servidor que quiera analizar, instale los siguientes agentes:<br />- [Microsoft Monitoring Agent (MMA)](../azure-monitor/agents/agent-windows.md)<br />- [Dependency Agent](../azure-monitor/agents/agents-overview.md#dependency-agent)<br /><br /> Si los servidores locales no están conectados a Internet, descargue e instale la puerta de enlace de Log Analytics en ellas.<br /><br /> Conozca más información sobre la instalación de [Dependency Agent](how-to-create-group-machine-dependencies.md#install-the-dependency-agent) y [MMA](how-to-create-group-machine-dependencies.md#install-the-mma).
**Área de trabajo de Log Analytics** | El área de trabajo debe estar en la misma suscripción que el proyecto.<br /><br /> Azure Migrate admite áreas de trabajo que se encuentran en las regiones Este de EE. UU., Sudeste Asiático y Oeste de Europa.<br /><br />  El área de trabajo debe estar en una región en la que [se admita Service Map](../azure-monitor/insights/vminsights-configure-workspace.md#supported-regions).<br /><br /> El área de trabajo de un proyecto no se puede modificar una vez que se ha agregado el área de trabajo.
**Costee** | La solución Service Map no genera ningún gasto durante los primeros 180 días (a partir del día que asocie el área de trabajo de Log Analytics al proyecto).<br /><br /> Transcurridos los 180 días, se aplicarán las tarifas normales de Log Analytics.<br /><br /> Si se usa una solución que no sea Service Map en el área de trabajo de Log Analytics asociada generará los [gastos estándar](https://azure.microsoft.com/pricing/details/log-analytics/) de Log Analytics.<br /><br /> Cuando se elimina el proyecto, el área de trabajo no se elimina automáticamente. Tras la eliminación del proyecto, el uso de Service Map no será gratuito y cada nodo se facturará conforme al plan de tarifa del área de trabajo de Log Analytics.<br /><br />Si tiene proyectos creados antes de que Azure Migrate estuviera disponible con carácter general (28 de febrero de 2018), puede que haya incurrido en cargos de Service Map adicionales. Para garantizar el cargo solo después de 180 días, es recomendable que cree un proyecto. Las áreas de trabajo creadas antes de la disponibilidad general siguen siendo facturables
**Administración** | Al registrar agentes en el área de trabajo, use el identificador y la clave proporcionados por el proyecto.<br /><br /> Puede usar el área de trabajo de Log Analytics fuera de Azure Migrate.<br /><br /> Si elimina el proyecto asociado, el área de trabajo no se elimina automáticamente. [Elimínela manualmente](../azure-monitor/logs/manage-access.md).<br /><br /> No elimine el área de trabajo creada por Azure Migrate, a menos que elimine el proyecto. Si lo hace, la característica de visualización de dependencias no funcionará según lo previsto.
**Conectividad de Internet** | Si los servidores no están conectados a Internet, instale la puerta de enlace de Log Analytics en ellos.
**Azure Government** | El análisis de dependencias basado en agente no se admite.

## <a name="next-steps"></a>Pasos siguientes

- Revise los [procedimientos recomendados de evaluación](best-practices-assessment.md).
- Aprenda a [prepararse para una evaluación de VMware](./tutorial-discover-vmware.md).