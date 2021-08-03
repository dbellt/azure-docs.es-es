---
title: Supervisión de Azure Cosmos DB con Información de Cosmos DB de Azure Monitor| Microsoft Docs
description: En este artículo se describe la característica Información de Cosmos DB de Azure Monitor que proporciona a los administradores de Cosmos DB un rápido conocimiento de los problemas de uso y rendimiento de sus cuentas de Cosmos DB.
author: lgayhardt
ms.author: lagayhar
ms.topic: conceptual
ms.date: 05/11/2020
ms.openlocfilehash: b6666e43e1dc3461bf7e0cc5849d26c1a8fef70e
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110095093"
---
# <a name="explore-azure-monitor-cosmos-db-insights"></a>Exploración de Información de Cosmos DB de Azure Monitor

Información de Cosmos DB proporciona una vista del rendimiento general, los errores, la capacidad y el estado operativo de todos los recursos de Azure Cosmos DB en una experiencia interactiva unificada. Este artículo le ayudará a comprender las ventajas de esta nueva experiencia de supervisión y cómo puede modificar y adaptar la experiencia para adaptarla a las necesidades únicas de su organización.   

## <a name="introduction"></a>Introducción

Antes de profundizar en la experiencia, debe entender cómo se presenta y se visualiza la información. 

Ofrece:

* **Perspectiva a escala** de los recursos de Azure Cosmos DB de todas las suscripciones en una sola ubicación, con la posibilidad de examinar de forma selectiva solo esas suscripciones y recursos que le interese evaluar.

* **Análisis detallado** de un recurso de Azure Cosmos DB determinado para ayudar a diagnosticar problemas o realizar análisis detallados por categoría: uso, errores, capacidad y operaciones. Al seleccionar cualquiera de esas opciones, se proporciona una vista detallada de las métricas de Azure Cosmos DB de interés.  

* **Personalizable**: esta experiencia se basa en las plantillas de libro de Azure Monitor que le permiten cambiar las métricas que se muestran, modificar o establecer umbrales que se alineen con los límites y, luego, guardarlos en un libro personalizado. Los gráficos del libro se pueden luego anclar a los paneles de Azure.  

Esta característica no requiere que habilite ni configure nada; estas métricas de Azure Cosmos DB se recopilan de forma predeterminada.

>[!NOTE]
>El acceso a esta característica no se cobra, solo paga por las características básicas de Azure Monitor esenciales que configure o habilite, como se describe en la página de detalles de [Precios de Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/).

## <a name="view-utilization-and-performance-metrics-for-azure-cosmos-db"></a>Visualización de las métricas de uso y rendimiento para Azure Cosmos DB

Para ver el uso y el rendimiento de las cuentas de almacenamiento en todas las suscripciones, realice los pasos siguientes.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

2. Busque **Monitor** y seleccione **Monitor**.

    ![Cuadro de búsqueda con la palabra "Monitor" y una lista desplegable donde pone Servicios "Monitor" con una imagen de un velocímetro](./media/cosmosdb-insights-overview/search-monitor.png)

3. Seleccione **Cosmos DB**.

    ![Captura de pantalla del libro de información general de Cosmos DB](./media/cosmosdb-insights-overview/cosmos-db.png)

### <a name="overview"></a>Información general

En **Información general**, la tabla muestra las métricas de Azure Cosmos DB interactivas. Puede filtrar los resultados en función de las opciones que seleccione en las siguientes listas desplegables:

* **Suscripciones**: solo se enumeran las suscripciones que tienen un recurso de Azure Cosmos DB.  

* **Cosmos DB**: puede seleccionar todos los recursos de Azure Cosmos DB, uno solo o un subconjunto de ellos.

* **Time Range** (Intervalo de tiempo): de forma predeterminada, muestra las últimas cuatro horas de información en función de las selecciones correspondientes realizadas.

El icono de contador que aparece en las listas desplegables acumula el número total de recursos de Azure Cosmos DB de las suscripciones seleccionadas. Existe codificación de color condicional o mapas térmicos para las columnas del libro que notifican las métricas de transacción. El color más profundo tiene el valor más alto y un color más claro indica los valores más bajos. 

