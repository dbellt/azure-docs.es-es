---
title: Configuración de la recopilación de datos para el agente de Azure Monitor (versión preliminar)
description: Se describe cómo crear una regla de recopilación de datos para recopilar datos de máquinas virtuales mediante el agente de Azure Monitor.
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 03/16/2021
ms.openlocfilehash: 884f048ac099cfc6b799fe266172a0eecef3db6f
ms.sourcegitcommit: afb79a35e687a91270973990ff111ef90634f142
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107478376"
---
# <a name="configure-data-collection-for-the-azure-monitor-agent-preview"></a>Configuración de la recopilación de datos para el agente de Azure Monitor (versión preliminar)

Las reglas de recopilación de datos (DCR) definen los datos que entran en Azure Monitor y especifican dónde se deben enviar. En este artículo se describe cómo crear una regla de recopilación de datos para recopilar datos de máquinas virtuales mediante el agente de Azure Monitor.

Para obtener una descripción completa de las reglas de recopilación de datos, consulte [Reglas de recopilación de datos en Azure Monitor (versión preliminar)](data-collection-rule-overview.md).

> [!NOTE]
> En este artículo se describe cómo configurar los datos de las máquinas virtuales con el agente de Azure Monitor que se encuentra actualmente en versión preliminar. Consulte [Información general sobre los agentes de Azure Monitor](agents-overview.md) para obtener una descripción de los agentes que están disponibles con carácter general y cómo usarlos para recopilar datos.

## <a name="data-collection-rule-associations"></a>Asociaciones de reglas de recopilación de datos

Para aplicar una reglas de recopilación de datos a una máquina virtual, cree una asociación para la máquina virtual. Una máquina virtual puede tener una asociación a varias reglas de recopilación de datos y una de estas reglas puede tener varias máquinas virtuales asociadas. Esto le permite definir un conjunto de reglas de recopilación de datos, cada una de las cuales coincide con un requisito determinado y aplicarlas solo a las máquinas virtuales en las que se aplican. 

Por ejemplo, imagine un entorno con un conjunto de máquinas virtuales que ejecutan una aplicación de línea de negocio y otras que se ejecutan SQL Server. Es posible que tenga una regla de recopilación de datos predeterminada que se aplique a todas las máquinas virtuales y a reglas de recopilación de datos independientes que recopilen datos específicamente para la aplicación de línea de negocio y SQL Server. Las asociaciones de las máquinas virtuales con las reglas de recopilación de datos tendrían un aspecto similar al siguiente diagrama.

![En el diagrama se muestran las máquinas virtuales que hospedan la aplicación de línea de negocio y SQL Server asociadas a las reglas de recopilación de datos denominadas central-i t-default y lob-app para la aplicación de línea de negocio y central-i t-default y s q l para SQL Server.](media/data-collection-rule-azure-monitor-agent/associations.png)



## <a name="create-rule-and-association-in-azure-portal"></a>Creación de reglas y asociaciones en Azure Portal

Puede usar Azure Portal para crear una regla de recopilación de datos y asociar las máquinas virtuales de su suscripción a esa regla. El agente de Azure Monitor se instalará automáticamente y se creará una identidad administrada para las máquinas virtuales que no lo tengan instalado.

> [!IMPORTANT]
> Actualmente hay un problema conocido por el que, si la regla de recopilación de datos crea una identidad administrada en una máquina virtual que ya tiene una identidad administrada asignada por el usuario, la identidad asignada por el usuario se deshabilita.

En el menú **Azure Monitor** de Azure Portal, seleccione las **reglas de recopilación de datos** en la sección **Configuración**. Haga clic en **Agregar** para agregar una nueva asignación y regla de recopilación de datos.

