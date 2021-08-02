---
title: Visualizaciones de cuadrículas de libro de Azure Monitor
description: Más información sobre todas las visualizaciones de cuadrículas de libro de Azure Monitor.
services: azure-monitor
author: lgayhardt
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.topic: conceptual
ms.date: 09/04/2020
ms.author: lagayhar
ms.openlocfilehash: 4b30c98bbaf7f7806feecdbccd2c8736d431943c
ms.sourcegitcommit: 67cdbe905eb67e969d7d0e211d87bc174b9b8dc0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111854349"
---
# <a name="grid-visualizations"></a>Visualizaciones de cuadrículas

Las cuadrículas o las tablas son una forma habitual de presentar datos a los usuarios. Los libros permiten a los usuarios aplicar estilo a las columnas de la cuadrícula de forma individual para proporcionar una interfaz de usuario enriquecida para sus informes.

En el ejemplo siguiente se muestra una cuadrícula que combina iconos, mapas térmicos y minibarras para presentar información compleja. El libro también proporciona un ordenado, un cuadro de búsqueda y un botón de "ir a análisis".

[![Captura de pantalla de una cuadrícula basada en registro](./media/workbooks-grid-visualizations/grid.png)](./media/workbooks-grid-visualizations/grid.png#lightbox)

## <a name="adding-a-log-based-grid"></a>Agregar una cuadrícula basada en registro

1. Para cambiar el libro al modo de edición, haga clic en el elemento de la barra de herramientas **Editar**.
2. Use el vínculo **Agregar consulta** para agregar un control de consulta de registro al libro.
3. Seleccione el tipo de consulta **Registro**, el tipo de recurso (por ejemplo, Application Insights) y los recursos de destino.
4. Utilice el editor de consultas para ingresar el KQL para el análisis (por ejemplo, máquinas virtuales con memoria por debajo de un umbral)
5. Establezca la visualización a **Cuadrícula**
6. Si es necesario, establezca otros parámetros, como el intervalo de tiempo, el tamaño, la paleta de colores y la leyenda.

[![Captura de pantalla de una cuadrícula basada en registro](./media/workbooks-grid-visualizations/grid-query.png)](./media/workbooks-grid-visualizations/grid-query.png#lightbox)

## <a name="log-chart-parameters"></a>Parámetros de gráfico de registro

| Parámetro | Explicación | Ejemplo |
| ------------- |:-------------|:-------------|
| `Query Type` | Tipo de consulta que se va a usar. | Registro, Resource Graph, etc. |
| `Resource Type` | Tipo de recurso de destino. | Application Insights, Log Analytics o Azure-first |
| `Resources` | Conjunto de recursos del que se obtendrá el valor de las métricas. | MyApp1 |
| `Time Range` | Periodo de tiempo para mostrar en el gráfico de registro. | Última hora, últimas 24 horas, etc. |
| `Visualization` | Visualización que se usará. | Cuadrícula |
| `Size` | Tamaño vertical del control. | Pequeño, mediano, grande o pantalla completa. |
| `Query` | Cualquier consulta de KQL que devuelva datos en el formato esperado por la visualización del gráfico. | _requests \| summarize Requests = count() by name_ |

## <a name="simple-grid"></a>Cuadrícula simple

Los libros pueden representar los resultados de KQL como una tabla simple. En la cuadrícula siguiente se muestra el número de solicitudes y usuarios únicos por tipo de solicitud en una aplicación.

```kusto
requests
| where name !endswith('.eot')
| summarize Requests = count(), Users = dcount(user_Id) by name
| order by Requests desc
```

[![Captura de pantalla de una cuadrícula basada en un registro en modo de edición](./media/workbooks-grid-visualizations/log-chart-simple-grid.png)](./media/workbooks-grid-visualizations/log-chart-simple-grid.png#lightbox)

## <a name="grid-styling"></a>Estilo de cuadrícula

Aunque una tabla normal muestra datos, es difícil de leer y la información no siempre es aparente. El estilo de la cuadrícula puede facilitar la lectura e interpretación de los datos.

A continuación se muestra la misma cuadrícula de la sección anterior con el estilo de mapas térmicos.

[![Captura de pantalla de una cuadrícula basada en registro con columnas con el estilo de mapas térmicos](./media/workbooks-grid-visualizations/log-chart-grid-heatmap.png)](./media/workbooks-grid-visualizations/log-chart-grid-heatmap.png#lightbox)

Aquí está la misma cuadrícula con estilo de barras: [![Captura de pantalla de una cuadrícula basada en registro con columnas con el estilo de barras](./media/workbooks-grid-visualizations/log-chart-grid-bar.png)](./media/workbooks-grid-visualizations/log-chart-grid-bar.png#lightbox)

### <a name="styling-a-grid-column"></a>Aplicar estilo a una columna de cuadrícula

1. Seleccione el botón **Configuración de columnas** en la barra de herramientas del control de consultas.
2. En *Editar la configuración de las columnas*, seleccione la columna a la que se va a aplicar un estilo.
3. Elija un representador de columnas (por ejemplo, mapa térmico, barra, barra debajo, etc.) y la configuración correspondiente para aplicar estilo a la columna.

A continuación puede ver un ejemplo en el que se muestran los estilos de la columna *Solicitudes* como una barra:

[![Captura de pantalla de una cuadrícula basada en registro con columnas Solicitudes con el estilo de barra.](./media/workbooks-grid-visualizations/log-chart-grid-column-settings-start.png)](./media/workbooks-grid-visualizations/log-chart-grid-column-settings-start.png#lightbox)

### <a name="column-renderers"></a>Representadores de columnas

| Representador de columnas | Explicación | Opciones adicionales |
|:------------- |:-------------|:-------------|
| `Automatic` | Es la opción predeterminada: usa el representador más adecuado basándose en el tipo de columna.  |  |
| `Text` | Representa los valores de columna como texto. | |
| `Right Aligned` | Similar a texto, salvo que está alineado a la derecha. | |
| `Date/Time` | Representa una cadena de fecha y hora legible. | |
| `Heatmap` | Colorea las celdas de la cuadrícula basándose en el valor de la celda. | Paleta de colores y valores mínimo y máximo usados para el escalado. |
| `Bar` | Representa una barra junto a la celda basándose en el valor de la celda. | Paleta de colores y valores mínimo y máximo usados para el escalado. |
| `Bar underneath` | Representa una barra junto a la parte inferior de la celda basándose en el valor de esta. | Paleta de colores y valores mínimo y máximo usados para el escalado. |
| `Composite bar` | Representa una barra compuesta usando las columnas especificadas de esa fila. Consulte [Barra compuesta](workbooks-composite-bar.md) para obtener detalles. | Columnas con los colores correspondientes para representar la barra y una etiqueta que se va a mostrar en la parte superior de dicha barra. |
| `Spark bars` | Representa una minibarra en la celda basándose en los valores de una matriz dinámica en la celda. Por ejemplo, la columna Tendencia de la captura de pantalla de la parte superior. | Paleta de colores y valores mínimo y máximo usados para el escalado. |
| `Spark lines` | Representa un minigráfico en la celda basándose en los valores de una matriz dinámica en la celda. | Paleta de colores y valores mínimo y máximo usados para el escalado. |
| `Icon` | Representa los iconos basándose en los valores de texto de la celda. Los valores compatibles incluyen: `cancelled`, `critical`, `disabled`, `error`, `failed`, `info`, `none`, `pending`, `stopped`, `question`, `success`, `unknown`, `warning` `uninitialized`, `resource`, `up`, `down`, `left`, `right`, `trendup`, `trenddown`, `4`, `3`, `2`, `1`, `Sev0`, `Sev1`, `Sev2`, `Sev3`, `Sev4`, `Fired`, `Resolved`, `Available`, `Unavailable`, `Degraded`, `Unknown` y `Blank`.|  |
| `Link` | Representa un vínculo en el que se hace clic o realiza una acción configurable. Úselo opción si *solo* desea que el elemento sea un vínculo.  Cualquiera de los demás tipos *también* puede ser un vínculo mediante el valor `Make this item a link`. Para más información, vea la sección [Acciones de vínculo](#link-actions) a continuación. |  |
| `Location` | Representa un nombre descriptivo de región de Azure basándose en los identificadores de región. |  |
| `Resource type` | Representa una cadena de tipo de recurso descriptivo basándose en un identificador de tipo de recurso.  |  |
| `Resource` | Representa un nombre de recurso descriptivo y un vínculo basándose en un identificador de recurso.  | Opción para mostrar el icono de tipo de recurso  |
| `Resource group` | Representa un nombre de recursos descriptivo y un vínculo basándose en un identificador de grupo de recursos. Si el valor de la celda no es un grupo de recursos, se convertirá en uno.  | Opción para mostrar el icono de grupo de recursos  |
| `Subscription` | Representa un nombre descriptivo de la suscripción y un vínculo basándose en un identificador de suscripción.  Si el valor de la celda no es una suscripción, se convertirá en una.  | Opción para mostrar el icono de la suscripción.  |
| `Hidden` | Oculta la columna de la cuadrícula. Resulta útil cuando la consulta predeterminada devuelve más columnas de las necesarias pero no se desea un operador project-away. |  |

### <a name="link-actions"></a>Acciones de vínculo

Si el representador `Link` está seleccionado o la casilla *Convertir este elemento en vínculo* está seleccionada, el autor puede configurar una acción de vínculo que se producirá al seleccionar la celda. Por lo general, esto lleva al usuario a otra vista con contexto que proviene de la celda o puede abrir una dirección URL.

### <a name="custom-formatting"></a>Formato personalizado

Los libros también permiten a los usuarios establecer el formato de número de sus valores de celda. Para ello, puede hacer clic en la casilla *Formato personalizado* cuando esté disponible.

| Opción de formato | Explicación |
|:------------- |:-------------|
| `Units` | Unidades de la columna: varias opciones para porcentaje, recuentos, tiempo, bytes, recuento/tiempo, bytes/hora, etc. Por ejemplo, la unidad para un valor de 1234 se puede establecer en milisegundos y se representa como 1234 s. |
| `Style` | Formato para representarlo: decimal, divida, porcentaje. |
| `Show group separator` | Casilla para mostrar los separadores de grupos. Presenta 1234 como 1,234 en Estados Unidos. |
| `Minimum integer digits` | Número mínimo de dígitos enteros que se van a usar (el valor predeterminado es 1). |
| `Minimum fractional digits` | Número mínimo de dígitos fraccionarios que se van a usar (el valor predeterminado es 0).  |
| `Maximum fractional digits` | Número máximo de dígitos fraccionarios que se van a usar. |
| `Minimum significant digits` | Número mínimo de dígitos significativos que se van a usar (el valor predeterminado es 1). |
| `Maximum significant digits` | Número máximo de dígitos significativos que se van a usar. |
| `Custom text for missing values` | Cuando un punto de datos no tiene ningún valor, se muestra este texto personalizado en lugar de un espacio en blanco. |

### <a name="custom-date-formatting"></a>Formato de fecha personalizado

Cuando el autor ha especificado que una columna está establecida en el representador de fecha y hora, el autor puede especificar opciones de formato de fecha personalizado mediante la casilla *Formato de fecha personalizado*.

| Opción de formato | Explicación |
|:------------- |:-------------|
| `Style` | Formato para representar una fecha con formatos cortos, largos, completos o una hora con formatos de hora corto o largo. |
| `Show time as` | Permite al autor decidir entre mostrar la hora en la hora local (valor predeterminado) o como UTC. Según el estilo de formato de fecha seleccionado, es posible que no se muestre la información de zona horaria o UTC. |

## <a name="custom-column-width-setting"></a>Valor de ancho de columna personalizado

El autor puede personalizar el ancho de cualquier columna de la cuadrícula mediante el campo *Custom Column Width* (Ancho de columna personalizado) en *Configuración de columna*.

![Captura de pantalla de la configuración de columnas con el campo de ancho de columna personalizado indicado en un cuadro rojo](./media/workbooks-grid-visualizations/custom-column-width-setting.png)

Si el campo se deja en blanco a la izquierda, el ancho se determinará automáticamente en función del número de caracteres de la columna y del número de columnas visibles. La unidad predeterminada es "ch" (caracteres).

Al seleccionar el botón azul **(Ancho actual)** de la etiqueta, se rellenará el campo de texto con el ancho actual de la columna seleccionada. Si hay valor presente en el campo de ancho personalizado sin unidad de medida, se usará el valor predeterminado.

Las unidades de medida disponibles son:

| Unidad de medida | Definición           |
|:--------------------|:---------------------|
| ch                  | caracteres (predeterminada) |
| px                  | píxeles               |
| fr                  | unidades fraccionarias     |
| %                   | percentage           |

Validación de entrada: si se produce un error en la validación, se mostrará un mensaje con instrucciones rojo debajo del campo, pero el usuario todavía puede aplicar el ancho. Si un valor está presente en la entrada, se analizará. Si no se encuentra ninguna unidad de medida válida, se usará el valor predeterminado.

No hay ningún ancho mínimo y máximo, ya que esto se deja a la discreción del autor. El campo de ancho de columna personalizado está deshabilitado para las columnas ocultas.

## <a name="examples"></a>Ejemplos

### <a name="spark-lines-and-bar-underneath"></a>Minigráficos y barra debajo

En el ejemplo siguiente se muestran recuentos de solicitudes y su tendencia por nombre de solicitud.

```kusto
requests
| make-series Trend = count() default = 0 on timestamp from ago(1d) to now() step 1h by name
| project-away timestamp
| join kind = inner (requests
    | summarize Requests = count() by name
    ) on name
| project name, Requests, Trend
| order by Requests desc
```

[![Captura de pantalla de una cuadrícula basada en un registro con una barra debajo y un minigráfico](./media/workbooks-grid-visualizations/log-chart-grid-spark-line.png)](./media/workbooks-grid-visualizations/log-chart-grid-spark-line.png#lightbox)

### <a name="heatmap-with-shared-scales-and-custom-formatting"></a>Mapa térmico con escalas compartidas y formato personalizado

Este ejemplo muestra varias métricas de duración de la solicitud y sus recuentos. El representador de mapa térmico usa los valores mínimos establecidos en la configuración o calcula un valor mínimo y máximo para la columna, y asigna un color de fondo de la paleta seleccionada para la celda basándose en el valor de la celda en relación al valor mínimo y máximo de la columna.

```
requests
| summarize Mean = avg(duration), (Median, p80, p95, p99) = percentiles(duration, 50, 80, 95, 99), Requests = count() by name
| order by Requests desc
```

[![Captura de pantalla de una cuadrícula basada en un registro con un mapa térmico que tiene una escala compartida entre columnas](./media/workbooks-grid-visualizations/log-chart-grid-shared-scale.png)](./media/workbooks-grid-visualizations/log-chart-grid-shared-scale.png#lightbox)

En el ejemplo anterior, se usa una paleta compartida (verde o roja) y la escala para colorear las columnas (media, mediana, p80, p95 y p99). Paleta independiente (azul) utilizada para la columna de solicitud.

#### <a name="shared-scale"></a>Escala compartida

Para obtener una escala compartida:

1. Use expresiones regulares para seleccionar más de una columna a la que aplicar una configuración. Por ejemplo, establezca el nombre de la columna en `Mean|Median|p80|p95|p99` para seleccionarlas todas.
2. Elimine la configuración predeterminada de las columnas individuales.

Esto hará que la nueva configuración de varias columnas aplique su configuración para incluir una escala compartida.

[![Captura de pantalla de una configuración de cuadrícula basada en un registro para obtener una escala compartida entre columnas](./media/workbooks-grid-visualizations/log-chart-grid-shared-scale-settings.png)](./media/workbooks-grid-visualizations/log-chart-grid-shared-scale-settings.png#lightbox)

### <a name="icons-to-represent-status"></a>Iconos para representar el estado

En el ejemplo siguiente se muestra el estado personalizado de las solicitudes en función de la duración de p95.

```
requests
| summarize p95 = percentile(duration, 95) by name
| order by p95 desc
| project Status = case(p95 > 5000, 'critical', p95 > 1000, 'error', 'success'), name, p95
```

[![Captura de pantalla de una cuadrícula basada en un registro con un mapa térmico que tiene una escala compartida entre columnas utilizando la consulta anterior.](./media/workbooks-grid-visualizations/log-chart-grid-icons.png)](./media/workbooks-grid-visualizations/log-chart-grid-icons.png#lightbox)

Los nombres de icono compatibles incluyen: `cancelled`, `critical`, `disabled`, `error`, `failed`, `info`, `none`, `pending`, `stopped`, `question`, `success`, `unknown`, `warning` `uninitialized`, `resource`, `up`, `down`, `left`, `right`, `trendup`, `trenddown`, `4`, `3`, `2`, `1`, `Sev0`, `Sev1`, `Sev2`, `Sev3`, `Sev4`, `Fired`, `Resolved`, `Available`, `Unavailable`, `Degraded`, `Unknown` y `Blank`.

### <a name="using-thresholds-with-links"></a>Uso de umbrales con vínculos

Las instrucciones siguientes le mostrarán cómo usar umbrales con vínculos para asignar iconos y abrir distintos libros. Cada vínculo de la cuadrícula abrirá una plantilla de libro diferente para ese recurso de Application Insights.

1. Para cambiar el libro al modo de edición, seleccione el elemento de la barra de herramientas *Editar*.
2. Seleccione **Agregar** y, luego, *Agregar consulta*.
3. Cambie el *Origen de datos* a "JSON" y *Visualización* a "Cuadrícula".
4. Escriba la siguiente consulta.

```json
[ 
    { "name": "warning", "link": "Community-Workbooks/Performance/Performance Counter Analysis" },
    { "name": "info", "link": "Community-Workbooks/Performance/Performance Insights" },
    { "name": "error", "link": "Community-Workbooks/Performance/Apdex" }
]
```

5. Ejecute una consulta.
6. Seleccione **Configuración de columnas** para abrir la configuración.
7. Seleccione "nombre" en *Columnas*.
8. En *Representador de columnas*, elija "Umbrales".
9.  Escriba y elija lo siguiente en *Configuración del umbral*.
   
    | Operator | Valor   | Iconos   |
    |----------|---------|---------|
    | ==       | warning | Advertencia |
    | ==       | error   | Con error  |

    ![Captura de pantalla de la pestaña Editar la configuración de las columnas con la configuración anterior.](./media/workbooks-grid-visualizations/column-settings.png)

    Mantenga la fila predeterminada tal cual. Puede escribir el texto que desee. La columna Texto toma un formato de cadena como entrada y lo rellena con el valor de columna y la unidad si se especifica. Por ejemplo, si advertencia es el valor de la columna y el texto "¡vínculo de {0}{1}!", se mostrará como "¡vínculo de advertencia!".
10. Active la casilla *Convertir este elemento en vínculo*.
    1. En *Vista para abrir*, elija "Libro (plantilla)".
    2. En *El valor de vínculo procede de*, elija "vínculo".
    3. Active la casilla *Abrir vínculo en hoja de contexto*.
    4. Elija la siguiente configuración en *Configuración de vínculos del libro*.
        1. En *El identificador de plantilla procede de*, elija "Columna".
        2. En *Columna* elija "vínculo".

    ![Captura de pantalla de la configuración de vínculos con la configuración anterior.](./media/workbooks-grid-visualizations/make-this-item-a-link.png)

11. Seleccione "vínculo" en *Columnas*. En Configuración junto a *Representador de columnas*, seleccione **(Ocultar columna)** .
1. Para cambiar el nombre para mostrar de la columna "nombre", seleccione la pestaña **Etiquetas**. En la fila con el "nombre" *Id. de columna*, en *Etiqueta de columna, escriba el nombre que desea mostrar.
2. Seleccione **Aplicar**.

![Captura de pantalla de un umbral en la cuadrícula con la configuración anterior](./media/workbooks-grid-visualizations/thresholds-workbooks-links.png)

## <a name="fractional-units-percentages"></a>Porcentajes de unidades fraccionarias

La unidad fraccionaria (fr) es una unidad de medida dinámica utilizada comúnmente en varios tipos de cuadrículas. A medida que cambia el tamaño y la resolución de la ventana, también cambia el ancho fr.

En la captura de pantalla siguiente se muestra una tabla con ocho columnas con un ancho de 1fr en todas y cada una de ellas. A medida que cambia el tamaño de la ventana, el ancho de cada columna cambia proporcionalmente.

[![Captura de pantalla de las columnas de la cuadrícula con un valor de ancho de columna de 1fr cada una](./media/workbooks-grid-visualizations/custom-column-width-fr.png)](./media/workbooks-grid-visualizations/custom-column-width-fr.png#lightbox)

En la imagen siguiente se muestra la misma tabla, salvo que la primera columna está establecida en un 50 % de ancho. Esto establecerá la columna en la mitad del ancho de la cuadrícula total dinámicamente. El cambio de tamaño de la ventana seguirá conservando el ancho del 50 % a menos que el tamaño de la ventana sea demasiado pequeño. Estas columnas dinámicas tienen un ancho mínimo en función de su contenido. El 50 % restante de la cuadrícula se divide entre las ocho unidades fraccionarias totales. La columna "Tipo" siguiente se establece en 2fr, por lo que ocupa una cuarta parte del espacio restante. Como las otras columnas son 1fr cada una, ocupan un octavo de la mitad derecha de la cuadrícula.

[![Captura de pantalla de columnas en la cuadrícula con una columna con un valor de ancho de columna del 50 % y el resto con 1fr cada una](./media/workbooks-grid-visualizations/custom-column-width-fr2.png)](./media/workbooks-grid-visualizations/custom-column-width-fr2.png#lightbox)

La combinación de los anchos fr, %, px y ch es posible y funciona de forma similar a los ejemplos anteriores. Los anchos establecidos por las unidades estáticas (ch y px) son constantes difíciles que no cambiarán aunque se cambie la ventana o resolución. Las columnas establecidas por % ocuparán su porcentaje según el ancho total de la cuadrícula (puede que no sea exacto debido a los anchos previamente mínimos). Las columnas establecidas con fr solo dividirán el espacio de la cuadrícula restante basándose en el número de unidades fraccionarias que se les asigne.

[![Captura de pantalla de columnas de la cuadrícula con una serie de unidades de ancho diferentes utilizadas](./media/workbooks-grid-visualizations/custom-column-width-fr3.png)](./media/workbooks-grid-visualizations/custom-column-width-fr3.png#lightbox)

## <a name="next-steps"></a>Pasos siguientes

* Obtenga información sobre cómo crear un [árbol en los libros](workbooks-tree-visualizations.md).
* Obtenga información sobre cómo crear [parámetros de texto del libro](workbooks-text.md).
