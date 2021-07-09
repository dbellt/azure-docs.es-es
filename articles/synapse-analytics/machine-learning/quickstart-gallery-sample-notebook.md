---
title: 'Inicio rápido: Uso de un cuaderno de ejemplo de la galería de Synapse Analytics'
description: Aprenda a usar un cuaderno de ejemplo de la galería de Synapse Analytics para explorar datos y crear un modelo de aprendizaje automático.
services: synapse-analytics
ms.service: synapse-analytics
ms.subservice: machine-learning
ms.topic: quickstart
ms.reviewer: ''
ms.date: 06/11/2021
author: garyericson
ms.author: garye
ms.openlocfilehash: 3fadfc3fdead7249480b2ce841601149f750b958
ms.sourcegitcommit: 23040f695dd0785409ab964613fabca1645cef90
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112063798"
---
# <a name="quickstart-use-a-sample-notebook-from-the-synapse-analytics-gallery"></a>Inicio rápido: Uso de un cuaderno de ejemplo de la galería de Synapse Analytics

En este inicio rápido, aprenderá a copiar un cuaderno de ejemplo de aprendizaje automático de la galería de Synapse Analytics en el área de trabajo, a modificarlo y a ejecutarlo.
El cuaderno de ejemplo ingiere un conjunto de datos de Open Datasets de trayectos en taxi de Nueva York y usa la visualización para ayudarle a preparar los datos. Luego, entrena un modelo para predecir si habrá una propina en un viaje determinado.

En este cuaderno se muestran los pasos básicos usados para crear un modelo: **importación de datos**, **preparación de datos**, **entrenamiento del modelo** y **evaluación**. Puede usar este ejemplo como punto de partida para crear un modelo con sus propios datos.

## <a name="prerequisites"></a>Requisitos previos

* Necesitará un [área de trabajo de Azure Synapse Analytics](../get-started-create-workspace.md) con una cuenta de almacenamiento de Azure Data Lake Storage Gen2 que esté configurada como almacenamiento predeterminado. Asegúrese de que es el *colaborador de datos de Storage Blob* en el sistema de archivos de Data Lake Storage Gen2 con el que trabaja.
* Un [grupo de Spark](../get-started-analyze-spark.md) en el área de trabajo de Azure Synapse Analytics.

## <a name="copy-the-notebook-to-your-workspace"></a>Copia del cuaderno en el área de trabajo

1. Abra el área de trabajo y seleccione **Aprender** en la página principal.
1. En **Centro de conocimientos**, seleccione **Examinar la galería**.
1. En la galería, seleccione **Cuadernos**.
1. Busque y seleccione el cuaderno "Data Exploration and ML Modeling - NYC taxi predict using Spark MLib".

   :::image type="content" source="media\quickstart-gallery-sample-notebook\gallery-select-ml-notebook.png" alt-text="Selección del cuaderno de ejemplo de aprendizaje automático de la galería.":::

1. Seleccione **Continuar**.
1. En la página de vista previa del cuaderno, seleccione **Abrir cuaderno**. El cuaderno de ejemplo se copia en el área de trabajo y se abre.

    :::image type="content" source="media\quickstart-gallery-sample-notebook\gallery-open-ml-notebook.png" alt-text="Apertura del cuaderno de ejemplo de aprendizaje automático en el área de trabajo.":::

1. En el menú **Asociar a** del cuaderno abierto, seleccione el grupo de Apache Spark.

## <a name="run-the-notebook"></a>Ejecución del cuaderno

El cuaderno se divide en varias celdas, cada una de las cuales realiza una función específica.
Puede ejecutar manualmente cada celda, ejecutar las celdas secuencialmente o seleccionar **Ejecutar todo** para ejecutar todas las celdas.

Estas son las descripciones de cada una de las celdas del cuaderno:

1. Importe las funciones de PySpark que usa el cuaderno.
1. **Ingest Date** (Fecha de ingesta): ingesta de datos del conjunto de datos **NycTlcYellow** de Azure Open Datasets en una trama de datos local para procesarlos. El código extrae los datos de un período de tiempo específico; puede modificar las fechas de inicio y finalización para obtener datos diferentes.
1. Muestree el conjunto de datos para acelerar el desarrollo. Puede modificar este paso para cambiar el tamaño de la muestra o la inicialización del muestreo.
1. **Exploratory Data Analysis** (Análisis exploratorio de los datos): presentación de los datos en vistas de gráficos. Así, puede tener una idea de qué preparación necesitan los datos antes de crear el modelo.
1. **Data Prep and Featurization** (Preparación y caracterización de datos): filtrado de los datos atípicos detectados a través de la visualización y creación de algunas variables derivadas útiles.
1. **Data Prep and Featurization Part 2** (Preparación y caracterización de datos, parte 2): eliminación de las columnas innecesarias y creación de algunas otras características.
1. **Encoding** (Codificación): conversión de las variables de cadena en números que espera el modelo de regresión logística.
1. **Generation of Testing and Training Data Sets** (Generación de conjuntos de datos de entrenamiento y pruebas): división de los datos en conjuntos de datos de entrenamiento por un lado y de pruebas por otro. Puede modificar la fracción y la inicialización de aleatorización que se usa para dividir los datos.
1. **Train the Model** (Entrenar el modelo): entrenamiento de un modelo de regresión logística y visualización de su métrica "Area under ROC" (Área bajo ROC) para ver cómo funciona el modelo. Este paso también guarda el modelo entrenado en caso de que quiera usarlo en otra parte.
1. **Evaluate and Visualize** (Evaluar y visualizar): trazado de la curva ROC del modelo para evaluarlo más a fondo.

## <a name="save-the-notebook"></a>Guardar el cuaderno

Para guardar el cuaderno, seleccione **Publicar** en la barra de comandos del área de trabajo.

## <a name="copying-the-sample-notebook"></a>Copia del cuaderno de ejemplo

Para realizar una copia de este cuaderno, haga clic en los puntos suspensivos de la barra de comandos superior y seleccione **Clonar** para crear una copia en el área de trabajo o **Exportar** para descargar una copia del archivo (`.ipynb`) del cuaderno.

:::image type="content" source="media\quickstart-gallery-sample-notebook\copy-notebook.png" alt-text="Realización de una copia del cuaderno con el comando Exportar o Clonar.":::

## <a name="clean-up-resources"></a>Limpieza de recursos

Para que la instancia de Spark se apague cuando haya terminado, finalice las sesiones conectadas (cuadernos). El grupo se cierra cuando se alcanza el **tiempo de inactividad** especificado en el grupo de Apache Spark. También puede decidir **finalizar la sesión** en la barra de estado en la parte superior derecha del cuaderno.

:::image type="content" source="media\quickstart-gallery-sample-notebook\stop-session.png" alt-text="Detención de la sesión.":::

## <a name="next-steps"></a>Pasos siguientes

* [Consulte más cuadernos de ejemplo de Synapse en GitHub](https://github.com/Azure-Samples/Synapse/tree/main/MachineLearning).
* [Machine Learning con Apache Spark](../spark/apache-spark-machine-learning-concept.md)
