---
title: Creación de un informe de Power BI a partir de datos de Azure Sentinel
description: Aprenda a crear un informe de Power BI mediante una consulta exportada desde Azure Sentinel Log Analytics. Comparta el informe con otros usuarios en el servicio Power BI y un canal de Teams.
author: batamig
ms.author: bagol
ms.service: azure-sentinel
ms.topic: conceptual
ms.date: 06/08/2021
ms.openlocfilehash: 1e641e878bc5370c9f9d7c0cc83be79008b2ebdd
ms.sourcegitcommit: a434cfeee5f4ed01d6df897d01e569e213ad1e6f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111852295"
---
# <a name="tutorial-create-a-power-bi-report-from-azure-sentinel-data"></a>Tutorial: creación de un informe de Power BI a partir de datos de Azure Sentinel

[Power BI](https://powerbi.microsoft.com/) es una plataforma de informes y análisis que convierte los datos en visualizaciones coherentes, envolventes e interactivas. Power BI permite conectarse con facilidad a orígenes de datos, visualizar y descubrir relaciones y compartir información con los usuarios que desee.

Puede basar los informes de Power BI en datos de áreas de trabajo de Azure Sentinel Log Analytics y compartir esos informes con usuarios que no tienen acceso a Azure Sentinel. Por ejemplo, es posible que quiera compartir información sobre los intentos de inicio de sesión con errores con los propietarios de la aplicación sin concederles acceso a Azure Sentinel. Las visualizaciones de Power BI pueden proporcionar los datos de un vistazo.

En este tutorial, hizo lo siguiente:

> [!div class="checklist"]
> * Exportar una consulta de Kusto de Log Analytics a una consulta de lenguaje M de Power BI.
> * Usar la consulta M en Power BI Desktop para crear visualizaciones y un informe.
> * Publicar el informe en el servicio Power BI y compartirlo con otros usuarios.
> * Agregar el informe a un canal de Teams.

Las personas a las que se concedió acceso en el servicio Power BI y los miembros del canal de Teams pueden ver el informe sin necesidad de permisos de Azure Sentinel.

## <a name="prerequisites"></a>Prerrequisitos

Para completar este tutorial, necesita:

- Al menos, acceso de lectura a un área de trabajo de Azure Sentinel Log Analytics que supervisa los intentos de inicio de sesión.
- Una cuenta de Power BI que tenga acceso de lectura al área de trabajo de Log Analytics.
- [Power BI Desktop instalado desde Microsoft Store](https://aka.ms/pbidesktopstore).

## <a name="export-a-query-from-log-analytics"></a>Exportación de una consulta desde Log Analytics

Cree, ejecute y exporte una consulta de Kusto en el área de trabajo de Azure Sentinel Log Analytics. 

1. Para crear una consulta simple, en el área de trabajo de Azure Sentinel Log Analytics, seleccione **Registros**. En el editor de consultas en **Nueva consulta 1**, escriba la siguiente consulta de Kusto:
   
   ```kusto
   SigninLogs
   |  where TimeGenerated >ago(7d)
   | summarize Attempts = count(), Failed=countif(ResultType !=0), Succeeded = countif(ResultType ==0) by AppDisplayName
   |  top 10 by Failed
   | sort by Failed
   ```
   
   También puede usar su consulta de Kusto de Azure Sentinel Log Analytics.
   
1. Seleccione **Ejecutar** para ejecutar la consulta y generar resultados.
   
   :::image type="content" source="media/powerbi/query.png" alt-text="Captura de pantalla que muestra la consulta y los resultados de Kusto de Log Analytics.":::
   
1. Para exportar la consulta a formato de consulta M de Power BI, seleccione **Exportar** y, a continuación, seleccione **Exportar a Power BI (consulta M)** . Log Analytics exporta la consulta a un archivo de texto denominado *PowerBIQuery.txt*.
   
   :::image type="content" source="media/powerbi/export.png" alt-text="Captura de pantalla que muestra la consulta Exportar en formato M de Power BI.":::
   
1. Copie el contenido del archivo exportado.

## <a name="get-the-data-in-power-bi-desktop"></a>Obtención de datos de Power BI Desktop

Ejecute la consulta M exportada en Power BI Desktop para obtener datos.

1. Abra Power BI Desktop e inicie sesión en la cuenta de Power BI que tenga acceso de lectura al área de trabajo de Log Analytics.
   
   :::image type="content" source="media/powerbi/sign-in.png" alt-text="Captura de pantalla que muestra el inicio de sesión en Power BI Desktop.":::
   
1. En la cinta de opciones de Power BI, seleccione **Obtener datos** y, a continuación, seleccione **Consulta en blanco**. Se abre el **Editor de Power Query**.
   
   :::image type="content" source="media/powerbi/blank-query.png" alt-text="Captura de pantalla que muestra la opción Consulta en blanco seleccionada en Obtener datos en Power BI Desktop.":::
   
1. En el **Editor de Power Query**, seleccione **Editor avanzado**.
   
1. Pegue el contenido copiado del archivo *PowerBIQuery.txt* exportado en la ventana **Editor avanzado** y, a continuación, seleccione **Listo**.
   
   :::image type="content" source="media/powerbi/advanced-editor.png" alt-text="Captura de pantalla que muestra la consulta M pegada en el Editor avanzado de Power BI.":::
   
1. En el **Editor de Power Query**, cambie el nombre de la consulta a *App_signin_stats* y, a continuación, seleccione **Cerrar y aplicar**.
   
   :::image type="content" source="media/powerbi/close-apply.png" alt-text="Captura de pantalla que muestra la consulta cuyo nombre ha cambiado y el comando Cerrar y aplicar en el Editor de Power Query.":::

## <a name="create-visualizations-from-the-data"></a>Creación de visualizaciones a partir de los datos

Ahora que los datos están en Power BI, puede crear visualizaciones para proporcionar información sobre los datos.

### <a name="create-a-table-visual"></a>Creación de un objeto visual de tabla

En primer lugar, cree una tabla que muestre todos los resultados de la consulta.

1. Para agregar una visualización de tabla al lienzo de Power BI Desktop, seleccione el icono de **tabla** en **Visualizaciones**.
   
   :::image type="content" source="media/powerbi/table.png" alt-text="Captura de pantalla que muestra el icono de tabla en Visualizaciones en Power BI Desktop.":::
   
1. En **Campos**, seleccione todos los campos de la consulta para que aparezcan todos en la tabla. Si la tabla no muestra todos los datos, amplíela arrastrando sus identificadores de selección.
   
   :::image type="content" source="media/powerbi/select-fields.png" alt-text="Captura de pantalla que muestra todos los campos seleccionados para la visualización de la tabla.":::
   
### <a name="create-a-pie-chart"></a>Creación de un gráfico circular

A continuación, cree un gráfico circular que muestre qué aplicaciones han tenido más intentos de inicio de sesión con errores.

1. Para anular la selección del objeto visual de tabla, haga clic en él o pulse fuera de él y, a continuación, en **Visualizaciones**, seleccione el icono de **gráfico circular**.
   
   :::image type="content" source="media/powerbi/pie-chart.png" alt-text="Captura de pantalla que muestra el icono de gráfico circular en Visualizaciones en Power BI Desktop.":::
   
1. Seleccione **AppDisplayName** en el cuadro **Leyenda** o arrástrelo desde el panel **Campos**. Seleccione **Error** en el cuadro **Valores** o arrástrelo desde **Campos**. El gráfico circular muestra ahora el número de intentos de inicio de sesión con errores por aplicación.
   
   :::image type="content" source="media/powerbi/failed.png" alt-text="Captura de pantalla que muestra el gráfico circular con el número de intentos de inicio de sesión con errores por aplicación.":::
   
### <a name="create-a-new-quick-measure"></a>Creación de una nueva medida rápida

También se espera que se muestre el porcentaje de intentos de inicio de sesión con errores para cada aplicación. Puesto que la consulta no tiene una columna de porcentaje, puede crear una nueva medida para mostrar esta información.

1. En **Visualizaciones**, seleccione el icono de **gráfico de columnas apiladas** para crear un gráfico de columnas apiladas.
   
   :::image type="content" source="media/powerbi/column-chart.png" alt-text="Captura de pantalla que muestra el icono de gráfico de columnas apiladas en Visualizaciones en Power BI Desktop.":::
   
1. Con la nueva visualización seleccionada, seleccione **Medida rápida** en la cinta de opciones.
   
1. En la ventana **Medidas rápidas**, en **Cálculo**, seleccione **División**. Arrastre **Error** desde **Campos** al campo **Numerador** y arrastre **Intentos** desde **Campos** al **Denominador**.
   
   :::image type="content" source="media/powerbi/quick-measures.png" alt-text="Captura de pantalla que muestra la configuración en la ventana Medidas rápidas.":::
   
1. Seleccione **Aceptar**. La nueva medida aparece en el panel **Campos**.
   
1. Seleccione la nueva medida en el panel **Campos** y, en **Formato** en la cinta de opciones, seleccione **Porcentaje**.
   
   :::image type="content" source="media/powerbi/percentage.png" alt-text="Captura de pantalla que muestra la nueva medida seleccionada en el panel Campos y Porcentaje seleccionado en Formato en la cinta de opciones.":::
   
1. Con la visualización del gráfico de columnas seleccionada en el lienzo, seleccione o arrastre el campo **AppDisplayName** al área **Eje** y la nueva medida de **Errores divididos por intentos** al área **Valores**. El gráfico muestra ahora el porcentaje de intentos de inicio de sesión con errores para cada aplicación.
   
   :::image type="content" source="media/powerbi/failed-percentage.png" alt-text="Captura de pantalla que muestra el gráfico de columnas con porcentaje de intentos con errores para cada aplicación.":::
   
### <a name="refresh-the-data-and-save-the-report"></a>Actualización de los datos y guardado del informe

1. Seleccione **Actualizar** para obtener los datos más recientes de Azure Sentinel.
   
   :::image type="content" source="media/powerbi/refresh.png" alt-text="Captura de pantalla que muestra el botón Actualizar en la cinta de opciones.":::
   
1. Seleccione **Archivo** > **Guardar** y guarde el informe de Power BI.

## <a name="create-a-power-bi-online-workspace"></a>Creación de un área de trabajo en línea de Power BI

Para crear un área de trabajo de Power BI para compartir el informe:

1. Inicie sesión en [powerbi.com](https://powerbi.com) con la misma cuenta que usó para el acceso de lectura de Power BI Desktop y Azure Sentinel.
   
1. En **Áreas de trabajo**, seleccione **Crear un área de trabajo**. Asígnele al área de trabajo el nombre *Informes de administración* y seleccione **Guardar**.
   
   :::image type="content" source="media/powerbi/create-workspace.png" alt-text="Captura de pantalla que muestra Creación de un área de trabajo en servicio Power BI.":::
   
1. Para conceder acceso a personas y grupos al área de trabajo, seleccione los puntos de **Más opciones** junto al nuevo nombre del área de trabajo y, a continuación, seleccione **Acceso al área de trabajo**.
   
   :::image type="content" source="media/powerbi/workspace-access.png" alt-text="Captura de pantalla que muestra el acceso al área de trabajo en el menú Más opciones del área de trabajo.":::
   
1. En el panel lateral de **acceso al área de trabajo**, puede agregar las direcciones de correo electrónico de los usuarios y asignar un rol a cada usuario. Los roles son Administrador, Miembro, Colaborador y Espectador.

## <a name="publish-the-power-bi-report"></a>Publicar el informe de Power BI

Ahora puede usar Power BI Desktop para publicar el informe de Power BI para que otras personas puedan verlo.

1. En el nuevo informe de Power BI Desktop, seleccione **Publicar**.
   
   :::image type="content" source="media/powerbi/publish.png" alt-text="Captura de pantalla en la que se muestra la cinta de opciones Publicar en Power BI Desktop.":::
   
1. Seleccione el área de trabajo **Informes de administración** en la que se publicará el informe y seleccione **Seleccionar**.
   
   :::image type="content" source="media/powerbi/select-workspace.png" alt-text="Captura de pantalla que muestra la selección del área de trabajo de informes de administración de Power BI en la que se publicará el informe.":::
   
## <a name="import-the-report-to-a-microsoft-teams-channel"></a>Importación del informe a un canal de Microsoft Teams

También se espera que los miembros del canal de administración de Teams puedan ver el informe. Para agregar el informe a un canal de Teams:

1. En el canal de administración de Teams, seleccione **+** para agregar una pestaña y, en la ventana **Agregar una pestaña**, busque **Power BI** y selecciónelo.

   :::image type="content" source="media/powerbi/add-tab.png" alt-text="Captura de pantalla que muestra Power BI en la ventana Agregar una pestaña de Teams.":::
   
1. Seleccione el nuevo informe en la lista de informes de Power BI y seleccione **Guardar**. El informe aparece en una nueva pestaña en el canal de Teams.
   
   :::image type="content" source="media/powerbi/teams.png" alt-text="Captura de pantalla que muestra el informe de Power BI en una pestaña del canal de Teams.":::

## <a name="schedule-report-refresh"></a>Programación de la actualización del informe

Actualice el informe de Power BI según una programación para que los datos actualizados siempre aparezcan en el informe.

1. En la servicio Power BI, seleccione el área de trabajo en la que publicó el informe.
   
1. Junto al conjunto de datos del informe, seleccione **Más opciones** > **Configuración**.
   
   :::image type="content" source="media/powerbi/settings.png" alt-text="Captura de pantalla que muestra Configuración en Más opciones en el conjunto de datos de informe de Power BI.":::
   
1. Seleccione **Editar credenciales** para proporcionar las credenciales de una cuenta que tenga acceso de lectura al área de trabajo de Log Analytics.
   
1. En **Actualización programada**, establezca el control deslizante en **Activada** y configure una programación de actualización para el informe.
   
   :::image type="content" source="media/powerbi/schedule.png" alt-text="Captura de pantalla que muestra la configuración Actualización programada para el conjunto de datos de informes de Power BI.":::

## <a name="next-steps"></a>Pasos siguientes

- [Importación de datos de registro de Azure Monitor en Power BI](/azure/azure-monitor/visualize/powerbi)
- [Lenguaje de fórmulas de Power Query M](/powerquery-m/)
