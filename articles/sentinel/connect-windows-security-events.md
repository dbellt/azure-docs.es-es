---
title: Conexión de datos de eventos de seguridad de Windows a Azure Sentinel (versión en pestañas) | Microsoft Docs
description: Aprenda a usar el conector de eventos de seguridad de Windows para transmitir todos los eventos de seguridad de sus sistemas Windows al área de trabajo de Azure Sentinel.
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.assetid: d51d2e09-a073-41c8-b396-91d60b057e6a
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 06/06/2021
ms.author: yelevin
ms.openlocfilehash: 364426e5b89915f51ec61e7c878e885045964554
ms.sourcegitcommit: 23040f695dd0785409ab964613fabca1645cef90
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112063457"
---
# <a name="connect-to-windows-servers-to-collect-security-events"></a>Conexión a servidores Windows para recopilar eventos de seguridad

[!INCLUDE [reference-to-feature-availability](includes/reference-to-feature-availability.md)]

El conector de eventos de seguridad de Windows permite transmitir eventos de seguridad desde cualquier servidor de Windows (físico o virtual, local o en cualquier nube) conectado al área de trabajo de Azure Sentinel. Esto le permite ver los eventos de seguridad de Windows en los paneles. De este modo, puede usarlos en la creación de alertas personalizadas y basarse en ellos para mejorar las investigaciones, lo que le proporciona más información sobre la red de su organización y amplía las funciones de las operaciones de seguridad. 

Ahora hay dos versiones de este conector: los **eventos de seguridad** son la versión heredada, que se basa en el agente de Log Analytics (también conocido como agente de OMS o MMA), y los **eventos de seguridad de Windows** son la nueva versión, que se encuentra actualmente en **versión preliminar** y se basa en el nuevo agente de Azure Monitor (AMA). En este documento se presenta información sobre ambos conectores. Puede elegir entre las pestañas siguientes para ver la información relevante para el conector seleccionado.

> [!NOTE]
> Para recopilar eventos de seguridad de cualquier sistema que no sea una máquina virtual de Azure, el sistema debe tener [**Azure Arc**](../azure-monitor/agents/azure-monitor-agent-install.md) instalado y habilitado *antes* de habilitar cualquiera de estos conectores.
>
> Esto incluye:
> - Servidores Windows instalados en máquinas físicas
> - Servidores Windows instalados en máquinas virtuales locales
> - Servidores Windows instalados en máquinas virtuales en nubes que no son de Azure

# <a name="log-analytics-agent-legacy"></a>[Agente de Log Analytics (heredado)](#tab/LAA)

Puede seleccionar los eventos que se transmitirán de entre los siguientes conjuntos: <a name="event-sets"></a>

- **Todos los eventos**: todos los eventos de seguridad de Windows y AppLocker.
- **Común**: conjunto estándar de eventos con fines de auditoría. En este conjunto se proporciona una pista de auditoría de usuario completa. Por ejemplo, contiene los eventos de inicio de sesión y cierre de sesión de usuario (identificadores de evento 4624, 4634). También hay acciones de auditoría como los cambios en los grupos de seguridad, las operaciones de Kerberos en el controlador de dominio de clave y otros tipos de eventos que siguen los procedimientos recomendados aceptados.

    El conjunto de eventos **común** puede contener algunos tipos de eventos que no son tan comunes.  Esto se debe a que el argumento principal del conjunto **común** es reducir el volumen de eventos a un nivel más fácil de administrar, a la vez que se mantiene una capacidad de registro de auditoría completa.

- **Mínimo**: conjunto reducido de eventos que pueden señalar posibles amenazas. Este conjunto no contiene un registro de auditoría completo. Abarca solamente los eventos que podrían indicar una brecha correcta, así como otros eventos importantes que tienen tasas de aparición muy bajas. Por ejemplo, contiene un inicio de sesión de usuario correcto y uno erróneo (identificadores de evento 4624 y 4625), pero no contiene información del cierre de sesión (4634), que es importante para la auditoría pero no lo es para la detección de brechas y tiene un volumen relativamente alto. La mayor parte del volumen de datos de este conjunto está conformado por los eventos de inicio de sesión y de creación de proceso (identificador de evento 4688).

- **Ninguno**: no se transmite ningún evento de seguridad o de AppLocker. (Esta configuración se usa para deshabilitar el conector).

