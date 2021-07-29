---
title: 'Inicio rápido: Transformación de datos mediante la definición de trabajo de Apache Spark'
description: En este tutorial se proporcionan instrucciones paso a paso para usar Azure Synapse Analytics con el objetivo de transformar los datos con la definición de trabajo de Apache Spark.
author: djpmsft
ms.author: daperlov
ms.reviewer: makromer
ms.service: synapse-analytics
ms.subservice: pipeline
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 05/13/2021
ms.openlocfilehash: f7a44ae2033d75c81f80bc9a37e17ed8fe2786b2
ms.sourcegitcommit: bb9a6c6e9e07e6011bb6c386003573db5c1a4810
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110501400"
---
# <a name="quickstart-transform-data-using-apache-spark-job-definition"></a>Inicio rápido: Transformación de datos mediante la definición de trabajo de Apache Spark.

En esta guía de inicio rápido, usará Azure Synapse Analytics para crear una canalización mediante la definición de trabajo de Apache Spark.

## <a name="prerequisites"></a>Prerrequisitos

* **Suscripción de Azure**: Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.
* **Área de trabajo de Azure Synapse**: Cree un área de trabajo de Synapse desde Azure Portal siguiendo las instrucciones que se indican en [Inicio rápido: Creación de un área de trabajo de Synapse](quickstart-create-workspace.md).
* **Definición de trabajo de Apache Spark**: cree una definición de trabajo de Apache Spark en el área de trabajo de Synapse siguiendo las instrucciones de [Tutorial:Creación de una definición de trabajo de Apache Spark en Synapse Studio](spark/apache-spark-job-definitions.md).


### <a name="navigate-to-the-synapse-studio"></a>Desplazamiento a Synapse Studio

Una vez creada el área de trabajo de Azure Synapse, hay dos maneras de abrir Synapse Studio:

* Abrir el área de trabajo de Synapse en [Azure Portal](https://ms.portal.azure.com/#home). Seleccione **Abrir** en la tarjeta Abrir Synapse Studio de Información general.
* Abra [Azure Synapse Analytics](https://web.azuresynapse.net/) e inicie sesión en su área de trabajo.

En este inicio rápido, se usa como ejemplo el área de trabajo denominada "sampletest". Se le dirigirá automáticamente a la página principal de Synapse Studio.

![página principal de Synapse Studio](media/quickstart-transform-data-using-spark-job-definition/synapse-studio-home.png)

## <a name="create-a-pipeline-with-an-apache-spark-job-definition"></a>Creación de una canalización con una definición de trabajo de Apache Spark

Una canalización contiene el flujo lógico para una ejecución de un conjunto de actividades. En esta sección, creará una canalización que contiene una actividad de definición de trabajo de Apache Spark.

1. Vaya a la ficha **Integrar**. Seleccione el icono de signo de más situado junto al encabezado de canalizaciones y seleccione Pipeline (Canalización).

     ![Creación de una canalización](media/doc-common-process/new-pipeline.png)

2. En la página de configuración **Propiedades** de la canalización, escriba **demo** en **Nombre**.

3. En *Synapse*, en el panel *Actividades*, arrastre la **definición del trabajo de Spark** al lienzo de la canalización.

     ![arrastre la definición de trabajos de Spark](media/quickstart-transform-data-using-spark-job-definition/drag-spark-job-definition.png)


## <a name="set-apache-spark-job-definition-canvas"></a>Establecimiento del lienzo de la definición de trabajo de Apache Spark

Una vez que haya creado la definición de trabajo de Apache Spark, se le enviará automáticamente al lienzo de la definición de trabajo de Spark.

### <a name="general-settings"></a>Configuración general

1. Seleccione el módulo de la definición de trabajo de Spark en el lienzo.

2. En la pestaña General, escriba **sample** en **Nombre**.

3. (Opción) También puede escribir una descripción.

4. Tiempo de espera: cantidad máxima de tiempo que se puede ejecutar una actividad. El valor predeterminado es siete días, que también es la cantidad máxima de tiempo permitida. El formato es D.HH:MM:SS.

5. Reintento: número máximo de reintentos.

6. Intervalo de reintento: número de segundos entre cada reintento.

7. Salida segura: cuando esté activada, la salida de la actividad no se capturará en el registro.

8. Entrada segura: cuando esté activada, la entrada de la actividad no se capturará en el registro.

     ![información general de la definición de trabajo de Spark](media/quickstart-transform-data-using-spark-job-definition/spark-job-definition-general.png)

### <a name="settings-tab"></a>Pestaña Settings 

En este panel, puede hacer referencia a la definición de trabajo de Spark que se va a ejecutar.

* Expanda la lista de definiciones de trabajo de Spark, puede elegir una definición de trabajo de Apache Spark existente. También puede crear una nueva definición de trabajo de Apache Spark al hacer clic en el botón Nuevo para hacer referencia a la definición de trabajo de Spark que se va a ejecutar.

* Puede agregar argumentos de línea de comandos al hacer clic en el botón **Nuevo**. Tome en cuenta que esto invalidará los argumentos de la línea de comandos definidos por la definición de trabajo de Spark. <br> *Ejemplo: `abfss://…/path/to/shakespeare.txt` `abfss://…/path/to/result`* <br>

     ![configuración de la canalización de la definición de trabajo de Spark](media/quickstart-transform-data-using-spark-job-definition/spark-job-definition-pipline-settings.png)

* Puede agregar contenido dinámico haciendo clic en el botón **Agregar contenido dinámico** o presionando la tecla de método abreviado <kbd>Alt</kbd>+<kbd>Mayús</kbd>+<kbd>D</kbd>. En la página **Agregar contenido dinámico**, puede usar cualquier combinación de expresiones, funciones y variables del sistema para agregar contenido dinámico.

     ![incorporación de contenido dinámico](media/quickstart-transform-data-using-spark-job-definition/add-dynamic-content.png)

### <a name="user-properties-tab"></a>Pestaña Propiedades de usuario

Puede agregar propiedades para la actividad de definición de trabajo de Apache Spark en este panel.

![propiedades de usuario](media/quickstart-transform-data-using-spark-job-definition/user-properties.png)

## <a name="next-steps"></a>Pasos siguientes

Avance a los artículos siguientes para aprender sobre la compatibilidad de Azure Synapse Analytics:

> [!div class="nextstepaction"]
> [Canalización y actividades](../data-factory/concepts-pipelines-activities.md?bc=%2fazure%2fsynapse-analytics%2fbreadcrumb%2ftoc.json&toc=%2fazure%2fsynapse-analytics%2ftoc.json)
> [Introducción a al flujo de datos de asignación](../data-factory/concepts-data-flow-overview.md?bc=%2fazure%2fsynapse-analytics%2fbreadcrumb%2ftoc.json&toc=%2fazure%2fsynapse-analytics%2ftoc.json)
> [Lenguaje de expresiones de flujo de datos](../data-factory/data-flow-expression-functions.md?bc=%2fazure%2fsynapse-analytics%2fbreadcrumb%2ftoc.json&toc=%2fazure%2fsynapse-analytics%2ftoc.json)