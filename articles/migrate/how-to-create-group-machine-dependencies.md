---
title: Configuración del análisis de dependencias basado en agente en Azure Migrate
description: En este artículo se describe cómo configurar el análisis de dependencias basado en agente en Azure Migrate.
author: rashi-ms
ms.author: rajosh
ms.manager: abhemraj
ms.topic: how-to
ms.date: 11/25/2020
ms.openlocfilehash: 0d935cac68fc1b9e8d97a875c4b6e48078324fb8
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108143618"
---
# <a name="set-up-dependency-visualization"></a>Configuración de la visualización de dependencias

En este artículo se describe cómo configurar el análisis de dependencias basado en agente en Azure Migrate: Discovery and assessment. El [análisis de dependencias](concepts-dependency-visualization.md) le ayuda a identificar y comprender las dependencias en los servidores que quiere evaluar y migrar a Azure.

## <a name="before-you-start"></a>Antes de comenzar

- Revise los requisitos de implementación y soporte técnico para el análisis de dependencias basado en agente de:
    - [Servidores del entorno de VMware](migrate-support-matrix-vmware.md#dependency-analysis-requirements-agent-based)
    - [Servidores físicos](migrate-support-matrix-physical.md#agent-based-dependency-analysis-requirements)
    - [Servidores del entorno de Hyper-V](migrate-support-matrix-hyper-v.md#agent-based-dependency-analysis-requirements)
- Asegúrese de lo siguiente:
    - Tener un proyecto de Azure Migrate. Si no lo tiene, es el momento de [crearlo](./create-manage-projects.md).
    - Compruebe que ha [agregado](how-to-assess.md) la herramienta Azure Migrate: Discovery and assessment al proyecto.
    - Configure un [dispositivo de Azure Migrate](migrate-appliance.md) para detectar los servidores locales. El dispositivo detecta los servidores locales y envía los metadatos y los datos de rendimiento a Azure Migrate: Discovery and assessment. Configuración de un dispositivo para:
        - [Servidores del entorno de VMware](how-to-set-up-appliance-vmware.md)
        - [Servidores del entorno de Hyper-V](how-to-set-up-appliance-hyper-v.md)
        - [Servidores físicos](how-to-set-up-appliance-physical.md)
- Si quiere usar la visualización de dependencias, debe asociar un [área de trabajo de Log Analytics](../azure-monitor/logs/manage-access.md) con un proyecto de Azure Migrate:
    - Solo puede asociar un área de trabajo después de configurar el dispositivo de Azure Migrate y de detectar los servidores en el proyecto de Azure Migrate.
    - Asegúrese de que tiene un área de trabajo en la suscripción que contiene el proyecto de Azure Migrate.
    - El área de trabajo debe residir en las regiones Este de EE. UU., Sudeste Asiático u Oeste de Europa. Las áreas de trabajo de otras regiones no se pueden asociar a un proyecto.
    - El área de trabajo debe estar en una región en la que [se admita Service Map](../azure-monitor/vm/vminsights-configure-workspace.md#supported-regions).
    - Puede asociar un área de trabajo de Log Analytics nueva o existente con un proyecto de Azure Migrate.
    - La primera vez que configure la visualización de dependencias para una servidor, deberá asociar el área de trabajo. El área de trabajo de un proyecto de Azure Migrate no se puede modificar una vez que se ha agregado.
    - En Log Analytics, el área de trabajo asociada con Azure Migrate se etiqueta con la clave del proyecto de migración y el nombre del proyecto.

## <a name="associate-a-workspace"></a>Asociación de un área de trabajo

1. Una vez que haya detectado los servidores que se van a evaluar, en **Servidores** > **Azure Migrate: Discovery and assessment**, haga clic en **Información general**.  
2. En **Azure Migrate: Discovery and assessment**, haga clic en **Essentials**.
3. En **Área de trabajo de OMS**, haga clic en **Requiere configuración**.

     ![Configuración de un área de trabajo de Log Analytics](./media/how-to-create-group-machine-dependencies/oms-workspace-select.png)   

4. En **Configurar área de trabajo de OMS**, especifique si quiere crear un área de trabajo nueva o utilizar una existente.
    - Puede seleccionar un área de trabajo existente de todas las áreas de trabajo de la suscripción del proyecto.
    - Necesita acceso de lectura al área de trabajo para asociarla.
5. Si crea una nueva área de trabajo, seleccione una ubicación para ella.

    ![Agregar una nueva área de trabajo](./media/how-to-create-group-machine-dependencies/workspace.png)

> [!Note]
> [Obtenga información](../azure-monitor/logs/private-link-security.md) sobre cómo configurar el área de trabajo de OMS para la conectividad de un punto de conexión privado.  

## <a name="download-and-install-the-vm-agents"></a>Descarga e instalación de los agentes en máquinas virtuales

Instale los agentes en cada servidor que desee analizar.

> [!NOTE]
> En el caso de los servidores que supervisa System Center Operations Manager 2012 R2 o versiones posteriores, no hay necesidad de instalar el agente MMA. Service Map se integra con Operations Manager. [Siga](../azure-monitor/vm/service-map-scom.md#prerequisites) la guía de integración.

1. En **Azure Migrate: Discovery and assessment**, haga clic en **Servidores detectados**.
2. Para cada servidor que desee analizar con la visualización de dependencias, en la columna **Dependencias**, haga clic en **Requiere la instalación del agente**.
3. En la página **Dependencias**, descargue Microsoft Monitoring Agent y Dependency Agent para Windows o Linux.
4. En **Configure MMA agent** (Configurar agente MMA), copie el id. del área de trabajo y la clave. Los necesitará para instalar el agente MMA.

    ![Instalación de los agentes](./media/how-to-create-group-machine-dependencies/dependencies-install.png)


## <a name="install-the-mma"></a>Instalación de MMA

Instale MMA en cada servidor Windows o Linux que desee analizar.

### <a name="install-mma-on-a-windows-server"></a>Instalación de MMA en Windows Server

Para instalar al agente en Windows Server, siga estos pasos:

1. Haga doble clic en el agente descargado.
2. En la página **principal**, haga clic en **Siguiente**. En la página **Términos de licencia**, haga clic en **Acepto** para aceptar la licencia.
3. En **Carpeta de destino**, mantenga o modifique la carpeta de instalación predeterminada y después haga clic en **Siguiente**.
4. En **Opciones de instalación del agente**, seleccione **Azure Log Analytics** > **Siguiente**.
5. Haga clic en **Agregar** para agregar un área de trabajo de Log Analytics nueva. Pegue la clave y el identificador de área de trabajo que ha copiado desde el portal. Haga clic en **Next**.

El agente se puede instalar desde la línea de comandos o mediante un método automatizado, como Configuration Manager o [Intigua](https://www.intigua.com/intigua-for-azure-migration).
- [Obtenga más información](../azure-monitor/agents/log-analytics-agent.md#installation-options) sobre el uso de estos métodos para instalar el agente MMA.
- También se puede instalar el agente de MMA mediante este [script](https://github.com/brianbar-MSFT/Install-MMA).
- [Más información](../azure-monitor/agents/agents-overview.md#supported-operating-systems) sobre la compatibilidad de MMA con sistemas operativos Windows.

### <a name="install-mma-on-a-linux-server"></a>Instalación de MMA en un servidor Linux

Para instalar MMA en un servidor Linux:

1. Transfiera el paquete adecuado (x86 o x x64) al equipo Linux mediante scp o sftp.
2. Instale el paquete mediante el argumento --install.

    ```sudo sh ./omsagent-<version>.universal.x64.sh --install -w <workspace id> -s <workspace key>```

[Más información](../azure-monitor/agents/agents-overview.md#supported-operating-systems) sobre la lista de compatibilidad de MMA con sistemas operativos Linux. 

## <a name="install-the-dependency-agent"></a>Instalación del agente de dependencia

1. Para instalar al agente de dependencias en Windows Server, haga doble clic en el archivo de instalación y siga los pasos del asistente.
2. Para instalar el agente de dependencias en un servidor Linux, realice la instalación como usuario raíz mediante el siguiente comando:

    ```sh InstallDependencyAgent-Linux64.bin```

- [Más información](../azure-monitor/vm/vminsights-enable-hybrid.md#dependency-agent) acerca de cómo puede utilizar scripts para instalar el agente de dependencia.
- [Obtenga más información](../azure-monitor/vm/vminsights-enable-overview.md#supported-operating-systems) sobre los sistemas operativos que admite el agente de dependencia.


## <a name="create-a-group-using-dependency-visualization"></a>Creación de un grupo mediante la visualización de dependencias

Cree ahora un grupo para su evaluación. 


> [!NOTE]
> Los grupos para los que desea visualizar las dependencias no deben contener más de 10 servidores. Si tiene más de 10 servidores, divídalos en grupos más pequeños.

1. En **Azure Migrate: Discovery and assessment**, haga clic en **Servidores detectados**.
2. En la columna **Dependencias**, haga clic en **Ver dependencias** para cada servidor que quiera revisar.
3. En el mapa de dependencias, puede ver lo siguiente:
    - Conexiones TCP de entrada (clientes) y de salida (servidores) hacia y desde el servidor.
    - Los servidores dependientes que no tienen instalados los agentes de dependencias se agrupan en función de los números de puerto.
    - Los servidores dependientes con los agentes de dependencias instalados se muestran como cuadros independientes.
    - Los procesos que se ejecutan en el servidor. Expanda el cuadro de cada servidor para ver los procesos.
    - Propiedades del servidor (incluido el FQDN, el sistema operativo y la dirección MAC). Haga clic en el cuadro de cada servidor para ver los detalles.

4. Puede consultar las dependencias de distintas duraciones; para ello, haga clic en la duración correspondiente en la etiqueta de intervalo de tiempo.
    - De forma predeterminada, el intervalo es una hora. 
    - Puede modificar el intervalo de tiempo o especificar las fechas de inicio y finalización, y la duración.
    - El intervalo de tiempo puede ser de hasta una hora. Si necesita un intervalo más amplio, use Azure Monitor para consultar los datos dependientes durante un período más prolongado.

5. Después de identificar los servidores dependientes que quiere agrupar, use Control+clic para seleccionar varios servidores en el mapa y haga clic en **Agrupar máquinas**.
6. Especifique un nombre para el grupo.
7. Compruebe que Azure Migrate detecta los servidores dependientes.

    - Si Azure Migrate: Discovery and assessment no detecta un servidor dependiente, no puede agregarlo al grupo.
    - Para agregar un servidor, vuelva a ejecutar la detección y compruebe que el servidor se ha detectado.

8. Si desea crear una evaluación para este grupo, seleccione la casilla para crear una evaluación para el grupo.
8. Haga clic en **Aceptar** para guardar el grupo.

Después de crear el grupo, le recomendamos que instale los agentes en todos los servidores del grupo y, a continuación, visualice las dependencias de todo el grupo.

## <a name="query-dependency-data-in-azure-monitor"></a>Consulta de datos de dependencia en Azure Monitor

Los datos de dependencia que captura Service Map se pueden consultar en el área de trabajo de Log Analytics asociada con el proyecto de Azure Migrate. Log Analytics se usa para escribir y ejecutar consultas de registros en Azure Monitor.

- [Obtenga información sobre cómo](../azure-monitor/vm/service-map.md#log-analytics-records) buscar datos de Service Map en Log Analytics.
- [Obtenga información general](../azure-monitor/logs/get-started-queries.md) sobre la escritura de consultas de registro en [Log Analytics](../azure-monitor/logs/log-analytics-tutorial.md).

Ejecute una consulta para los datos de dependencia como se indica a continuación:

1. Después de instalar los agentes, vaya al portal y haga clic en **Introducción**.
2. En **Azure Migrate: Discovery and assessment**, haga clic en **Información general**. Haga clic en la flecha abajo para expandir **Información esencial**.
3. En el **Área de trabajo de OMS**, haga clic en el nombre del área de trabajo.
3. En la página del área de trabajo de Log Analytics > **General**, haga clic en **Registros**.
4. Escriba la consulta y haga clic en **Ejecutar**.

### <a name="sample-queries"></a>Consultas de ejemplo

A continuación se muestran algunas consultas de ejemplo que puede utilizar para extraer datos de dependencia.

- Puede modificar las consultas para extraer los puntos de datos preferidos.
- [Revise](../azure-monitor/vm/service-map.md#log-analytics-records) una lista completa de los registros de datos de dependencia.
- [Revise](../azure-monitor/vm/service-map.md#sample-log-searches) ejemplos consultas adicionales.

#### <a name="sample-review-inbound-connections"></a>Sample: revisión de conexiones entrantes

Revise las conexiones entrantes de un conjunto de servidores.

- Los registros de la tabla de métricas de conexión (VMConnection) no representan conexiones de red físicas individuales.
- Varias conexiones de red físicas se agrupan en una conexión lógica.
- [Obtenga más información](../azure-monitor/vm/service-map.md#connections) sobre cómo se agregan los datos de conexión de red física en VMConnection.

```
// the servers of interest
let ips=materialize(ServiceMapComputer_CL
| summarize ips=makeset(todynamic(Ipv4Addresses_s)) by MonitoredMachine=ResourceName_s
| mvexpand ips to typeof(string));
let StartDateTime = datetime(2019-03-25T00:00:00Z);
let EndDateTime = datetime(2019-03-30T01:00:00Z);
VMConnection
| where Direction == 'inbound'
| where TimeGenerated > StartDateTime and TimeGenerated  < EndDateTime
| join kind=inner (ips) on $left.DestinationIp == $right.ips
| summarize sum(LinksEstablished) by Computer, Direction, SourceIp, DestinationIp, DestinationPort
```

#### <a name="sample-summarize-sent-and-received-data"></a>Sample: resumen de los datos enviados y recibidos

En este ejemplo se resume el volumen de datos enviados y recibidos en las conexiones salientes entre un conjunto de servidores.

```
// the servers of interest
let ips=materialize(ServiceMapComputer_CL
| summarize ips=makeset(todynamic(Ipv4Addresses_s)) by MonitoredMachine=ResourceName_s
| mvexpand ips to typeof(string));
let StartDateTime = datetime(2019-03-25T00:00:00Z);
let EndDateTime = datetime(2019-03-30T01:00:00Z);
VMConnection
| where Direction == 'inbound'
| where TimeGenerated > StartDateTime and TimeGenerated  < EndDateTime
| join kind=inner (ips) on $left.DestinationIp == $right.ips
| summarize sum(BytesSent), sum(BytesReceived) by Computer, Direction, SourceIp, DestinationIp, DestinationPort
```

## <a name="next-steps"></a>Pasos siguientes

[Cree una evaluación](how-to-create-assessment.md) para un grupo.