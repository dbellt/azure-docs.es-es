---
title: Supervisión de flujos de datos de asignación
description: Cómo supervisar visualmente los flujos de datos de asignación en Azure Data Factory
author: kromerm
ms.author: makromer
ms.service: data-factory
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 04/11/2021
ms.openlocfilehash: 82aba428627cba1a3df26fc67c5da0cde52d368c
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107309074"
---
# <a name="monitor-data-flows"></a>Supervisión de flujos de datos

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Después de haber completado la compilación y depuración del flujo de datos, desea programarlo para ejecutarse según una programación en el contexto de una canalización. Puede programar la canalización de Azure Data Factory mediante desencadenadores. También puede usar la opción Desencadenar ahora desde el generador de canalización de Azure Data Factory para ejecutar una sola ejecución con el fin de probar el flujo de datos dentro del contexto de canalización.

Cuando se ejecuta la canalización, puede supervisar la canalización y todas las actividades contenidas en la canalización, incluida la actividad de Data Flow. Haga clic en el icono de supervisión en el panel izquierdo de la interfaz de usuario de Azure Data Factory. Puede ver una pantalla similar a la siguiente. Los iconos resaltados permiten profundizar en las actividades de la canalización, incluidas las actividades de Data Flow.

![Captura de pantalla que muestra los iconos que se deben seleccionar para las canalizaciones a fin de obtener más información.](media/data-flow/mon001.png "Supervisión de Data Flow")

Se muestran estadísticas en este nivel, así como los tiempos de ejecución y estado. El identificador de ejecución en el nivel de actividad es diferente al del nivel de canalización. El identificador de ejecución en el nivel anterior es para la canalización. Al seleccionar el icono de las gafas, se obtienen detalles de la ejecución del flujo de datos.

![Captura de pantalla que muestra el icono de las gafas para ver los detalles de la ejecución del flujo de datos.](media/data-flow/monitoring-details.png "Supervisión de Data Flow")

En la vista de supervisión del nodo gráfico, puede ver una versión simplificada de solo lectura del gráfico de flujo de datos.

![Captura de pantalla que muestra la versión de solo lectura del gráfico.](media/data-flow/mon003.png "Supervisión de Data Flow")

Este es un vídeo de información general sobre la supervisión del rendimiento de los flujos de datos desde la pantalla de supervisión de ADF:

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE4u4mH]

## <a name="view-data-flow-execution-plans"></a>Visualización de planes de ejecución de Data Flow

Cuando se ejecuta Data Flow en Spark, Azure Data Factory determina las rutas de acceso de código óptimo según la integridad del flujo de datos. Además, las rutas de ejecución pueden producirse en distintos nodos de escalabilidad horizontal y particiones de datos. Por lo tanto, el gráfico de supervisión representa el diseño del flujo, teniendo en cuenta la ruta de acceso de ejecución de las transformaciones. Al seleccionar los nodos individuales, puede ver "agrupaciones" que representan el código que se ejecutó juntos en el clúster. Los intervalos y recuentos que ve representan esos grupos en lugar de los pasos individuales del diseño.

![Captura de pantalla que muestra la página de un flujo de datos.](media/data-flow/mon004.png "Supervisión de Data Flow")

* Al seleccionar el espacio abierto de la ventana de supervisión, las estadísticas del panel inferior muestran los recuentos de filas y el tiempo de cada receptor y las transformaciones que dieron lugar a los datos de receptor de linaje de transformación.

* Cuando se seleccionan transformaciones individuales, recibe más comentarios en el panel derecho que muestra estadísticas de partición, recuentos de columna, sesgo (con qué uniformidad se distribuyen los datos entre particiones) y curtosis (cuántos picos tienen los datos).

* Al seleccionar el receptor en la vista del nodo, puede ver el linaje de columna. Hay tres métodos diferentes que las columnas acumulan a lo largo de su flujo de datos para colocarse en el receptor. Son las siguientes:

  * Calculado: Use la columna para el procesamiento condicional o dentro de una expresión en el flujo de datos, pero no la coloque en el receptor.
  * Derivado: La columna es una columna nueva que generó en el flujo, es decir, no estaba presente en el origen.
  * Asignada: La columna se origina desde el origen y la asigna a un campo de receptor.
  * Estado del flujo de datos: el estado actual de la ejecución.
  * Tiempo de inicio del clúster: la cantidad de tiempo necesaria para adquirir el entorno de proceso de Spark JIT para la ejecución del flujo de datos.
  * Número de transformaciones: el número de pasos de transformación que se ejecutan en el flujo
  
