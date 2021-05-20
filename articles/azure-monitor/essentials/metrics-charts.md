---
title: Características avanzadas del Explorador de métricas de Azure
description: Obtenga información sobre los usos avanzados del Explorador de métricas de Azure.
author: vgorbenko
services: azure-monitor
ms.topic: conceptual
ms.date: 01/22/2019
ms.author: vitalyg
ms.openlocfilehash: 8564f96ad311a07ccf4582f274e44e6ebd1e0c85
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108758538"
---
# <a name="advanced-features-of-the-azure-metrics-explorer"></a>Características avanzadas del Explorador de métricas de Azure

> [!NOTE]
> En este artículo se da por supuesto que está familiarizado con las características básicas de la característica Explorador de métricas de Azure de Azure Monitor. Si es un nuevo usuario y desea aprender a crear su primer gráfico de métricas, consulte [Introducción al Explorador de métricas](./metrics-getting-started.md).

En Azure Monitor, las [métricas](data-platform-metrics.md) son una serie de valores medidos y recuentos que se recopilan y se almacenan con el tiempo. Las métricas pueden ser estándar (también denominadas "plataforma") o personalizadas. 

La plataforma Azure proporciona métricas estándar. Estas reflejan las estadísticas de uso y mantenimiento de los recursos de Azure. 

## <a name="resource-scope-picker"></a>Selector de ámbitos de recursos
El selector de ámbitos de recursos permite ver las métricas en uno o varios recursos. En las secciones siguientes se explica cómo usar el selector de ámbitos de recursos. 

### <a name="select-a-single-resource"></a>Seleccione un solo recursos
Seleccione **Métricas** en el menú de **Azure Monitor** o en la sección **Supervisión** del menú de un recurso. A continuación, elija **Seleccionar un ámbito** para abrir el selector de ámbitos. 

Use el selector de ámbitos para seleccionar los recursos cuyas métricas desea ver. El ámbito se debe rellenar al abrir el Explorador de métricas de Azure desde el menú de un recurso. 

![Captura de pantalla que muestra cómo abrir el selector de ámbitos de recursos.](./media/metrics-charts/scope-picker.png)

En el caso de algunos recursos, solo puede ver las métricas de un recurso a la vez. En el menú **Tipos de recursos**, estos recursos se encuentran en la sección **Todos los tipos de recursos**.

![Captura de pantalla que muestra un solo recurso.](./media/metrics-charts/single-resource-scope.png)

Después de seleccionar un recurso, verá todas las suscripciones y los grupos de recursos que contienen ese recurso.

![Captura de pantalla que muestra los recursos disponibles.](./media/metrics-charts/available-single-resource.png)

> [!TIP]
> Si desea que la funcionalidad vea las métricas de varios recursos al mismo tiempo o las métricas de una suscripción o un grupo de recursos, seleccione **Votar a favor**.

Cuando esté satisfecho con la selección, seleccione **Aplicar**.

### <a name="view-metrics-across-multiple-resources"></a>Visualización de métricas en varios recursos
Algunos tipos de recursos pueden consultar las métricas de varios recursos. Los recursos deben estar en la misma suscripción y ubicación. Busque estos tipos de recursos en la parte superior del menú **Tipos de recursos**. 