> [!NOTE]
> La recopilación de eventos de seguridad dentro del contexto de un área de trabajo única se puede configurar desde Azure Security Center o desde Azure Sentinel, pero no con ambas herramientas. Si va a incorporar Azure Sentinel a un área de trabajo que ya recibe alertas de Azure Defender desde Azure Security Center y está configurado para recopilar eventos de seguridad, tiene dos opciones:
> - Deje la recopilación de eventos de seguridad en Azure Security Center tal y como está. Podrá consultar y analizar estos eventos tanto en Azure Sentinel como en Azure Defender. Sin embargo, no podrá supervisar el estado de conectividad del conector ni cambiar su configuración en Azure Sentinel. Si esta acción le parece importante, considere la segunda opción.
>
> - [Deshabilite la recopilación de eventos de seguridad](../security-center/security-center-enable-data-collection.md) en Azure Security Center y, a continuación, agregue el conector de eventos de seguridad en Azure Sentinel. Al igual que con la primera opción, podrá consultar y analizar eventos en Azure Sentinel y Azure Defender (o ASC), pero ahora podrá supervisar el estado de conectividad del conector o cambiar su configuración únicamente en Azure Sentinel.

# <a name="azure-monitor-agent-new"></a>[Agente de Azure Monitor (nuevo)](#tab/AMA)

> [!IMPORTANT]
>
> - Actualmente, el conector de datos de eventos de seguridad de Windows basado en el agente de Azure Monitor (AMA) se encuentra en **Versión preliminar**. Consulte [Términos de uso complementarios para las Versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) para conocer los términos legales adicionales que se aplican a las características de Azure que se encuentran en la versión beta, en versión preliminar o que todavía no se han publicado para que estén disponibles con carácter general.

El [agente de Azure Monitor](../azure-monitor/agents/azure-monitor-agent-overview.md) usa **reglas de recopilación de datos (DCR)** para definir los datos que se recopilan de cada agente. Las reglas de recopilación de datos permiten administrar la configuración de la recopilación a gran escala y, al mismo tiempo, habilitan configuraciones únicas con ámbito para subconjuntos de máquinas. Son independientes del área de trabajo e independientes de la máquina virtual, lo que significa que se pueden definir una vez y reutilizarlas en distintas máquinas y entornos. Consulte [Configuración de la recopilación de datos para el agente de Azure Monitor](../azure-monitor/agents/data-collection-rule-azure-monitor-agent.md).

Además de los conjuntos de eventos seleccionados previamente (**Todos los eventos**,**Mínimo** o **Común**) que podría optar por ingerir con el conector anterior, las **reglas de recopilación de datos** permiten crear filtros personalizados para elegir los eventos exactos que quiere ingerir. El agente de Azure Monitor usa estas reglas para filtrar los datos *en el origen* e ingerir solo los eventos que quiera, a la vez que se olvida del resto. Esto puede ahorrar mucho dinero en costos de ingesta de datos.

En este documento se muestra cómo crear reglas de recopilación de datos.

> [!NOTE]
> **Coexistencia con otros agentes**
> 
> El agente de Azure Monitor puede coexistir con los agentes existentes para que pueda seguir usando el conector heredado durante la evaluación o la migración. Esto es especialmente importante mientras el nuevo conector está en versión preliminar, debido a la compatibilidad limitada con las soluciones existentes. Sin embargo, debe tener cuidado al recopilar datos duplicados, ya que esto podría sesgar los resultados de las consultas y generar cargos adicionales por la ingesta y retención de datos.

---

## <a name="set-up-the-windows-security-events-connector"></a>Configuración del conector de eventos de seguridad de Windows

Para recopilar los eventos de seguridad de Windows en Azure Sentinel, haga lo siguiente:

# <a name="log-analytics-agent-legacy"></a>[Agente de Log Analytics (heredado)](#tab/LAA)

1. En el menú de navegación de Azure Sentinel, seleccione **Data connectors** (Conectores de datos). En la lista de conectores, haga clic en **Eventos de seguridad** y, a continuación, en el botón **Open connector page** (Abrir página del conector) en la parte inferior derecha. A continuación, siga las instrucciones en pantalla debajo de la pestaña **Instrucciones**, tal como se describe en el resto de esta sección.

1. Compruebe que tiene los permisos adecuados, tal como se describe en la sección **Requisitos previos** de la página del conector.

