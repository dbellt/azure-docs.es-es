---
title: Etiquetado de imágenes y de texto
titleSuffix: Azure Machine Learning
description: Aprenda a crear y ejecutar proyectos para etiquetar imágenes o texto. Para ayudar con la tarea, use el etiquetado asistido por ML o el etiquetado con intervención humana.
author: sdgilley
ms.author: sgilley
ms.service: machine-learning
ms.subservice: core
ms.topic: how-to
ms.date: 04/29/2021
ms.custom: data4ml
ms.openlocfilehash: c4edd4317bf125b4aa8dd8ebf404613c7fab3ba8
ms.sourcegitcommit: fc9fd6e72297de6e87c9cf0d58edd632a8fb2552
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108290499"
---
# <a name="create-a-data-labeling-project-and-export-labels"></a>Creación de un proyecto de etiquetado de datos y exportación de etiquetas

Aprenda a crear y ejecutar proyectos para etiquetar imágenes o datos de texto en Azure Machine Learning.  Para ayudar con la tarea, use el etiquetado asistido por aprendizaje automático o el etiquetado con intervención humana.


## <a name="data-labeling-capabilities"></a>Funcionalidades del etiquetado de datos

> [!Important]
> Las imágenes de datos o texto deben estar disponibles en un almacén de datos de blobs de Azure. (Si no tiene un almacén de datos existente, puede cargar los archivos durante la creación del proyecto).

Los datos de imagen pueden ser archivos de cualquiera de estos tipos: ".jpg", ".jpeg", ".png", ".jpe", ".jfif", ".bmp", ".tif", ".tiff". Cada archivo es un elemento que se va a etiquetar.
Los datos de texto pueden ser archivos ".txt" o ".csv".

* En el caso de los archivos ".txt", cada archivo representa un elemento que se va a etiquetar.
* En el de los archivos ".csv", cada fila del archivo es un elemento que se va a etiquetar.

El etiquetado de datos de Azure Machine Learning es una ubicación central para crear, administrar y supervisar proyectos de etiquetado:

- Coordine los datos, las etiquetas y los miembros del equipo para administrar de forma eficaz las tareas de etiquetado. 
- Realiza un seguimiento del progreso y mantiene la cola de tareas de etiquetado incompletas.
- Inicie y detenga el proyecto y controle el progreso de etiquetado.
- Examine los datos etiquetados y expórtelos en formato COCO o como un conjunto de datos de Azure Machine Learning.

## <a name="prerequisites"></a>Requisitos previos

- Los datos que quiere etiquetar, ya sea en archivos locales o en el almacenamiento de blobs de Azure.
- Conjunto de etiquetas que quiere aplicar.
- Instrucciones para el etiquetado.
- Suscripción a Azure. Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://aka.ms/AMLFree) antes de empezar.
- Un área de trabajo de Machine Learning. Consulte [Creación de un área de trabajo de Azure Machine Learning](how-to-manage-workspace.md).

## <a name="create-a-data-labeling-project"></a>Creación de un proyecto de etiquetado de datos

Los proyectos de etiquetado se administran desde Azure Machine Learning. Use la página **Proyectos de etiquetado** para administrar sus proyectos.

Si los datos ya están en el almacenamiento de blobs de Azure, debe hacer que estén disponibles como un almacén de datos antes de crear el proyecto de etiquetado. Para obtener un ejemplo del uso de un almacén de datos, consulte [Tutorial: Creación de un proyecto de etiquetado para la clasificación de imágenes](tutorial-labeling.md).

Para crear un proyecto, seleccione **Agregar proyecto**. Asigne un nombre adecuado al proyecto y seleccione **Tipo de tarea de etiquetado**. El nombre del proyecto no se puede reutilizar, ni siquiera si el proyecto se elimina en el futuro.

### <a name="image-labeling-project"></a>Proyecto de etiquetado de imágenes

* Seleccione **Imagen** para crear un proyecto de etiquetado de imágenes.

    :::image type="content" source="media/how-to-create-labeling-projects/labeling-creation-wizard.png" alt-text="Creación de un proyecto de etiquetado para el etiquetado de imágenes":::

  * Elija un proyecto de tipo **Clasificación de imágenes con varias clases** cuando quiera aplicar una *sola etiqueta* de un conjunto de etiquetas a una imagen.
  * Elija un proyecto de tipo **Clasificación de imágenes con varias etiquetas** cuando quiera aplicar *una o varias* etiquetas de un conjunto de etiquetas a una imagen. Por ejemplo, una fotografía de un perro podría etiquetarse como *perro* y *diurno*.
  * Elija un proyecto de tipo **Identificación del objeto (rectángulo de selección)** cuando quiera asignar una etiqueta y un rectángulo de selección a cada objeto de una imagen.
  * Elija un proyecto de tipo **Segmentación de instancias (polígono)** cuando desee asignar una etiqueta y dibujar un polígono alrededor de cada objeto dentro de una imagen.

    