Al seleccionar una flecha desplegable junto a uno de los recursos de Azure Cosmos DB se mostrará un desglose de las métricas de rendimiento en el nivel de contenedor de base de datos individual:

![Lista desplegable expandida que revela los contenedores de base de datos individuales y el desglose del rendimiento asociado](./media/cosmosdb-insights-overview/container-view.png)

Al seleccionar el nombre del recurso de Azure Cosmos DB resaltado en azul, se le dirigirá a la pestaña **Información general** de la cuenta de Azure Cosmos DB asociada. 

### <a name="failures"></a>Errores

Seleccione **Errores** en la parte superior de la página; se abrirá la sección **Errores** de la plantilla de libro. Se muestra el número total de solicitudes con la distribución de las respuestas que componen esas solicitudes:

![Captura de pantalla de errores con desglose por tipo de solicitud HTTP](./media/cosmosdb-insights-overview/failures.png)

| Código |  Descripción       | 
|-----------|:--------------------|
| `200 OK`  | Una de las siguientes operaciones REST se realizó correctamente: </br>-GET en un recurso. </br> -PUT en un recurso. </br> -POST en un recurso. </br> -POST en un recurso de procedimiento almacenado para ejecutar el procedimiento almacenado.|
| `201 Created` | Una operación POST para crear un recurso se ha realizado correctamente. |
| `404 Not Found` | La operación está intentando actuar en un recurso que ya no existe. Por ejemplo, puede que el recurso ya se haya eliminado. |

Para obtener una lista completa de códigos de estado, consulte el artículo [Códigos de estado HTTP de Azure Cosmos DB](/rest/api/cosmos-db/http-status-codes-for-cosmosdb).

### <a name="capacity"></a>Capacity

Seleccione **Capacidad** en la parte superior de la página; se abre la sección **Capacidad** de la plantilla de libro. Se muestra el número de documentos que tiene, el crecimiento de documentos con el tiempo, el uso de los datos y la cantidad total de almacenamiento disponible que ha dejado.  Esta información se puede usar para identificar posibles problemas de almacenamiento y uso de datos.

![Libro de capacidad](./media/cosmosdb-insights-overview/capacity.png) 

Al igual que en el libro de información general, al seleccionar la lista desplegable junto a un recurso de Azure Cosmos DB en la columna **Suscripción** se mostrará un desglose por los contenedores individuales que componen la base de datos.

### <a name="operations"></a>Operaciones

Seleccione **Operaciones** en la parte superior de la página; se abre la plantilla de libro **Operaciones**. Esta plantilla le ofrece la posibilidad de ver las solicitudes desglosadas por el tipo de solicitudes realizadas.

De modo que, en el ejemplo siguiente, verá que `eastus-billingint` está recibiendo principalmente solicitudes de lectura, pero con un pequeño número de solicitudes upsert y de creación. Considerando que `westeurope-billingint` es de solo lectura desde una perspectiva de la solicitud, al menos en las últimas cuatro horas el libro ha estado limitado mediante su parámetro de intervalo de tiempo.

![Libro de operaciones](./media/cosmosdb-insights-overview/operation.png)

## <a name="view-from-an-azure-cosmos-db-resource"></a>Visualización desde un recurso de Azure Cosmos DB

1. Busque o seleccione cualquiera de las cuentas de Azure Cosmos DB existentes.

:::image type="content" source="./media/cosmosdb-insights-overview/cosmosdb-search.png" alt-text="HTAP para Azure Cosmos DB" border="true":::

2. Cuando se haya desplazado a su cuenta de Azure Cosmos DB, en la sección Supervisión, seleccione **Conclusiones (versión preliminar)** o **Libros** para realizar análisis adicionales sobre el rendimiento, las solicitudes, el almacenamiento, la disponibilidad, la latencia, el sistema y la administración de cuentas.

:::image type="content" source="./media/cosmosdb-insights-overview/cosmosdb-overview.png" alt-text="Información general sobre Conclusiones de Cosmos DB." border="true":::

### <a name="time-range"></a>Intervalo de horas

De forma predeterminada, el campo **Intervalo de tiempo** muestra los datos de las **últimas 24 horas**. Puede modificar el intervalo de tiempo para mostrar los datos en cualquier lugar desde los últimos cinco minutos hasta los últimos siete días. El selector de intervalos de tiempo también incluye un modo **personalizado** que le permite escribir las fechas de inicio y finalización para ver un intervalo de tiempo personalizado en función de los datos disponibles de la cuenta seleccionada.