1. Descargue e instale el [agente de Log Analytics](../azure-monitor/agents/log-analytics-agent.md) (también conocido como Microsoft Monitoring Agent o MMA) en los equipos desde los que quiere transmitir eventos de seguridad a Azure Sentinel.

    Para las máquinas virtuales de Azure:
    
    1. Haga clic en **Install agent on Azure Windows Virtual Machine** (Instalar el agente en la máquina virtual de Windows de Azure) y, después, en el vínculo que aparece a continuación.
    1. Haga clic en el nombre de cada máquina virtual que quiera conectar en la lista que aparece a la derecha y, a continuación, haga clic en **Conectar**.

    Para las máquinas Windows que no son de Azure (físicas, virtuales locales o virtuales en otra nube):

    1. Haga clic en **Install agent on non-Azure Windows Machine** (Instalar el agente en una máquina de Windows que no es de Azure) y, después, en el vínculo que aparece a continuación.
    1. Haga clic en los vínculos de descarga correspondientes que aparecen a la derecha, debajo de **Equipos Windows**.
    1. Con el archivo ejecutable descargado, instale el agente en los sistemas Windows que quiera y configúrelo con el **Id. y las claves de área de trabajo** que aparecen debajo de los vínculos de descarga mencionados anteriormente.

    > [!NOTE]
    >
    > Para que los sistemas Windows sin la conectividad a Internet necesaria puedan seguir transmitiendo eventos a Azure Sentinel, use el vínculo de la esquina inferior derecha para descargar e instalar la **puerta de enlace de OMS** en una máquina independiente, que actuará como proxy.  Tendrá que instalar el agente de Log Analytics en todos los sistemas Windows cuyos eventos quiera recopilar.
    >
    > Para obtener más información sobre este escenario, consulte la [documentación de la **puerta de enlace de Log Analytics**](../azure-monitor/agents/gateway.md).

    Para obtener más información sobre las opciones de instalación adicionales y otros detalles, consulte la [documentación del **agente de Log Analytics**](../azure-monitor/agents/agent-windows.md).

