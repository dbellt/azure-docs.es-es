---
title: Acceso a datos basado en identidad para los servicios de almacenamiento de Azure
titleSuffix: Machine Learning
description: Aprenda a utilizar el acceso a datos basado en identidades para conectarse a los servicios de almacenamiento de Azure con almacenes de datos de Azure Machine Learning y el SDK de Azure Machine Learning para Python.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: how-to
ms.author: yogipandey
author: ynpandey
ms.reviewer: nibaccam
ms.date: 02/22/2021
ms.custom: contperf-fy21q1, devx-track-python, data4ml
ms.openlocfilehash: 2ca7eec3bcc40d5c0f93c43cc32b9395feb1e5d0
ms.sourcegitcommit: 32ee8da1440a2d81c49ff25c5922f786e85109b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109790422"
---
# <a name="connect-to-storage-by-using-identity-based-data-access-preview"></a>Conexión al almacenamiento mediante el acceso a datos basado en identidades (versión preliminar)

>[!IMPORTANT]
> Las características que se presentan en este artículo están en versión preliminar. Por tanto, deben considerarse características [experimentales](/python/api/overview/azure/ml/#stable-vs-experimental) que pueden cambiar en cualquier momento.

En este artículo, aprenderá a conectarse a los servicios de almacenamiento de Azure utilizando el acceso a datos basado en identidades y los almacenes de datos de Azure Machine Learning con el [SDK de Azure Machine Learning para Python](/python/api/overview/azure/ml/intro).  

Normalmente, los almacenes de datos usan el acceso a datos basado en credenciales para confirmar que dispone de permiso para acceder al servicio de almacenamiento. Guardan la información de la conexión, como el identificador de la suscripción y la autorización del token, en el [almacén de claves](https://azure.microsoft.com/services/key-vault/) asociado al área de trabajo. Cuando cree un almacén de datos que utilice el acceso a datos basado en identidades, su cuenta de Azure ([token de Azure Active Directory](../active-directory/fundamentals/active-directory-whatis.md)) se utilizará para confirmar que tiene permiso para acceder al servicio de almacenamiento. En este escenario, no se guardarán las credenciales de autenticación. Lo único que se guardará en el almacén de datos será la información de la cuenta de almacenamiento. 

Para crear almacenes de datos que usen la autenticación basada en credenciales, como claves de acceso o entidades de servicio, consulte [Conexión a los servicios de almacenamiento en Azure](how-to-access-data.md).

## <a name="identity-based-data-access-in-azure-machine-learning"></a>Acceso a datos basado en identidad en Azure Machine Learning

Hay dos escenarios en los que se puede aplicar el acceso a datos basado en identidades de Azure Machine Learning. Estos escenarios son una buena opción para utilizar el acceso basado en identidades cuando hay datos confidenciales de por medio y es preciso administrar el acceso a los datos de forma más pormenorizada:

- Acceso a servicios de almacenamiento
- Entrenamiento de modelos de aprendizaje automático con datos privados

### <a name="accessing-storage-services"></a>Acceso a servicios de almacenamiento

Puede conectarse a los servicios de almacenamiento mediante el acceso a datos basado en identidad con almacenes de datos de Azure Machine Learning o [conjuntos de datos de Azure Machine Learning](how-to-create-register-datasets.md). 

Normalmente, las credenciales de autenticación se guardan en un almacén de datos, que se usa para comprobar que un usuario tiene permiso para acceder al servicio de almacenamiento. Cuando estas credenciales se registran utilizando almacenes de datos, cualquier usuario que tenga el rol Lector en el área de trabajo podrá recuperarlas. Ese nivel de acceso puede ser un problema de seguridad para algunas organizaciones. [Consulte este artículo para más información sobre el rol Lector del área de trabajo.](how-to-assign-roles.md#default-roles) 

Cuando se usa el acceso a datos basado en identidades, Azure Machine Learning solicita el token de Azure Active Directory para autenticar el acceso a los datos en lugar de guardar las credenciales en el almacén de datos. Este enfoque permite administrar el acceso a los datos en el nivel de almacenamiento y preserva la confidencialidad de las credenciales. 

Este mismo comportamiento se aplica al:

* [Crear un conjunto de datos directamente a partir de las direcciones URL de almacenamiento](#use-data-in-storage). 
* Trabajar con los datos de forma interactiva utilizando un cuaderno de Jupyter Notebook en el equipo local o en la [instancia de proceso](concept-compute-instance.md).

> [!NOTE]
> Las credenciales que se almacenan con la autenticación basada en credenciales incluyen: los identificadores de la suscripción, los tokens de firma de acceso compartido (SAS), las claves de acceso de almacenamiento y la información de la entidad de servicio, como los identificadores de cliente y los identificadores de inquilino.

### <a name="model-training-on-private-data"></a>Entrenamiento de modelos con datos privados

Algunos escenarios de aprendizaje automático implican el entrenamiento de modelos con datos privados. En estos casos, los científicos de datos deben ejecutar los flujos de trabajo de entrenamiento sin exponer los datos de entrada confidenciales. En este escenario, se usa una identidad administrada de la instancia de proceso de entrenamiento para la autenticación del acceso a datos. De este modo, los administradores del almacenamiento pueden conceder el acceso como Lector de datos de Storage Blob a la identidad administrada que la instancia de proceso de entrenamiento utiliza para ejecutar el trabajo de entrenamiento. No es necesario conceder acceso a cada científico de datos. Para más información, consulte ese artículo sobre la [configuración de identidades administradas en un clúster de proceso](how-to-create-attach-compute-cluster.md#managed-identity).


## <a name="prerequisites"></a>Requisitos previos

- Suscripción a Azure. Si no tiene una suscripción de Azure, cree una cuenta gratuita antes de empezar. Pruebe la [versión gratuita o de pago de Azure Machine Learning](https://aka.ms/AMLFree).

- Una cuenta de Azure Storage con un tipo de almacenamiento compatible. Los tipos de almacenamiento admitidos en la versión preliminar son los siguientes: 
    - [Azure Blob Storage](../storage/blobs/storage-blobs-overview.md)
    - [Azure Data Lake Storage Gen1](../data-lake-store/index.yml)
    - [Azure Data Lake Storage Gen2](../storage/blobs/data-lake-storage-introduction.md)
    - [Azure SQL Database](../azure-sql/database/sql-database-paas-overview.md)

- El [SDK de Azure Machine Learning para Python](/python/api/overview/azure/ml/install).

- Un área de trabajo de Azure Machine Learning.
  
  [Cree un área de trabajo de Azure Machine Learning](how-to-manage-workspace.md) o use una [existente mediante el SDK para Python](how-to-manage-workspace.md#connect-to-a-workspace). 

## <a name="storage-access-permissions"></a>Permisos de acceso al almacenamiento

Para ayudar a garantizar la seguridad al conectarse al servicio de almacenamiento de Azure, en Azure Machine Learning es obligatorio tener permiso para poder acceder al almacenamiento de datos correspondiente.

El acceso a datos basado en identidades solo permite conectarse a los siguientes servicios de almacenamiento:

* Azure Blob Storage
* Azure Data Lake Storage Gen1
* Azure Data Lake Storage Gen2
* Azure SQL Database

Para poder acceder a estos servicios de almacenamiento, debe tener como mínimo acceso de [Lector de datos de Storage Blob](../role-based-access-control/built-in-roles.md#storage-blob-data-reader). Solo los propietarios de la cuenta de almacenamiento pueden [cambiar el nivel de acceso mediante Azure Portal](../storage/common/storage-auth-aad-rbac-portal.md).

Si va a entrenar un modelo en un destino de proceso remoto, es necesario conceder a la identidad de proceso al menos el rol Lector de datos de Storage Blob en el servicio de almacenamiento. Aprenda a [configurar la identidad administrada en un clúster de proceso](how-to-create-attach-compute-cluster.md#managed-identity).

## <a name="work-with-virtual-networks"></a>Trabajo con redes virtuales

De forma predeterminada, Azure Machine Learning no puede comunicarse con una cuenta de almacenamiento que esté detrás de un firewall o en una red virtual.

Puede configurar las cuentas de almacenamiento para permitir el acceso exclusivamente desde redes virtuales específicas. Esto requiere algunos pasos adicionales para evitar que los datos se filtren fuera de la red. Este comportamiento es igual en el acceso a datos basado en credenciales. Para más información, consulte este artículo sobre la [configuración de escenarios de red virtual](how-to-access-data.md#virtual-network). 

## <a name="create-and-register-datastores"></a>Creación y registro de almacenes de datos

Cuando se registra un servicio de almacenamiento de Azure como almacén de datos, se crea y registra automáticamente ese almacén de datos en un área de trabajo específica. Consulte [Permisos de acceso al almacenamiento](#storage-access-permissions) para obtener instrucciones sobre los tipos de permisos necesarios. Consulte este artículo sobre el [uso de las redes virtuales](#work-with-virtual-networks) para obtener más detalles acerca de cómo conectarse a los almacenamientos de datos que están detrás de redes virtuales.

En el código siguiente, observe que faltan los parámetros de autenticación, como `sas_token`, `account_key` y `subscription_id`, y la entidad de servicio `client_id`. Esta omisión indica que Azure Machine Learning utiliza el acceso a datos basado en identidades para realizar la autenticación. Normalmente, los almacenes de datos se crean de forma interactiva en un cuaderno o mediante Studio. Por tanto, se utiliza el token de Azure Active Directory para autenticar el acceso a los datos.

> [!NOTE]
> Los nombres de los almacenes de datos solo pueden contener letras minúsculas, números y caracteres de subrayado. 

### <a name="azure-blob-container"></a>Contenedor de blobs de Azure

Para registrar un almacén de datos de un contenedor de blobs de Azure, use [`register_azure_blob_container()`](/python/api/azureml-core/azureml.core.datastore%28class%29#register-azure-blob-container-workspace--datastore-name--container-name--account-name--sas-token-none--account-key-none--protocol-none--endpoint-none--overwrite-false--create-if-not-exists-false--skip-validation-false--blob-cache-timeout-none--grant-workspace-access-false--subscription-id-none--resource-group-none-).

El código siguiente crea el almacén de datos `credentialless_blob`, lo registra en el área de trabajo `ws` y lo asigna a la variable `blob_datastore`. Este almacén de datos accede al contenedor de blobs `my_container_name` de la cuenta de almacenamiento `my-account-name`.

```Python
# Create blob datastore without credentials.
blob_datastore = Datastore.register_azure_blob_container(workspace=ws,
                                                      datastore_name='credentialless_blob',
                                                      container_name='my_container_name',
                                                      account_name='my_account_name')
```

### <a name="azure-data-lake-storage-gen1"></a>Azure Data Lake Storage Gen1

Utilice [register_azure_data_lake()](/python/api/azureml-core/azureml.core.datastore.datastore#register-azure-data-lake-workspace--datastore-name--store-name--tenant-id-none--client-id-none--client-secret-none--resource-url-none--authority-url-none--subscription-id-none--resource-group-none--overwrite-false--grant-workspace-access-false-) para registrar un almacén de datos que se conecte a Azure Data Lake Storage Gen1.

El código siguiente crea el almacén de datos `credentialless_adls1`, lo registra en el área de trabajo `workspace` y lo asigna a la variable `adls_dstore`. Este almacén de datos accede a la cuenta `adls_storage` de Azure Data Lake Storage.

```Python
# Create Azure Data Lake Storage Gen1 datastore without credentials.
adls_dstore = Datastore.register_azure_data_lake(workspace = workspace,
                                                 datastore_name='credentialless_adls1',
                                                 store_name='adls_storage')

```

### <a name="azure-data-lake-storage-gen2"></a>Azure Data Lake Storage Gen2

Utilice [register_azure_data_lake_gen2()](/python/api/azureml-core/azureml.core.datastore.datastore#register-azure-data-lake-gen2-workspace--datastore-name--filesystem--account-name--tenant-id--client-id--client-secret--resource-url-none--authority-url-none--protocol-none--endpoint-none--overwrite-false-) para registrar un almacén de datos que se conecte a Azure Data Lake Storage Gen2.

El código siguiente crea el almacén de datos `credentialless_adls2`, lo registra en el área de trabajo `ws` y lo asigna a la variable `adls2_dstore`. Este almacén de datos accede al sistema de archivos `tabular` de la cuenta de almacenamiento `myadls2`.  

```python
# Create Azure Data Lake Storage Gen2 datastore without credentials.
adls2_dstore = Datastore.register_azure_data_lake_gen2(workspace=ws, 
                                                       datastore_name='credentialless_adls2', 
                                                       filesystem='tabular', 
                                                       account_name='myadls2')
```

## <a name="use-data-in-storage"></a>Uso de los datos del almacenamiento

Le recomendamos que utilice [conjuntos de datos de Azure Machine Learning](how-to-create-register-datasets.md) cuando interactúe con los datos de almacenamiento en Azure Machine Learning. 

Los conjuntos de datos empaquetan los datos en un objeto consumible que se evalúa de forma diferida para tareas de aprendizaje automático, como el entrenamiento. Además, con los conjuntos de datos, se pueden [descargar o montar](how-to-train-with-datasets.md#mount-vs-download) archivos con cualquier formato desde los servicios de almacenamiento de Azure, como Azure Blob Storage y Azure Data Lake Storage, en un destino de proceso.


Para crear conjuntos de datos con acceso a datos basado en identidad, tiene las siguientes opciones. Cuando se crean conjuntos de datos de este modo, se utiliza el token de Azure Active Directory para autenticar el acceso a los datos. 

*  Hacer referencia a rutas de acceso de los almacenes de datos que también usan el acceso a datos basado en identidad. 
<br>En el ejemplo siguiente, `blob_datastore` ya existe y usa el acceso a datos basado en identidades.   

    ```python
    blob_dataset = Dataset.Tabular.from_delimited_files(blob_datastore,'test.csv') 
    ```

* Omita la creación del almacén de datos y cree conjuntos de datos directamente desde las direcciones URL de almacenamiento. Actualmente, esta funcionalidad solo permite utilizar blobs de Azure, así como Azure Data Lake Storage Gen1 y Gen2.

    ```python
    blob_dset = Dataset.File.from_files('https://myblob.blob.core.windows.net/may/keras-mnist-fashion/')
    ```

Sin embargo, cuando se envía un trabajo de entrenamiento que consume un conjunto de datos creado con el acceso a datos basado en identidades, se utiliza la identidad administrada de la instancia de proceso de entrenamiento para autenticar el acceso a datos. No se utiliza el token de Azure Active Directory. En este escenario, asegúrese de que la identidad administrada de la instancia de proceso recibe al menos el rol Lector de datos de Storage Blob en el servicio de almacenamiento. Para más información, consulte este artículo sobre la [configuración de identidades administradas en el clúster de proceso](how-to-create-attach-compute-cluster.md#managed-identity). 

## <a name="next-steps"></a>Pasos siguientes

* [Creación de conjuntos de datos de Azure Machine Learning](how-to-create-register-datasets.md)
* [Entrenamiento con conjuntos de datos](how-to-train-with-datasets.md)
* [Conexión a los servicios de almacenamiento en Azure](how-to-access-data.md)
