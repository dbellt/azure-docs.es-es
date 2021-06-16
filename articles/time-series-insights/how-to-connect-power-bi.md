---
title: 'Conexión de un entorno a Power BI: Azure Time Series Insights | Microsoft Docs'
description: Aprenda a conectar Azure Time Series Insights a Power BI para compartir, representar y mostrar datos en toda una organización.
author: shreyasharmamsft
ms.author: shresha
manager: cnovak
ms.reviewer: orspodek
services: time-series-insights
ms.service: time-series-insights
ms.topic: conceptual
ms.date: 12/14/2020
ms.openlocfilehash: cb9f0531c7f4d0fc865e9e832a71065220d3805b
ms.sourcegitcommit: 6323442dbe8effb3cbfc76ffdd6db417eab0cef7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110615702"
---
# <a name="visualize-data-from-azure-time-series-insights-in-power-bi"></a>Visualización de datos de Azure Time Series Insights en Power BI

Azure Time Series Insights es una plataforma para almacenar, administrar, consultar y visualizar datos de series temporales en la nube. [Power BI](https://powerbi.microsoft.com) es una herramienta de análisis de negocio con funcionalidades de visualización enriquecidas que permiten compartir información y resultados en toda la organización. Ahora se pueden integrar ambos servicios, lo que permite aumentar el eficaz análisis de Azure Time Series Insights con las sólidas funcionalidades de visualización de datos y facilidad de uso compartido de Power BI.

Aprenderá a:

* Conectar Azure Time Series Insights a Power BI mediante el conector nativo de Time Series Insights.
* Crear objetos visuales con datos de una serie temporal en Power BI.
* Publicar el informe en Power BI y compartirlo con el resto de la organización.

## <a name="prerequisites"></a>Prerrequisitos

* Regístrese para obtener [suscripción gratuita a Azure](https://azure.microsoft.com/free/), en caso de que no tenga ninguna.
* Descargar e instalar la versión más reciente de [Power BI Desktop](https://powerbi.microsoft.com/downloads/)
* Tener o crear un [entorno de Azure Time Series Insights Gen2](./how-to-provision-manage.md)

Examine las [directivas de acceso del entorno](./concepts-access-policies.md) y asegúrese de que tiene acceso directo o acceso de invitado al entorno de Azure Time Series Insights Gen2.

> [!IMPORTANT]
> Descargue e instale la versión más reciente de [Power BI Desktop](https://powerbi.microsoft.com/downloads/). Para seguir los pasos de este artículo, asegúrese de que tiene instalada al menos la versión de diciembre 2020 (2.88.321.0) de Power BI Desktop.

## <a name="export-data-from-azure-time-series-insights-into-power-bi-desktop"></a>Exportación de datos de Azure Time Series Insights a Power BI Desktop

Primeros pasos:

1. Abra el Explorador de Azure Time Series Insights Gen2 y mantenga los datos. Estos son los datos que se van a exportar en Power BI.
1. Una vez que haya creado una vista con la que esté conforme, vaya al menú desplegable **Más acciones** y seleccione **Conectar con Power BI**.

    [Exportación del Explorador de Azure Time Series Insights Gen2![](media/how-to-connect-power-bi/export-from-explorer.jpg)](media/how-to-connect-power-bi/export-from-explorer.jpg#lightbox)

1. Establezca los parámetros de la exportación:

   * **Formato de datos**: Elija si desea exportar **datos agregados** o **eventos sin formato** a Power BI.

       > [!NOTE]
       > Si exporta eventos sin formato, puede agregar esos datos más adelante en Power BI. Sin embargo, si exporta datos agregados, no podrá revertirlos a datos sin procesar en Power BI. Hay un límite de recuento de eventos de 250 000 para los datos de nivel de evento sin formato.

   * **Intervalo de tiempo**: Elija si desea ver un intervalo de tiempo **fijo** o los datos **más recientes**  en Power BI. La elección del intervalo de tiempo fijo implica que los datos del intervalo de búsqueda que ha incluido en el gráfico se exportarán a Power BI. La elección del intervalo de tiempo más reciente implica que Power BI va a obtener los datos más recientes para el intervalo de búsqueda que ha elegido (por ejemplo, si crea un gráfico con una hora de datos y elige el valor "más reciente", el conector de Power BI siempre realizará consultas de la última hora de datos).

   * **Tipo de almacén**: Elija si desea ejecutar la consulta seleccionada en **Almacenamiento intermedio** o en **Almacenamiento en reposo**. Si ha seleccionado un intervalo que abarca los almacenes en frío e intermedio, la consulta se enrutará al almacén en frío de forma predeterminada, ya que el almacén intermedio solo contendrá los datos más recientes. Se permite cambiar manualmente el parámetro storeType, pero, para una mejor experiencia, no se recomienda.

    > [!TIP]
    > El Explorador de Azure Time Series Insights seleccionará automáticamente los parámetros recomendados en función del intervalo de búsqueda y la vista de los datos que haya elegido exportar.

1. Una vez que haya realizado la configuración, seleccione **Copiar consulta en el Portapapeles**.

    [![Modal de exportación del Explorador de Azure Time Series Insights](media/how-to-connect-power-bi/choose-explorer-parameters.jpg)](media/how-to-connect-power-bi/choose-explorer-parameters.jpg#lightbox)

1. Inicie Power BI Desktop.

1. En Power BI Desktop, en la pestaña **Inicio**, seleccione **Obtener datos** en la esquina superior izquierda y luego **Más**.

    [![Obtención de datos en Power BI](media/how-to-connect-power-bi/get-data-power-bi.jpg)](media/how-to-connect-power-bi/get-data-power-bi.jpg#lightbox)

1. Busque **Azure Time Series Insights**, seleccione **Azure Time Series Insights (Beta)** y, después, **Conectar**.

    [![Conexión de Power BI a Azure Time Series Insights](media/how-to-connect-power-bi/select-tsi-connector.jpg)](media/how-to-connect-power-bi/select-tsi-connector.jpg#lightbox)

    También puede ir a la pestaña **Azure**, seleccionar **Azure Time Series Insights (Beta)** y, después, **Conectar**.

1. Pegue la consulta que copió del Explorador de Time Series Insights en el campo **Consulta personalizada** y presione **Aceptar**.

    [![Pegar la consulta personalizada y seleccionar Aceptar](media/how-to-connect-power-bi/custom-query-load.png)](media/how-to-connect-power-bi/custom-query-load.png#lightbox)

1. Se cargará la tabla de datos. Pulse **Cargar** para realizar la carga en Power BI. Para realizar cualquier transformación en los datos, haga clic en **Transformar datos**. Los datos también se pueden transformar los datos después de cargarlos.

    [![Examinar los datos de la tabla y seleccionar Cargar](media/how-to-connect-power-bi/review-the-loaded-data-table.png)](media/how-to-connect-power-bi/review-the-loaded-data-table.png#lightbox)

## <a name="create-a-report-with-visuals"></a>Crear un informe con objetos visuales

Ahora que ha importado los datos en Power BI, es el momento de crear un informe con objetos visuales.

1. En el lado izquierdo de la ventana, asegúrese de que ha seleccionado la vista **Informe**.

   [![Captura de pantalla que muestra el icono de la vista Informe.](media/how-to-connect-power-bi/select-the-report-view.png)](media/how-to-connect-power-bi/select-the-report-view.png#lightbox)

1. En la columna **Visualizaciones**, seleccione el objeto visual que prefiera. Por ejemplo, seleccione **Gráfico de líneas**. Se agregará un gráfico de líneas en blanco al lienzo.

1. En la lista **Campos**, seleccione **Marca de tiempo** y arrástrelo al campo **Eje** para ver el tiempo en el eje X. Asegúrese de cambiar a **_Marca de tiempo** como el valor de **Eje** (el valor predeterminado es **Jerarquía de fechas**).

   [![Seleccionar _Marca de tiempo](media/how-to-connect-power-bi/select-timestamp.png)](media/how-to-connect-power-bi/select-timestamp.png#lightbox)

1. En la lista **Campos**, seleccione la variable que desea representar y arrástrela hasta el campo **Valores** para mostrar los valores en el eje Y. Seleccione el valor de Id. de serie temporal y arrástrelo hasta el campo **Leyenda** para crear varias líneas en el gráfico, una por cada identificador de serie temporal. Esto representará una vista similar a la que proporciona el Explorador de Azure Time Series Insights.

   [![Crear un gráfico de líneas básico](media/how-to-connect-power-bi/power-bi-line-chart.png)](media/how-to-connect-power-bi/power-bi-line-chart.png#lightbox)

1. Para agregar otro gráfico al lienzo, seleccione cualquier lugar del lienzo fuera del gráfico de líneas y repita este proceso.

   [![Crear gráficos adicionales para compartir](media/how-to-connect-power-bi/power-bi-additional-charts.png)](media/how-to-connect-power-bi/power-bi-additional-charts.png#lightbox)

Una vez que haya creado el informe, puede publicarlo en Reporting Services de Power BI y compartirlo con otros usuarios de su organización.

## <a name="advanced-editing"></a>Edición avanzada

Si ya ha cargado un conjunto de datos en Power BI pero desea modificar la consulta (por ejemplo, los parámetros de fecha y hora o del identificador del entorno), puede hacerlo a través de la funcionalidad Editor avanzado de Power BI. Consulte la [documentación de Power BI](/power-bi/desktop-query-overview) para obtener más información sobre cómo realizar cambios mediante el **Editor de Power Query**.

## <a name="next-steps"></a>Pasos siguientes

* Más información acerca de [Power BI Desktop](/power-bi/desktop-query-overview).

* Aprenda a [consultar datos](concepts-query-overview.md) en Azure Time Series Insights Gen2.