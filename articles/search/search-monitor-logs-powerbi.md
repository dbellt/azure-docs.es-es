---
title: Visualización de registros y métricas con Power BI
description: Visualice registros y métricas de Azure Cognitive Search con Power BI.
author: MarkHeff
ms.author: maheff
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 04/07/2021
ms.openlocfilehash: fbbeb861e50abfd393b416ddc46ff147fffb7b8e
ms.sourcegitcommit: d63f15674f74d908f4017176f8eddf0283f3fac8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "106581639"
---
# <a name="visualize-azure-cognitive-search-logs-and-metrics-with-power-bi"></a>Visualización de registros y métricas de Azure Cognitive Search con Power BI

[Azure Cognitive Search](./search-what-is-azure-search.md) puede enviar registros de operaciones y métricas de servicio a una cuenta de Azure Storage, que puede visualizar en Power BI. En este artículo se explican los pasos y el procedimiento para usar una aplicación de plantilla de Power BI para visualizar los datos. La plantilla puede ayudarle a obtener información detallada sobre su servicio de búsqueda, incluida información acerca de las métricas de consultas, indexación, operaciones y servicio.

Puede buscar la aplicación de plantilla de Power BI **Azure Cognitive Search: Análisis de registros y métricas** en [Marketplace de aplicaciones de Power BI](https://appsource.microsoft.com/marketplace/apps).

## <a name="set-up-the-app"></a>Configuración de la aplicación

1. Habilite el registro de métricas y recursos en el servicio de búsqueda:

    1. Cree o identifique una [cuenta de Azure Storage](../storage/common/storage-account-create.md) existente donde pueda archivar los registros.
    1. Vaya al servicio Azure Cognitive Search en Azure Portal.
    1. En la sección Supervisión de la columna izquierda, seleccione **Configuración de diagnóstico**.

        :::image type="content" source="media/search-monitor-logs-powerbi/diagnostic-settings.png" alt-text="Captura de pantalla que muestra cómo seleccionar Configuración de diagnóstico en la sección Supervisión del servicio Azure Cognitive Search." border="false":::

    1. Seleccione **+ Agregar configuración de diagnóstico**.
    1. Marque **Archivar en una cuenta de almacenamiento**, proporcione su información de la cuenta de almacenamiento y marque **OperationLogs** y **AllMetrics**.

        :::image type="content" source="media/search-monitor-logs-powerbi/add-diagnostic-setting.png" alt-text="Captura de pantalla que muestra cómo hacer selecciones para las métricas y el registro de recursos en la página Configuración de diagnóstico.":::
    1. Seleccione **Guardar**.

1. Una vez habilitado el registro, use el servicio de búsqueda para empezar a generar registros y métricas. Transcurre hasta una hora antes de que los contenedores aparezcan en Blob Storage con estos registros. Verá un contenedor **insights-logs-operationlogs** para los registros de tráfico de búsqueda y un contenedor **insights-metrics-pt1m** para las métricas.

1. Busque la aplicación de Power BI de Azure Cognitive Search en [Marketplace de aplicaciones de Power BI](https://appsource.microsoft.com/marketplace/apps) e instálela en una nueva área de trabajo o un área de trabajo existente. La aplicación se llama **Azure Cognitive Search: Análisis de registros y métricas**.

1. Después de instalar la aplicación, seleccione la aplicación en su lista de aplicaciones de Power BI.

    :::image type="content" source="media/search-monitor-logs-powerbi/azure-search-app-tile.png" alt-text="Captura de pantalla que muestra la aplicación Cognitive Search de Azure para seleccionar en la lista de aplicaciones.":::

1. Seleccione **Conectar** para conectar sus datos.

    :::image type="content" source="media/search-monitor-logs-powerbi/get-started-with-your-new-app.png" alt-text="Captura de pantalla que muestra cómo conectarse a los datos en la aplicación Azure Cognitive Search.":::

1. Escriba el nombre de la cuenta de almacenamiento que contiene sus registros y métricas. De forma predeterminada, la aplicación revisará los últimos 10 días de datos, pero este valor puede cambiarse con el parámetro **Days**.

    :::image type="content" source="media/search-monitor-logs-powerbi/connect-to-storage-account.png" alt-text="Captura de pantalla que muestra cómo especificar el nombre de la cuenta de almacenamiento y el número de días que se consultan en la página Conexión a Azure Cognitive Search.":::

1. Seleccione **Clave** como método de autenticación y proporcione su clave de cuenta de almacenamiento. Seleccione **Privado** como nivel de privacidad. Haga clic en Iniciar sesión para iniciar el proceso de carga.

    :::image type="content" source="media/search-monitor-logs-powerbi/connect-to-storage-account-step-two.png" alt-text="Captura de pantalla que muestra cómo especificar el método de autenticación, la clave de cuenta y el nivel de privacidad en la página Conexión a Azure Cognitive Search.":::

1. Espere a que se actualicen los datos. Esto puede tardar algún tiempo en función de la cantidad de datos. Puede ver si los datos siguen actualizándose en función del indicador siguiente.

    :::image type="content" source="media/search-monitor-logs-powerbi/workspace-view-refreshing.png" alt-text="Captura de pantalla que muestra cómo leer la información de la página Actualización de datos.":::

1. Una vez que se haya completado la actualización de datos, seleccione **Informe de Azure Cognitive Search** para ver el informe.

    :::image type="content" source="media/search-monitor-logs-powerbi/workspace-view-select-report.png" alt-text="Captura de pantalla que muestra cómo seleccionar el informe de Azure Cognitive Search en la página de actualización de datos.":::

1. Asegúrese de actualizar la página después de abrir el informe para que se abra con sus datos.

    :::image type="content" source="media/search-monitor-logs-powerbi/powerbi-search.png" alt-text="Captura de pantalla del informe de Power BI en Azure Cognitive Search":::

## <a name="modify-app-parameters"></a>Modificación de los parámetros de la aplicación

Si desea visualizar datos en otra cuenta de almacenamiento o cambiar el número de días de datos que se van a consultar, siga los pasos siguientes para cambiar los parámetros **Days** y **StorageAccount**.

1. Vaya a sus aplicaciones de Power BI, busque su aplicación de Azure Cognitive Search y seleccione el botón **Editar aplicación** para ver el área de trabajo.

    :::image type="content" source="media/search-monitor-logs-powerbi/azure-search-app-tile-edit.png" alt-text="Captura de pantalla que muestra cómo seleccionar el botón Editar aplicación para la aplicación Azure Cognitive Search.":::

1. Seleccione **Configuración** en las opciones de conjunto de datos.

    :::image type="content" source="media/search-monitor-logs-powerbi/workspace-view-select-settings.png" alt-text="Captura de pantalla que muestra cómo seleccionar Configuración en las opciones del conjunto de datos de Azure Cognitive Search.":::

1. En la pestaña Conjuntos de datos, cambie los valores del parámetro y seleccione **Aplicar**. Si hay un problema con la conexión, actualice las credenciales de origen de datos en la misma página.

1. Vuelva al área de trabajo y seleccione **Actualizar ahora** en las opciones de conjunto de datos.

    :::image type="content" source="media/search-monitor-logs-powerbi/workspace-view-select-refresh-now.png" alt-text="Captura de pantalla que muestra cómo seleccionar Actualizar ahora en las opciones del conjunto de datos de Azure Cognitive Search.":::

1. Abra el informe para ver los datos actualizados. También es posible que tenga que actualizar el informe para ver los datos más recientes.

## <a name="troubleshooting-report-issues"></a>Solución de problemas de informe

Si ve que no puede ver sus datos, siga estos pasos para solucionar problemas:

1. Abra el informe y actualice la página para asegurarse de que está viendo los datos más recientes. Hay una opción en el informe para actualizar los datos. Selecciónela para obtener los datos más recientes.

1. Asegúrese de que el nombre de la cuenta de almacenamiento y la clave de acceso que proporcionó son correctos. El nombre de la cuenta de almacenamiento debe corresponder a la cuenta configurada con sus registros de servicios de búsqueda.

1. Confirme que su cuenta de almacenamiento contiene los contenedores **insights-logs-operationlogs** e **insights-metrics-pt1m** y que cada contenedor tiene datos. Los registros y las métricas estarán dentro de un par de niveles de carpetas.

1. Compruebe si sigue actualizándose el conjunto de datos. El indicador de estado de actualización se muestra en el paso 8 anterior. Si se sigue actualizando, espere a que se haya completado la actualización para abrir y actualizar el informe.

## <a name="next-steps"></a>Pasos siguientes

+ [Supervisión de operaciones de búsqueda y actividad](search-monitor-usage.md)
+ [¿Qué es Power BI?](/power-bi/fundamentals/power-bi-overview)
+ [Conceptos básicos para diseñadores en el servicio Power BI](/power-bi/service-basic-concepts)