[![Reglas de recopilación de datos](media/data-collection-rule-azure-monitor-agent/data-collection-rules.png)](media/data-collection-rule-azure-monitor-agent/data-collection-rules.png#lightbox)

Haga clic en **Agregar** para crear una nueva regla y un conjunto de asociaciones. Dé un **nombre de regla** y especifique una **suscripción** y **un grupo de recursos**. Esto especifica dónde se creará la regla de recopilación de datos. Las máquinas virtuales y sus asociaciones pueden estar en cualquier suscripción o grupo de recursos del inquilino.

[![Nombre de la regla de recopilación de datos](media/data-collection-rule-azure-monitor-agent/data-collection-rule-basics.png)](media/data-collection-rule-azure-monitor-agent/data-collection-rule-basics.png#lightbox)

En la pestaña **Máquinas virtuales**, agregue las máquinas virtuales que deben tener aplicada la regla de recopilación de datos. Deben mostrarse las máquinas virtuales de Azure y los servidores habilitados para Azure Arc en su entorno. El agente de Azure Monitor se instalará en las máquinas virtuales que no lo tengan instalado.

[![Máquinas virtuales de reglas de recopilación de datos](media/data-collection-rule-azure-monitor-agent/data-collection-rule-virtual-machines.png)](media/data-collection-rule-azure-monitor-agent/data-collection-rule-virtual-machines.png#lightbox)

En la pestaña **Collect and deliver** (Recopilar y entregar), haga clic en **Add data source** (Agregar origen de datos) para agregar un origen de datos y un conjunto de destino. Seleccione un **tipo de origen de datos** y se mostrarán los detalles correspondientes. Para los contadores de rendimiento, puede seleccionar entre un conjunto predefinido de objetos y su frecuencia de muestreo. En el caso de los eventos, puede seleccionar entre un conjunto de registros o instalaciones y el nivel de gravedad. 

[![Origen de datos básico](media/data-collection-rule-azure-monitor-agent/data-collection-rule-data-source-basic.png)](media/data-collection-rule-azure-monitor-agent/data-collection-rule-data-source-basic.png#lightbox)


Para especificar otros registros y contadores de rendimiento de los [orígenes de datos admitidos actualmente](azure-monitor-agent-overview.md#data-sources-and-destinations) o para filtrar eventos mediante consultas XPath, seleccione **Custom** (Personalizado). Después, puede especificar un [XPath](https://www.w3schools.com/xml/xpath_syntax.asp) para los valores específicos que se van a recopilar. Consulte la [regla de recopilación de datos de ejemplo](data-collection-rule-overview.md#sample-data-collection-rule) para ver ejemplos.

[![Origen de datos personalizado](media/data-collection-rule-azure-monitor-agent/data-collection-rule-data-source-custom.png)](media/data-collection-rule-azure-monitor-agent/data-collection-rule-data-source-custom.png#lightbox)

En la pestaña **Destination** (Destino), agregue uno o varios destinos para el origen de datos. Los orígenes de datos de eventos de Windows y Syslog solo pueden enviar a registros de Azure Monitor. Los contadores de rendimiento pueden enviar a métricas de Azure Monitor y registros de Azure Monitor.

[![Destination](media/data-collection-rule-azure-monitor-agent/data-collection-rule-destination.png)](media/data-collection-rule-azure-monitor-agent/data-collection-rule-destination.png#lightbox)

Haga clic en **Agregar origen de datos** (Agregar origen de datos) y, después, en **Revisar y crear** para revisar los detalles de la regla de recopilación de datos y la asociación con el conjunto de máquinas virtuales. Haga clic en **Crear** para crearlo.

> [!NOTE]
> Una vez creadas las asociaciones y la regla de recopilación de datos, los datos pueden tardar hasta cinco minutos en enviarse a los destinos.

## <a name="limit-data-collection-with-custom-xpath-queries"></a>Limitar la recopilación de datos con consultas XPath personalizadas
Dado que se le cobran los datos recopilados en un área de trabajo Log Analytics, debe recopilar solo los datos que necesite. Con la configuración básica en el Azure Portal, solo tiene una capacidad limitada para filtrar los eventos que se van a recopilar. En el caso de los registros del sistema y de la aplicación, se trata de todos los registros con una gravedad determinada. En el caso de los registros de seguridad, se trata de todos los registros de auditoría correcta o error de auditoría.

Para especificar filtros adicionales, debe usar la configuración personalizada y especificar un XPath que filtre los eventos que no tiene. Las entradas XPath se escriben en el formulario `LogName!XPathQuery`. Por ejemplo, puede que desee devolver solo los eventos del registro de eventos de aplicación con el identificador de evento 1035. El XPathQuery para estos eventos sería `*[System[EventID=1035]]`. Dado que desea recuperar los eventos del registro de eventos de la aplicación, el XPath sería `Application!*[System[EventID=1035]]`

Vea [limitaciones de XPath 1.0](/windows/win32/wes/consuming-events#xpath-10-limitations) para obtener una lista de las limitaciones de XPath que admite el registro de eventos de Windows.

> [!TIP]
> Use el cmdlet de PowerShell `Get-WinEvent` con el `FilterXPath` parámetro para probar la validez de un XPathQuery. El script siguiente muestra un ejemplo de la edición.
> 
> ```powershell
> $XPath = '*[System[EventID=1035]]'
> Get-WinEvent -LogName 'Application' -FilterXPath $XPath
> ```
>
> - Si se devuelven eventos, la consulta es válida.
> - Si recibe el mensaje *No se encontraron eventos que coincidan con los criterios de selección especificados.* , la consulta puede ser válida, pero no hay eventos coincidentes en el equipo local.
> - Si recibe el mensaje *La consulta especificada no es válida*, la sintaxis de la consulta no es válida. 

En la tabla siguiente se muestran ejemplos de filtrado de eventos mediante un XPath personalizado.

| Descripción |  XPath |
|:---|:---|
| Recopilar solo eventos del Sistema con id. de Evento = 4648 |  `System!*[System[EventID=4648]]`
| Recopilar solo eventos del Sistema con id. de evento = 4648 y un nombre de proceso de consent.exe | `Security!*[System[(EventID=4648)]] and *[EventData[Data[@Name='ProcessName']='C:\Windows\System32\consent.exe']]` |
| Recopilar todos los eventos críticos, de error, de advertencia y de información del registro de eventos del Sistema, excepto el id. de evento = 6 (controlador cargado) |  `System!*[System[(Level=1 or Level=2 or Level=3) and (EventID != 6)]]` |
| Recopilación de todos los eventos de seguridad correctos y erróneos excepto el id. de evento 4624 (inicio de sesión correcto) |  `Security!*[System[(band(Keywords,13510798882111488)) and (EventID != 4624)]]` |


## <a name="create-rule-and-association-using-rest-api"></a>Creación de reglas y asociaciones mediante una API REST

Siga los pasos que se indican a continuación para crear una regla de recopilación de datos y asociaciones mediante la API REST.

1. Cree manualmente el archivo de regla de recopilación de datos con el formato JSON que se muestra en [DCR de ejemplo](data-collection-rule-overview.md#sample-data-collection-rule).

2. Cree la regla con la [API REST](/rest/api/monitor/datacollectionrules/create#examples).

3. Cree una asociación para cada máquina virtual a la regla de recopilación de datos mediante la [API REST](/rest/api/monitor/datacollectionruleassociations/create#examples).


## <a name="create-association-using-resource-manager-template"></a>Creación de una asociación mediante plantillas de Resource Manager

Puede crear una asociación entre una máquina virtual de Azure o un servidor habilitado para Azure Arc mediante una plantilla de Resource Manager. Consulte [Ejemplos de plantillas de Resource Manager para reglas de recopilación de datos en Azure Monitor](./resource-manager-data-collection-rules.md) para obtener plantillas de ejemplo.



## <a name="next-steps"></a>Pasos siguientes

- Más información sobre el [agente de Azure Monitor](azure-monitor-agent-overview.md).
- Más información sobre las [reglas de recopilación de datos](data-collection-rule-overview.md).
