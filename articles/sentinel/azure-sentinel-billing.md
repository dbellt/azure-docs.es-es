---
title: Costos y facturación de Azure Sentinel | Microsoft Docs
description: Obtenga información sobre la estimación y administración de los costos de Azure Sentinel y su modelo de precios, además de sobre sus costos y facturas.
services: sentinel
author: batamig
ms.author: bagol
ms.service: azure-sentinel
ms.topic: conceptual
ms.date: 06/03/2021
ms.openlocfilehash: 0d9c016741fcdb36a2803eaf9705e9dc2ea1782b
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111950307"
---
# <a name="azure-sentinel-costs-and-billing"></a>Costos y facturación de Azure Sentinel

Azure Sentinel usa un amplio lenguaje de consulta para obtener y analizar información a partir de grandes volúmenes de datos operativos en cuestión de segundos e interactuar con ella. Azure Sentinel almacena los datos para analizarlos en áreas de trabajo de Log Analytics de Azure Monitor.

Cuando Azure Sentinel se habilita en un área de trabajo de Log Analytics, analiza automáticamente todos los datos que ingiere el área de trabajo y factura el volumen de datos que el área de trabajo ingiere y almacena. En este artículo se describe de qué maneras puede supervisar y comprender el uso y los costos de Azure Sentinel, y las áreas de trabajo de Log Analytics asociadas y como ahorrar en ello.

## <a name="azure-sentinel-pricing-model"></a>Modelo de precios de Azure Sentinel