* Seleccione **Siguiente** cuando esté listo para continuar.

### <a name="text-labeling-project-preview"></a>Proyecto de etiquetado de texto (versión preliminar)

> [!IMPORTANT]
> El etiquetado de texto se encuentra actualmente en versión preliminar pública.
> Se ofrece la versión preliminar sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas.
> Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

* Seleccione **Texto** para crear un proyecto de etiquetado de texto.

    :::image type="content" source="media/how-to-create-labeling-projects/text-labeling-creation-wizard.png" alt-text="Creación de un proyecto de etiquetado para el etiquetado de texto":::

    * Elija **Clasificación de texto con varias clases (versión preliminar)** cuando quiera aplicar una *sola etiqueta* de un conjunto de etiquetas a cada fragmento de texto.
    * Elija **Clasificación de texto con varias etiquetas (versión preliminar)** cuando quiera aplicar *una o varias* etiquetas de un conjunto de etiquetas a un fragmento de texto.

* Seleccione **Siguiente** cuando esté listo para continuar.

## <a name="specify-the-data-to-label"></a>Especificación de los datos que se van a etiquetar

Si ya ha creado un conjunto de datos que contiene los datos, selecciónelo en la lista desplegable **Seleccione un conjunto de datos existente**. O bien, seleccione **Crear un conjunto de datos** para usar un almacén de información de Azure existente o cargar archivos locales.

> [!NOTE]
> Un proyecto no puede contener más de 500 000 archivos.  Si el conjunto de datos supera ese número, solo se cargarán los primeros 500 000 archivos.  

### <a name="create-a-dataset-from-an-azure-datastore"></a>Creación de un conjunto de datos a partir de un almacén de datos de Azure