Para obtener más información, consulte [Seleccionar varios recursos](./metrics-dynamic-scope.md#select-multiple-resources).

![Captura de pantalla que muestra tipos entre recursos.](./media/metrics-charts/multi-resource-scope.png)

En el caso de los tipos que son compatibles con varios recursos, puede consultar las métricas en una suscripción o en varios grupos de recursos. Para obtener más información, consulte [Selección de un grupo de recursos o suscripción](./metrics-dynamic-scope.md#select-a-resource-group-or-subscription).

## <a name="multiple-metric-lines-and-charts"></a>Varias líneas y gráficos de métricas

En el Explorador de métricas de Azure, puede crear gráficos que tracen varias líneas de métricas o muestren varios gráficos de métricas al mismo tiempo. Esta funcionalidad le permite:

- Poner en correlación métricas relacionadas en el mismo gráfico para ver la relación de un valor con otro.
- Mostrar métricas que usen diferentes unidades de medida muy cercanas.
- Agregar y comparar visualmente métricas de varios recursos.

Por ejemplo, imagine que tiene cinco cuentas de almacenamiento y desea saber cuánto espacio consumen conjuntamente. Puede crear un gráfico de áreas (apiladas) que muestre los valores individuales y la suma de todos los valores en momentos concretos.

### <a name="multiple-metrics-on-the-same-chart"></a>Varias métricas en el mismo gráfico

Para ver varias métricas en el mismo gráfico, [cree un nuevo gráfico](./metrics-getting-started.md#create-your-first-metric-chart) en primer lugar. A continuación, seleccione **Agregar métrica**. Repita este paso para agregar otra métrica en el mismo gráfico.

> [!NOTE]
> Normalmente, los gráficos no deben mezclar métricas que utilicen unidades de medida diferentes. Por ejemplo, evite mezclar una métrica que use milisegundos con otra que utilice kilobytes. Además, evite mezclar métricas cuyas escalas difieran significativamente. 
>
> En estos casos, considere la posibilidad de usar varios gráficos en su lugar. En el Explorador de métricas, seleccione **Nuevo gráfico** para crear un nuevo gráfico.

![Captura de pantalla que muestra varias métricas.](./media/metrics-charts/multiple-metrics-chart.png)

### <a name="multiple-charts"></a>Varios gráficos

Para crear otro gráfico que use una métrica diferente, seleccione **Nuevo gráfico**.

Para reordenar o eliminar varios gráficos, seleccione el botón de puntos suspensivos ( **...** ) para abrir el menú del gráfico. A continuación, elija **Subir**, **Bajar** o **Eliminar**.

![Captura de pantalla que muestra varios gráficos.](./media/metrics-charts/multiple-charts.png)

## <a name="time-range-controls"></a>Controles del intervalo de tiempo

Además de cambiar el intervalo de tiempo mediante el [panel del selector de tiempo](metrics-getting-started.md#select-a-time-range), también puede desplazarse por los controles del área de gráfico y hacer zoom en ellos.
### <a name="pan"></a>Movimiento panorámico

Para desplazarse, haga clic en las flechas izquierda y derecha del borde del gráfico.  Esto moverá la mitad del intervalo de tiempo del gráfico seleccionado hacia atrás y hacia delante.  Por ejemplo, si está consultando las últimas 24 horas, al hacer clic en la flecha izquierda el intervalo de tiempo se desplazará hasta abarcar un día y medio de hace 12 horas.

La mayoría de las métricas admiten 93 días de retención, pero solo permiten ver 30 días a la vez.  Con los controles de desplazamiento lateral, puede examinar los últimos 30 días y, a continuación, recorrer con facilidad 15 días a la vez para ver el resto del período de retención.

![GIF animado que muestra los controles de desplazamiento lateral izquierdo y derecho.](./media/metrics-charts/metrics-pan-controls.gif)

### <a name="zoom"></a>Zoom

Puede hacer clic y arrastrar el gráfico para acercar una sección de este.  El zoom actualizará el intervalo de tiempo del gráfico para así abarcar la selección, y seleccionará un intervalo de agregación más pequeño si el intervalo de tiempo se establece en "automático".  El nuevo intervalo de tiempo se aplicará a todos los gráficos de las métricas.

![GIF animado que muestra la característica de zoom de las métricas.](./media/metrics-charts/metrics-zoom-control.gif)

## <a name="aggregation&quot;></a>Agregación

Al agregar una métrica a un gráfico, el Explorador de métricas aplica automáticamente una agregación predeterminada. El valor predeterminado tiene sentido en escenarios básicos. Sin embargo, puede usar una agregación diferente para obtener más información sobre la métrica. 

Antes de usar agregaciones diferentes en un gráfico, debe comprender cómo las administra el Explorador de métricas. Las métricas son una serie de medidas (o &quot;valores de métricas") que se capturan durante un período de tiempo. Al trazar un gráfico, los valores de la métrica seleccionada se agregan por separado en el *intervalo de agregación*. 

Puede seleccionar el tamaño del intervalo de agregación mediante el [panel selector de tiempo](./metrics-getting-started.md#select-a-time-range) del Explorador de métricas. Si no selecciona explícitamente el intervalo de agregación, el intervalo de tiempo seleccionado actualmente se utiliza de forma predeterminada. Una vez determinado el intervalo de agregación, los valores de métricas que se capturaron durante cada intervalo de agregación se agregan en el gráfico, un punto de datos por intervalo de agregación.

Por ejemplo, supongamos que un gráfico muestra la métrica *Tiempo de respuesta del servidor*. Usa la agregación *media* en el intervalo de tiempo de las *últimas 24 horas*. En este ejemplo:

- Si la granularidad de tiempo se establece en 30 minutos, el gráfico se elabora a partir de 48 puntos de datos agregados. Es decir, el gráfico de líneas conecta 48 puntos en el área de trazado del gráfico (24 horas x 2 puntos de datos por hora). Cada punto de datos representa la *media* de todos los tiempos de respuesta capturados para las solicitudes de servidor que se produjeron durante cada uno de los períodos de tiempo de 30 minutos correspondientes.
- Si cambia la granularidad de tiempo a 15 minutos, obtendrá 96 puntos de datos agregados.  Es decir, obtiene 24 horas x 4 puntos de datos por hora.

El Explorador de métricas tiene cinco tipos de agregación estadística básicos: suma, recuento, mínima, máxima y promedio. A veces se llama a la agregación *suma* agregación *total*. En el caso de muchas métricas, el Explorador de métricas oculta las agregaciones que no son pertinentes y no se pueden usar.

* **Suma**: la suma de todos los valores capturados durante el intervalo de agregación.

    ![Captura de pantalla de una solicitud de suma.](./media/metrics-charts/request-sum.png)

* **Recuento**: el número de medidas capturadas durante el intervalo de agregación. 
    
    Cuando la métrica se captura siempre con el valor 1, la agregación recuento es igual a la agregación suma. Este escenario es común cuando la métrica realiza un seguimiento del recuento de diferentes eventos y cada medida representa un evento. El código emite un registro de métricas cada vez que llega una nueva solicitud.

    ![Captura de pantalla de una solicitud de recuento.](./media/metrics-charts/request-count.png)

* **Average**: el promedio de los valores de la métrica capturados durante el intervalo de agregación.

    ![Captura de pantalla de una solicitud de promedio.](./media/metrics-charts/request-avg.png)

* **Mínimo**: el menor valor capturado durante el intervalo de agregación.

    ![Captura de pantalla de una solicitud de mínima.](./media/metrics-charts/request-min.png)

* **Máximo**: el mayor valor capturado durante el intervalo de agregación.

    ![Captura de pantalla de una solicitud de máxima.](./media/metrics-charts/request-max.png)

## <a name="filters"></a>Filtros

Puede aplicar filtros a gráficos cuyas métricas tengan dimensiones. Por ejemplo, imagine una métrica "Recuento de transacciones" que tiene una dimensión "Tipo de respuesta". Esta dimensión indica si la respuesta de las transacciones era correcta o errónea. Si filtra por esta dimensión, verá una línea de gráfico solo para transacciones correctas (o solo para transacciones erróneas). 

### <a name="add-a-filter"></a>Agregar un filtro

1. Encima del gráfico, seleccione **Agregar filtro**.

2. Seleccione una dimensión (propiedad) para filtrar.

   ![Captura de pantalla que muestra las dimensiones (propiedades) que se pueden filtrar.](./media/metrics-charts/028.png)

3. Seleccione el operador que quiere aplicar a la dimensión (propiedad). El operador predeterminado es = (es igual a)

   ![Captura de pantalla que muestra el operador que puede usar con el filtro.](./media/metrics-charts/filter-operator.png)

4. Seleccione qué valores de dimensión quiere aplicar al filtro al trazar el gráfico (en este ejemplo se muestra el filtrado excluyente de las transacciones de almacenamiento correctas):

   ![Captura de pantalla que muestra las transacciones de almacenamiento filtradas correctas.](./media/metrics-charts/029.png)

5. Después de seleccionar los valores de filtro, haga clic fuera del Selector de filtro para cerrarlo. Ahora el gráfico muestra cuántas transacciones de almacenamiento han tenido error:

   ![Captura de pantalla que muestra cuántas transacciones de almacenamiento han tenido error.](./media/metrics-charts/030.png)

6. Puede repetir los pasos del 1 al 5 para aplicar varios filtros a los mismos gráficos.


## <a name="metric-splitting"></a>División de métricas

Puede dividir una métrica por dimensión para visualizar cómo se comparan los distintos segmentos de la métrica. La división también puede ayudarle a identificar los segmentos aislados de una dimensión.

### <a name="apply-splitting"></a>Aplicación de la división

1. Encima del gráfico, seleccione **Aplicar división**.
 
   > [!NOTE]
   > Los gráficos que tienen varias métricas no pueden usar la funcionalidad de división. Además, aunque un gráfico puede tener varios filtros, solo puede tener una dimensión de división.

2. Elija una dimensión en la que segmentar el gráfico:

   ![Captura de pantalla que muestra la dimensión seleccionada en la que segmentar el gráfico.](./media/metrics-charts/031.png)

   Ahora el gráfico muestra varias líneas, una para cada segmento de dimensión:

   ![Captura de pantalla que muestra varias líneas, una para cada segmento de dimensión.](./media/metrics-charts/segment-dimension.png)
   
3. Elija un límite para el número de valores que se mostrarán después de dividir la dimensión seleccionada. El límite predeterminado es 10, tal como se muestra en el gráfico anterior. El intervalo de límite es de 1 a 50.
   
   ![Captura de pantalla que muestra el límite de la división, que restringe el número de valores después de la división.](./media/metrics-charts/segment-dimension-limit.png)
   
4. Elija el criterio de ordenación de los segmentos: Ascendente o descendente. El valor predeterminado es descendente.
   
   ![Captura de pantalla que muestra el criterio de ordenación de los valores divididos.](./media/metrics-charts/segment-dimension-sort.png)

5. Haga clic fuera del **Selector de agrupación** para cerrarlo.
   

   > [!NOTE]
   > Para ocultar los segmentos que no son pertinentes para su escenario y facilitar la lectura de los gráficos, use el filtrado y la división en la misma dimensión.

## <a name="locking-the-range-of-the-y-axis"></a>Bloqueo del intervalo del eje y

El bloqueo del intervalo del eje de valores (Y) cobra importancia en los gráficos que muestran fluctuaciones pequeñas de valores grandes. 

Por ejemplo, una caída en el volumen de solicitudes correctas del 99,99 % al 99,5 % podría representar una reducción significativa de la calidad del servicio. Sin embargo, percibir una pequeña fluctuación de un valor numérico sería difícil, o incluso imposible, si usa la configuración predeterminada del gráfico. En este caso, podría bloquear el límite inferior del gráfico al 99 % para que una pequeña caída fuera más evidente. 

Otro ejemplo es una fluctuación en la memoria disponible. En este escenario, el valor técnicamente nunca llegará a 0. El ajuste del intervalo a un valor mayor podría facilitar la detección de caídas en la memoria disponible. 

Para controlar el intervalo del eje y, abra el menú del gráfico ( **...** ). A continuación, seleccione **Configuración del gráfico** para acceder a la configuración avanzada del gráfico.

![Captura de pantalla que resalta la selección de la configuración del gráfico.](./media/metrics-charts/033.png)

Modifique los valores de la sección **Intervalo del eje Y** o seleccione **Auto** para volver a los valores predeterminados.
 
 ![Captura de pantalla que resalta la sección del rango del eje Y.](./media/metrics-charts/034.png)

> [!WARNING]
> Si tiene que bloquear los límites del eje y para los gráficos que realizan el seguimiento de los recuentos o sumas durante un período de tiempo (mediante el uso de agregaciones recuento, suma, mínima o máxima), normalmente debe especificar una granularidad de tiempo fijo. En este caso, no debe confiar en los valores predeterminados automáticos. 
>
> Se elige una granularidad de tiempo fijo porque los valores del gráfico cambian cuando la granularidad de tiempo se modifica automáticamente después de que un usuario cambie el tamaño de una ventana del explorador o cambie la resolución de pantalla. El cambio resultante en la granularidad de tiempo afecta al aspecto del gráfico, lo que invalida la selección actual del intervalo del eje y.

## <a name="line-colors"></a>Colores de línea

Después de configurar los gráficos, se asigna automáticamente a las líneas del gráfico un color de la paleta predeterminada. Puede cambiar esos colores.

Para cambiar el color de una línea del gráfico, seleccione la barra coloreada de la leyenda correspondiente al gráfico. Se abre el cuadro de diálogo Selector de colores. Use el selector de colores para configurar el color de línea.

![Captura de pantalla que muestra cómo cambiar el color.](./media/metrics-charts/035.png)

Los colores personalizados se conservan al anclar el gráfico a un panel. En la sección siguiente se muestra cómo anclar un gráfico.

## <a name="pinning-to-dashboards"></a>Anclar a paneles 

Después de configurar un gráfico, puede que desee agregarlo a un panel. Al anclar un gráfico a un panel, puede hacer que sea accesible para el equipo. También puede obtener información mediante su visualización en el contexto de otros tipos de telemetría de supervisión.

Para anclar un gráfico configurado a un panel, en la esquina superior derecha del gráfico, seleccione **Anclar al panel**.

![Captura de pantalla que muestra cómo anclar un gráfico a un panel.](./media/metrics-charts/036.png)

## <a name="alert-rules"></a>Las reglas de alertas

Puede usar sus criterios de visualización para crear una regla de alerta basada en métricas. La nueva regla de alerta incluirá su recurso de destino, métrica, división y dimensiones de filtro del gráfico. Puede modificar esta configuración mediante el panel de creación de reglas de alerta.

Para empezar, seleccione **Nueva regla de alertas**.

![Captura de pantalla que muestra el botón Nueva regla de alertas resaltado en rojo.](./media/metrics-charts/042.png)

Se abre el panel de creación de reglas de alerta. En el panel, verá las dimensiones de métricas del gráfico. Los campos del panel se rellenan previamente para ayudarle a personalizar la regla.

![Captura de pantalla que muestra el panel creación de reglas.](./media/metrics-charts/041.png)

Para obtener más información, consulte [Creación, visualización y administración de alertas de métricas](../alerts/alerts-metric.md).

## <a name="correlate-metrics-to-logs"></a>Correlación de métricas con registros
Para ayudar a los clientes a diagnosticar la causa principal de las anomalías de su gráfico de métricas, hemos creado la característica Obtener detalles de los registros. Obtener detalles de los registros permite a los clientes correlacionar los picos de su gráfico de métricas con registros y consultas. 

Antes de sumergirnos en la experiencia, queremos introducir los distintos tipos de registros y consultas proporcionados. 

| Término             | Definición  | 
|------------------|-------------|
| Registros de actividad    | Proporciona una visión general de las operaciones de cada recurso de Azure de la suscripción desde fuera (en el plano de administración) y de las actualizaciones de los eventos de Service Health. Use el registro de actividad para determinar qué, quién y cuándo para las operaciones de escritura (PUT, POST, DELETE) realizadas en los recursos de la suscripción. Hay un único registro de actividad para cada suscripción de Azure.  |   
| Registro de diagnóstico   | Proporcionan conclusiones de las operaciones realizadas dentro de un recurso de Azure (en el plano de datos), por ejemplo, obtener un secreto de un almacén de claves o realizar una solicitud en una base de datos. El contenido de estos registros de recurso varía según el servicio de Azure y el tipo de recurso. **Nota:** Lo debe proporcionar el servicio y habilitar el cliente.  | 
| Registro recomendado | Consultas basadas en escenario que el cliente puede aprovechar para investigar anomalías en el explorador de métricas.  |

Actualmente, Obtener detalles de los registros está disponible para los proveedores de recursos seleccionados. Los proveedores de recursos que tienen la experiencia Obtener detalles de los registros completa son: 

* Application Insights 
* Escalado automático 
* Servicios de aplicaciones  
* Storage  

A continuación se muestra un ejemplo de experiencias para el proveedor de recursos de Application Insights.

![Pico de errores en la hoja de métricas de App Insights](./media/metrics-charts/drill-into-log-ai.png)

Para diagnosticar el pico en las solicitudes con error, haga clic en "Obtener detalles de los registros".

![Captura de pantalla de la lista desplegable Obtener detalles de los registros](./media/metrics-charts/drill-into-logs-dropdown.png)

Al hacer clic en la opción de error, se le conducirá a una hoja de error personalizada que le proporciona las operaciones con errores, los tipos de excepciones principales y las dependencias. 

![Captura de pantalla de la hoja de error de App Insights](./media/metrics-charts/ai-failure-blade.png)

### <a name="common-problems-with-drill-into-logs"></a>Problemas comunes con Obtener detalles de los registros

* El registro y las consultas están deshabilitados: para ver los registros y las consultas recomendados, debe redirigir los registros de diagnóstico a Log Analytics. Lea [este documento](./diagnostic-settings.md) para obtener información sobre cómo hacerlo. 
* Solo se proporcionan registros de actividad: la característica Obtener detalles de los registros solo está disponible para los proveedores de recursos seleccionados. De forma predeterminada, se proporcionan registros de actividad. 

 
## <a name="troubleshooting"></a>Solución de problemas

Si no ve ningún dato en el gráfico, revise la siguiente información de solución de problemas:

* Los filtros se aplican a todos los gráficos del panel. Mientras se centra en un gráfico, asegúrese de que no establece un filtro que excluya todos los datos de otro gráfico.

* Para establecer filtros diferentes en diversos gráficos, cree los gráficos en hojas distintas. A continuación, guarde los gráficos como favoritos independientes. Si lo desea, puede anclar los gráficos al panel para que pueda verlos juntos.

* Si segmenta un gráfico mediante una propiedad que no define la métrica, el gráfico no muestra ningún contenido. Intente borrar la segmentación (división) o elija otro nombre de propiedad.

## <a name="next-steps"></a>Pasos siguientes

Para crear paneles que se puedan accionar mediante métricas, consulte [Creación de paneles personalizados de KPI](../app/tutorial-app-dashboards.md).
