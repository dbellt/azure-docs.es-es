---
title: Agentes de implementación automática de Azure Security Center | Microsoft Docs
description: En este artículo se describe cómo configurar el aprovisionamiento automático del agente de Log Analytics y de otros agentes y extensiones utilizados por Azure Security Center.
author: memildin
manager: rkarlin
ms.service: security-center
ms.topic: quickstart
ms.date: 03/04/2021
ms.author: memildin
ms.openlocfilehash: f7b09c47d94443edc81a66028fa687aa48e843e7
ms.sourcegitcommit: a038863c0a99dfda16133bcb08b172b6b4c86db8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/29/2021
ms.locfileid: "113002349"
---
# <a name="configure-auto-provisioning-for-agents-and-extensions-from-azure-security-center"></a>Configuración del aprovisionamiento automático de agentes y extensiones desde Azure Security Center

Azure Security Center recopila datos de los recursos con el agente o las extensiones correspondientes para ese recurso y el tipo de recopilación de datos que ha habilitado. Siga los procedimientos que se indican a continuación para asegurarse de que los recursos tengan los agentes y las extensiones necesarios utilizados por Security Center.

## <a name="prerequisites"></a>Prerrequisitos
Para empezar a trabajar con el Centro de seguridad, debe disponer de una suscripción a Microsoft Azure. Si no tiene ninguna suscripción, puede registrarse para obtener una [cuenta gratuita](https://azure.microsoft.com/pricing/free-trial/).

## <a name="availability"></a>Disponibilidad

| Aspecto                  | Detalles                                                                                                                                                                                                                      |
|-------------------------|:-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Estado de la versión:          | **Característica**: El aprovisionamiento automático está en fase de disponibilidad general (GA)<br>**Agente y extensiones**: El agente de Log Analytics agente para las máquinas virtuales de Azure está en fase de disponibilidad general, el agente de Microsoft Dependency está en versión preliminar y el complemento de Policy para Kubernetes está en fase de disponibilidad general                |
| Precios:                | Gratuito                                                                                                                                                                                                                         |
| Destinos admitidos: | ![Sí](./media/icons/yes-icon.png) Máquinas de Azure<br>![No](./media/icons/no-icon.png) Máquinas de Azure Arc<br>![No](./media/icons/no-icon.png) Nodos de Kubernetes<br>![No](./media/icons/no-icon.png) Virtual Machine Scale Sets |
| Nubes:                 | ![Sí](./media/icons/yes-icon.png) Nubes comerciales<br>![Sí](./media/icons/yes-icon.png) US Gov, Azure China                                                                                                      |
|                         |                                                                                                                                                                                                                              |

## <a name="how-does-security-center-collect-data"></a>¿Cómo recopila los datos Security Center?

Security Center recopila datos de las máquinas virtuales de Azure, los conjuntos de escalado de máquinas virtuales, los contenedores de IaaS y de las máquinas que no son de Azure (incluidas las máquinas locales) para supervisar las amenazas y vulnerabilidades de seguridad. 

La recopilación de datos es necesaria para proporcionar visibilidad sobre actualizaciones que faltan, valores de seguridad del sistema operativo mal configurados, estado de la protección de punto de conexión y protección contra amenazas y del mantenimiento. La recopilación de datos solo es necesaria para los recursos de proceso, como máquinas virtuales, conjuntos de escalado de máquinas virtuales, contenedores de IaaS y equipos que no son de Azure. 

Puede aprovechar las ventajas de Azure Security Center incluso si no aprovisiona agentes. Sin embargo, tendrá una seguridad limitada y no se admitirán las funcionalidades indicadas anteriormente.  

Los datos se recopilan mediante:

- El **agente de Log Analytics**, que lee distintas configuraciones relacionadas con la seguridad y registros de eventos de la máquina, y copia los datos en el área de trabajo para analizarlos. Estos son algunos ejemplos de dichos datos: tipo y versión del sistema operativo, registros del sistema operativo (registros de eventos de Windows), procesos en ejecución, nombre de la máquina, direcciones IP y usuario conectado.
- Las **extensiones de seguridad** como, por ejemplo, el [complemento de Azure Policy para Kubernetes](../governance/policy/concepts/policy-for-kubernetes.md), el cual también puede proporcionar datos a Security Center relacionados con tipos de recursos especializados.

> [!TIP]
> A medida que Security Center ha crecido, los tipos de recursos que se pueden supervisar también han crecido. El número de extensiones también ha crecido. El aprovisionamiento automático se ha ampliado para admitir tipos de recursos adicionales que aprovechan las funcionalidades de Azure Policy.

## <a name="why-use-auto-provisioning"></a>Motivos del uso del aprovisionamiento automático
Cualquiera de los agentes y extensiones que se describen en esta página *puede* instalarse manualmente (consulte [Instalación manual del agente de Log Analytics](#manual-agent)). Sin embargo, el **aprovisionamiento automático** reduce la sobrecarga de administración mediante la instalación de todos los agentes y extensiones necesarios en las máquinas existentes, y en las nuevas, para garantizar una cobertura de seguridad más rápida para todos los recursos admitidos. 

Se recomienda habilitar el aprovisionamiento automático, pero está deshabilitado de forma predeterminada.

## <a name="how-does-auto-provisioning-work"></a>¿Cómo funciona el aprovisionamiento automático?
La configuración del aprovisionamiento automático de Security Center tiene una opción de alternancia para cada tipo de extensión admitida. Al habilitar el aprovisionamiento automático de una extensión, asigne la directiva **Implementar si no existe** adecuada. Este tipo de directiva garantiza que la extensión esté aprovisionada en todos los recursos existentes y futuros de ese tipo.

> [!TIP]
> Obtenga más información sobre los efectos de Azure Policy, incluida la directiva "Implementar si no existe" en [Comprender los efectos de Azure Policy](../governance/policy/concepts/effects.md).


## <a name="enable-auto-provisioning-of-the-log-analytics-agent-and-extensions"></a>Habilitación del aprovisionamiento automático del agente y las extensiones de Log Analytics <a name="auto-provision-mma"></a>

Si el aprovisionamiento automático está activado para el agente de Log Analytics, Security Center implementará este agente en todas las máquinas virtuales de Azure compatibles y en las nuevas que se hayan creado. Consulte [Plataformas compatibles con Azure Security Center](security-center-os-coverage.md) para ver una lista de plataformas compatibles.

Para habilitar el aprovisionamiento automático del agente de Log Analytics:

1. En el menú de Security Center, seleccione **Precios y configuración**.
1. Seleccione la suscripción correspondiente.
1. En la página **Aprovisionamiento automático**, establezca el estado del agente de Log Analytics en **Activo**.

    :::image type="content" source="./media/security-center-enable-data-collection/enable-automatic-provisioning.png" alt-text="Habilitación del aprovisionamiento automático del agente de Log Analytics.":::

1. En el panel Opciones de configuración, defina el área de trabajo que se va a usar.

    :::image type="content" source="./media/security-center-enable-data-collection/log-analytics-agent-deploy-options.png" alt-text="Opciones de configuración para el aprovisionamiento automático de agentes de Log Analytics en máquinas virtuales." lightbox="./media/security-center-enable-data-collection/log-analytics-agent-deploy-options.png":::

    - **Conexión de máquinas virtuales de Azure a las áreas de trabajo predeterminadas que crea Security Center**: Security Center crea un nuevo grupo de recursos y un área de trabajo predeterminada en esa geolocalización y la conecta al agente. Si una suscripción contiene máquinas virtuales de varias geolocalizaciones, Security Center crea varias áreas de trabajo para garantizar el cumplimiento de los requisitos de privacidad de los datos.

        La convención de nomenclatura del área de trabajo y el grupo de recursos es: 
        - Área de trabajo: DefaultWorkspace-[subscription-ID]-[geo] 
        - Grupo de recursos: DefaultResourceGroup-[geo] 

        Security Center habilita automáticamente una solución de Security Center en el área de trabajo de acuerdo al plan de tarifa establecido para la suscripción. 

        > [!TIP]
        > Si tiene preguntas relacionadas con las áreas de trabajo predeterminadas, consulte:
        >
        > - [¿Se me facturan los registros de Azure Monitor en las áreas de trabajo que creada Security Center?](/azure/security-center/faq-data-collection-agents#am-i-billed-for-azure-monitor-logs-on-the-workspaces-created-by-security-center-)
        > - [¿Dónde se crea el área de trabajo Log Analytics predeterminada?](/azure/security-center/faq-data-collection-agents#where-is-the-default-log-analytics-workspace-created-)
        > - [¿Puedo eliminar las áreas de trabajo predeterminadas creadas por Security Center?](/azure/security-center/faq-data-collection-agents#can-i-delete-the-default-workspaces-created-by-security-center-)

    - **Conexión de máquinas virtuales de Azure a un área de trabajo diferente**: en la lista desplegable, seleccione el área de trabajo donde se van a almacenar los datos recopilados. La lista desplegable incluye todas las áreas de trabajo de todas las suscripciones. Puede usar esta opción para recopilar datos de máquinas virtuales que se ejecutan en distintas suscripciones y almacenarlos en el área de trabajo seleccionada.  

        Si ya tiene un área de trabajo de Log Analytics, es posible que desee utilizarla (necesita permisos de lectura y escritura sobre ella). Esta opción es útil si está usando un área de trabajo centralizada en la organización y desea usarla para la recopilación de datos de seguridad. Puede obtener más información en [Administración del acceso a los datos de registro y las áreas de trabajo en Azure Monitor](../azure-monitor/logs/manage-access.md).

        Si el área de trabajo seleccionada ya tiene una solución "Security" o "SecurityCenterFree" habilitada, los precios se establecerán automáticamente. En caso contrario, instale una solución de Security Center en el área de trabajo:

        1. En el menú de Security Center, abra **Precios y configuración**.
        1. Seleccione el área de trabajo a la que se conectarán los agentes.
        1. Seleccione **Azure Defender está activado** o **Azure Defender está desactivado**.

1. En la opción **Eventos de seguridad de Windows**, seleccione la cantidad de datos de eventos sin procesar que se van a almacenar:
    - **Ninguno**: deshabilita el almacenamiento de eventos de seguridad. Esta es la configuración predeterminada.
    - **Mínima**: un pequeño conjunto de eventos para cuando quiere reducir el volumen de eventos.
    - **Normal**: un conjunto de eventos que satisface a la mayoría de los clientes y les proporciona un registro de auditoría completo.
    - **Todos los eventos**: para clientes que quieren asegurarse de que se almacenan todos los eventos.

    > [!TIP]
    > Para establecer estas opciones en el nivel del área de trabajo, consulte [Configuración de las opciones de eventos de seguridad en el nivel de área de trabajo](#setting-the-security-event-option-at-the-workspace-level).
    > 
    > Para más información acerca de estas opciones, consulte [Opciones de eventos de seguridad de Windows para el agente de Log Analytics](#data-collection-tier).

1. Seleccione **Aplicar** en el panel de configuración.

1. Para habilitar el aprovisionamiento automático de una extensión distinta al agente de Log Analytics: 

    1. Si va a habilitar el aprovisionamiento automático para Microsoft Dependency Agent, asegúrese de que el agente de Log Analytics está establecido en implementación automática.
    1. Cambie el estado a **Activo** para la extensión que corresponda.

        :::image type="content" source="./media/security-center-enable-data-collection/toggle-kubernetes-add-on.png" alt-text="Opción para habilitar o deshabilitar el aprovisionamiento automático para el complemento de directivas de K8s.":::

    1. Seleccione **Guardar**. Se asigna la directiva de Azure y se crea una tarea de corrección.

        |Extensión  |Directiva  |
        |---------|---------|
        |Complemento de Policy para Kubernetes|[Implementación del complemento de Azure Policy en los clústeres de Azure Kubernetes Service](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2fa8eff44f-8c92-45c3-a3fb-9880802d67a7)|
        |Microsoft Dependency Agent (versión preliminar) (Máquinas virtuales Windows)|[Implementación de Dependency Agent para máquinas virtuales Windows](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2f1c210e94-a481-4beb-95fa-1571b434fb04)         |
        |Microsoft Dependency Agent (versión preliminar) (Máquinas virtuales Linux)|[Implementación de Dependency Agent para máquinas virtuales Linux](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2f4da21710-ce6f-4e06-8cdb-5cc4c93ffbee)|
        |||

1. Seleccione **Guardar**. Si es necesario aprovisionar un área de trabajo, la instalación del agente puede tardar hasta 25 minutos.

1. Se le preguntará si quiere volver a configurar las máquinas virtuales supervisadas que estaban anteriormente conectadas a un área de trabajo predeterminada:

    :::image type="content" source="./media/security-center-enable-data-collection/reconfigure-monitored-vm.png" alt-text="Revisión de las opciones para volver a configurar máquinas virtuales supervisadas.":::

    - **No**: la nueva configuración del área de trabajo solo se aplica a aquellas máquinas virtuales recién detectadas que no tienen instalado el agente de Log Analytics.
    - **Sí**: la nueva configuración del área de trabajo se aplicará a todas las máquinas virtuales y aquellas máquinas virtuales conectadas actualmente a un área de trabajo de Security Center se volverán a conectar a la nueva área de trabajo de destino.

   > [!NOTE]
   > Si selecciona **Sí**, no elimine las áreas de trabajo creadas por Security Center hasta que todas las máquinas virtuales se hayan vuelto a conectar a la nueva área de trabajo de destino. Esta operación no se lleva a cabo si se elimina un área de trabajo demasiado pronto.


## <a name="windows-security-event-options-for-the-log-analytics-agent"></a>Opciones de eventos de seguridad de Windows para el agente de Log Analytics <a name="data-collection-tier"></a> 

La selección de un nivel de recopilación de datos en Azure Security Center solo afecta al *almacenamiento* de eventos de seguridad en el área de trabajo de Log Analytics. El agente de Log Analytics sigue recopilando y analizando los eventos de seguridad necesarios para la protección contra amenazas de Security Center, con independencia del nivel de eventos de seguridad que elija almacenar en el área de trabajo. Si elige almacenar los eventos de seguridad, permitirá la investigación, búsqueda y auditoría de esos eventos en el área de trabajo.

### <a name="requirements"></a>Requisitos 
Azure Defender es necesario para almacenar los datos de eventos de seguridad de Windows. [Más información sobre Azure Defender](azure-defender.md).

El almacenamiento de datos en Log Analytics podría incurrir en cargos adicionales por almacenamiento de datos. Consulte la [página de precios](https://azure.microsoft.com/pricing/details/security-center/)para obtener más información.

### <a name="information-for-azure-sentinel-users"></a>Información para usuarios de Azure Sentinel 
Usuarios de Azure Sentinel: tenga en cuenta que la recopilación de eventos de seguridad dentro del contexto de un área de trabajo única se puede configurar desde el Azure Defender o desde Azure Sentinel, pero no con ambas herramientas. Si tiene previsto agregar Azure Sentinel a un área de trabajo que ya recibe alertas de Azure Security Center y está configurada para recopilar eventos de seguridad, tiene dos opciones:
- Deje la recopilación de eventos de seguridad en Azure Security Center tal y como está. Podrá consultar y analizar estos eventos tanto en Azure Sentinel como en Azure Defender. Sin embargo, no podrá supervisar el estado de conectividad del conector ni cambiar su configuración en Azure Sentinel. Si esta acción le parece importante, considere la segunda opción.
- Deshabilite la recopilación de eventos de seguridad en Azure Security Center (estableciendo **Eventos de seguridad de Windows** en **Ninguno** en la configuración del agente de Log Analytics). Después, agregue el conector de eventos de seguridad en Azure Sentinel. Al igual que con la primera opción, podrá consultar y analizar eventos en Azure Sentinel y Azure Defender (o ASC), pero ahora podrá supervisar el estado de conectividad del conector o cambiar su configuración únicamente en Azure Sentinel.

### <a name="what-event-types-are-stored-for-common-and-minimal"></a>¿Qué tipos de evento se almacenan para los niveles "Normal" y "Mínima"?
Estos conjuntos se han diseñado para abordar escenarios típicos. Asegúrese de evaluar cuál se ajusta a sus necesidades antes de implementarlo.

Para determinar los eventos de las opciones **Normal** y **Mínima**, hemos colaborado con los clientes y los estándares del sector para obtener información sobre la frecuencia sin filtrar de cada evento y su uso. En este proceso se han empleado las siguientes directrices:

- **Mínimo**: Asegúrese de que este conjunto abarque solamente los eventos que podrían indicar una brecha correcta y eventos importantes que tengan un volumen muy bajo. Por ejemplo, este conjunto contiene un inicio de sesión de usuario correcto y uno erróneo (identificadores de evento 4624 y 4625), pero no contiene el cierre de sesión, que es importante para la auditoría pero no lo es para la detección y tiene un volumen relativamente alto. La mayor parte del volumen de datos de este conjunto son los eventos de inicio de sesión y el evento de creación de proceso (Id. de evento 4688).
- **Común**: Proporcione una pista de auditoría de usuario completa en este conjunto. Por ejemplo, este conjunto contiene los inicios de sesión y los cierres de sesión de usuario (identificador de evento 4634). Se incluyen acciones de auditoría como cambios en los grupos de seguridad, operaciones de Kerberos en los controladores de dominio de clave y otros eventos que recomiendan las organizaciones del sector.

Los eventos que tienen un volumen muy bajo se han incluido en el conjunto común, ya que la motivación principal para elegirlo respecto de todos los eventos pasa por reducir el volumen, y no por filtrar eventos específicos.

A continuación se muestra un desglose completo de los identificadores de evento de seguridad y de AppLocker para cada conjunto:

| Capa de datos | Indicadores de eventos recopilados |
| --- | --- |
| mínimo | 1102,4624,4625,4657,4663,4688,4700,4702,4719,4720,4722,4723,4724,4727,4728,4732,4735,4737,4739,4740,4754,4755, |
| | 4756,4767,4799,4825,4946,4948,4956,5024,5033,8001,8002,8003,8004,8005,8006,8007,8222 |
| Comunes | 1,299,300,324,340,403,404,410,411,412,413,431,500,501,1100,1102,1107,1108,4608,4610,4611,4614,4622, |
| |  4624,4625,4634,4647,4648,4649,4657,4661,4662,4663,4665,4666,4667,4688,4670,4672,4673,4674,4675,4689,4697, |
| | 4700,4702,4704,4705,4716,4717,4718,4719,4720,4722,4723,4724,4725,4726,4727,4728,4729,4733,4732,4735,4737, |
| | 4738,4739,4740,4742,4744,4745,4746,4750,4751,4752,4754,4755,4756,4757,4760,4761,4762,4764,4767,4768,4771, |
| | 4774,4778,4779,4781,4793,4797,4798,4799,4800,4801,4802,4803,4825,4826,4870,4886,4887,4888,4893,4898,4902, |
| | 4904,4905,4907,4931,4932,4933,4946,4948,4956,4985,5024,5033,5059,5136,5137,5140,5145,5632,6144,6145,6272, |
| | 6273,6278,6416,6423,6424,8001,8002,8003,8004,8005,8006,8007,8222,26401,30004 |

> [!NOTE]
> - Si se usa un objeto de directiva de grupo (GPO), se recomienda habilitar el evento 4688 de creación de procesos de las directivas de auditoría y el campo *CommandLine* del evento 4688. Para más información acerca del evento 4688 de creación de procesos, consulte las [preguntas más frecuentes](/azure/security-center/faq-data-collection-agents#what-happens-when-data-collection-is-enabled-) acerca de Security Center. Para más información acerca de estas directivas de auditoría, consulte [Recomendaciones de la directiva de auditoría](/windows-server/identity/ad-ds/plan/security-best-practices/audit-policy-recommendations).
> -  Para habilitar la recopilación de datos para [Controles de aplicación adaptables](security-center-adaptive-application.md), Security Center configura una directiva de AppLocker local en el modo de auditoría que permite todas las aplicaciones. Con esto, AppLocker generará eventos que luego Security Center recopilará y aprovechará. Es importante tener en cuenta que esta directiva no se configurará en las máquinas en las que ya se haya configurado una directiva de AppLocker. 
> - Para recopilar Windows Filtering Platform [Event ID 5156](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventID=5156), tiene que habilitar [Auditar conexión de Plataforma de filtrado](/windows/security/threat-protection/auditing/audit-filtering-platform-connection) (Auditpol /set /subcategory:"Filtering Platform Connection" /Success:Enable)
>

### <a name="setting-the-security-event-option-at-the-workspace-level"></a>Configuración de la opción Evento de seguridad en el nivel del área de trabajo

Puede definir el nivel de datos de eventos de seguridad que se van a almacenar en el nivel del área de trabajo.

1. En el menú de Security Center de Azure Portal, seleccione **Precios y configuración**.
1. Seleccione el área de trabajo correspondiente. Los únicos eventos de recopilación de datos de un área de trabajo son los eventos de seguridad de Windows que se describen en esta página.

    :::image type="content" source="media/security-center-enable-data-collection/event-collection-workspace.png" alt-text="Configuración de los datos de eventos de seguridad que se van a almacenar en un área de trabajo.":::

1. Seleccione la cantidad de datos de eventos sin procesar que se van a almacenar y seleccione **Guardar**.

## <a name="manual-agent-provisioning"></a>Aprovisionamiento manual del agente <a name="manual-agent"></a>
 
Para instalar manualmente el agente de Log Analytics, haga lo siguiente:

1. Deshabilite el aprovisionamiento automático.

1. Opcionalmente, cree un área de trabajo.

1. Habilite Azure Defender en el área de trabajo en la que va a instalar el agente de Log Analytics:

    1. En el menú de Security Center, seleccione **Precios y configuración**.

    1. Establezca el área de trabajo en la que va a instalar el agente. Asegúrese de que el área de trabajo está en la misma suscripción que se usa en Security Center y que tiene permisos de lectura/escritura en el área de trabajo.

    1. Seleccione **Azure Defender está activado** y **Guardar**.

       >[!NOTE]
       >Si el área de trabajo ya tiene una solución **Security** o **SecurityCenterFree** habilitada, los precios se establecerán automáticamente. 

1. Para implementar agentes en las nuevas máquinas virtuales mediante una plantilla de Resource Manager, instale el agente de Log Analytics:

   - [Instalación del agente de Log Analytics para Windows](../virtual-machines/extensions/oms-windows.md)
   - [Instalación del agente de Log Analytics para Linux](../virtual-machines/extensions/oms-linux.md)

1. Para implementar agentes en las máquinas virtuales existentes, siga las instrucciones que se indican en [Recopilación de datos acerca de máquinas virtuales de Azure](../azure-monitor/vm/quick-collect-azurevm.md) (la sección **Recopilación de datos de eventos y rendimiento** es opcional).

1. Para usar PowerShell para implementar los agentes, siga las instrucciones de la documentación de máquinas virtuales:

    - [Para máquinas Windows](../virtual-machines/extensions/oms-windows.md?toc=%2fazure%2fazure-monitor%2ftoc.json#powershell-deployment)
    - [Para máquinas Linux](../virtual-machines/extensions/oms-linux.md?toc=%2fazure%2fazure-monitor%2ftoc.json#azure-cli-deployment)

> [!TIP]
> Para obtener instrucciones sobre cómo incorporar Security Center mediante PowerShell, consulte [Automatización de la incorporación de Azure Security Center mediante PowerShell](security-center-powershell-onboarding.md).


## <a name="automatic-provisioning-in-cases-of-a-pre-existing-agent-installation"></a>Aprovisionamiento automático en los casos de una instalación de agente ya existente <a name="preexisting"></a> 

Los siguientes casos de uso especifican cómo funciona el aprovisionamiento automático en los casos en los que ya hay un agente o una extensión instalados. 

- **El agente de Log Analytics está instalado en la máquina, pero no como una extensión (agente directo)** : si el agente de Log Analytics está instalado directamente en la máquina virtual (no como una extensión de Azure), Security Center instalará la extensión del agente de Log Analytics y puede que la actualice a la versión más reciente.
El agente instalado continuará informando a las áreas de trabajo ya configuradas y además informará al área de trabajo configurada en Security Center (el hospedaje múltiple es compatible en máquinas Windows).
Si el área de trabajo configurada es un área de trabajo de usuario (no el área de trabajo predeterminada de Security Center), debe instalar la solución "Security" o "SecurityCenterFree" para que Security Center empiece a procesar eventos de máquinas virtuales y equipos que notifican a esa área de trabajo.

    El hospedaje múltiple del agente todavía no es compatible con las máquinas Linux; por lo tanto, si se detecta una instalación de agente existente, no se producirá el aprovisionamiento automático y no se modificará la configuración de la máquina.

    Para las máquinas existentes en suscripciones incorporadas a Security Center antes del 17 de marzo de 2019, cuando se detecte un agente existente, no se instalará la extensión del agente de Log Analytics y la máquina no se modificará. Para estas máquinas, consulte la recomendación "Resolver incidencias de supervisión de estado del agente en las máquinas" con el fin de resolver las incidencias de instalación del agente en estas máquinas.
  
- **El agente de System Center Operations Manager está instalado en la máquina**: Security Center instalará la extensión del agente de Log Analytics en paralelo a la versión existente de Operations Manager. El agente de Operations Manager existente continuará enviando informes con normalidad al servidor de Operations Manager. El agente de Operations Manager y el agente de Log Analytics comparten bibliotecas en tiempo de ejecución, las cuales se actualizarán a la versión más reciente durante este proceso. Nota: si está instalada la versión 2012 del agente de Operations Manager, **no** habilite el aprovisionamiento automático.

- **Está presente una extensión de máquina virtual existente**:
    - Cuando se instala Monitoring Agent como una extensión, la configuración de extensión permite enviar informes a una sola área de trabajo. Security Center no invalida las conexiones existentes con áreas de trabajo de usuario. Security Center almacenará datos de seguridad de la máquina virtual en el área de trabajo que ya está conectada, siempre que se haya instalado en ella la solución "Security" o "SecurityCenterFree". Durante este proceso, Security Center puede actualizar la versión de extensión a la versión más reciente.
    - Para ver el área de trabajo, a la que la extensión existente está enviando datos, ejecute la prueba de [Validación de la conectividad con Azure Security Center](/archive/blogs/yuridiogenes/validating-connectivity-with-azure-security-center). También puede abrir las áreas de trabajo de Log Analytics, seleccionar un área de trabajo, seleccionar la máquina virtual y examinar la conexión del agente de Log Analytics.
    - Si tiene un entorno donde esté instalado el agente de Log Analytics en estaciones de trabajo de cliente y esté informando a un área de trabajo de Log Analytics existente, revise la lista de [sistemas operativos compatibles con Azure Security Center](security-center-os-coverage.md) para asegurarse de que el sistema operativo es compatible. Para más información, consulte [Clientes existentes de análisis de registros](./faq-azure-monitor-logs.yml).
 

## <a name="disable-auto-provisioning"></a>Deshabilitar aprovisionamiento automático <a name="offprovisioning"></a>

Si deshabilita el aprovisionamiento automático, los agentes no se aprovisionarán en nuevas máquinas virtuales.

Para desactivar el aprovisionamiento automático de un agente:

1. En el menú de Security Center del portal, seleccione **Precios y configuración**.
1. Seleccione la suscripción correspondiente.
1. Seleccione **Aprovisionamiento automático**.
1. Cambie el estado a **Desactivado** para el agente que corresponda.

    :::image type="content" source="./media/security-center-enable-data-collection/agent-toggles.png" alt-text="Botones de alternancia para deshabilitar el aprovisionamiento automático por tipo de agente.":::

1. Seleccione **Guardar**. Cuando el aprovisionamiento automático está deshabilitado, no se muestra la sección de configuración del área de trabajo predeterminada:

    :::image type="content" source="./media/security-center-enable-data-collection/empty-configuration-column.png" alt-text="Cuando el aprovisionamiento automático está deshabilitado, la celda de configuración está vacía.":::


> [!NOTE]
>  La deshabilitación del aprovisionamiento automático no quita el agente de Log Analytics de las máquinas virtuales de Azure en las que se aprovisionó el agente. Para más información acerca de cómo quitar la extensión de OMS, consulte [¿Cómo quito extensiones OMS instaladas por Security Center?](/azure/security-center/faq-data-collection-agents#how-do-i-remove-oms-extensions-installed-by-security-center-).
>


## <a name="troubleshooting"></a>Solución de problemas

-   Para identificar problemas de instalación del aprovisionamiento automático, consulte [Supervisión de problemas de Agent Health](security-center-troubleshooting-guide.md#mon-agent).
-  Para identificar los requisitos de red de agente de supervisión, consulte [Solución de problemas de los requisitos de red del agente de supervisión](security-center-troubleshooting-guide.md#mon-network-req).
-   Para identificar problemas de incorporación manual, consulte [Cómo solucionar problemas de incorporación de Operations Management Suite](https://support.microsoft.com/help/3126513/how-to-troubleshoot-operations-management-suite-onboarding-issues).



## <a name="next-steps"></a>Pasos siguientes
En esta página se explica cómo habilitar el aprovisionamiento automático para el agente de Log Analytics y otras extensiones de Security Center. También se describe cómo definir un área de trabajo de Log Analytics en la que almacenar los datos recopilados. Ambas operaciones son necesarias para habilitar la recopilación de datos. El almacenamiento de datos en Log Analytics, independientemente de si usa un área de trabajo nueva o existente, puede incurrir en más cargos por almacenamiento de datos. Para obtener información sobre los precios en la moneda de su elección y según su región, consulte [Precios de Security Center](https://azure.microsoft.com/pricing/details/security-center/).