En muchos casos, basta con cargar archivos locales. Sin embargo, [Explorador de Azure Storage](https://azure.microsoft.com/features/storage-explorer/) es una forma más rápida y eficaz de transferir una gran cantidad de datos. Se recomienda usar Explorador de Storage de forma predeterminada para migrar archivos.

Para crear un conjunto de datos a partir de los datos que ya ha almacenado en el almacenamiento de blobs de Azure:

1. Seleccione **Crear un conjunto de datos** > **De almacén de datos**.
1. En **Nombre**, asigne un nombre al conjunto de datos.
1. Elija el **tipo de conjunto de datos**.  Solo se admiten tipos de conjunto de datos de archivo para imágenes. Los tipos archivo y tabular están disponibles para el etiquetado de texto.
1. Seleccione el almacén de datos.
1. Si los datos están en una subcarpeta del almacenamiento de blobs, elija **Examinar** para seleccionar la ruta de acceso.
    * Anexe "/**" a la ruta de acceso para incluir todos los archivos que haya en las subcarpetas de la ruta de acceso seleccionada.
    * Anexe "* */* .*" para incluir todos los datos que haya en el contenedor actual y sus subcarpetas.
1. Proporcione una descripción para el conjunto de datos.
1. Seleccione **Next** (Siguiente).
1. Confirme los detalles. Seleccione **Atrás** para modificar la configuración o **Crear** para crear el conjunto de datos.

### <a name="create-a-dataset-from-uploaded-data"></a>Creación de un conjunto de datos a partir de los datos cargados

Para cargar los datos directamente:

1. Seleccione **Crear un conjunto de datos** > **De archivos locales**.
1. En **Nombre**, asigne un nombre al conjunto de datos.
1. Elija el **tipo de conjunto de datos**.  Solo se admiten tipos de conjunto de datos de archivo para imágenes. Los tipos archivo y tabular están disponibles para el etiquetado de texto.
1. *Opcional:* Seleccione **Configuración avanzada** para personalizar el almacén de datos, el contenedor y la ruta de acceso a los datos.
1. Seleccione **Examinar** para seleccionar los archivos locales que se van a cargar.
1. Proporcione una descripción para el conjunto de datos.
1. Seleccione **Next** (Siguiente).
1. Confirme los detalles. Seleccione **Atrás** para modificar la configuración o **Crear** para crear el conjunto de datos.

Los datos se cargan en el almacén de blobs predeterminado ("workspaceblobstore") del área de trabajo de Machine Learning.

## <a name="configure-incremental-refresh"></a><a name="incremental-refresh"> </a>Configuración de la actualización incremental

Si planea agregar nuevos archivos al conjunto de datos, utilice la actualización incremental para agregar estos archivos al proyecto.   Cuando la opción de **actualización incremental**  está habilitada, se comprueba periódicamente si hay nuevas imágenes el conjunto de datos que puedan agregarse a un proyecto en función de la tasa de finalización de la etiqueta.   La comprobación de los nuevos datos se detiene cuando el proyecto alcanza el número máximo de 500 000 archivos.

Para agregar más archivos al proyecto, use el [Explorador de Azure Storage](https://azure.microsoft.com/features/storage-explorer/) para cargarlas en la carpeta adecuada del almacenamiento de blobs. 

Active la casilla **Habilitar la actualización incremental** cuando desee que el proyecto supervise continuamente los nuevos datos en el almacén de datos. Estos datos se extraerán en el proyecto una vez día, cuando se habilita, por lo que tras agregar nuevos datos al almacén de datos tendrá que esperar un tiempo para que se muestren en el proyecto.  Puede ver una marca de tiempo de la última vez que se actualizaron los datos vez en la sección **Actualización incremental** de la pestaña **Detalles** del proyecto.

Anule la selección de esta casilla si no desea que los archivos nuevos que aparecen en el almacén de datos se agreguen a su proyecto.

## <a name="specify-label-classes"></a>Especificación de clases de etiquetas

En la página **Clases de etiquetas**, especifique el conjunto de clases para clasificar los datos. La precisión y velocidad de los etiquetadores resultan afectadas por su capacidad para elegir entre las clases. Por ejemplo, en lugar de deletrear el género y la especie completos para plantas o animales, use códigos de campo o abrevie el género.

Escriba una etiqueta por fila. Use el botón **+** para agregar una nueva fila. Si tiene más de 3 o 4 etiquetas pero menos de 10, quizás quiera añadir a los nombres un prefijo de número ("1:", "2:") para que los etiquetadores puedan usar las teclas numéricas para acelerar su trabajo.

## <a name="describe-the-data-labeling-task"></a>Descripción de la tarea de etiquetado de datos

Es importante explicar claramente la tarea de etiquetado. En la página **Instrucciones de etiquetado**, puede agregar un vínculo a un sitio externo que contenga las instrucciones de etiquetado o bien incluir instrucciones en el cuadro de edición de la página. Mantenga las instrucciones orientadas a tareas y adecuadas para el público. Tenga en cuenta estas preguntas:

* ¿Cuáles son las etiquetas que verán y cómo las elegirán? ¿Hay un texto de referencia que puedan consultar?
* ¿Qué debe hacer si ninguna etiqueta parece adecuada?
* ¿Qué debe hacer si varias etiquetas parecen adecuadas?
* ¿Qué umbral de confianza debe aplicarse a una etiqueta? ¿Quiere que use una conjetura si no está seguro?
* ¿Qué debe hacer con los objetos de interés ocluidos o superpuestos?
* ¿Qué debe hacer si un objeto de interés se corta con el borde de la imagen?
* ¿Qué deben hacer después de enviar una etiqueta si creen que han cometido un error?

En el caso de los rectángulos de selección, estas son algunas preguntas importantes:

* ¿Cómo se define el cuadro de límite para esta tarea? ¿Debe estar totalmente en el interior del objeto o en el exterior? ¿Debe recortarse lo más cerca posible o hay algo de margen?
* ¿Qué nivel de cuidado y coherencia espera que los etiquetadores apliquen para definir los rectángulos de selección?
* ¿Cómo se etiqueta el objeto que se muestra parcialmente en la imagen? 
* ¿Cómo se etiqueta el objeto parcialmente cubierto por otro objeto?

>[!NOTE]
> Recuerde que los etiquetadores podrán seleccionar las 9 primeras etiquetas usando las claves numéricas de 1 a 9.

## <a name="use-ml-assisted-data-labeling"></a>Uso del etiquetado de datos asistido por Machine Learning

La página **Etiquetado asistido por ML** permite desencadenar modelos de Machine Learning automáticos para acelerar la tarea de etiquetado. Solo está disponible para el etiquetado de imágenes.

Al principio del proyecto de etiquetado, los elementos se presentan en orden aleatorio para reducir el posible sesgo. Sin embargo, los sesgos presentes en el conjunto de datos se reflejarán en el modelo entrenado. Por ejemplo, si el 80 % de los elementos son de una sola clase, aproximadamente el 80 % de los datos usados para entrenar el modelo serán de esa clase. Este entrenamiento no incluye el aprendizaje activo.

Seleccione *Habilitar el etiquetado con asistencia de ML* y especifique una GPU para habilitar el etiquetado con asistencia, que consta de dos fases:

* Agrupación en clústeres (para el etiquetado de imágenes)
* Etiquetado previo 

El número exacto de datos con etiqueta necesarios para iniciar el etiquetado con asistencia no es un número fijo.  Puede variar significativamente de un proyecto de etiquetado a otro. En algunos proyectos, a veces es posible ver tareas previas de etiquetado o de agrupación en clústeres después de que se hayan etiquetado manualmente 300 elementos. El etiquetado con asistencia de ML usa una técnica llamada *transferencia de aprendizaje*, que usa un modelo entrenado previamente para iniciar el proceso de entrenamiento. Si las clases del conjunto de datos son similares a las del modelo entrenado previamente, puede haber etiquetas previas disponibles después de unos pocos cientos de elementos etiquetados manualmente. Si el conjunto de datos es significativamente diferente de los datos utilizados para entrenar previamente el modelo, puede tardar mucho más.

Como las etiquetas finales se siguen basando en la entrada del etiquetador, a veces esta tecnología se denomina etiquetado *con intervención humana*.

> [!NOTE]
> El etiquetado de datos asistido mediante ML no es compatible con las cuentas de almacenamiento predeterminadas que estén protegidas en una [red virtual](how-to-network-security-overview.md). Debe usar una cuenta de almacenamiento no predeterminada para el etiquetado de datos asistidos mediante ML. La cuenta de almacenamiento no predeterminada se puede proteger en la red virtual.

### <a name="clustering"></a>Agrupación en clústeres

Después de haber enviado un determinado número de etiquetas, el modelo de clasificación de Machine Learning empezará a agrupar elementos similares.  Estas imágenes similares se presentan a los etiquetadores en la misma pantalla, para acelerar el etiquetado manual. La agrupación en clústeres es especialmente útil cuando el etiquetador está viendo una cuadrícula de 4, 6 o 9 imágenes.

Una vez que se ha entrenado un modelo de Machine Learning con los datos etiquetados manualmente, el modelo se trunca a su última capa totalmente conectada. A continuación, las imágenes sin etiquetar pasan a través del modelo truncado en un proceso comúnmente conocido como "incrustación" o "caracterización". El proceso incrusta cada imagen en un espacio de alta dimensión definido por esta capa de modelo. Las imágenes más próximas en el espacio se usan para las tareas de agrupación en clústeres. 

La fase de agrupación en clústeres no aparece para los modelos de detección de objetos o clasificación de texto.

### <a name="prelabeling"></a>Etiquetado previo

Después de haber enviado un número suficiente de etiquetas, se usará un modelo de clasificación para predecir las etiquetas. O bien, se utilizará un modelo de detección de objetos para predecir los rectángulos de selección. Ahora el etiquetador ve las páginas que contienen las etiquetas predichas ya presentes en cada elemento. También se muestran cuadros de predicción para la detección de objetos. La tarea siguiente consiste en revisar estas predicciones y corregir cualquier imagen que se haya etiquetado incorrectamente antes de enviar la página.  

Una vez que se ha entrenado un modelo de Machine Learning en los datos etiquetados manualmente, el modelo se evalúa en un conjunto de prueba de elementos etiquetados manualmente para determinar su precisión en distintos umbrales de confianza. Este proceso de evaluación se usa para determinar un umbral de confianza por encima del cual el modelo es lo suficientemente preciso como para mostrar las etiquetas previas. A continuación, el modelo se evalúa con datos sin etiquetar. Los elementos con predicciones más confiables que este umbral se usan para la etiquetado previo.

## <a name="initialize-the-data-labeling-project"></a>Inicialización del proyecto de etiquetado de datos

Una vez inicializado el proyecto de etiquetado, algunos aspectos del proyecto son inmutables. No se puede cambiar el tipo de tarea ni el conjunto de datos. Se *pueden* modificar las etiquetas y la dirección URL de la descripción de la tarea. Repase atentamente la configuración antes de crear el proyecto. Después de enviar el proyecto, volverá a la página principal **Data Labelling** (Etiquetado de datos), que muestra el proyecto como **Initializing** (Inicializando).

> [!NOTE]
> Puede que esta página no se actualice automáticamente. Espere unos momentos y actualice la página manualmente para ver el estado del proyecto como **Creado**.

## <a name="run-and-monitor-the-project"></a>Ejecutar y supervisar el proyecto
Después de inicializar el proyecto, Azure comenzará a ejecutarlo. Seleccione el proyecto en la página **Etiquetado de datos** para ver los detalles del proyecto.

Para pausar o reiniciar el proyecto, alterne el estado **En ejecución** en la parte superior derecha. Solo se pueden etiquetar datos cuando el proyecto se está ejecutando.

### <a name="dashboard"></a>Panel

En la pestaña **Panel** se muestra el progreso de la tarea de etiquetado.

:::image type="content" source="media/how-to-create-labeling-projects/labeling-dashboard.png" alt-text="Panel de etiquetado de datos":::

El gráfico de progreso muestra cuántos elementos se han etiquetado y cuántos no todavía.  Es posible que los elementos pendientes:

* Todavía no se hayan agregado a una tarea.
* Se hayan incluido en una tarea que está asignada a un etiquetador, pero que aún no se hayan completado. 
* Se encuentren en la cola de tareas que todavía se deben asignar.

En la sección central se muestra la cola de tareas que todavía se deben asignar. Cuando el etiquetado con asistencia de ML esté desactivado, en esta sección se mostrará el número de tareas manuales que se van a asignar. Cuando el etiquetado con asistencia de ML esté activado, también se mostrará:

* Tareas que contienen elementos agrupados en la cola
* Tareas que contienen elementos etiquetados previamente en la cola

Además, cuando el etiquetado con asistencia de ML está habilitado, se muestra una pequeña barra de progreso cuando se produce la siguiente ejecución de entrenamiento.  En la sección Experimentos se proporcionan vínculos para cada una de las ejecuciones de aprendizaje automático.

* Entrenamiento: Entrena un modelo para predecir las etiquetas.
* Validación: Determina si se usará la predicción de este modelo para etiquetar previamente los elementos. 
* Inferencia: Ejecución de predicción para nuevos elementos.
* Caracterización: Elementos de clúster (solo para proyectos de clasificación de imágenes).

En el lado derecho, se muestra una distribución de las etiquetas de las tareas que se han completado.  Recuerde que en algunos tipos de proyecto, un elemento puede tener varias etiquetas, en cuyo caso, el número total de etiquetas puede ser mayor que el número total de elementos.

### <a name="data-tab"></a>Pestaña Datos

En la pestaña **Datos**, puede ver el conjunto de datos y revisar los datos etiquetados. Si ve datos etiquetados incorrectamente, selecciónelos y elija **Rechazar**; se quitarán las etiquetas y los datos se volverán a colocar en la cola sin etiquetar.

### <a name="details-tab"></a>Pestaña Detalles

Vea los detalles del proyecto.  En esta pestaña, puede:

* Ver los detalles del proyecto y los conjuntos de datos de entrada
* Habilitar la actualización incremental
* Ver detalles del contenedor de almacenamiento usado para almacenar salidas etiquetadas en el proyecto
* Agregar etiquetas al proyecto
* Editar las instrucciones que proporcione a las etiquetas
* Editar los detalles del etiquetado con asistencia de ML, incluido habilitar o deshabilitar

### <a name="access-for-labelers"></a>Acceso para etiquetadores

Cualquier usuario que tenga acceso al área de trabajo puede etiquetar los datos del proyecto.  También puede personalizar los permisos de los etiquetadores para que puedan acceder al etiquetado, pero no a otras partes del área de trabajo o del proyecto de etiquetado.  Para obtener más información, consulte [Administración del acceso a un área de trabajo de Azure Machine Learning](how-to-assign-roles.md) y aprenda a crear el [rol personalizado de etiquetador](how-to-assign-roles.md#labeler).

## <a name="add-new-label-class-to-a-project"></a>Incorporación de una nueva clase de etiqueta a un proyecto

Durante el proceso de etiquetado de datos es posible que se necesiten más etiquetas para clasificar las imágenes.  Por ejemplo, puede que desee agregar una etiqueta "Desconocido" u "Otro" para indicar que las imágenes son confusas.

Siga estos pasos para agregar una o varias etiquetas a un proyecto:

1. Seleccione el proyecto en la página principal **Data Labeling** (Etiquetado de datos).
1. En la parte superior derecha de la página, cambie **En ejecución** a **En pausa** para detener la actividad de los etiquetadores.
1. Seleccione la pestaña **Details** (Detalles).
1. En la lista de la izquierda, seleccione **Label classes** (Clases de etiquetas).
1. En la parte superior de la lista, seleccione **+ Add Labels** (+ Agregar etiquetas) ![Add a label](media/how-to-create-labeling-projects/add-label.png) (Agregar una etiqueta).
1. En el formulario, agregue la nueva etiqueta y elija cómo continuar.  Como ha cambiado las etiquetas disponibles, elija cómo tratar los datos ya etiquetados:
    * Vuelva a empezar y quite todas las etiquetas existentes.  Elija esta opción si desea empezar a etiquetar desde el principio con el nuevo conjunto de etiquetas completo. 
    * Vuelva a empezar y conserve todas las etiquetas existentes.  Elija esta opción para marcar todos los datos como sin etiquetar, pero mantener las etiquetas existentes como predeterminadas para las imágenes etiquetadas previamente.
    * Continúe y conserve todas las etiquetas existentes. Elija esta opción para mantener todos los datos ya etiquetados como están y empezar a usar la nueva etiqueta para los datos que aún no se hayan etiquetado.
1. Modifique la página de instrucciones según sea necesario para las nuevas etiquetas.
1. Una vez que haya agregado todas las etiquetas nuevas, en la parte superior derecha de la página, cambie **En pausa** a **En ejecución** para reiniciar el proyecto.  

## <a name="export-the-labels"></a>Exportar las etiquetas
 
Use el botón **Exportar** en la página **Detalles del proyecto** del proyecto de etiquetado. En cualquier momento, puede exportar los datos de etiquetas para realizar experimentos de Machine Learning. 

* Las etiquetas de texto se pueden exportar como:
    * Un archivo CSV. El archivo CSV se crea en el almacén de blobs predeterminado del área de trabajo de Azure Machine Learning, en una carpeta dentro de *Labeling/export/csv*. 
    * Un [conjunto de datos de Azure Machine Learning con etiquetas](how-to-use-labeled-dataset.md). 

* Las etiquetas de imagen se pueden exportar como:
    * [Formato COCO](http://cocodataset.org/#format-data). El archivo COCO se crea en el almacén de blobs predeterminado del área de trabajo de Azure Machine Learning, en una carpeta dentro de *Labeling/export/coco*. 
    * Un [conjunto de datos de Azure Machine Learning con etiquetas](how-to-use-labeled-dataset.md). 

Acceda al conjunto de datos exportado de Azure Machine Learning en la sección **Conjuntos de datos** de Machine Learning. La página de detalles del conjunto de datos también proporciona código de ejemplo para acceder a las etiquetas desde Python.

![Conjunto de datos exportado](./media/how-to-create-labeling-projects/exported-dataset.png)

## <a name="troubleshooting"></a>Solución de problemas

Si observa alguno de estos problemas, use estas sugerencias.

|Problema  |Resolución  |
|---------|---------|
|Solo se pueden usar los conjuntos de datos creados en almacenes de datos de blobs.     |  Se trata de una limitación conocida de la versión actual.       |
|Después de la creación, el proyecto muestra el mensaje "Initializing" (Inicializando) durante mucho tiempo.     | Actualice manualmente la página. La inicialización debería continuar aproximadamente en 20 puntos de datos por segundo. La falta de actualización automática es un problema conocido.         |
|Al revisar imágenes, no se muestran las imágenes recién etiquetadas.     |   Para cargar todas las imágenes etiquetadas, elija el botón **Primera**. El botón **Primera** le llevará al principio de la lista, pero carga todos los datos etiquetados.      |
|Al presionar la tecla ESC mientras se etiqueta para la detección de objetos, se crea una etiqueta de tamaño cero en la esquina superior izquierda. El envío de etiquetas en este estado produce un error.     |   Haga clic en la cruz junto a la etiqueta para eliminarla.  |
|No se puede asignar un conjunto de tareas a un etiquetador concreto.     |   Se trata de una limitación conocida de la versión actual.  |

## <a name="next-steps"></a>Pasos siguientes

* [Tutorial: Creación de un proyecto de etiquetado para la clasificación de imágenes](tutorial-labeling.md).
* Etiquetado de imágenes para la [clasificación de imágenes o la detección de objetos](how-to-label-data.md)
* Obtenga más información sobre [Azure Machine Learning y Machine Learning Studio (clásico)](./overview-what-is-machine-learning-studio.md#ml-studio-classic-vs-azure-machine-learning-studio)
