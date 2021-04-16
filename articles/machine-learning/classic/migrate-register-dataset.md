---
title: 'ML Studio (clásico): Migración a Azure Machine Learning - Recompilación de conjuntos de datos'
description: Recompile los conjuntos de datos de Studio (clásico) en el diseñador de Azure Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: studio-classic
ms.topic: how-to
author: xiaoharper
ms.author: zhanxia
ms.date: 02/04/2021
ms.openlocfilehash: 9604fc9d862d94ba5e566753d2186d7d28aa37ee
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107308853"
---
# <a name="migrate-a-studio-classic-dataset-to-azure-machine-learning"></a>Migración de un conjunto de datos de Studio a Azure Machine Learning

En este artículo, aprenderá a migrar un conjunto de datos de Studio (clásico) a Azure Machine Learning. Para obtener más información sobre cómo migrar desde Studio (clásico), consulte el [artículo de información general sobre la migración](migrate-overview.md).

Cuenta con tres opciones para migrar un conjunto de datos a Azure Machine Learning. Lea cada sección para determinar cuál es la que mejor se adapta a su escenario.


|¿Donde están los datos? | Opción de migración  |
|---------|---------|
|En Studio (clásico)     |  Opción 1: [descargue el conjunto de datos de Studio (clásico) y cárguelo en Azure Machine Learning](#download-the-dataset-from-studio-classic).      |
|El almacenamiento en la nube     | Opción 2: [registre un conjunto de datos de un origen en la nube](#import-data-from-cloud-sources). <br><br>  Opción 3: [use el módulo de importación de datos para obtener datos de un origen en la nube](#import-data-from-cloud-sources).        |

> [!NOTE]
> Azure Machine Learning también admite [flujos de trabajo de código primero](../how-to-create-register-datasets.md) para crear y administrar conjuntos de datos. 

## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Un área de trabajo de Azure Machine Learning. [Cree un área de trabajo de Azure Machine Learning](../how-to-manage-workspace.md#create-a-workspace).
- El conjunto de datos de Studio (clásico) que se va a migrar.


## <a name="download-the-dataset-from-studio-classic"></a>Descarga del conjunto de datos de Studio (clásico)

La manera más sencilla de migrar un conjunto de datos de Studio (clásico) a Azure Machine Learning es descargar el conjunto de datos y registrarlo en Azure Machine Learning. Así se crea una nueva copia del conjunto de datos que se carga en un almacén de datos de Azure Machine Learning.

Puede descargar los siguientes tipos de conjuntos de datos de Studio (clásico) directamente.

* Texto sin formato (.txt)
* Valores separados por coma (CSV) con un encabezado (.csv) o sin encabezado (.nh.csv)
* Valores separados con tabulaciones (TSV) con un encabezado (.tsv) o sin encabezado (.nh.tsv)
* Archivo de Excel
* Archivo ZIP (.zip)

Para descargar los conjuntos de datos directamente, siga estos pasos:
1. Vaya al área de trabajo de Studio (clásico) ([https://studio.azureml.net](https://studio.azureml.net)).
1. En la barra de navegación izquierda, seleccione la pestaña **Conjuntos de datos**.
1. Seleccione los conjuntos de datos que desea descargar.
1. En la barra de acciones inferior, seleccione **Descargar**.

    ![Captura de pantalla que muestra cómo se descarga un conjunto de datos en Studio (clásico)](./media/migrate-register-dataset/download-dataset.png)

Con los siguientes tipos de datos debe utilizar el módulo **Convert to CSV**  (Convertir a CSV) para descargar conjuntos de datos.

* Datos de SVMLight (.svmlight) 
* Datos de formato de archivo de relación de atributos (.arff) 
* Archivo de área de trabajo u objeto de R (.RData)
* Tipo de conjunto de datos (.data). Es el tipo de datos interno de Studio (clásico) para la salida del módulo.

Para convertir el conjunto de datos en un archivo CSV y descargar los resultados, siga estos pasos:

1. Vaya al área de trabajo de Studio (clásico) ([https://studio.azureml.net](https://studio.azureml.net)).
1. Cree un experimento nuevo.
1. Arrastre y coloque el conjunto de datos que desea descargar en el lienzo.
1. Agregue un módulo **Convert to CSV** (Convertir a CSV).
1. Conecte el puerto de entrada de **Convert to CSV** (Convertir a CSV) al puerto de salida del conjunto de datos.
1. Ejecute el experimento.
1. Haga clic con el botón derecho en el módulo **Convert to CSV** (Convertir a CSV).
1. Seleccione **Results dataset** > **Download** (Conjunto de datos de resultados > Descargar).

    ![Captura de pantalla que muestra cómo configurar una canalización de conversión a CSV](./media/migrate-register-dataset/csv-download-dataset.png)

### <a name="upload-your-dataset-to-azure-machine-learning"></a>Carga del conjunto de datos en Azure Machine Learning

Después de descargar el archivo de datos, puede registrar el conjunto de datos en Azure Machine Learning:

1. Vaya a Estudio de Azure Machine Learning en [ml.azure.com](https://ml.azure.com).
1. En el panel de navegación izquierdo, seleccione la pestaña **Conjuntos de datos**.
1. Seleccione **Crear conjunto de datos** > **De archivos locales**.
    ![Captura de pantalla que muestra la pestaña de conjuntos de datos y el botón para crear un archivo local](./media/migrate-register-dataset/register-dataset.png)
1. Escriba un nombre y una descripción.
1. En **Tipo de conjunto de datos**, seleccione **Tabular**.

    > [!NOTE]
    > También puede cargar archivos ZIP como conjuntos de datos. Para cargar un archivo ZIP, seleccione **Archivo** en **Tipo de conjunto de datos**.

1. En el formulario de **selección de almacén de datos y archivos**, seleccione el almacén de datos en el que desea cargar el archivo de conjunto de datos.

    De manera predeterminada, Azure Machine Learning almacena el conjunto de datos en el almacén de blobs del área de trabajo predeterminada. Para más información sobre los almacenes de datos, consulte [Conexión a los servicios de almacenamiento en Azure](../how-to-access-data.md).

1. Establezca la configuración de análisis de datos y el esquema del conjunto de datos. A continuación, confirme la configuración.

## <a name="import-data-from-cloud-sources"></a>Importación de datos de orígenes en la nube

Si los datos ya están en un servicio de almacenamiento en nube y desea conservarlos en su ubicación nativa, puede usar una de estas dos opciones:

|Método de ingesta|Descripción|
|---| --- |
|Registro de un conjunto de datos de Azure Machine Learning|Ingiere datos de orígenes de datos locales y en línea (blob, ADLS Gen1, ADLS Gen2, recurso compartido de archivos, SQL DB). <br><br>Crea una referencia al origen de datos, que se evalúa de forma diferida en tiempo de ejecución. Utilice esta opción si accede repetidamente a este conjunto de datos y desea habilitar características de datos avanzadas como el control de versiones y la supervisión de los datos.
|Módulo Importación de datos|Ingiere datos de orígenes de datos en línea (blob, ADLS Gen1, ADLS Gen2, recurso compartido de archivos, SQL DB). <br><br> El conjunto de datos solo se importa en la ejecución de canalización del diseñador actual.


>[!Note]
> Los usuarios de Studio (clásico) deben tener en cuenta que los siguientes orígenes en la nube no se admiten de forma nativa en Azure Machine Learning:
> - Consulta de Hive
> - tabla de Azure
> - Azure Cosmos DB
> - SQL Database en el entorno local
>
> Se recomienda que los usuarios migren sus datos a los servicios de almacenamiento compatibles mediante Azure Data Factory.  

### <a name="register-an-azure-machine-learning-dataset"></a>Registro de un conjunto de datos de Azure Machine Learning

Siga los pasos que se indican a continuación para registrar un conjunto de datos en Azure Machine Learning desde un servicio en la nube: 

1. [Cree un almacén de datos](../how-to-connect-data-ui.md#create-datastores) que vincule el servicio de almacenamiento en nube a su área de trabajo de Azure Machine Learning. 

1. [Registre un conjunto de datos](../how-to-connect-data-ui.md#create-datasets). Si va a migrar un conjunto de datos de Studio (clásico), seleccione el valor de conjunto de datos **Tabular**.

Después de registrar un conjunto de datos en Azure Machine Learning, puede usarlo en el diseñador:
 
1. Cree un nuevo borrador de canalización en el diseñador.
1. En la paleta de módulos de la izquierda, expanda la sección **Conjuntos de datos**.
1. Arrastre el conjunto de datos registrado al lienzo. 

### <a name="use-the-import-data-module"></a>Uso del módulo Import Data (Importar datos)

Siga estos pasos para importar datos directamente a la canalización del diseñador:

1. [Cree un almacén de datos](https://github.com/MicrosoftDocs/azure-docs-pr/blob/master/articles/machine-learning/how-to-connect-data-ui.md#create-datastores) que vincule el servicio de almacenamiento en nube a su área de trabajo de Azure Machine Learning. 

Después de crear el almacén de datos, puede usar el módulo [**Import Data**](../algorithm-module-reference/import-data.md) (Importar datos) en el diseñador para ingerir datos de él:

1. Cree un nuevo borrador de canalización en el diseñador.
1. En la paleta de módulos de la izquierda, localice el módulo **Import Data**  (Importar datos) y arrástrelo al lienzo.
1. Seleccione el módulo **Import Data** (Importar datos) y use la configuración del panel derecho para configurar el origen de datos.

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha aprendido a migrar un conjunto de datos de Studio (clásico) a Azure Machine Learning. El siguiente paso consiste en [recompilar una canalización de entrenamiento de Studio (clásico)](migrate-rebuild-experiment.md).


Consulte los demás artículos de la serie de migración de Studio (clásico):

1. [Información general sobre la migración](migrate-overview.md).
1. **Migración de conjuntos de datos**.
1. [Recompilación de una canalización de entrenamiento de Studio (clásico)](migrate-rebuild-experiment.md).
1. [Recompilación de un servicio web de Studio (clásico)](migrate-rebuild-web-service.md).
1. [Integración de un servicio web de Azure Machine Learning con aplicaciones cliente](migrate-rebuild-integrate-with-client-app.md).
1. [Migración de los módulos Execute R Script (Ejecutar script R) de Studio (Clásico)](migrate-execute-r-script.md).
