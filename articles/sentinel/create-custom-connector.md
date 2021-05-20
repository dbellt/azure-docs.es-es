---
title: Recursos para crear conectores personalizados de Azure Sentinel | Microsoft Docs
description: Obtenga información sobre los recursos disponibles para crear conectores personalizados para Azure Sentinel. Los métodos incluyen el agente y API de Log Analytics, Logstash, Logic Apps, PowerShell y Azure Functions.
services: sentinel
documentationcenter: na
author: batamig
manager: rkarlin
editor: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: conceptual
ms.custom: mvc
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 02/09/2021
ms.author: bagol
ms.openlocfilehash: d964dc25fd450f244e34140080e77cdc0e319941
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108749436"
---
# <a name="resources-for-creating-azure-sentinel-custom-connectors"></a>Recursos para crear conectores personalizados de Azure Sentinel

Azure Sentinel proporciona una amplia gama de [conectores integrados para los servicios de Azure y soluciones externas](connect-data-sources.md), y también admite la ingesta de datos desde algunos orígenes sin un conector dedicado.

Si no puede conectar el origen de datos a Azure Sentinel mediante cualquiera de las soluciones existentes disponibles, considere la posibilidad de crear su propio conector de origen de datos.

Para obtener una lista completa de los conectores admitidos, consulte la entrada de blog [Azure Sentinel: Generalidades de los conectores (CEF, Syslog, directo, agente, personalizado y más)](https://techcommunity.microsoft.com/t5/azure-sentinel/azure-sentinel-the-connectors-grand-cef-syslog-direct-agent/ba-p/803891).

## <a name="compare-custom-connector-methods"></a>Comparación de los métodos de conectores personalizados

En la tabla siguiente se comparan los detalles esenciales de cada método para crear los conectores personalizados que se describen en este artículo. Seleccione los vínculos de la tabla para obtener más detalles sobre cada método.

|Descripción del método  |Capacidad | Sin servidor    |Complejidad  |
|---------|---------|---------|---------|
|**[Agente de Log Analytics](#connect-with-the-log-analytics-agent)** <br>El mejor para recopilar archivos de orígenes locales y de IaaS   | Solo recopilación de archivos  |   No      |Bajo         |
|**[Logstash](#connect-with-logstash)** <br>El mejor para orígenes locales y de IaaS, cualquier origen para el que haya un complemento disponible, y organizaciones que ya estén familiarizadas con Logstash.  | Complementos disponibles, más complemento personalizado, las funcionalidades proporcionan una gran flexibilidad.   |   No; requiere la ejecución de una VM o un clúster de VM.           |   Baja; admite muchos escenarios con complementos.      |
|**[Logic Apps](#connect-with-logic-apps)** <br>Costo alto; se evitan datos de gran volumen. <br>El mejor para orígenes en la nube de bajo volumen.  | La programación sin código permite una flexibilidad limitada, sin compatibilidad para la implementación de algoritmos.<br><br> Si no hay ninguna acción disponible que ya admita sus requisitos, la creación de una acción personalizada puede sumar complejidad.    |    Sí         |   Baja; desarrollo sencillo y sin código.      |
|**[PowerShell](#connect-with-powershell)** <br>El mejor para la creación de prototipos y cargas de archivos periódicas. | Compatibilidad directa con la recopilación de archivos. <br><br>PowerShell se puede usar para recopilar más orígenes, pero requiere codificación y configuración del script como servicio.      |No               |  Bajo       |
|**[API de Log Analytics](#connect-with-the-log-analytics-api)** <br>El mejor para los ISV que implementan la integración y para los requisitos de recopilación únicos.   | Admite todas las funcionalidades disponibles con el código.  | Depende de la implementación.           |     Alto    |
|**[Azure Functions](#connect-with-azure-functions)** El mejor para los orígenes en la nube de gran volumen y para los requisitos de recopilación únicos  | Admite todas las funcionalidades disponibles con el código.  |  Sí             |     Alta; requiere conocimientos de programación.    |
|     |         |                |

> [!TIP]
> Para obtener comparaciones del uso de Logic Apps y Azure Functions para el mismo conector, consulte:
> 
> - [Ingesta de registros de Web Application Firewall con rapidez en Azure Sentinel](https://techcommunity.microsoft.com/t5/azure-sentinel/ingest-fastly-web-application-firewall-logs-into-azure-sentinel/ba-p/1238804)
> - Office 365 (comunidad de Azure Sentinel en GitHub): [Conector Logic App](https://github.com/Azure/Azure-Sentinel/tree/master/Playbooks/Get-O365Data) | [Conector Azure Function](https://github.com/Azure/Azure-Sentinel/tree/master/DataConnectors/O365%20Data)
> 

## <a name="connect-with-the-log-analytics-agent"></a>Conexión con el agente de Log Analytics

Si el origen de datos entrega eventos en archivos, se recomienda usar el agente de Log Analytics de Azure Monitor para crear el conector personalizado.

- Para obtener más información, consulte [Recopilación de registros personalizados en Azure Monitor](../azure-monitor/agents/data-sources-custom-logs.md).

- Para ver un ejemplo de este método, consulte [Recopilación de orígenes de datos JSON personalizados con el agente de Log Analytics para Linux en Azure Monitor](../azure-monitor/agents/data-sources-json.md).

## <a name="connect-with-logstash"></a>Conexión con Logstash

Si está familiarizado con [Logstash](https://www.elastic.co/logstash), puede que quiera usar Logstash con el [complemento de salida de Logstash para Azure Sentinel](connect-logstash.md) para crear el conector personalizado.

Con el complemento de salida de Logstash de para Azure Sentinel, puede usar cualquier complemento de entrada y filtrado de Logstash, y configurar Azure Sentinel como salida para una canalización de Logstash. Logstash tiene una gran biblioteca de complementos que permiten la entrada desde diversos orígenes, como Event Hubs, Apache Kafka, archivos, bases de datos y servicios en la nube. Use complementos de filtrado para analizar eventos, filtrar eventos innecesarios, ofuscar valores y mucho más.

Para obtener ejemplos del uso de Logstash como conector personalizado, consulte:

- [Búsqueda de tácticas, técnicas y procedimientos (TTP) de vulneraciones de Capital one en los registros de AWS con Azure Sentinel](https://techcommunity.microsoft.com/t5/azure-sentinel/hunting-for-capital-one-breach-ttps-in-aws-logs-using-azure/ba-p/1019767) (blog)
- [Guía de implementación de Azure Sentinel de Radware](https://support.radware.com/ci/okcsFattach/get/1025459_3)

Para obtener ejemplos de complementos de Logstash útiles, consulte:

- [Complemento de entrada de Cloudwatch](https://www.elastic.co/guide/en/logstash/current/plugins-inputs-cloudwatch.html)
- [Complemento de Azure Event Hubs](https://www.elastic.co/guide/en/logstash/current/plugins-inputs-azure_event_hubs.html)
- [Complemento de entrada de Google Cloud Storage](https://www.elastic.co/guide/en/logstash/current/plugins-inputs-google_cloud_storage.html)
- [Complemento de entrada Google_pubsub](https://www.elastic.co/guide/en/logstash/current/plugins-inputs-google_pubsub.html)

> [!TIP]
> Logstash también permite la recopilación de datos escalados mediante un clúster. Para obtener más información, consulte [Uso de una VM de Logstash con equilibrio de carga a gran escala](https://techcommunity.microsoft.com/t5/azure-sentinel/scaling-up-syslog-cef-collection/ba-p/1185854).
>

## <a name="connect-with-logic-apps"></a>Conexión con Logic Apps

Use [Azure Logic Apps](../logic-apps/index.yml) para crear un conector personalizado y sin servidor para Azure Sentinel.

> [!NOTE]
> Aunque puede ser conveniente crear conectores sin servidor con Logic Apps, el uso de Logic Apps para los conectores puede ser costoso si trabaja con grandes volúmenes de datos.
>
> Se recomienda usar este método solo para orígenes de datos de bajo volumen o enriquecer las cargas de datos.
>

1. **Use uno de los siguientes desencadenadores para iniciar Logic Apps**:

    |Desencadenador  |Descripción  |
    |---------|---------|
    |**Una tarea recurrente**     |   Por ejemplo, programe la instancia de Logic Apps para que recupere datos periódicamente de archivos, bases de datos o API externas. <br>Para obtener más información, consulte [Creación, programación y ejecución de tareas y flujos de trabajo periódicos en Azure Logic Apps](../connectors/connectors-native-recurrence.md).      |
    |**Desencadenamiento a petición**     | Ejecute la instancia de Logic Apps a petición para la recopilación de datos y las pruebas manuales. <br>Para obtener más información, consulte [Llamada, desencadenamiento o anidación de aplicaciones lógicas mediante puntos de conexión HTTPS](../logic-apps/logic-apps-http-endpoint.md).        |
    |**Punto de conexión HTTP/S**     |  Se recomienda para el streaming, y si el sistema de origen puede iniciar la transferencia de datos. <br>Para obtener más información, consulte [Llamada a puntos de conexión de servicio mediante HTTP o HTTPS](../connectors/connectors-native-http.md).       |
    |     |         |

1. **Use cualquiera de los conectores de Logic Apps que leen la información para obtener los eventos**. Por ejemplo:

    - [Conexión a una API REST](/connectors/custom-connectors/)
    - [Conectarse a un servidor SQL Server](/connectors/sql/)
    - [Conexión a un sistema de archivos](/connectors/filesystem/)

    > [!TIP]
    > Los conectores personalizados a las API REST, instancias de SQL Server y sistemas de archivos también admiten la recuperación de datos de orígenes de datos locales. Para obtener más información, consulte la documentación sobre [Instalación de la puerta de enlace de datos local](/connectors/filesystem/).
    >

1. **Prepare la información que quiera recuperar**.

    Por ejemplo, use la [acción analizar JSON](../logic-apps/logic-apps-perform-data-operations.md#parse-json-action) para acceder a las propiedades de contenido JSON, lo que le permite seleccionar esas propiedades en la lista de contenido dinámico cuando especifique entradas para la instancia de Logic Apps.

    Para obtener más información, consulte [Realización de operaciones de datos en Azure Logic Apps](../logic-apps/logic-apps-perform-data-operations.md).

1. **Escriba los datos en Log Analytics**.

    Para obtener más información, consulte la documentación del [Recopilador de datos de Azure Log Analytics](/connectors/azureloganalyticsdatacollector/).

Para ver ejemplos de cómo puede crear un conector personalizado para Azure Sentinel mediante Logic Apps, consulte:

- [Creación de una canalización de datos con Data Collector API](/connectors/azureloganalyticsdatacollector/)
- [Conector Logic App para Prisma de Palo Alto mediante un webhook](https://github.com/Azure/Azure-Sentinel/tree/master/Playbooks/Ingest-Prisma) (comunidad de Azure Sentinel en GitHub)
- [Protección de las llamadas de Microsoft Teams con activación programada](https://techcommunity.microsoft.com/t5/azure-sentinel/secure-your-calls-monitoring-microsoft-teams-callrecords/ba-p/1574600) (blog)
- [Ingesta de indicadores de amenazas de AlienVault OTX en Azure Sentinel](https://techcommunity.microsoft.com/t5/azure-sentinel/ingesting-alien-vault-otx-threat-indicators-into-azure-sentinel/ba-p/1086566) (blog)

## <a name="connect-with-powershell"></a>Conexión con PowerShell

El [script Upload-AzMonitorLog de PowerShell](https://www.powershellgallery.com/packages/Upload-AzMonitorLog/) permite usar PowerShell para transmitir eventos o información de contexto a Azure Sentinel desde la línea de comandos. Este streaming crea eficazmente un conector personalizado entre el origen de datos y Azure Sentinel.

Por ejemplo, el siguiente script carga un archivo CSV en Azure Sentinel:

``` PowerShell
Import-Csv .\testcsv.csv
| .\Upload-AzMonitorLog.ps1
-WorkspaceId '69f7ec3e-cae3-458d-b4ea-6975385-6e426'
-WorkspaceKey $WSKey
-LogTypeName 'MyNewCSV'
-AddComputerName
-AdditionalDataTaggingName "MyAdditionalField"
-AdditionalDataTaggingValue "Foo"
```

El [script Upload-AzMonitorLog de PowerShell](https://www.powershellgallery.com/packages/Upload-AzMonitorLog/) usa los siguientes parámetros:

|Parámetro  |Descripción  |
|---------|---------|
|**WorkspaceId**     |   El identificador del área de trabajo de Azure Sentinel, donde almacenará los datos.  [Busque el identificador y la clave del área de trabajo](#find-your-workspace-id-and-key).  |
|**WorkspaceKey**     |   La clave principal o secundaria del área de trabajo de Azure Sentinel donde almacenará los datos. [Busque el identificador y la clave del área de trabajo](#find-your-workspace-id-and-key).  |
|**LogTypeName**     |    El nombre de la tabla de registro personalizada donde quiere almacenar los datos. Se agregará automáticamente un sufijo **_CL** al final del nombre de la tabla.  |
|**AddComputerName**     |   Cuando este parámetro existe, el script agrega el nombre del equipo actual a cada entrada del registro, en un campo denominado **Computer**.      |
|**TaggedAzureResourceId**     | Cuando este parámetro existe, el script asocia todas las entradas del registro cargadas con el recurso de Azure especificado. <br><br>Esta asociación permite las entradas del registro cargadas para las consultas de contexto de recursos, y cumple con el control de acceso basado en roles centrado en recursos.       |
|**AdditionalDataTaggingName**     |      Cuando este parámetro existe, el script agrega otro campo a cada entrada del registro, con el nombre configurado y el valor configurado para el parámetro **AdditionalDataTaggingValue**. <br><br>En este caso, **AdditionalDataTaggingValue** no debe estar vacío. |
|**AdditionalDataTaggingValue**     |   Cuando este parámetro existe, el script agrega otro campo a cada entrada del registro, con el valor configurado y el nombre del campo configurado para el parámetro **AdditionalDataTaggingName**. <br><br>Si el parámetro **AdditionalDataTaggingName** está vacío, pero hay configurado un valor, el nombre de campo predeterminado es **DataTagging**.       |
|     |         |

### <a name="find-your-workspace-id-and-key"></a>Búsqueda del identificador y la clave del área de trabajo

Busque los detalles de los parámetros **WorkspaceID** y **WorkspaceKey** en Azure Sentinel:

1. En Azure Sentinel, seleccione **Configuración** a la izquierda y, a continuación, seleccione la pestaña **Configuración del área de trabajo**.

1. En **Introducción a Log Analytics** > **1 Conectar un origen de datos**, seleccione **Windows and Linux agents management** (Administración de agentes de Windows y Linux).

1. Busque el identificador del área de trabajo, la clave principal y la clave secundaria en las pestañas **Servidores de Windows**.
## <a name="connect-with-the-log-analytics-api"></a>Conexión con la API de Log Analytics

Puede transmitir eventos a Azure Sentinel mediante la API de recopilador de datos de Log Analytics para llamar directamente a un punto de conexión de RESTful.

Aunque llamar directamente a un punto de conexión de RESTful requiere más programación, también proporciona mayor flexibilidad.

Para obtener más información, consulte la [API de recopilador de datos de Log Analytics](../azure-monitor/logs/data-collector-api.md), especialmente los ejemplos siguientes:

- [C#](../azure-monitor/logs/data-collector-api.md#c-sample)
- [Python 2](../azure-monitor/logs/data-collector-api.md#python-2-sample)

## <a name="connect-with-azure-functions"></a>Conexión con Azure Functions

Use Azure Functions junto con una API RESTful y varios lenguajes de codificación, como [PowerShell](../azure-functions/functions-reference-powershell.md), para crear un conector personalizado sin servidor.

Para obtener ejemplos de este método, consulte:

- [Conexión de datos de VMware Carbon Black Cloud Endpoint Standard a Azure Sentinel a Azure Functions](connect-vmware-carbon-black.md)
- [Conexión del inicio de sesión único de Okta a Azure Sentinel con Azure Functions](connect-okta-single-sign-on.md)
- [Conexión de Proofpoint TAP a Azure Sentinel con Azure Functions](connect-proofpoint-tap.md)
- [Conexión de Qualys Vulnerability Management a Azure Sentinel con Azure Functions](connect-qualys-vm.md)
- [Ingesta de XML, CSV u otros formatos de datos](../azure-monitor/logs/create-pipeline-datacollector-api.md#ingesting-xml-csv-or-other-formats-of-data)
- [Supervisión de Zoom con Azure Sentinel](https://techcommunity.microsoft.com/t5/azure-sentinel/monitoring-zoom-with-azure-sentinel/ba-p/1341516) (blog)
- [Implementación de un aplicación de funciones para obtener datos de la API de administración de Office 365 en Azure Sentinel](https://github.com/Azure/Azure-Sentinel/tree/master/DataConnectors/O365%20Data) (comunidad de Azure Sentinel en GitHub)

## <a name="parse-your-custom-connector-data"></a>Análisis de los datos de un conector personalizado

Puede usar la técnica de análisis integrada del conector personalizado para extraer la información pertinente y rellenar los campos pertinentes en Azure Sentinel.

Por ejemplo:

- **Si ha usado Logstash**, use el complemento de filtro [Grok](https://www.elastic.co/guide/en/logstash/current/plugins-filters-grok.html) para analizar los datos.
- **Si ha usado una instancia de Azure Functions**, analice los datos con código. Para obtener más información, consulte los [Analizadores](normalization.md#parsers).

Azure Sentinel admite el análisis en el tiempo de consulta. El análisis en tiempo de consulta permite introducir datos en el formato original y, a continuación, analizar a petición, cuando sea necesario.

El análisis en el tiempo de consulta también significa que no es necesario conocer la estructura exacta de los datos con anterioridad, cuando se crea el conector personalizado, incluso tampoco la información que tiene que extraer. En su lugar, analice los datos en cualquier momento, incluso durante una investigación.

> [!NOTE]
> La actualización del analizador también se aplica a los datos que ya ha ingerido en Azure Sentinel.
> 
## <a name="next-steps"></a>Pasos siguientes

Use los datos ingeridos en Azure Sentinel para proteger su entorno con cualquiera de los siguientes procesos:

- [Obtención de visibilidad sobre las alertas](quickstart-get-visibility.md)
- [Visualizar y supervisar los datos](tutorial-monitor-your-data.md)
- [Investigación de incidentes](tutorial-investigate-cases.md)
- [Detectar amenazas](tutorial-detect-threats-built-in.md)
- [Prevención automática de amenazas](tutorial-respond-threats-playbook.md)
- [Búsqueda de amenazas](hunting.md)