:::image type="content" source="./media/cosmosdb-insights-overview/cosmosdb-time-range.png" alt-text="Intervalo de tiempo de Cosmos DB" border="true":::

### <a name="insights-overview"></a>Introducción a Conclusiones

En la pestaña **Información general** se proporcionan las métricas más comunes para la cuenta de Azure Cosmos DB seleccionada, como las siguientes:

* Total de solicitudes
* Solicitudes con error (429)
* Consumo de RU normalizado (máx.)
* Uso de datos e índices
* Métricas de cuenta de Cosmos DB por recopilación

**Total de solicitudes:** este gráfico proporciona una vista de las solicitudes totales de la cuenta desglosadas por código de estado. Las unidades de la parte inferior del gráfico son una suma de las solicitudes totales para el período.

:::image type="content" source="./media/cosmosdb-insights-overview/cosmosdb-total-requests.png" alt-text="Gráfico de solicitudes totales de Cosmos DB" border="true":::

**Solicitudes con error (429)** : este gráfico proporciona una vista de las solicitudes con error con el código de estado 429. Las unidades situadas en la parte inferior del gráfico son la suma del total de solicitudes con error para el período.

:::image type="content" source="./media/cosmosdb-insights-overview/cosmosdb-429.png" alt-text="Gráfico de solicitudes con error de Cosmos DB." border="true":::

**Consumo de RU normalizado (máx.)** : este gráfico proporciona el porcentaje máximo entre el 0-100 % de unidades de consumo de RU normalizadas durante el período especificado.

:::image type="content" source="./media/cosmosdb-insights-overview/cosmosdb-normalized-ru.png" alt-text="Consumo de RU normalizado de Cosmos DB" border="true":::

## <a name="pin-export-and-expand"></a>Anclaje, exportación y expansión

Cualquiera de las secciones de métricas se puede anclar a un panel de [Azure](../../azure-portal/azure-portal-dashboards.md); para ello, seleccione el icono de chincheta en la parte superior derecha de la sección.

![Ejemplo de la sección de métricas anclada al panel](./media/cosmosdb-insights-overview/pin.png)

Para exportar los datos a formato de Excel, seleccione el icono de flecha abajo situado a la izquierda del icono de chincheta.

![Icono de exportación del libro](./media/cosmosdb-insights-overview/export.png)

Para expandir o contraer todas las vistas desplegables del libro, seleccione el icono de expansión situado a la izquierda del icono de exportación:

![Icono de expansión del libro](./media/cosmosdb-insights-overview/expand.png)

## <a name="customize-cosmos-db-insights"></a>Personalización de Información de Cosmos DB

Puesto que esta experiencia se basa en plantillas de Azure Monitor, tiene la posibilidad de **personalizar** > **editar** y **guardar** una copia de la versión modificada en un libro personalizado. 

![Barra de personalización](./media/cosmosdb-insights-overview/customize.png)

Los libros se guardan en un recurso compartido, bien en la sección **Mis informes** o en la sección **Informes compartidos** accesibles a cualquiera con acceso al grupo de recursos. Después de guardar el libro personalizado, debe ir a la galería de libros para iniciarlo.

![Inicio de la galería de libros desde la barra de comandos](./media/cosmosdb-insights-overview/gallery.png)

## <a name="troubleshooting"></a>Solución de problemas

Para obtener instrucciones para la solución de problemas, consulte el [artículo dedicado de solución de problemas](troubleshoot-workbooks.md) de conclusiones basadas en libros.

## <a name="next-steps"></a>Pasos siguientes

* Configure [alertas de métricas](../alerts/alerts-metric.md) y [notificaciones de estado del servicio](../../service-health/alerts-activity-log-service-notifications-portal.md) para generar alertas automáticas que ayuden a detectar los problemas.

* Conozca los escenarios para los que están concebidos los libros, cómo crear informes y personalizar los ya existentes y otros muchos temas en el artículo [Crear informes interactivos con libros de Azure Monitor](../visualize/workbooks-overview.md).
