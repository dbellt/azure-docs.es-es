---
title: Creación de paneles personalizados en Azure Application Insights | Microsoft Docs
description: Tutorial para crear paneles de indicadores clave de rendimiento (KPI) personalizados con Azure Application Insights.
ms.topic: tutorial
author: lgayhardt
ms.author: lagayhar
ms.date: 09/30/2020
ms.custom: mvc, contperf-fy21q1
ms.openlocfilehash: 7bad0b17bb2fed9d9f54cdd44042ec88708f35fd
ms.sourcegitcommit: 5f482220a6d994c33c7920f4e4d67d2a450f7f08
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/08/2021
ms.locfileid: "107103732"
---
# <a name="create-custom-kpi-dashboards-using-azure-application-insights"></a>Creación de paneles de indicadores clave de rendimiento (KPI) personalizados con Azure Application Insights

Puede crear varios paneles en Azure Portal de manera que cada uno incluya iconos en los que se visualicen datos procedentes de varios recursos de Azure en diferentes suscripciones y grupos de recursos.  Puede anclar distintos gráficos y vistas de visión de Azure Application Insights para crear paneles personalizados que le proporcionen una descripción completa del estado y el rendimiento de la aplicación. Este tutorial le guía a través de la creación de un panel personalizado que incluye varios tipos de datos y visualizaciones de Azure Application Insights.

 Aprenderá a:

> [!div class="checklist"]
> * Crear un panel personalizado en Azure
> * Agregar un icono desde la Galería de iconos
> * Agregar métricas estándar de Application Insights al panel
> * Agregar un gráfico de métricas personalizado de Application Insights al panel
> * Agregar los resultados de una consulta de Registros (Analytics) al panel

## <a name="prerequisites"></a>Prerequisites

Para completar este tutorial:

- Implemente una aplicación de .NET en Azure y [habilite el SDK de Application Insights](../app/asp-net.md).