![Captura de pantalla que muestra la opción Actualizar.](media/data-flow/monitornew.png "Nueva supervisión de Data Flow")

## <a name="total-sink-processing-time-vs-transformation-processing-time"></a>Diferencias entre tiempo total de procesamiento de receptores y tiempo de procesamiento de la transformación

Cada fase de transformación incluye un tiempo total para que se complete, y el tiempo de ejecución de todas las particiones se suma. Al hacer clic en el receptor, verá "Tiempo de procesamiento del receptor". Este valor incluye el total del tiempo de transformación *más* el tiempo de E/S que se tarda en escribir los datos en el almacén de destino. La diferencia entre el tiempo de procesamiento del receptor y el total de la transformación es el tiempo de E/S para escribir los datos.

También puede ver el tiempo detallado del paso de transformación de cada partición si abre la salida JSON desde la actividad de flujo de datos en la vista de supervisión de la canalización de ADF. El archivo JSON contiene el tiempo en milisegundos de cada partición, mientras que la vista de supervisión de la experiencia de usuario es un intervalo agregado de las particiones sumadas:

```
 {
     "stage": 4,
     "partitionTimes": [
          14353,
          14914,
          14246,
          14912,
          ...
         ]
}
```

### <a name="sink-processing-time"></a>Tiempo de procesamiento del receptor

Cuando seleccione un icono de transformación del receptor en el mapa, el panel deslizante de la derecha mostrará un punto de datos adicional denominado "Tiempo de posprocesamiento" en la parte inferior. Esta es la cantidad de tiempo que se dedica a ejecutar el trabajo en el clúster de Spark *después* de cargar, transformar y escribir los datos. Esta cantidad puede incluir el cierre de grupos de conexiones, el apagado de controladores, la eliminación de archivos, la fusión de archivos, etc. Al realizar acciones en el flujo como "migrar archivos" y "enviar a un solo archivo", es probable que se muestre un aumento en el valor de tiempo de posprocesamiento.

* Duración de la fase de escritura: el tiempo para escribir los datos en una ubicación de almacenamiento provisional para Synapse SQL.
* Duración de SQL de operación de tabla: el tiempo dedicado a mover datos de tablas temporales a la tabla de destino.
* Duración de SQL previo y posterior: el tiempo empleado en ejecutar comandos SQL previos y posteriores.
* Duración de los comandos previos y posteriores: el tiempo dedicado a ejecutar cualquier operación previa o posterior para el origen o los receptores basados en archivos. Por ejemplo, mover o eliminar archivos después del procesamiento.
* Duración de la combinación: el tiempo empleado en combinar el archivo, los archivos de combinación se usan para los receptores basados en archivos al escribir en un solo archivo o cuando se usa el "nombre de archivo como datos de columna". Si se dedica mucho tiempo a esta métrica, debe evitar el uso de estas opciones.
  
## <a name="error-rows"></a>Filas de error

La habilitación del control de filas de error en el receptor de flujo de datos se reflejará en la salida de supervisión. Cuando se establece el receptor en "report success on error" (notificar éxito cuando hay error), la salida de supervisión mostrará el número de filas correctas y con error al hacer clic en el nodo de supervisión de receptores.

![La captura de pantalla muestra las filas de error.](media/data-flow/error-row-2.png "Error Row Monitoring Success (Éxito de supervisión de filas)")

Al seleccionar "report failure on error" (informar de error en caso de error), la misma salida solo se mostrará en el texto de salida de la supervisión de la actividad. Esto se debe a que la actividad de flujo de datos devolverá un error de ejecución y la vista de supervisión detallada no estará disponible.

![Captura de pantalla que muestra las filas de error en la actividad.](media/data-flow/error-rows-4.png "Error Row Monitoring Failure (Error de supervisión de filas)")

## <a name="monitor-icons"></a>Iconos de supervisión

Este icono significa que los datos de transformación se almacenaron en caché en el clúster, por lo que los intervalos y la ruta de acceso de ejecución se han tenido en cuenta:

![Captura de pantalla que muestra el icono de disco.](media/data-flow/mon005.png "Supervisión de Data Flow")

También ve los iconos de círculo verde en la transformación. Representan un recuento del número de receptores en los que fluyen los datos.
