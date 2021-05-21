---
title: Conexión a los datos de los servicios de almacenamiento en Azure
titleSuffix: Azure Machine Learning
description: Cree almacenes de datos y conjuntos de datos para conectarse de forma segura a los datos de los servicios de almacenamiento de Azure con Azure Machine Learning Studio.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: how-to
ms.author: yogipandey
author: ynpandey
ms.reviewer: nibaccam
ms.date: 09/22/2020
ms.custom: data4ml
ms.openlocfilehash: b85be179fa12e29ef17fe9fa488b22c3d8a3c69b
ms.sourcegitcommit: 32ee8da1440a2d81c49ff25c5922f786e85109b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109785122"
---
# <a name="connect-to-data-with-the-azure-machine-learning-studio"></a>Conexión a los datos con Azure Machine Learning Studio

En este artículo, aprenderá a acceder a los datos con [Azure Machine Learning Studio](overview-what-is-machine-learning-studio.md). Conéctese a los datos de los servicios de almacenamiento en Azure con [almacenes de datos de Azure Machine Learning](how-to-access-data.md) y luego empaquete esos datos para las tareas de los flujos de trabajo de ML con [conjuntos de datos de Azure Machine Learning](how-to-create-register-datasets.md).

En la tabla siguiente se definen y resumen las ventajas de los almacenes de datos y los conjuntos de datos. 