1. Seleccione el conjunto de eventos (todos, común o mínimo) que quiere transmitir. Consulte las [listas de los identificadores de eventos incluidos](#event-id-reference) en los conjuntos de eventos mínimos y comunes.

1. Haga clic en **Update**(Actualizar).

1. Para usar el esquema correspondiente en Log Analytics para encontrar los eventos de seguridad de Windows, escriba `SecurityEvent` en la ventana de consulta.

# <a name="azure-monitor-agent-new"></a>[Agente de Azure Monitor (nuevo)](#tab/AMA)

1. En el menú de navegación de Azure Sentinel, seleccione **Conectores de datos**. En la lista de conectores, haga clic en **Eventos de seguridad de Windows (versión preliminar)** (Windows Security Events [Preview]) y, a continuación, en el botón **Open connector page** (Abrir página del conector) en la parte inferior derecha. A continuación, siga las instrucciones en pantalla debajo de la pestaña **Instrucciones**, tal como se describe en el resto de esta sección.

1. Compruebe que tiene los permisos adecuados, tal como se describe en la sección **Requisitos previos** de la página del conector.

    1. Debe tener permisos de lectura y escritura en el área de trabajo y en todos los orígenes de datos desde los que va a ingerir eventos de seguridad de Windows.

    1. Para recopilar eventos de máquinas Windows que no sean máquinas virtuales de Azure, las máquinas (físicas o virtuales) deben tener Azure Arc instalado y habilitado. [Más información](../azure-monitor/agents/azure-monitor-agent-install.md).

1. En **Configuración,** seleccione **+Agregar regla de recopilación de datos**. El **Asistente para crear reglas de recopilación de datos** se abrirá a la derecha.

1. En **Aspectos básicos,** escriba una **regla de nombre** y especifique una **suscripción** y un **grupo de recursos** donde se creará la regla de recopilación de datos (DCR). *No* tiene que ser el mismo grupo de recursos o suscripción en el que se encuentran las máquinas supervisadas y sus asociaciones, siempre y cuando estén en el mismo inquilino.

1. En la pestaña **Recursos,** seleccione **+Agregar recursos** para agregar máquinas a las que se aplicará la regla de recopilación de datos. Se abrirá el cuadro de diálogo **Seleccionar un ámbito** y verá una lista de suscripciones disponibles. Expanda una suscripción para ver sus grupos de recursos y expanda un grupo de recursos para ver las máquinas disponibles. Verá las máquinas virtuales de Azure y los servidores habilitados para Azure Arc en la lista. Puede marcar las casillas de suscripciones o grupos de recursos para seleccionar todas las máquinas que contienen, o puede seleccionar máquinas individuales. Seleccione **Aplicar** cuando haya elegido todas las máquinas. Al final de este proceso, el agente de Azure Monitor se instalará en las máquinas seleccionadas que aún no lo tengan instalado.

1. En la pestaña **Recopilar**. elija el [conjunto de eventos](#event-id-reference) que quiere recopilar o seleccione **Personalizado** para especificar otros registros o filtrar eventos mediante [consultas XPath](../azure-monitor/agents/data-collection-rule-azure-monitor-agent.md#limit-data-collection-with-custom-xpath-queries). Escriba expresiones en el cuadro que se evalúen como criterios XML específicos para los eventos que se recopilan y, a continuación, seleccione **Agregar**. Puede escribir hasta 20 expresiones en un solo cuadro y hasta 100 cuadros en una regla.

    Obtenga más información sobre las [reglas de recopilación de datos](../azure-monitor/agents/data-collection-rule-overview.md#create-a-dcr) en la documentación de Azure Monitor.

    > [!NOTE]
    > - Asegúrese de consultar solo los registros de seguridad de Windows y AppLocker. Es posible que los eventos de otros registros de Windows o de registros de seguridad de otros entornos no se adhieran al esquema de eventos de seguridad de Windows y no se analicen correctamente, en cuyo caso no se ingieren en el área de trabajo.
    >
    > - El agente de Azure Monitor admite consultas XPath solo para la **[versión 1.0 de XPath](/windows/win32/wes/consuming-events#xpath-10-limitations)** .

1. Cuando haya agregado todas las expresiones de filtro que quiera, seleccione **Siguiente: Revisar y crear**.

1. Cuando vea el mensaje "Validación superada", seleccione **Crear**. 

Verá todas las reglas de recopilación de datos (incluidas las creadas a través de la API) en **Configuración** en la página del conector. Desde allí, puede editar o eliminar las reglas existentes.

> [!TIP]
> Use el cmdlet de PowerShell **Get-WinEvent** con el parámetro *-FilterXPath* para probar la validez de una consulta XPath. En el script siguiente se muestra un ejemplo:
> ```powershell
> $XPath = '*[System[EventID=1035]]'
> Get-WinEvent -LogName 'Application' -FilterXPath $XPath
> ```
> - Si se devuelven eventos, la consulta es válida.
> - Si recibe el mensaje No se encontraron eventos que coincidan con los criterios de selección especificados. , la consulta puede ser válida, pero no hay eventos coincidentes en el equipo local.
> - Si recibe el mensaje La consulta especificada no es válida, la sintaxis de la consulta no es válida.

### <a name="create-data-collection-rules-using-the-api"></a>Creación de reglas de recopilación de datos mediante la API

También puede crear reglas de recopilación de datos mediante la API ([consulte el esquema](/rest/api/monitor/data-collection-rules)), lo que le puede facilitar la vida si va a crear muchas reglas (por ejemplo, si es un MSSP). Este es un ejemplo que puede usar como plantilla para crear una regla:

**Encabezado y dirección URL de solicitud**

```http
PUT https://management.azure.com/subscriptions/703362b3-f278-4e4b-9179-c76eaf41ffc2/resourceGroups/myResourceGroup/providers/Microsoft.Insights/dataCollectionRules/myCollectionRule?api-version=2019-11-01-preview
```
**Encabezado y dirección URL de solicitud**

```json
{
    "location": "eastus",
    "properties": {
        "dataSources": {
            "windowsEventLogs": [
                {
                    "streams": [
                        "Microsoft-SecurityEvent"
                    ],
                    "xPathQueries": [
                        "Security!*[System[(EventID=) or (EventID=4688) or (EventID=4663) or (EventID=4624) or (EventID=4657) or (EventID=4100) or (EventID=4104) or (EventID=5140) or (EventID=5145) or (EventID=5156)]]"
                    ],
                    "name": "eventLogsDataSource"
                }
            ]
        },
        "destinations": {
            "logAnalytics": [
                {
                    "workspaceResourceId": "/subscriptions/703362b3-f278-4e4b-9179-c76eaf41ffc2/resourceGroups/myResourceGroup/providers/Microsoft.OperationalInsights/workspaces/centralTeamWorkspace",
                    "name": "centralWorkspace"
                }
            ]
        },
        "dataFlows": [
            {
                "streams": [
                    "Microsoft-SecurityEvent"
                ],
                "destinations": [
                    "centralWorkspace"
                ]
            }
        ]
    }
}
```



---

## <a name="validate-connectivity"></a>Validar conectividad

Hasta que los registros empiecen a aparecer en Log Analytics, pueden transcurrir unos 20 minutos. 

### <a name="configure-the-security-events--windows-security-events-connector-for-anomalous-rdp-login-detection"></a>Configuración del conector de eventos de seguridad o de eventos de seguridad de Windows para la detección de inicios de sesión de RDP anómalos

> [!IMPORTANT]
> La detección de inicios de sesión de RDP anómalos se encuentra actualmente en versión preliminar pública.
> Esta característica se ofrece sin contrato de nivel de servicio y no se recomienda para cargas de trabajo de producción.
> Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Azure Sentinel puede aplicar aprendizaje automático (ML) a los datos de eventos de seguridad para identificar una actividad de inicio de sesión de protocolo de escritorio remoto (RDP) anómala. Los escenarios incluyen:

- **IP inusual**: la dirección IP no se ha observado nunca o casi nunca en los últimos 30 días

- **Ubicación geográfica inusual**: la dirección IP, la ciudad, el país y el ASN no se han observado nunca o casi nunca en los últimos 30 días

- **Nuevo usuario**: un nuevo usuario inicia sesión desde una dirección IP o una ubicación geográfica que no se esperaba ver de acuerdo con los datos de los 30 días anteriores.

**Instrucciones de configuración**

1. Debe recopilar los datos de inicio de sesión de RDP (identificador de evento 4624) mediante los conector de datos de **eventos de seguridad** o de **eventos de seguridad de Windows**. Asegúrese de que ha seleccionado un [conjunto de eventos](#event-id-reference) que no sea "Ninguno" o que ha creado una regla de recopilación de datos que incluye este identificador de evento para la transmisión a Azure Sentinel.

1. En el portal de Azure Sentinel, haga clic en **Analytics** (Análisis) y, después, haga clic en la pestaña **Rule templates** (Plantillas de reglas). Seleccione la regla **(Preview) Anomalous RDP Login Detection** [(Versión preliminar) Detección de inicio de sesión RDP anómalo] y mueva el regulador **Status** (Estado) a **Enabled** (Habilitado).

    > [!NOTE]
    > Dado que el algoritmo de aprendizaje automático requiere datos de 30 días para crear un perfil de base de referencia del comportamiento de los usuarios, debe permitir que se recopilen los datos de eventos de seguridad de Windows de 30 días antes de que se puedan detectar incidentes.

## <a name="event-id-reference"></a>Referencia de Id. de evento

En la lista siguiente se muestra un desglose completo de los identificadores de evento de seguridad y de AppLocker para cada conjunto:

| Conjunto de eventos | Identificadores de eventos recopilados |
| --- | --- |
| **Mínimo** | 1102, 4624, 4625, 4657, 4663, 4688, 4700, 4702, 4719, 4720, 4722, 4723, 4724, 4727, 4728, 4732, 4735, 4737, 4739, 4740, 4754, 4755, 4756, 4767, 4799, 4825, 4946, 4948, 4956, 5024, 5033, 8001, 8002, 8003, 8004, 8005, 8006, 8007, 8222 |
| **Común** | 1, 299, 300, 324, 340, 403, 404, 410, 411, 412, 413, 431, 500, 501, 1100, 1102, 1107, 1108, 4608, 4610, 4611, 4614, 4622, 4624, 4625, 4634, 4647, 4648, 4649, 4657, 4661, 4662, 4663, 4665, 4666, 4667, 4688, 4670, 4672, 4673, 4674, 4675, 4689, 4697, 4700, 4702, 4704, 4705, 4716, 4717, 4718, 4719, 4720, 4722, 4723, 4724, 4725, 4726, 4727, 4728, 4729, 4733, 4732, 4735, 4737, 4738, 4739, 4740, 4742, 4744, 4745, 4746, 4750, 4751, 4752, 4754, 4755, 4756, 4757, 4760, 4761, 4762, 4764, 4767, 4768, 4771, 4774, 4778, 4779, 4781, 4793, 4797, 4798, 4799, 4800, 4801, 4802, 4803, 4825, 4826, 4870, 4886, 4887, 4888, 4893, 4898, 4902, 4904, 4905, 4907, 4931, 4932, 4933, 4946, 4948, 4956, 4985, 5024, 5033, 5059, 5136, 5137, 5140, 5145, 5632, 6144, 6145, 6272, 6273, 6278, 6416, 6423, 6424, 8001, 8002, 8003, 8004, 8005, 8006, 8007, 8222, 26401, 30004 |
|

## <a name="next-steps"></a>Pasos siguientes
En este documento, ha aprendido a conectar eventos de seguridad de Windows a Azure Sentinel. Para más información sobre Azure Sentinel, consulte los siguientes artículos:
- Aprenda a [obtener visibilidad de los datos y de posibles amenazas](quickstart-get-visibility.md).
- Empiece a detectar amenazas con Azure Sentinel mediante las reglas [integradas](tutorial-detect-threats-built-in.md) o [personalizadas](tutorial-detect-threats-custom.md).