> [!NOTE]
> Los permisos necesarios para trabajar con paneles se describen en el artículo [Descripción del control de acceso para los paneles](../../azure-portal/azure-portal-dashboard-share-access.md#understanding-access-control-for-dashboards).

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en Azure Portal en [https://portal.azure.com](https://portal.azure.com).

## <a name="create-a-new-dashboard"></a>Creación de un nuevo panel

> [!WARNING]
> Si mueve el recurso de Application Insights a un grupo de recursos o una suscripción diferentes, tendrá que actualizar manualmente el panel; para ello, deberá quitar los iconos antiguos y anclar otros nuevos del mismo recurso de Application Insights en la nueva ubicación.

Un solo panel puede contener recursos de varias aplicaciones, grupos de recursos y suscripciones.  Comience el tutorial creando un nuevo panel para la aplicación.  

1. En Azure Portal, en el menú desplegable de la izquierda, seleccione **Panel**.

    ![Menú desplegable de Azure Portal](media/tutorial-app-dashboards/dashboard-from-menu.png)

2. En el panel, seleccione **Nuevo panel** y, luego, **Panel en blanco**.

   ![Nuevo panel](media/tutorial-app-dashboards/new-dashboard.png)

3. Escriba un nombre para el panel.
4. Eche un vistazo a la **Galería de iconos** para ver la variedad de iconos que se pueden agregar al panel.  Además de agregar iconos de la galería, puede anclar gráficos y otras vistas directamente desde Application Insights al panel.
5. Busque el icono **Markdown** y arrástrelo hasta el panel.  Este icono le permite agregar texto en formato Markdown, que es idóneo para agregar texto descriptivo al panel. Para más información, consulte [Uso de un icono de Markdown en los paneles de Azure para mostrar contenido personalizado](../../azure-portal/azure-portal-markdown-tile.md).
6. Agregue texto a las propiedades del icono y cambie su tamaño en el lienzo del panel.

    [![Edición del mosaico de Markdown](media/tutorial-app-dashboards/markdown.png)](media/tutorial-app-dashboards/markdown.png#lightbox)

7. Seleccione **Personalización finalizada** en la parte superior de la pantalla para salir del modo de personalización de iconos.

## <a name="add-health-overview"></a>Adición de información general de estado

Un panel con texto estático no resulta muy interesante; así pues, agregue ahora un icono de Application Insights para mostrar información acerca de la aplicación. Puede agregar iconos de Application Insights desde la Galería de iconos, o puede anclarlos directamente desde las pantallas de Application Insights. Esto le permite configurar gráficos y vistas con las que ya está familiarizado antes de anclarlos al panel.  Empiece por agregar la información general de estado estándar para la aplicación.  Esto no requiere configuración y permite una personalización mínima en el panel.


1. Seleccione el recurso de **Application Insights** en la pantalla de inicio.
2. En el panel **Información general**, seleccione el icono de chincheta ![icono de chincheta](media/tutorial-app-dashboards/pushpin.png) para agregarlo a un panel.
3. En la pestaña "Anclar al panel", seleccione el panel al que quiere agregar el icono o cree uno.
 
3. En la esquina superior derecha aparecerá una notificación que indica que el icono estaba anclado al panel.  Seleccione **Anclado al panel** en la notificación para volver a su panel o use la pestaña de panel.
4. El icono ya se ha agregado al panel. Seleccione **Editar** para cambiar el posicionamiento del icono. Selecciónelo y arrástrelo a su posición y, luego, elija **Personalización finalizada**. Ahora, el panel tiene un icono con información útil.

    [![Panel en modo de edición](media/tutorial-app-dashboards/dashboard-edit-mode.png)](media/tutorial-app-dashboards/dashboard-edit-mode.png#lightbox)

## <a name="add-custom-metric-chart"></a>Adición del gráfico de métricas personalizado

El panel **Métricas** panel le permite representar una métrica recopilada por Application Insights a lo largo del tiempo con filtros opcionales y agrupación.  Al igual que todo lo demás en Application Insights, puede agregar este gráfico al panel.  Para ello se requiere antes un poco de personalización.

1. Seleccione el recurso de **Application Insights** en la pantalla de inicio.
1. Seleccione **Métricas**.  
2. Ya se ha creado un gráfico vacío, y se le pedirá que agregue una métrica.  Agregue una métrica al gráfico y, opcionalmente, agregue un filtro y una agrupación.  En el ejemplo siguiente se muestra el número de solicitudes de servidor agrupadas según se hayan completado correctamente o no.  De este modo se obtiene una vista de ejecución de las solicitudes correctas e incorrectas.

    [![Adición de métrica](media/tutorial-app-dashboards/metrics.png)](media/tutorial-app-dashboards/metrics.png#lightbox)

4. Seleccione **Anclar al panel** a la derecha.

3.  En la esquina superior derecha aparecerá una notificación que indica que el icono estaba anclado al panel. Seleccione **Anclado al panel** en la notificación para volver a su panel o use la pestaña de panel.

4. El icono ya se ha agregado al panel. Seleccione **Editar** para cambiar el posicionamiento del icono. Seleccione y arrastre el icono a su posición y, luego, elija **Personalización finalizada**.

## <a name="add-logs-query"></a>Adición de consulta de registros

Los registros de Azure Application Insights proporcionan un lenguaje de consulta completo que le permite analizar todos los datos recopilados por Application Insights. Al igual que los gráficos y otras vistas, puede agregar el resultado de una consulta de registros al panel.

1. Seleccione el recurso de **Application Insights** en la pantalla de inicio.
2. Seleccione **Registros** a la izquierda en "Supervisión" para abrir la pestaña Registros.
3. Escriba la siguiente consulta, que devuelve las 10 páginas más solicitadas y su número de solicitudes:

    ``` Kusto
    requests
    | summarize count() by name
    | sort by count_ desc
    | take 10
    ```

4. Seleccione **Ejecutar** para validar los resultados de la consulta.
5. Seleccione el icono de anclaje ![Icono de chincheta](media/tutorial-app-dashboards/pushpin.png) y seleccione el nombre del panel.

5. Antes de volver al panel, agregue otra consulta, pero represéntela como un gráfico para ver las distintas maneras de visualizar una consulta de registros en un panel. Comience con la siguiente consulta, que resume las 10 operaciones principales con la mayoría de las excepciones.

    ``` Kusto
    exceptions
    | summarize count() by operation_Name
    | sort by count_ desc
    | take 10
    ```

6. Seleccione **Gráfico** y, a continuación, cambie a **Anillo** para visualizar el resultado.

    [![Gráfico de anillos con la consulta anterior](media/tutorial-app-dashboards/logs-doughnut.png)](media/tutorial-app-dashboards/logs-doughnut.png#lightbox)

6. Seleccione el icono de anclaje ![Icono de chincheta](media/tutorial-app-dashboards/pushpin.png) en la parte superior derecha para anclar el gráfico al panel y, luego, regrese al panel.
7. Los resultados de las consultas se agregan ahora al panel en el formato que seleccionó. Seleccione y arrastre todos ellos a su posición y, luego, elija **Personalización finalizada**.
8. Seleccione el icono de lápiz ![Icono de lápiz](media/tutorial-app-dashboards/pencil.png) en cada título para darles un título descriptivo.

## <a name="share-dashboard"></a>Compartir panel

1. En la parte superior del panel, seleccione **Compartir** para publicar los cambios.
2. Si lo desea, puede definir los usuarios específicos que deben tener acceso al panel. Para más información, consulte [Uso compartido de paneles de Azure mediante el control de acceso basado en rol de Azure](../../azure-portal/azure-portal-dashboard-share-access.md).
3. Seleccione **Publicar**.

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha aprendido a crear paneles personalizados, eche un vistazo al resto de la documentación de Application Insights, que incluye un caso práctico.

> [!div class="nextstepaction"]
> [Diagnósticos detallados](../app/devops.md)