Azure Sentinel ofrece un modelo de precios flexible y predecible. Para más información, consulte la [página de precios de Azure Sentinel](https://azure.microsoft.com/pricing/details/azure-sentinel/). Para ver los cargos de Log Analytics relacionados, consulte los [precios de Log Analytics de Azure Monitor](https://azure.microsoft.com/pricing/details/log-analytics/).

### <a name="pay-as-you-go-and-commitment-tiers"></a>Pago por uso y Niveles de compromiso

Hay dos maneras de pagar el servicio de Azure Sentinel: Pago por uso y Niveles de compromiso.

Pago por uso es el modelo predeterminado, basado en el volumen real de datos almacenados y, de forma opcional, para la retención de datos más allá de 90 días. El volumen de datos se mide en GB (10^9 bytes).

Log Analytics y Azure Sentinel también tienen el sistema de precios "Nivel de compromiso", anteriormente denominado Reservas de capacidad, que es más predecible y ahorra hasta un 65 % en comparación con el sistema de precios de Pago por uso. Con el sistema de precios de nivel de compromiso, puede adquirir un compromiso a partir de 100 GB al día. Cualquier uso por encima del nivel de compromiso se factura según la tarifa del nivel de compromiso que haya seleccionado. Por ejemplo, un nivel de compromiso de 100 GB al día le factura por el volumen de datos de 100 GB al día con el que se ha comprometido, y cualquier GB por día adicional se le factura a la tarifa con descuento para ese nivel.

Puede aumentar el nivel de compromiso en cualquier momento y reducirlo cada 31 días para optimizar los costos a medida que aumenta o disminuye el volumen de datos. Para ver su plan de tarifa actual de Azure Sentinel, seleccione **Configuración** en el panel de navegación izquierdo de Azure Sentinel y, a continuación, seleccione la pestaña **Precios**. El plan de tarifa actual está marcado como **Nivel actual**.

Para establecer y cambiar el nivel de compromiso, consulte [Establecer o cambiar el plan de tarifa](#set-or-change-pricing-tier).

### <a name="costs-for-other-services"></a>Costos de otros servicios

Azure Sentinel se integra con muchos otros servicios de Azure para proporcionar funcionalidades mejoradas. Estos servicios son: Azure Logic Apps, Azure Notebooks y la plataforma "Aporte sus propios modelos de aprendizaje automático" (BYOML). Algunos de estos servicios pueden tener cargos adicionales. Algunos de los conectores y soluciones de datos de Azure Sentinel usan Azure Functions para la ingesta de datos, que también tiene un costo asociado aparte.

Para más información sobre los precios de estos servicios, consulte los siguientes documentos:

- [Precios de Automation-Logic Apps](https://azure.microsoft.com/pricing/details/logic-apps/)
- [Precios de Notebooks](https://azure.microsoft.com/pricing/details/machine-learning/)
- [Precios de BYOML](https://azure.microsoft.com/pricing/details/machine-learning-studio/)
- [Precios de Azure Functions](https://azure.microsoft.com/en-us/pricing/details/functions/)

Cualquier otro servicio que use podría tener costos asociados.

### <a name="data-retention-costs"></a>Costos de retención de datos

Después de habilitar Azure Sentinel en un área de trabajo de Log Analytics, puede conservar todos los datos ingeridos en el área de trabajo sin cargo alguno durante los primeros 90 días. La retención de datos más allá de 90 días se cobra según los precios estándar de [retención de Log Analytics](https://azure.microsoft.com/pricing/details/monitor/).

Puede especificar diferentes valores de retención para tipos de datos individuales. Para más información, consulte [Retención por tipo de datos](../azure-monitor/logs/manage-cost-storage.md#retention-by-data-type).

### <a name="free-trial"></a>Evaluación gratuita

Puede habilitar Azure Sentinel en un área de trabajo de Log Analytics nueva o existente sin costo adicional durante los primeros 31 días. Los cargos relacionados con Log Analytics, Automation y BYOML también se aplican durante la evaluación gratuita. La utilización más allá de los primeros 31 días se cobra según los [precios de Azure Sentinel](https://azure.microsoft.com/pricing/details/azure-sentinel). 

### <a name="free-data-sources"></a>Orígenes de datos gratuitos

Los siguientes orígenes de datos son gratuitos con Azure Sentinel:

- Registros de actividad de Azure.
- Registros de auditoría de Office 365, incluidas todas las actividades de SharePoint, la actividad del administrador de Exchange, y Teams.
- Alertas de Microsoft Defender, incluidas las alertas de Azure Defender, Microsoft 365 Defender, Microsoft Defender para Office 365, Microsoft Defender para Identity y Microsoft Defender para Endpoint.
- Alertas de Azure Security Center y Microsoft Cloud App Security (MCAS). Sin embargo, los registros sin procesar de algunos tipos de datos de Microsoft 365 Defender, MCAS, Azure Active Directory (Azure AD) y Azure Information Protection (AIP) son de pago.

En la tabla siguiente se enumeran los orígenes de datos gratuitos que puede habilitar en Azure Sentinel. Algunos de los conectores de datos, como Microsoft 365 Defender y MCAS, incluyen tanto tipos de datos gratuitos como de pago.

| Conector de datos de Azure Sentinel   | Tipo de datos | Gratuitos o de pago |
|-------------------------------------|--------------------------------|------------------|
| **Registros de actividad de Azure**         | AzureActivity                  | Gratuito             |
| **Office 365**                     | OfficeActivity (SharePoint)    | Gratuito|
|| OfficeActivity (Exchange)|Gratuito|
|| OfficeActivity (Teams)          | Gratuito|
| **Azure Defender**                  | SecurityAlert (ASC)             | Gratuito             |
| **Azure Defender para IoT**          | SecurityAlert (ASC para IoT)     | Gratuito             |
| **Microsoft 365 Defender**          | SecurityIncident | Gratuito|
||SecurityAlert| Gratuito|
||DeviceEvents                    | De pago|
||DeviceFileEvents                | De pago|
||DeviceImageLoadEvents           | De pago|
||DeviceInfo                      | De pago|
||DeviceLogonEvents               | De pago|
||DeviceNetworkEvents             | De pago|
||DeviceNetworkInfo               | De pago|
||DeviceProcessEvents             | De pago|
||DeviceRegistryEvents            | De pago|
||DeviceFileCertificateInfo       | De pago|
| **Microsoft Defender para punto de conexión** | SecurityAlert (MDATP)          | Gratuito             |
| **Microsoft Defender for Identity** | SecurityAlert (AATP)           | Gratuito             |
| **Microsoft Cloud App Security**   | SecurityAlert (MCAS)           | Gratuito             |
||McasShadowItReporting           | De pago|

En el caso de los conectores de datos que incluyen tipos de datos gratuitos y de pago, puede seleccionar los tipos de datos que desea habilitar.

![Instantánea que muestra la página del conector de datos para MCAS, con las alertas de seguridad gratuitas seleccionadas y el MCASShadowITReporting de pago sin seleccionar.](media/billing/data-types.png)

## <a name="estimate-azure-sentinel-costs"></a>Haga una estimación de los costos de Azure Sentinel

Si aún no dispone de Azure Sentinel, puede usar la [calculadora de precios de Azure Sentinel](https://azure.microsoft.com/pricing/calculator/?service=azure-sentinel) para hacer una estimación del costo potencial que tendría usar Azure Sentinel. Escriba *Azure Sentinel* en el cuadro de búsqueda y seleccione el icono Azure Sentinel que aparece. La calculadora de precios le ayuda a hacer una estimación de sus costos en función de la retención y la ingesta de datos previstos.

Por ejemplo, puede especificar cuántos GB de datos diarios prevé ingerir en Azure Sentinel y la región para el área de trabajo. La calculadora proporciona el costo mensual agregado de estos componentes:

- Ingesta de datos de Log Analytics 
- Análisis de datos de Azure Sentinel 
- Retención de datos de Log Analytics 

## <a name="manage-azure-sentinel-costs"></a>Administración de los costos de Azure Sentinel

Hay varias maneras de comprender y administrar la utilización y los costos de Azure Sentinel.

Administrar la ingesta y retención de datos:

- [Use los precios del nivel de compromiso para optimizar los costos](#set-or-change-pricing-tier) en función del volumen de ingesta de datos.
- [Defina un límite de volumen de datos de Log Analytics](#define-a-data-volume-cap-in-log-analytics) para administrar la ingesta, aunque los datos de seguridad se excluyen del límite.
- [Optimice los costos de Log Analytics con clústeres dedicados](#optimize-log-analytics-costs-with-dedicated-clusters).
- [Coloque los datos que no están relacionados con la seguridad en un área de trabajo diferente](#separate-non-security-data-in-a-different-workspace).
- [Reduzca los costos de retención de datos a largo plazo con Azure Data Explorer (ADX)](#reduce-long-term-data-retention-costs-with-adx).

Comprender, supervisar y alertar sobre la ingesta de datos y los cambios de costos:

- [Ejecute consultas para comprender la ingesta de datos](#run-queries-to-understand-your-data-ingestion).
- [Implemente un libro para visualizar la ingesta de datos](#deploy-a-workbook-to-visualize-data-ingestion).
- [Use un cuaderno de estrategias de administración de costos](#use-a-playbook-for-cost-management-alerts) que pueda enviar una alerta cuando la ingesta supere un umbral definido previamente.
- [Comprenda la ingesta de datos de formato de evento común (CEF)](#understand-cef-ingestion-volume).

### <a name="manage-data-ingestion-and-retention"></a>Administrar la ingesta y retención de datos

Use los métodos siguientes para administrar la ingesta y retención de datos para el área de trabajo de Azure Sentinel.

#### <a name="set-or-change-pricing-tier"></a>Establecer o cambiar el plan de tarifa

Para lograr el mayor ahorro, supervise el volumen de ingesta y asegúrese de que tiene el nivel de compromiso que mejor se adapta a sus patrones de volumen de ingesta. Puede aumentar o disminuir el nivel de compromiso para adaptarse a los cambios de los volúmenes de datos.

Puede aumentar el nivel de compromiso en cualquier momento, que reinicia el período de compromiso de 31 días. Sin embargo, para volver al plan de pago por uso o a un nivel de compromiso inferior, debe esperar a que finalice el período de compromiso de 31 días. La facturación de los niveles de compromiso se realiza a diario.

Para ver su plan de tarifa actual de Azure Sentinel, seleccione **Configuración** en el panel de navegación izquierdo de Azure Sentinel y, a continuación, seleccione la pestaña **Precios**. El plan de tarifa actual está marcado como **Nivel actual**.

Para cambiar el compromiso de su plan de tarifa, seleccione uno de los otros niveles en la página de precios y, a continuación, seleccione **Aplicar**. Debe tener el rol de **Colaborador** o **Propietario** en Azure Sentinel para cambiar el plan de tarifa.

![Instantánea que muestra la página de precios en la Configuración de Azure Sentinel, con la opción de Pago por uso como plan de tarifa actual.](media/billing/pricing.png)

> [!NOTE]
> Los volúmenes de ingesta de datos de Azure Sentinel aparecen en **Información de seguridad** en gráficos de utilización de algunos portales.

Los planes de tarifa de Azure Sentinel no incluyen cargos de Log Analytics. Para cambiar el compromiso de su plan de tarifa para Log Analytics, consulte [Cambio del plan de tarifa](../azure-monitor/logs/manage-cost-storage.md#changing-pricing-tier).

#### <a name="define-a-data-volume-cap-in-log-analytics"></a>Definir un límite de volumen de datos en Log Analytics

En Log Analytics, puede habilitar un límite de volumen diario que restrinja la ingesta diaria del área de trabajo. El límite diario puede ayudarle a administrar aumentos inesperados en el volumen de datos, mantenerse dentro del límite y restringir los cargos no previstos.

Para definir un límite de volumen diario, seleccione **Utilización y costos estimados** en el panel de navegación izquierdo del área de trabajo de Log Analytics y, a continuación, seleccione **Límite diario**. Seleccione **Activado**, escriba una cantidad máxima de volumen diaria y, a continuación, seleccione **Aceptar**.

![Instantánea que muestra la pantalla de Utilización y costos estimados y la ventana de Límite diario.](media/billing/daily-cap.png)

La pantalla **Uso y costos estimados** también muestra la tendencia del volumen de datos ingeridos en los últimos 31 días y el volumen total de datos retenidos.

> [!IMPORTANT]
> El límite diario no restringe la recopilación de todos los tipos de datos. Para más información sobre cómo administrar el límite diario en Log Analytics, consulte [Administración del volumen de datos diario máximo](../azure-monitor/logs/manage-cost-storage.md#manage-your-maximum-daily-data-volume).

#### <a name="optimize-log-analytics-costs-with-dedicated-clusters"></a>Optimice los costos de Log Analytics con clústeres dedicados

Si ingiere al menos 1 TB al día en el área de trabajo o las áreas de trabajo de Azure Sentinel de la misma región, considere la posibilidad de pasar a un clúster dedicado de Log Analytics para reducir los costos. Un nivel de compromiso de clúster dedicado de Log Analytics agrega el volumen de datos en áreas de trabajo que ingieren colectivamente un total de 1 TB al día o más.

Los clústeres dedicados de Log Analytics no se aplican a los niveles de compromiso de Azure Sentinel. Los costos de Azure Sentinel se siguen aplicando por área de trabajo en el clúster dedicado.

Puede agregar varias áreas de trabajo de Azure Sentinel a un clúster dedicado de Log Analytics. Hay un par de ventajas en usar un clúster dedicado de Log Analytics para Azure Sentinel:

- Las consultas entre áreas de trabajo se ejecutan más rápido si todas las áreas de trabajo implicadas en la consulta están en el clúster dedicado. Sigue siendo mejor tener el menor número de áreas de trabajo posible en el entorno y un clúster dedicado que conserva el [límite de 100 áreas de trabajo](../azure-monitor/logs/cross-workspace-query.md) para incluirlas en una sola consulta entre áreas de trabajo.

- Todas las áreas de trabajo del clúster dedicado pueden compartir el nivel de compromiso de Log Analytics establecido en el clúster. No tener que confirmar niveles de compromiso de Log Analytics independientes para cada área de trabajo puede permitir ahorrar costos y eficiencias. Al habilitar un clúster dedicado, confirma un nivel de compromiso mínimo de Log Analytics de ingesta de 1 TB al día.

Estas son otras consideraciones para pasar a un clúster dedicado para la optimización de costos:

- El número máximo de clústeres por región y suscripción es dos.
- Todas las áreas de trabajo vinculadas a un clúster deben estar en la misma región.
- El número máximo de áreas de trabajo vinculadas a un clúster es 1000.
- Puede desvincular un área de trabajo vinculada de su clúster. El número de operaciones de vinculación en un área de trabajo determinada en un período de 30 días se limita a dos.
- No puede mover un área de trabajo existente a un clúster de claves administradas por el cliente (CMK). Debe crear el área de trabajo en el clúster.
- Actualmente no se admite el traslado de un clúster a otro grupo de recursos o a otra suscripción.
- Se produce un error en un vínculo de área de trabajo a un clúster si el área de trabajo está vinculada a otro clúster.

Para más información sobre los clústeres dedicados, consulte [Clústeres dedicados de Log Analytics](../azure-monitor/logs/manage-cost-storage.md#log-analytics-dedicated-clusters).

#### <a name="separate-non-security-data-in-a-different-workspace"></a>Coloque los datos que no están relacionados con la seguridad en un área de trabajo diferente

Azure Sentinel analiza todos los datos ingeridos en áreas de trabajo de Log Analytics habilitadas para Azure Sentinel. Es mejor tener un área de trabajo independiente para los datos de operaciones no relacionados con la seguridad, para garantizar que no incurran en costos de Azure Sentinel.

Al buscar o investigar amenazas en Azure Sentinel, es posible que tenga que acceder a los datos operativos almacenados en estas áreas de trabajo de Azure Log Analytics independientes. Puede acceder a estos datos mediante consultas entre áreas de trabajo en la experiencia de exploración de registros y los libros. Sin embargo, no puede usar reglas de análisis entre áreas de trabajo y consultas de búsqueda a menos Azure Sentinel esté habilitado en todas las áreas de trabajo.

#### <a name="reduce-long-term-data-retention-costs-with-adx"></a>Reducir los costos de retención de datos a largo plazo con ADX

La retención de datos de Azure Sentinel es gratuita durante los primeros 90 días. Para ajustar el período de tiempo de retención de datos en Log Analytics, seleccione **Utilización y costos estimados** en el panel de navegación izquierdo, seleccione **Retención de datos** y, después, ajuste el control deslizante.

Los datos de seguridad de Azure Sentinel podrían perder parte de su valor después de unos meses. Es posible que los usuarios del centro de operaciones de seguridad (SOC) no necesiten acceder a los datos más antiguos con tanta frecuencia como los datos más recientes, pero puede que necesiten acceder a los datos con fines de auditoría o investigaciones esporádicas. Para reducir los costos de retención de datos de Azure Sentinel, puede usar Azure Data Explorer para la retención de datos a largo plazo a un costo menor. ADX proporciona el saldo adecuado de costo y facilidad de uso para datos antiguos que ya no necesitan inteligencia de seguridad de Azure Sentinel.

Con ADX, puede almacenar datos a un precio más bajo, pero seguir explorando los datos con las mismas consultas del lenguaje de consulta Kusto (KQL) que en Azure Sentinel. También puede usar la característica de proxy ADX para realizar consultas multiplataforma. Estas consultas agregan y correlacionan datos distribuidos en ADX, Application Insights, Azure Sentinel y Log Analytics.

Para más información, consulte [Integración de Azure Data Explorer para la retención de registros a largo plazo](store-logs-in-azure-data-explorer.md).

### <a name="understand-monitor-and-alert-for-changes-in-data-ingestion-and-costs"></a>Comprender, supervisar y alertar sobre los cambios en la ingesta de datos y los costos

Use los métodos siguientes para comprender, supervisar y alertar sobre los cambios en el área de trabajo de Azure Sentinel.

#### <a name="run-queries-to-understand-your-data-ingestion"></a>Ejecute consultas para comprender la ingesta de datos

Estas son algunas consultas que puede usar para comprender el volumen de ingesta de datos.

Ejecute la siguiente consulta para mostrar el volumen de ingesta de datos por solución: 

```kusto
Usage
| where StartTime >= startofday(ago(31d)) and EndTime < startofday(now())
| where IsBillable == true
| summarize BillableDataGB = sum(Quantity) / 1000. by bin(StartTime, 1d), Solution
| extend Solution = iif(Solution == "SecurityInsights", "AzureSentinel", Solution)
| render columnchart
```

Ejecute la siguiente consulta para mostrar el volumen de ingesta de datos por tipo de datos: 

```kusto
Usage
| where StartTime >= startofday(ago(31d)) and EndTime < startofday(now())
| where IsBillable == true
| summarize BillableDataGB = sum(Quantity) / 1000. by bin(StartTime, 1d), DataType
| render columnchart
```
Ejecute la siguiente consulta para mostrar el volumen de ingesta de datos por solución y por tipo de datos: 

```kusto
Usage
| where TimeGenerated > ago(32d)
| where StartTime >= startofday(ago(31d)) and EndTime < startofday(now())
| where IsBillable == true
| summarize BillableDataGB = sum(Quantity) by Solution, DataType
| extend Solution = iif(Solution == "SecurityInsights", "AzureSentinel", Solution)
| sort by Solution asc, DataType asc
```

#### <a name="deploy-a-workbook-to-visualize-data-ingestion"></a>Implemente un libro para visualizar la ingesta de datos

El **libro sobre el informe de utilización del área de trabajo** proporciona las estadísticas de consumo, costo y utilización de datos del área de trabajo. El libro proporciona el estado de ingesta de datos del área de trabajo y la cantidad de datos gratuitos y facturables. Puede usar la lógica del libro para supervisar los datos ingeridos y costos, y crear vistas personalizadas y alertas basadas en reglas.

Este libro también proporciona detalles de ingesta granulares. El libro desglosa los datos del área de trabajo por tabla de datos y proporciona volúmenes por tabla y entrada para ayudarle a comprender mejor los patrones de ingesta.

Para habilitar el libro de informe de uso del área de trabajo, haga lo siguiente:

1. En el panel de navegación izquierdo de Azure Sentinel, seleccione **Administración de amenazas** > **Libros**.
1. Escriba *uso del área de trabajo* en la barra de búsqueda y, a continuación, seleccione **Informe de uso del área de trabajo**.
1. Seleccione **Ver plantilla** para usar el libro tal cual, o bien seleccione **Guardar** para crear una copia modificable del libro. Si guarda una copia, seleccione **Ver libro guardado**.
1. En el libro, seleccione los valores de **Suscripción** y **Área de trabajo** que desea ver y, a continuación, establezca **TimeRange** en el período de tiempo que desea ver. Puede establecer el botón de alternancia **Mostrar ayuda** en **Sí** para mostrar explicaciones en contexto en el libro.

#### <a name="use-a-playbook-for-cost-management-alerts"></a>Use un cuaderno de estrategias para alertas de administración de costos

Para ayudarle a controlar el presupuesto de Azure Sentinel, puede crear un cuaderno de estrategias de administración de costos. El cuaderno de estrategias le envía una alerta si su área de trabajo de Azure Sentinel supera el presupuesto que defina dentro de un período de tiempo determinado.

La comunidad GitHub de Azure Sentinel proporciona el cuaderno de estrategias de administración de costos [Send-IngestionCostAlert](https://github.com/iwafula025/Azure-Sentinel/tree/master/Playbooks/Send-IngestionCostAlert) en GitHub. Este cuaderno de estrategias se activa mediante un desencadenador de periodicidad y proporciona un alto nivel de flexibilidad. Puede controlar la frecuencia de ejecución, el volumen de ingesta y el mensaje que se va a desencadenar, según sus requisitos.

#### <a name="understand-cef-ingestion-volume"></a>Comprender el volumen de ingesta de CEF

CEF es un formato de eventos de Syslog compatible de Azure Sentinel. Puede usar CEF para traer información de seguridad valiosa de una variedad de orígenes al área de trabajo de Azure Sentinel. Los registros CEF llegan a la tabla CommonSecurityLog en Azure Sentinel, que incluye todos los campos CEF estándar actualizados.

Muchos dispositivos y orígenes de datos permiten registrar campos más allá del esquema CEF estándar. Estos campos adicionales llegan a la tabla AdditionalExtensions. Estos campos podrían tener volúmenes de ingesta más altos que los campos CEF estándar, ya que el contenido del evento dentro de estos campos puede ser variable.

## <a name="understand-your-azure-sentinel-costs-and-bill"></a>Comprender los costos y la facturación de Azure Sentinel

Es importante comprender y realizar un seguimiento de los costos de Azure Sentinel. El centro [Azure Cost Management + Billing](../cost-management-billing/costs/quick-acm-cost-analysis.md) proporciona una funcionalidad útil. Después de abrir **Cost Management + Billing** en Azure Portal, seleccione **Cost Management** en el panel de navegación izquierdo y, a continuación, seleccione el [ámbito](..//cost-management-billing/costs/understand-work-scopes.md) o el conjunto de recursos que desea investigar, como una suscripción o un grupo de recursos de Azure.

Para ver la factura de Azure, seleccione **Análisis de costos** en el panel de navegación izquierdo de **Cost Management + Billing**. En la pantalla **Análisis de costos**, seleccione el cuadro de diálogo desplegable en el campo **Ver** y seleccione **Detalles de la factura**.

> [!NOTE]
> Los costos que se muestran en esta imagen son solo para fines de ejemplo. No están diseñados para reflejar los costos reales.

![Instantánea que muestra la sección de Azure Sentinel de una factura de ejemplo de Azure.](media/billing/sample-bill.png)

Los cargos de Azure Sentinel y Log Analytics aparecen como elementos de línea independientes en la factura de Azure, basados en el plan de precios seleccionado. Si supera la utilización del nivel de compromiso del área de trabajo en un mes determinado, la factura de Azure muestra un elemento de línea para el nivel de compromiso con su costo fijo asociado y un elemento de línea independiente para la ingesta más allá del nivel de compromiso, facturado a la misma tarifa del nivel de compromiso.

En la tabla siguiente se muestran los costos de Azure Sentinel y Log Analytics en las columnas **Nombre del servicio** y **Medidor** de la factura de Azure:

|Coste|Nombre del servicio|Medidor|
|----|------------|----------------|
|Nivel de compromiso de Azure Sentinel|**sentinel**|**nivel de compromiso de `n` gb**|
|Nivel de compromiso de Log Analytics|**azure monitor**|**nivel de compromiso de `n` gb**|
|Uso por encima del límite del nivel de compromiso de Azure Sentinel o Pago por uso|**sentinel**|**análisis**|
|Uso por encima del límite del nivel de compromiso de Log Analytics o Pago por uso|**log analytics**|**ingesta de datos**|

Para más información sobre cómo ver y descargar la factura de Azure, consulte [Información de costos y facturación de Azure](../cost-management-billing/understand/download-azure-daily-usage.md).

La pantalla **Análisis de costos** también muestra vistas detalladas de la utilización y costos de Azure, con la opción de aplicar una variedad de controles y filtros.

Por ejemplo, para ver gráficos de los costos diarios de un período de tiempo determinado, haga lo siguiente:
1. Seleccione el símbolo de intercalación desplegable en el campo **Ver** y seleccione **Costos acumulados** o **Costos diarios**.
1. Seleccione el símbolo de intercalación desplegable en el campo de fecha y seleccione un intervalo de fechas.
1. Seleccione el símbolo de intercalación desplegable situado junto a **Granularidad** y seleccione **Diariamente**.

> [!NOTE]
> Los costos que se muestran en esta imagen son solo para fines de ejemplo. No están diseñados para reflejar los costos reales.

![Instantánea que muestra una pantalla de análisis de costos de Cost Management + Billing.](media/billing/cost-management.png)

También puede aplicar más controles. Por ejemplo, para ver solo los costos asociados con Azure Sentinel, seleccione **Agregar filtro**, seleccione **Nombre de servicio** y, a continuación, seleccione los nombres de servicio **sentinel**, **log analytics** y **azure monitor**.

## <a name="next-steps"></a>Pasos siguientes
Para obtener más sugerencias sobre cómo reducir el volumen de datos de Log Analytics, consulte [Sugerencias para reducir el volumen de datos](../azure-monitor/logs/manage-cost-storage.md#tips-for-reducing-data-volume).