---
title: Conexión a los servicios de almacenamiento en Azure
titleSuffix: Azure Machine Learning
description: Aprenda a usar los almacenes de datos para conectarse de forma segura a los servicios de Azure Storage durante el entrenamiento con Azure Machine Learning
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: how-to
ms.author: yogipandey
author: ynpandey
ms.reviewer: nibaccam
ms.date: 11/03/2020
ms.custom: contperf-fy21q1, devx-track-python, data4ml
ms.openlocfilehash: f0fb8ee681bfb3056547eed712d0197100b8ec08
ms.sourcegitcommit: 32ee8da1440a2d81c49ff25c5922f786e85109b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109785248"
---
# <a name="connect-to-storage-services-on-azure"></a>Conexión a los servicios de almacenamiento en Azure

En este artículo, aprenderá a conectarse a los servicios de almacenamiento de datos de Azure con almacenes de datos de Azure Machine Learning y el [SDK de Azure Machine Learning para Python](/python/api/overview/azure/ml/intro).

Los almacenes de datos se conectan de forma segura al servicio de almacenamiento de Azure sin poner en riesgo las credenciales de autenticación ni la integridad del origen de datos original. Almacenan información de conexión, como el identificador de suscripción y la autorización de tokens de la instancia de [Key Vault](https://azure.microsoft.com/services/key-vault/) asociada con el área de trabajo, para que pueda acceder de forma segura al almacenamiento sin tener que codificarla de forma rígida en los scripts. Puede crear almacenes de datos que se conecten a [estas soluciones de almacenamiento de Azure](#matrix).

Para comprender el lugar de los almacenes de datos en el flujo de trabajo global de acceso a datos de Azure Machine Learning, consulte el artículo [Acceso seguro a los datos](concept-data.md#data-workflow).

Para tener una experiencia con poco código, consulte cómo usar [Estudio de Azure Machine Learning para crear y registrar almacenes de datos](how-to-connect-data-ui.md#create-datastores).

>[!TIP]
> En este artículo se supone que desea conectarse al servicio de almacenamiento con credenciales de autenticación basada en credenciales, como una entidad de servicio o un token de firma de acceso compartido (SAS). Tenga en cuenta que, si las credenciales se registran con almacenes de datos, todos los usuarios con el rol *Lector* del área de trabajo podrán recuperar estas credenciales. [Más información sobre el rol *Lector* del área de trabajo](how-to-assign-roles.md#default-roles). <br><br>Si esto supone un problema, aprenda a [conectarse a los servicios de almacenamiento con el acceso basado en identidad](how-to-identity-based-data-access.md). <br><br>Esta funcionalidad es una característica [experimental](/python/api/overview/azure/ml/#stable-vs-experimental) en versión preliminar y puede cambiar en cualquier momento. 

## <a name="prerequisites"></a>Requisitos previos

- Suscripción a Azure. Si no tiene una suscripción de Azure, cree una cuenta gratuita antes de empezar. Pruebe la [versión gratuita o de pago de Azure Machine Learning](https://aka.ms/AMLFree).

- Una cuenta de Azure Storage con un [tipo de almacenamiento compatible](#matrix).

- El [SDK de Azure Machine Learning para Python](/python/api/overview/azure/ml/intro).

- Un área de trabajo de Azure Machine Learning.
  
  [Cree un área de trabajo de Azure Machine Learning](how-to-manage-workspace.md) o use una existente mediante el SDK de Python. 

    Importe las clases `Workspace` y `Datastore`, y cargue la información de la suscripción desde el archivo `config.json` con la función `from_config()`. De forma predeterminada, busca el archivo JSON en el directorio actual, pero también puede especificar un parámetro de ruta de acceso para que apunte al archivo mediante `from_config(path="your/file/path")`.

   ```Python
   import azureml.core
   from azureml.core import Workspace, Datastore
        
   ws = Workspace.from_config()
   ```

    Al crear un área de trabajo, se registran automáticamente un contenedor de blobs de Azure y un recurso compartido de archivos de Azure como almacenes de datos en el área de trabajo. Se denominan `workspaceblobstore` y `workspacefilestore`, respectivamente. `workspaceblobstore` se usa para almacenar los artefactos del área de trabajo y los registros del experimento de aprendizaje automático. También se establece como el **almacén de datos predeterminado** y no se puede eliminar del área de trabajo. `workspacefilestore` se usa para almacenar cuadernos y scripts de R autorizados a través de la [instancia de proceso](./concept-compute-instance.md#accessing-files).
    
    > [!NOTE]
    > El diseñador de Azure Machine Learning creará automáticamente un almacén de datos denominado **azureml_globaldatasets** al abrir un ejemplo en la página principal del diseñador. Este almacén de datos solo contiene conjuntos de datos de ejemplo. **No** use este almacén de datos para el acceso a datos confidenciales.

<a name="matrix"></a>

## <a name="supported-data-storage-service-types"></a>Tipos de servicio de almacenamiento de datos admitidos

Los almacenes de datos admiten actualmente el almacenamiento de la información de conexión a los servicios de almacenamiento que se enumeran en la siguiente matriz. 

> [!TIP]
> **En el caso de las soluciones de almacenamiento no compatibles** y para ahorrar el costo de salida durante los experimentos de ML, [mueva los datos](#move) a una solución de almacenamiento de Azure compatible. 

| Tipo de&nbsp;almacenamiento | Tipo de&nbsp;autenticación | [Azure&nbsp;Machine&nbsp;Learning Studio](https://ml.azure.com/) | SDL de Python de [Azure&nbsp;Machine&nbsp;Learning Studio&nbsp;](/python/api/overview/azure/ml/intro) |  Cli de [Azure&nbsp;Machine&nbsp;Learning CLI](reference-azure-machine-learning-cli.md) | API Rest de [Azure&nbsp;Machine&nbsp;Learning&nbsp;](/rest/api/azureml/) | Código de VS
---|---|---|---|---|---|---
[Azure&nbsp;Blob&nbsp;Storage](../storage/blobs/storage-blobs-overview.md)| Clave de cuenta <br> Token de SAS | ✓ | ✓ | ✓ |✓ |✓
[Recurso compartido de &nbsp;archivos de&nbsp;Azure](../storage/files/storage-files-introduction.md)| Clave de cuenta <br> Token de SAS | ✓ | ✓ | ✓ |✓|✓
[Azure&nbsp;Data Lake&nbsp;Storage Gen&nbsp;1](../data-lake-store/index.yml)| Entidad de servicio| ✓ | ✓ | ✓ |✓|
[Azure&nbsp;Data Lake&nbsp;Storage Gen&nbsp;2](../storage/blobs/data-lake-storage-introduction.md)| Entidad de servicio| ✓ | ✓ | ✓ |✓|
[Azure&nbsp;SQL&nbsp;Database](../azure-sql/database/sql-database-paas-overview.md)| Autenticación SQL <br>Entidad de servicio| ✓ | ✓ | ✓ |✓|
[Azure&nbsp;PostgreSQL](../postgresql/overview.md) | Autenticación SQL| ✓ | ✓ | ✓ |✓|
[Azure&nbsp;Database&nbsp;for&nbsp;MySQL](../mysql/overview.md) | Autenticación SQL|  | ✓* | ✓* |✓*|
[Sistema&nbsp; de archivos de &nbsp;Databricks](/azure/databricks/data/databricks-file-system)| Sin autenticación | | ✓** | ✓ ** |✓** |

\* MySQL solo se admite para la canalización [DataTransferStep](/python/api/azureml-pipeline-steps/azureml.pipeline.steps.datatransferstep).<br />
\*\* Databricks solo se admite para la canalización [DatabricksStep](/python/api/azureml-pipeline-steps/azureml.pipeline.steps.databricks_step.databricksstep).


### <a name="storage-guidance"></a>Orientación sobre el almacenamiento

Le recomendamos que cree un almacén de datos para un [contenedor de blobs de Azure](../storage/blobs/storage-blobs-introduction.md). El almacenamiento Estándar y Premium están disponibles para blobs. Aunque el almacenamiento premium sea más costoso, su mayor velocidad de rendimiento puede mejorar la velocidad de las ejecuciones de entrenamiento, sobre todo si usa un gran conjunto de datos. Para obtener información sobre el costo de las cuentas de almacenamiento, vea la [Calculadora de precios de Azure](https://azure.microsoft.com/pricing/calculator/?service=machine-learning-service).

[Azure Data Lake Storage Gen2](../storage/blobs/data-lake-storage-introduction.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json) se basa en Azure Blob Storage y se ha diseñado para el análisis de macrodatos empresarial. Parte fundamental de Data Lake Storage Gen2 es la incorporación de un [espacio de nombres jerárquico](../storage/blobs/data-lake-storage-namespace.md) en Blob Storage. El espacio de nombres jerárquico organiza los objetos o archivos en una jerarquía de directorios para un acceso eficaz a los datos.

## <a name="storage-access-and-permissions"></a>Permisos y acceso a Storage

Para asegurarse de que se conecta de forma segura a su servicio Azure Storage, Azure Machine Learning requiere que tenga permiso para obtener acceso al contenedor de almacenamiento de datos correspondiente. Este acceso depende de las credenciales de autenticación usadas para registrar el almacén de datos. 

> [!NOTE]
> Esta guía también se aplica a los [almacenes de datos creados con el acceso a datos basado en identidad (versión preliminar)](how-to-identity-based-data-access.md). 

### <a name="virtual-network"></a>Virtual network 

Azure Machine Learning requiere pasos de configuración adicionales para comunicarse con una cuenta de almacenamiento que esté detrás de un firewall o dentro de una red virtual. Si la cuenta de almacenamiento está detrás de un firewall, puede [permitir enumerar la dirección IP a través de Azure Portal](../storage/common/storage-network-security.md#managing-ip-network-rules).

Azure Machine Learning puede recibir solicitudes de clientes fuera de la red virtual. Para asegurarse de que la entidad que solicita los datos del servicio es segura, [configure Azure Private Link para el área de trabajo](how-to-configure-private-link.md).

**En el caso de los usuarios del SDK para Python**, para acceder a los datos mediante el script de entrenamiento en un destino de proceso, el destino de proceso debe estar dentro de la misma red virtual y subred de almacenamiento. 

**En el caso de los usuarios de Estudio de Azure Machine Learning**, varias características se basan en la capacidad de leer datos desde un conjunto de datos, como las vistas previas del conjunto de datos, los perfiles y el aprendizaje automático automatizado. Para que estas características funcionen con el almacenamiento detrás de redes virtuales, use una [identidad administrada del área de trabajo en el Estudio](how-to-enable-studio-virtual-network.md) para permitir que Azure Machine Learning acceda a la cuenta de almacenamiento desde fuera de la red virtual. 

> [!NOTE]
> Si el almacenamiento de datos es una instancia de Azure SQL Database detrás de una red virtual, asegúrese de establecer *Denegación del acceso público* en **No** a través de [Azure Portal](https://ms.portal.azure.com/) para permitir que Azure Machine Learning acceda a la cuenta de almacenamiento.

### <a name="access-validation"></a>Validación de acceso

**Como parte del proceso de creación y registro del almacén de datos inicial**, Azure Machine Learning valida automáticamente que el servicio de almacenamiento subyacente exista y que la entidad de seguridad proporcionada por el usuario (nombre de usuario, entidad de servicio o token de SAS) tenga acceso al almacenamiento especificado.

**Una vez creado el almacén de datos**, esta validación solo se realiza para los métodos que requieren acceso al contenedor de almacenamiento subyacente, y **no** cada vez que se recuperan objetos del almacén de datos. Por ejemplo, la validación se produce si quiere descargar archivos del almacén de archivos. Sin embargo, no se produce si solo quiere cambiar el almacén de datos predeterminado.

Para autenticar su acceso al servicio de almacenamiento subyacente, puede proporcionar su clave de cuenta, tokens de firmas de acceso compartido (SAS) o entidad de servicio en el método `register_azure_*()` correspondiente del tipo de almacén de datos que desea crear. La [matriz de tipo de almacenamiento](#matrix) muestra los tipos de autenticación admitidos que corresponden a cada tipo de almacén de datos.

Encontrará información sobre la clave de cuenta, el token de SAS y la entidad de servicio en [Azure Portal](https://portal.azure.com).

* Si tiene previsto usar una clave de cuenta o un token de SAS para la autenticación, seleccione **Cuentas de almacenamiento** en el panel izquierdo y elija la cuenta de almacenamiento que quiere registrar. 
  * La página **Información general** proporciona información como el nombre de la cuenta, el contenedor y el nombre del recurso compartido de archivos. 
      1. En el caso de las claves de cuenta, vaya a **Claves de acceso** en el panel **Configuración**. 
      1. En el caso de los tokens de SAS, vaya a **Firmas de acceso compartido** en el panel **Configuración**.

* Si tiene previsto usar una entidad de servicio para la autenticación, vaya a **Registros de aplicaciones** y seleccione la aplicación que quiere usar. 
    * Su página de **información general** correspondiente contendrá la información necesaria, como el id. de inquilino y de cliente.

> [!IMPORTANT]
> * Si necesita cambiar las claves de acceso de una cuenta de Azure Storage (clave de cuenta o token de SAS), asegúrese de sincronizar las credenciales nuevas con el área de trabajo y los almacenes de datos conectados a ella. Obtenga información sobre cómo [sincronizar las credenciales actualizadas](how-to-change-storage-access-key.md). 
### <a name="permissions"></a>Permisos

Para el almacenamiento de Azure Data Lake Gen 2 y del contenedor de blobs de Azure, asegúrese de que las credenciales de autenticación tengan acceso al **Lector de datos de Storage Blob**. Obtenga más información sobre el [Lector de datos de Storage Blob](../role-based-access-control/built-in-roles.md#storage-blob-data-reader). Un token de SAS de cuenta no tiene de forma predeterminada ningún permiso. 
* Para el **acceso de lectura** de datos, las credenciales de autenticación deben tener un número mínimo de permisos de enumeración y lectura para contenedores y objetos. 

* Para el **acceso de escritura** de datos, también se necesitan los permisos de escritura y adición.

<a name="python"></a>

## <a name="create-and-register-datastores"></a>Creación y registro de almacenes de datos

Cuando se registra una solución de Azure Storage como almacén de datos, se crea y registra automáticamente ese almacén de datos en un área de trabajo específica. Revise la sección [Permisos y acceso a Storage](#storage-access-and-permissions) para obtener ayuda sobre los escenarios de red virtual y dónde encontrar las credenciales de autenticación necesarias. 

En esta sección encontrará ejemplos de cómo crear y registrar un almacén de datos a través del SDK de Python para los siguientes tipos de almacenamiento. Los parámetros proporcionados en estos ejemplos son los **parámetros obligatorios** para crear y registrar un almacén de datos.

* [Contenedor de blobs de Azure](#azure-blob-container)
* [Recurso compartido de archivos de Azure](#azure-file-share)
* [Azure Data Lake Storage Generation 2](#azure-data-lake-storage-generation-2)

 A fin de crear almacenes de datos para otros servicios de almacenamiento admitidos, consulte la [documentación de referencia de los métodos `register_azure_*` aplicables](/python/api/azureml-core/azureml.core.datastore.datastore#methods).

Si prefiere una experiencia de código bajo, consulte la sección sobre cómo [conectar almacenes de datos en Azure Machine Learning Studio](how-to-connect-data-ui.md).
>[!IMPORTANT]
> Si anula el registro y vuelve a registrar un almacén de datos con el mismo nombre y se produce un error, es posible que la instancia de Azure Key Vault del área de trabajo no tenga habilitada la eliminación temporal. De manera predeterminada, la eliminación temporal está habilitada para la instancia del almacén de claves que creó el área de trabajo, pero podría no estar habilitada si usó un almacén de claves existente, o si creó el área de trabajo antes de octubre de 2020. Para obtener información sobre cómo habilitar la eliminación temporal, consulte [Activación de la eliminación temporal de un almacén de claves existente](../key-vault/general/soft-delete-change.md#turn-on-soft-delete-for-an-existing-key-vault).


> [!NOTE]
> El nombre del almacén de datos solo puede contener letras minúsculas, dígitos y caracteres de subrayado. 

### <a name="azure-blob-container"></a>Contenedor de blobs de Azure

Para registrar un almacén de datos de un contenedor de blobs de Azure, use [`register_azure_blob_container()`](/python/api/azureml-core/azureml.core.datastore%28class%29#register-azure-blob-container-workspace--datastore-name--container-name--account-name--sas-token-none--account-key-none--protocol-none--endpoint-none--overwrite-false--create-if-not-exists-false--skip-validation-false--blob-cache-timeout-none--grant-workspace-access-false--subscription-id-none--resource-group-none-).

En el código siguiente se crea y registra el almacén de datos `blob_datastore_name` en el área de trabajo `ws`. Este almacén de datos tiene acceso al contenedor de blobs `my-container-name` en la cuenta de almacenamiento `my-account-name` con la clave de acceso a la cuenta proporcionada. Revise la sección [Permisos y acceso a Storage](#storage-access-and-permissions) para obtener ayuda sobre los escenarios de red virtual y dónde encontrar las credenciales de autenticación necesarias. 

```Python
blob_datastore_name='azblobsdk' # Name of the datastore to workspace
container_name=os.getenv("BLOB_CONTAINER", "<my-container-name>") # Name of Azure blob container
account_name=os.getenv("BLOB_ACCOUNTNAME", "<my-account-name>") # Storage account name
account_key=os.getenv("BLOB_ACCOUNT_KEY", "<my-account-key>") # Storage account access key

blob_datastore = Datastore.register_azure_blob_container(workspace=ws, 
                                                         datastore_name=blob_datastore_name, 
                                                         container_name=container_name, 
                                                         account_name=account_name,
                                                         account_key=account_key)
```

### <a name="azure-file-share"></a>Recurso compartido de archivos de Azure

Para registrar un recurso compartido de archivos de Azure como almacén de datos, use [`register_azure_file_share()`](/python/api/azureml-core/azureml.core.datastore%28class%29#register-azure-file-share-workspace--datastore-name--file-share-name--account-name--sas-token-none--account-key-none--protocol-none--endpoint-none--overwrite-false--create-if-not-exists-false--skip-validation-false-). 

En el código siguiente se crea y registra el almacén de datos `file_datastore_name` en el área de trabajo `ws`. Este almacén de datos tiene acceso al recurso compartido de archivos `my-fileshare-name` en la cuenta de almacenamiento `my-account-name` con la clave de acceso a la cuenta proporcionada. Revise la sección [Permisos y acceso a Storage](#storage-access-and-permissions) para obtener ayuda sobre los escenarios de red virtual y dónde encontrar las credenciales de autenticación necesarias. 

```Python
file_datastore_name='azfilesharesdk' # Name of the datastore to workspace
file_share_name=os.getenv("FILE_SHARE_CONTAINER", "<my-fileshare-name>") # Name of Azure file share container
account_name=os.getenv("FILE_SHARE_ACCOUNTNAME", "<my-account-name>") # Storage account name
account_key=os.getenv("FILE_SHARE_ACCOUNT_KEY", "<my-account-key>") # Storage account access key

file_datastore = Datastore.register_azure_file_share(workspace=ws,
                                                     datastore_name=file_datastore_name, 
                                                     file_share_name=file_share_name, 
                                                     account_name=account_name,
                                                     account_key=account_key)
```

### <a name="azure-data-lake-storage-generation-2"></a>Azure Data Lake Storage Generation 2

Para un almacén de datos de Azure Data Lake Storage Generation 2 (ADLS Gen 2), utilice [register_azure_data_lake_gen2()](/python/api/azureml-core/azureml.core.datastore.datastore#register-azure-data-lake-gen2-workspace--datastore-name--filesystem--account-name--tenant-id--client-id--client-secret--resource-url-none--authority-url-none--protocol-none--endpoint-none--overwrite-false-) para registrar un almacén de datos de credenciales conectado a un almacenamiento Azure DataLake Gen 2 con [permisos de entidad de servicio](../active-directory/develop/howto-create-service-principal-portal.md).  

Para poder utilizar la entidad de servicio, debe [registrar la aplicación](../active-directory/develop/app-objects-and-service-principals.md) y conceder acceso a los datos de dicha entidad mediante el control de acceso basado en roles de Azure (Azure RBAC) o las listas de control de acceso (ACL). Aprenda más sobre la [configuración del control de acceso en ADLS Gen 2](../storage/blobs/data-lake-storage-access-control-model.md). 

En el código siguiente se crea y registra el almacén de datos `adlsgen2_datastore_name` en el área de trabajo `ws`. Este almacén de datos accede al sistema de archivos `test` en la cuenta de almacenamiento `account_name` con las credenciales de la entidad de servicio facilitadas. Revise la sección [Permisos y acceso a Storage](#storage-access-and-permissions) para obtener ayuda sobre los escenarios de red virtual y dónde encontrar las credenciales de autenticación necesarias. 

```python 
adlsgen2_datastore_name = 'adlsgen2datastore'

subscription_id=os.getenv("ADL_SUBSCRIPTION", "<my_subscription_id>") # subscription id of ADLS account
resource_group=os.getenv("ADL_RESOURCE_GROUP", "<my_resource_group>") # resource group of ADLS account

account_name=os.getenv("ADLSGEN2_ACCOUNTNAME", "<my_account_name>") # ADLS Gen2 account name
tenant_id=os.getenv("ADLSGEN2_TENANT", "<my_tenant_id>") # tenant id of service principal
client_id=os.getenv("ADLSGEN2_CLIENTID", "<my_client_id>") # client id of service principal
client_secret=os.getenv("ADLSGEN2_CLIENT_SECRET", "<my_client_secret>") # the secret of service principal

adlsgen2_datastore = Datastore.register_azure_data_lake_gen2(workspace=ws,
                                                             datastore_name=adlsgen2_datastore_name,
                                                             account_name=account_name, # ADLS Gen2 account name
                                                             filesystem='test', # ADLS Gen2 filesystem
                                                             tenant_id=tenant_id, # tenant id of service principal
                                                             client_id=client_id, # client id of service principal
                                                             client_secret=client_secret) # the secret of service principal
```



## <a name="create-datastores-with-other-azure-tools"></a>Creación de almacenes de datos con otras herramientas de Azure
Además de crear almacenes de datos con el SDK para Python y el Estudio, también puede usar plantillas de Azure Resource Manager o la extensión de VS Code de Azure Machine Learning. 

<a name="arm"></a>
### <a name="azure-resource-manager"></a>Azure Resource Manager

Existen varias plantillas en [https://github.com/Azure/azure-quickstart-templates/tree/master/101-machine-learning-datastore-create-*](https://github.com/Azure/azure-quickstart-templates/tree/master/) que se pueden usar para crear almacenes de datos.

Para información sobre el uso de estas plantillas, consulte [Uso de una plantilla de Azure Resource Manager para crear un área de trabajo para Azure Machine Learning](how-to-create-workspace-template.md).

### <a name="vs-code-extension"></a>Extensión de VS Code

Si prefiere crear y administrar almacenes de datos mediante la extensión de VS Code de Azure Machine Learning, consulte [Administración de recursos de Azure Machine Learning con la extensión de VS Code (versión preliminar)](how-to-manage-resources-vscode.md#datastores) para más información.
<a name="train"></a>
## <a name="use-data-in-your-datastores"></a>Uso de datos en los almacenes de datos

Una vez que cree un almacén de datos, [cree un conjunto de datos de Azure Machine Learning](how-to-create-register-datasets.md) para interactuar con los datos. Los conjuntos de datos empaquetan sus datos en un objeto consumible evaluado de forma diferida para tareas de aprendizaje automático, como un curso. 

Con los conjuntos de datos, puede [descargar o montar](how-to-train-with-datasets.md#mount-vs-download) archivos de cualquier formato desde los servicios de almacenamiento de Azure para el entrenamiento del modelo en un destino de proceso. [Más información sobre cómo entrenar modelos de aprendizaje automático con conjuntos de datos](how-to-train-with-datasets.md).

<a name="get"></a>

## <a name="get-datastores-from-your-workspace"></a>Obtención de almacenes de almacenamiento del área de trabajo

Para obtener un almacén de datos específico registrado en el área de trabajo actual, utilice el método estático [`get()`](/python/api/azureml-core/azureml.core.datastore%28class%29#get-workspace--datastore-name-) en la clase `Datastore`:

```Python
# Get a named datastore from the current workspace
datastore = Datastore.get(ws, datastore_name='your datastore name')
```
Para obtener la lista de almacenes de datos registrados con un área de trabajo determinada, puede utilizar la propiedad [`datastores`](/python/api/azureml-core/azureml.core.workspace%28class%29#datastores) en un objeto del área de trabajo:

```Python
# List all datastores registered in the current workspace
datastores = ws.datastores
for name, datastore in datastores.items():
    print(name, datastore.datastore_type)
```

Para obtener el almacén de datos predeterminado del área de trabajo, use esta línea:

```Python
datastore = ws.get_default_datastore()
```
También puede cambiar el almacén de datos predeterminado con el código siguiente. Esta capacidad solo se admite a través del SDK. 

```Python
 ws.set_default_datastore(new_default_datastore)
```

## <a name="access-data-during-scoring"></a>Acceso a los datos durante la puntuación

Azure Machine Learning dispone de varios métodos para usar los modelos para puntuación. Algunos de estos métodos no proporcionan acceso a los almacenes de datos. Use la tabla siguiente para saber qué métodos le permiten acceder a los almacenes de datos durante la puntuación:

| Método | Acceso a almacén de datos | Descripción |
| ----- | :-----: | ----- |
| [Predicción por lotes](./tutorial-pipeline-batch-scoring-classification.md) | ✔ | Realice predicciones sobre grandes cantidades de datos asincrónicamente. |
| [Servicio web](how-to-deploy-and-where.md) | &nbsp; | Implemente modelos como servicios web. |
| [Módulo de Azure IoT Edge](how-to-deploy-and-where.md) | &nbsp; | Implemente modelos en dispositivos IoT Edge. |

En situaciones en las que el SDK no proporciona acceso a los almacenes de datos, es posible que pueda crear código personalizado mediante el SDK de Azure correspondiente para obtener acceso a los datos. Por ejemplo, el [SDK de Azure Storage para Python](https://github.com/Azure/azure-storage-python) es una biblioteca cliente que puede usar para acceder a los datos almacenados en blobs o archivos.

<a name="move"></a>

## <a name="move-data-to-supported-azure-storage-solutions"></a>Movimiento de datos a soluciones de Azure Storage compatibles

Azure Machine Learning admite el acceso a datos desde Azure Blob Storage, Azure Files, Azure Data Lake Storage Gen1, Azure Data Lake Storage Gen2, Azure SQL Database y Azure Database for PostgreSQL. Si usa un almacenamiento que no es compatible, le recomendamos que mueva los datos a soluciones de Azure Storage compatibles mediante [Azure Data Factory y estos pasos](../data-factory/quickstart-create-data-factory-copy-data-tool.md). El movimiento de datos a un almacenamiento compatible puede ayudarle a ahorrar costos de salida de datos durante los experimentos de aprendizaje automático. 

Azure Data Factory proporciona una transferencia de datos eficaz y resistente con más de 80 conectores pregenerados sin costo adicional. Estos conectores incluyen servicios de datos de Azure, orígenes de datos locales, Amazon S3 y Redshift, y Google BigQuery.

## <a name="next-steps"></a>Pasos siguientes

* [Creación de un conjunto de datos de Azure Machine Learning](how-to-create-register-datasets.md)
* [Entrenamiento de un modelo](how-to-set-up-training-targets.md)
* [Implementar un modelo](how-to-deploy-and-where.md)