|Object|Descripción| Ventajas|   
|---|---|---|
|Almacenes de datos| Conéctese de forma segura al servicio de almacenamiento en Azure y almacene la información de conexión, como el identificador de suscripción y la autorización de token de su instancia de [Key Vault](https://azure.microsoft.com/services/key-vault/) asociada el área de trabajo. | Dado que la información se almacena de forma segura: <br><br> <li> No&nbsp;ponga&nbsp;en peligro las &nbsp;credenciales&nbsp; de &nbsp;autenticación&nbsp;o los orígenes de datos nativos. <li> Ya no es necesario codificarlos de forma rígida en los scripts.
|Conjuntos de datos| Mediante la creación de un conjunto de datos, puede crear una referencia a la ubicación del origen de datos, junto con una copia de sus metadatos. Con los conjuntos de datos puede: <br><br><li> Acceder a los datos durante el entrenamiento del modelo.<li> Compartir datos y colaborar con otros usuarios.<li> Aprovechar las bibliotecas de código abierto, como Pandas, para la exploración de datos. | Dado que los conjuntos de datos se evalúan de forma diferida y los datos permanecen en su ubicación existente, usted: <br><br><li>Mantener una sola copia de los datos en el almacenamiento.<li> No generar ningún costo de almacenamiento adicional. <li> No se arriesgará de forma no intencionada cambiando sus orígenes de datos originales.<li>Mejorará las velocidades de rendimiento del flujo de trabajo de ML. 

Para comprender dónde encajan los almacenes de datos y los conjuntos de datos en el flujo de trabajo global de acceso a datos de Azure Machine Learning, consulte el artículo [Acceso seguro a los datos](concept-data.md#data-workflow).

Para obtener una experiencia de Code First, consulte los artículos siguientes para usar el [SDK de Python de Azure Machine Learning](/python/api/overview/azure/ml/) para:
* [Conectarse a servicios de Azure Storage con almacenes de datos](how-to-access-data.md). 
* [Crear conjuntos de datos de Azure Machine Learning](how-to-create-register-datasets.md). 

## <a name="prerequisites"></a>Requisitos previos

- Suscripción a Azure. Si no tiene una suscripción de Azure, cree una cuenta gratuita antes de empezar. Pruebe la [versión gratuita o de pago de Azure Machine Learning](https://aka.ms/AMLFree).

- Acceso a [Azure Machine Learning Studio](https://ml.azure.com/).

- Un área de trabajo de Azure Machine Learning. [Cree un área de trabajo de Azure Machine Learning](how-to-manage-workspace.md).

    -  Al crear un área de trabajo, se registran automáticamente un contenedor de blobs de Azure y un recurso compartido de archivos de Azure como almacenes de datos en el área de trabajo. Se denominan `workspaceblobstore` y `workspacefilestore`, respectivamente. Si el almacenamiento de blobs es suficiente para sus necesidades, `workspaceblobstore` se establece como almacén de datos predeterminado y ya está configurado para su uso. De lo contrario, necesita una cuenta de almacenamiento en Azure con un [tipo de almacenamiento compatible](how-to-access-data.md#matrix).
    

## <a name="create-datastores"></a>Creación de almacenes de datos

Puede crear almacenes de datos a partir de [estas soluciones de Azure Storage](how-to-access-data.md#matrix). **En el caso de las soluciones de almacenamiento no compatibles**, y para ahorrar el costo de salida durante los experimentos de ML, debe [mover los datos](how-to-access-data.md#move) a una solución de Azure Storage compatible. [Más información sobre los almacenes de datos](how-to-access-data.md). 

Cree un nuevo almacén de datos en unos cuantos pasos con Azure Machine Learning Studio.

> [!IMPORTANT]
> Si la cuenta de almacenamiento de datos se encuentra en una red virtual, se necesitan pasos de configuración adicionales para garantizar que Studio tenga acceso a los datos. Vea [Aislamiento de red y privacidad](how-to-enable-studio-virtual-network.md) para asegurarse de que se aplican los pasos de configuración adecuados.

1. Inicie sesión en [Azure Machine Learning Studio](https://ml.azure.com/).
1. Seleccione **Almacenes de datos** en el panel izquierdo en **Administrar**.
1. Seleccione **+ Nuevo almacén de datos**.
1. Rellene el formulario para crear y registrar un nuevo almacén de datos. El formulario se actualiza de forma inteligente según las selecciones de tipo de Azure Storage y de autenticación. Consulte la [sección Permisos y acceso a Storage](#access-validation) para saber dónde encontrar las credenciales de autenticación que necesita para rellenar este formulario.

En el ejemplo siguiente se muestra el aspecto que tendría el formulario al crear un **almacén de Azure Blob**:

![Formulario para un nuevo almacén de datos](media/how-to-connect-data-ui/new-datastore-form.png)

## <a name="create-datasets"></a>Creación de conjuntos de datos

Después de crear un almacén de datos, cree un conjunto de datos para interactuar con los datos. Los conjuntos de datos empaquetan sus datos en un objeto consumible evaluado de forma diferida para tareas de aprendizaje automático, como un curso. [Más información sobre los conjuntos de datos](how-to-create-register-datasets.md).

Hay dos tipos de conjuntos de datos, FileDataset y TabularDataset. 
Los objetos [FileDataset](how-to-create-register-datasets.md#filedataset) crean referencias a uno o varios archivos o direcciones URL públicas. Mientras que los objetos [TabularDataset](how-to-create-register-datasets.md#tabulardataset) representan los datos en formato tabular. Puede crear objetos TabularDataset a partir de archivos .csv, .tsv, .parquet y .jsonl, así como de los resultados de una consulta SQL.

Los siguientes pasos y la animación muestran cómo crear un conjunto de datos en [Azure Machine Learning Studio](https://ml.azure.com).

> [!Note]
> Los conjuntos de datos creados mediante Azure Machine Learning Studio se registran automáticamente en el área de trabajo.

![Creación de un conjunto de datos con la interfaz de usuario](./media/how-to-connect-data-ui/create-dataset-ui.gif)

Para crear un conjunto de datos en el estudio:
1. Inicie sesión en [Azure Machine Learning Studio](https://ml.azure.com/).
1. Seleccione **Conjuntos de datos** en la sección **Recursos** del panel de la izquierda.
1. Seleccione **Crear conjunto de datos** para elegir el origen del conjunto de datos. Este origen puede ser un archivo local, un almacén de datos, una dirección URL pública o [Azure Open Datasets](../open-datasets/how-to-create-azure-machine-learning-dataset-from-open-dataset.md).
1. Seleccione **Tabular** o **Archivo** para el tipo de conjunto de datos.
1. Seleccione **Siguiente** para abrir el formulario **Datastore and file selection** (Almacén de datos y selección de archivos). En este formulario, seleccione dónde desea mantener el conjunto de datos después de su creación, así como los archivos de datos que se usarán para el conjunto de datos.
    1. Habilite la omisión de la validación si los datos están en una red virtual. Obtenga más información acerca del [aislamiento de red virtual y la privacidad](how-to-enable-studio-virtual-network.md).
    1. Para los conjuntos de datos tabulares, puede especificar un rasgo de "serie temporal" para permitir operaciones relacionadas con el tiempo en el conjunto datos. Aprenda a [agregar el rasgo de serie temporal al conjunto de datos](how-to-monitor-datasets.md#studio-dataset).
1. Seleccione **Siguiente** para rellenar los formularios **Settings and preview** (Configuración y versión preliminar) y **Esquema**; se rellenan de forma inteligente en función del tipo de archivo y se puede configurar el conjunto de archivos antes de la creación en estos formularios. 
1. Seleccione **Siguiente** para revisar el formulario **Confirmar detalles**. Compruebe sus selecciones y cree un perfil de datos opcional para el conjunto de datos. Más información acerca de la [generación de perfiles de datos](#profile).
1. Seleccione **Crear** para completar la creación del conjunto de datos.

<a name="profile"></a>

### <a name="data-profile-and-preview"></a>Perfil de datos y vista previa

Después de crear el conjunto de datos, compruebe que puede ver el perfil y la vista previa en Studio con los pasos siguientes. 

1. Inicie sesión en [Azure Machine Learning Studio](https://ml.azure.com/).
1. Seleccione **Conjuntos de datos** en la sección **Recursos** del panel de la izquierda.
1. Seleccione el nombre del conjunto de datos que quiere ver. 
1. Seleccione la pestaña **Explore** (Explorar). 
1. Seleccione la pestaña **Vista previa** o **Perfil**. 

![Vista del perfil y vista previa del conjunto de datos](./media/how-to-connect-data-ui/dataset-preview-profile.gif)

Puede obtener una gran variedad de estadísticas de resumen en el conjunto de datos para comprobar si dicho conjunto está listo para ML. Para las columnas no numéricas, solo incluyen estadísticas básicas, como mínimo, máximo y recuento de errores. Para las columnas numéricas, también puede revisar sus momentos estadísticos y los cuantiles estimados. 

En concreto, el perfil de datos del conjunto de datos de Azure Machine Learning incluye:

>[!NOTE]
> Aparecen entradas en blanco para las características con tipos irrelevantes.

|Estadísticas|Descripción
|------|------
|Característica| Nombre de la columna que se está resumiendo.
|Perfil| Visualización en línea según el tipo inferido. Por ejemplo, las cadenas, los tipos booleanos y las fechas tendrán recuentos de valores, mientras que los tipos decimales (valores numéricos) tendrán histogramas aproximados. Esto le permite obtener una descripción rápida de la distribución de los datos.
|Distribución de tipo| Recuento de valor en línea de los tipos dentro de una columna. Los valores Null son su propio tipo, por lo que esta visualizaicón es útil para detectar los valores impares o que faltan.
|Tipo|Tipo inferido de la columna. Los valores posibles incluyen: cadenas, valores booleanos, fechas y decimales.
|Min| Valor mínimo de la columna. Aparecen entradas en blanco para características cuyo tipo no tiene una ordenación inherente (por ejemplo, valores booleanos).
|Max| Valor máximo de la columna. 
|Count| Número total de entradas que faltan y que no faltan en la columna.
|No falta el recuento| Número de entradas de la columna que no faltan. Las cadenas vacías y los errores se tratan como valores, por lo que no contribuirán a la lista "No falta el recuento".
|Cuantiles| Valores aproximados en cada cuantil para proporcionar una idea de la distribución de los datos.
|Media| Media aritmética o promedio de la columna.
|Desviación estándar| Medida de la cantidad de dispersión o variación de los datos de esta columna.
|Variance| Medida de la diferencia de los datos de esta columna con respecto a su valor medio. 
|Asimetría| Medida de la diferencia entre los datos de esta columna y la distribución normal.
|Curtosis| La medida de la cantidad de datos en cola de esta columna se compara con una distribución normal.

## <a name="storage-access-and-permissions"></a>Permisos y acceso a Storage

Para asegurarse de que se conecta de forma segura a su servicio Azure Storage, Azure Machine Learning requiere que tenga permiso para acceder al contenedor de almacenamiento de datos correspondiente. Este acceso depende de las credenciales de autenticación usadas para registrar el almacén de datos.

### <a name="virtual-network"></a>Virtual network

Si la cuenta de almacenamiento de datos se encuentra en una **red virtual**, se necesitan pasos de configuración adicionales para garantizar que Azure Machine Learning tenga acceso a los datos. Consulte [Aislamiento de red y privacidad](how-to-enable-studio-virtual-network.md) para asegurarse de que se aplican los pasos de configuración adecuados al crear y registrar el almacén de datos.  

### <a name="access-validation"></a>Validación de acceso

**Como parte del proceso de creación y registro del almacén de datos inicial**, Azure Machine Learning valida automáticamente que el servicio de almacenamiento subyacente exista y que la entidad de seguridad proporcionada por el usuario (nombre de usuario, entidad de servicio o token de SAS) tenga acceso al almacenamiento especificado.

**Una vez creado el almacén de datos**, esta validación solo se realiza para los métodos que requieren acceso al contenedor de almacenamiento subyacente, y **no** cada vez que se recuperan objetos del almacén de datos. Por ejemplo, la validación se produce si quiere descargar archivos del almacén de archivos. Sin embargo, no se produce si solo quiere cambiar el almacén de datos predeterminado.

Para autenticar su acceso al servicio de almacenamiento subyacente, puede proporcionar la clave de cuenta, tokens de firmas de acceso compartido (SAS) o una entidad de servicio en función del tipo de almacén de datos que quiere crear. La [matriz de tipo de almacenamiento](how-to-access-data.md#matrix) muestra los tipos de autenticación admitidos que corresponden a cada tipo de almacén de datos.

Encontrará información sobre la clave de cuenta, el token de SAS y la entidad de servicio en [Azure Portal](https://portal.azure.com).

* Si tiene previsto usar una clave de cuenta o un token de SAS para la autenticación, seleccione **Cuentas de almacenamiento** en el panel izquierdo y elija la cuenta de almacenamiento que quiere registrar.
  * La página **Información general** proporciona información como el nombre de la cuenta, el contenedor y el nombre del recurso compartido de archivos.
      1. En el caso de las claves de cuenta, vaya a **Claves de acceso** en el panel **Configuración**.
      1. En el caso de los tokens de SAS, vaya a **Firmas de acceso compartido** en el panel **Configuración**.

* Si piensa usar una [entidad de servicio](../active-directory/develop/howto-create-service-principal-portal.md) para la autenticación, vaya a **Registros de aplicaciones** y seleccione la aplicación que quiere usar.
    * Su página de **información general** correspondiente contendrá la información necesaria, como el id. de inquilino y de cliente.

> [!IMPORTANT]
> * Si necesita cambiar las claves de acceso de una cuenta de Azure Storage (clave de cuenta o token de SAS), asegúrese de sincronizar las credenciales nuevas con el área de trabajo y los almacenes de datos conectados a ella. Obtenga información sobre cómo [sincronizar las credenciales actualizadas](how-to-change-storage-access-key.md). <br> <br>
> * Si anula el registro y vuelve a registrar un almacén de datos con el mismo nombre y se produce un error, es posible que la instancia de Azure Key Vault del área de trabajo no tenga habilitada la eliminación temporal. De manera predeterminada, la eliminación temporal está habilitada para la instancia del almacén de claves que creó el área de trabajo, pero podría no estar habilitada si usó un almacén de claves existente, o si creó el área de trabajo antes de octubre de 2020. Para obtener información sobre cómo habilitar la eliminación temporal, consulte [Activación de la eliminación temporal de un almacén de claves existente](../key-vault/general/soft-delete-change.md#turn-on-soft-delete-for-an-existing-key-vault).

### <a name="permissions"></a>Permisos

Para el almacenamiento de Azure Data Lake Gen 2 y del contenedor de blobs de Azure, las credenciales de autenticación deben tener acceso al **Lector de datos de Storage Blob**. Obtenga más información sobre el [Lector de datos de Storage Blob](../role-based-access-control/built-in-roles.md#storage-blob-data-reader). Un token de SAS de cuenta no tiene de forma predeterminada ningún permiso. 
* Para el **acceso de lectura** de datos, las credenciales de autenticación deben tener un número mínimo de permisos de enumeración y lectura para contenedores y objetos. 

* Para el **acceso de escritura** de datos, también se necesitan los permisos de escritura y agregación.

## <a name="train-with-datasets"></a>Entrenamiento con conjuntos de datos

Use sus conjuntos de datos en los experimentos de aprendizaje automático para entrenar modelos de aprendizaje automático. [Más información sobre cómo entrenar con conjuntos de datos](how-to-train-with-datasets.md).

## <a name="next-steps"></a>Pasos siguientes

* [Ejemplo de entrenamiento paso a paso con TabularDatasets y ML automatizado](tutorial-first-experiment-automated-ml.md).

* [Entrenamiento de un modelo](how-to-set-up-training-targets.md).

* Para ver más ejemplos de entrenamiento de conjuntos de datos, consulte los [cuadernos de ejemplo](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/work-with-data/).
