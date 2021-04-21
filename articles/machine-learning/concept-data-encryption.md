---
title: Cifrado de datos con Azure Machine Learning
titleSuffix: Azure Machine Learning
description: Obtenga información sobre cómo los procesos y almacenes de datos de Azure Machine Learning proporcionan cifrado de datos en reposo y en tránsito.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: jhirono
author: jhirono
ms.reviewer: larryfr
ms.date: 11/09/2020
ms.openlocfilehash: 31b2a993267e6e8abbf7184d745889b74e587c50
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107311777"
---
# <a name="data-encryption-with-azure-machine-learning"></a>Cifrado de datos con Azure Machine Learning

Azure Machine Learning usa una variedad de servicios de almacenamiento de datos y recursos de proceso de Azure al entrenar modelos y realizar inferencias. Cada uno de ellos tiene su propia historia sobre cómo proporcionan el cifrado de datos en reposo y en tránsito. En este artículo, obtendrá información sobre cada uno de ellos y cuál es la mejor opción para su escenario.

> [!IMPORTANT]
> Para el cifrado de nivel de producción durante el __aprendizaje__, Microsoft recomienda usar el clúster de proceso de Azure Machine Learning. Para el cifrado de nivel de producción durante la __inferencia__, Microsoft recomienda usar Azure Kubernetes Service.
>
> La instancia de proceso de Azure Machine Learning es un entorno de desarrollo/pruebas. Cuando se usa, se recomienda almacenar los archivos, como cuadernos s y scripts, en un recurso compartido de archivos. Los datos deben almacenarse en un almacén de datos.

## <a name="encryption-at-rest"></a>Cifrado en reposo

> [!IMPORTANT]
> Si su área de trabajo contiene datos confidenciales, se recomienda establecer el parámetro [hbi_workspace](/python/api/azureml-core/azureml.core.workspace%28class%29#create-name--auth-none--subscription-id-none--resource-group-none--location-none--create-resource-group-true--sku--basic---friendly-name-none--storage-account-none--key-vault-none--app-insights-none--container-registry-none--cmk-keyvault-none--resource-cmk-uri-none--hbi-workspace-false--default-cpu-compute-target-none--default-gpu-compute-target-none--exist-ok-false--show-output-true-) a la hora de crearla. La marca `hbi_workspace` solo se puede establecer cuando se crea un área de trabajo. No se puede cambiar en un área de trabajo existente.

La marca `hbi_workspace` controla la cantidad de [datos que Microsoft recopila para fines de diagnóstico](#microsoft-collected-data) y permite el [cifrado adicional en entornos administrados por Microsoft](../security/fundamentals/encryption-atrest.md). Además, permite las siguientes acciones:

* Inicia el cifrado del disco temporal local en el clúster de proceso de Azure Machine Learning, siempre que no haya creado ningún clúster anterior en esa suscripción. En caso contrario, debe generar una incidencia de soporte técnico para habilitar el cifrado del disco temporal de los clústeres de proceso. 
* Limpia el disco temporal local entre ejecuciones.
* Pasa de forma segura las credenciales de la cuenta de almacenamiento, el registro de contenedor y la cuenta SSH desde la capa de ejecución a los clústeres de proceso mediante el almacén de claves.
* Habilita el filtrado de IP para asegurarse de que los servicios externos que no sean AzureMachineLearningService no puedan llamar a los grupos de lotes subyacentes.
* En el área de trabajo HBI se admiten las instancias de proceso.

### <a name="azure-blob-storage"></a>Azure Blob Storage

Azure Machine Learning almacena instantáneas, salidas y registros en la cuenta de Azure Blob Storage vinculada al área de trabajo de Azure Machine Learning y la suscripción. Todos los datos almacenados en Azure Blob Storage se cifran en reposo con claves administradas por Microsoft.

Para más información sobre el uso de claves propias para los datos almacenados en Azure Blob Storage, consulte [Cifrado de Azure Storage con claves administradas por el cliente en Azure Key Vault](../storage/common/customer-managed-keys-configure-key-vault.md).

Los datos de entrenamiento también se almacenan en Azure Blob Storage, de modo que están accesibles para los destinos de proceso de entrenamiento. Este almacenamiento no está administrado por Azure Machine Learning, sino que está montado en destinos de proceso como un sistema de archivos remoto.

Si necesita __girar o revocar__ la clave, puede hacerlo en cualquier momento. Al rotar una clave, la cuenta de almacenamiento comenzará a usar la nueva clave (versión más reciente) para cifrar los datos en reposo. Al revocar (deshabilitar) una clave, la cuenta de almacenamiento se encarga de las solicitudes con error. Normalmente, se tarda una hora para que la rotación o la revocación surtan efecto.

Para más información sobre cómo regenerar una clave de acceso, vea el artículo [Regeneración de las claves de la cuenta de almacenamiento](how-to-change-storage-access-key.md).

### <a name="azure-cosmos-db"></a>Azure Cosmos DB

Azure Machine Learning almacena metadatos en una instancia de Azure Cosmos DB. Esta instancia está asociada a una suscripción de Microsoft administrada por Azure Machine Learning. Todos los datos almacenados en Azure Cosmos DB se cifran en reposo con claves administradas por Microsoft.

Si quiere usar sus propias claves (administradas por el cliente) para cifrar su instancia de Azure Cosmos DB, puede crear una instancia de Cosmos DB dedicada para usarla con el área de trabajo. Recomendamos este método si quiere almacenar los datos, como la información del historial de ejecución, fuera de la instancia de Cosmos DB multiinquilino hospedada en la suscripción de Microsoft. 

Para habilitar el aprovisionamiento de una instancia de Cosmos DB en su suscripción con claves administradas por el cliente, realice estas acciones:

* Si aún no lo ha hecho, registre los proveedores de recursos de Microsoft.MachineLearning y Microsoft.DocumentDB en su suscripción.

* Use los parámetros que se indican más abajo al crear el área de trabajo de Azure Machine Learning. Ambos parámetros son obligatorios y se admiten en SDK, CLI, API REST y plantillas de Resource Manager.

    * `resource_cmk_uri`: Este parámetro es el URI de recurso completo de la clave administrada por el cliente en el almacén de claves e incluye la [información de versión de la clave](../key-vault/general/about-keys-secrets-certificates.md#objects-identifiers-and-versioning). 

    * `cmk_keyvault`: Este parámetro es el identificador de recurso del almacén de claves de su suscripción. Este almacén de claves debe estar en la misma región y suscripción que usará para el área de trabajo de Azure Machine Learning. 
    
        > [!NOTE]
        > Esta instancia del almacén de claves puede ser diferente a la creada por Azure Machine Learning al aprovisionar el área de trabajo. Si quiere usar la misma instancia del almacén de claves para el área de trabajo, pase el mismo almacén de claves al aprovisionar el área de trabajo mediante el [parámetro key_vault](/python/api/azureml-core/azureml.core.workspace%28class%29#create-name--auth-none--subscription-id-none--resource-group-none--location-none--create-resource-group-true--sku--basic---friendly-name-none--storage-account-none--key-vault-none--app-insights-none--container-registry-none--cmk-keyvault-none--resource-cmk-uri-none--hbi-workspace-false--default-cpu-compute-target-none--default-gpu-compute-target-none--exist-ok-false--show-output-true-). 

[!INCLUDE [machine-learning-customer-managed-keys.md](../../includes/machine-learning-customer-managed-keys.md)]

Si necesita __girar o revocar__ la clave, puede hacerlo en cualquier momento. Al rotar una clave, Cosmos DB comenzará a usar la nueva clave (versión más reciente) para cifrar los datos en reposo. Al revocar (deshabilitar) una clave, Cosmos DB se encarga de las solicitudes con error. Normalmente, se tarda una hora para que la rotación o la revocación surtan efecto.

Para más información sobre las claves administradas por el cliente con Cosmos DB, vea el artículo [Configuración de claves administradas por el cliente para la cuenta de Azure Cosmos DB](../cosmos-db/how-to-setup-cmk.md).

### <a name="azure-container-registry"></a>Azure Container Registry

Todas las imágenes del contenedor en el registro (Azure Container Registry) se cifran en reposo. Azure cifra automáticamente una imagen antes de almacenarla y la descifra cuando Azure Machine Learning la extrae.

Para usar sus propias claves (administradas por el cliente) para cifrar su instancia de Azure Container Registry, debe crear un ACR y adjuntarlo al aprovisionar el área de trabajo, o bien cifrar la instancia predeterminada que se crea en el momento del aprovisionamiento del área de trabajo.

> [!IMPORTANT]
> Azure Machine Learning requiere que la cuenta de administrador esté habilitada en el Azure Container Registry. De forma predeterminada, esta opción se deshabilita al crear un registro de contenedor. Para información sobre cómo habilitar la cuenta de administrador, consulte [Cuenta de administrador](../container-registry/container-registry-authentication.md#admin-account).
>
> Cuando se ha creado una instancia de Azure Container Registry para un área de trabajo, no la elimine. Si lo hace, se interrumpirá el área de trabajo de Azure Machine Learning.

Para ver ejemplos de cómo crear un área de trabajo con una instancia existente de Azure Container Registry, vea los siguientes artículos:

* [Creación de un área de trabajo para Azure Machine Learning con la CLI de Azure](how-to-manage-workspace-cli.md).
* [Creación de un área de trabajo con el SDK de Python](how-to-manage-workspace.md?tabs=python#create-a-workspace).
* [Uso de una plantilla de Azure Resource Manager para crear un área de trabajo para Azure Machine Learning](how-to-create-workspace-template.md).

### <a name="azure-container-instance"></a>Azure Container Instances

Puede cifrar un recurso de Azure Container Instance (ACI) implementado mediante claves administradas por el cliente. La clave administrada por el cliente que se usa para ACI puede almacenarse en el almacén Azure Key Vault del área de trabajo. Para obtener información sobre cómo generar una clave, consulte [Cifrado de datos con una clave administrada por el cliente](../container-instances/container-instances-encrypt-data.md#generate-a-new-key).

Para usar la clave al implementar un modelo en una instancia de Azure Container Instances, cree una nueva configuración de implementación mediante `AciWebservice.deploy_configuration()`. Proporcione la información de clave con los parámetros siguientes:

* `cmk_vault_base_url`: La dirección URL del almacén de claves que contiene la clave.
* `cmk_key_name`: El nombre de la clave.
* `cmk_key_version`: La versión de la clave.

Para obtener más información sobre la creación y el uso de una configuración de implementación, vea los siguientes artículos:

* Referencia de [AciWebservice.deploy_configuration()](/python/api/azureml-core/azureml.core.webservice.aci.aciwebservice#deploy-configuration-cpu-cores-none--memory-gb-none--tags-none--properties-none--description-none--location-none--auth-enabled-none--ssl-enabled-none--enable-app-insights-none--ssl-cert-pem-file-none--ssl-key-pem-file-none--ssl-cname-none--dns-name-label-none--primary-key-none--secondary-key-none--collect-model-data-none--cmk-vault-base-url-none--cmk-key-name-none--cmk-key-version-none-)
* [Implementación de modelos con Azure Machine Learning](how-to-deploy-and-where.md)
* [Implementación de un modelo en Azure Container Instances](how-to-deploy-azure-container-instance.md)

Para obtener más información sobre el uso de una clave administrada por el cliente con ACI, consulte [Cifrado de datos con una clave administrada por el cliente](../container-instances/container-instances-encrypt-data.md#encrypt-data-with-a-customer-managed-key).

### <a name="azure-kubernetes-service"></a>Azure Kubernetes Service

Puede cifrar un recurso implementado de Azure Kubernetes Service mediante claves administradas por el cliente en cualquier momento. Para más información, consulte [Traiga sus propias claves (BYOK) con discos de Azure en Azure Kubernetes Service (AKS)](../aks/azure-disk-customer-managed-keys.md). 

Este proceso permite cifrar los datos y el disco del sistema operativo de las máquinas virtuales implementadas en el clúster de Kubernetes.

> [!IMPORTANT]
> Este proceso solo funciona con la versión 1.17 de AKS K8s o con versiones posteriores. Azure Machine Learning agregó compatibilidad con AKS 1.17 el 13 de enero de 2020.

### <a name="machine-learning-compute"></a>Proceso de Machine Learning

El disco del sistema operativo de cada nodo de proceso almacenado en Azure Storage se cifra mediante claves administradas por Microsoft en las cuentas de almacenamiento de Azure Machine Learning. Este destino de proceso es efímero y, por lo general, los clústeres se reducen verticalmente cuando no hay ninguna ejecución en cola. La máquina virtual subyacente se desaprovisiona y el disco del sistema operativo se elimina. Azure Disk Encryption no se admite con el disco del sistema operativo.

Cada máquina virtual tiene también un disco local temporal para las operaciones del sistema operativo. Si quiere, puede usar el disco para almacenar temporalmente los datos de entrenamiento. El disco se cifra de forma predeterminada para las áreas de trabajo con el parámetro `hbi_workspace` establecido en `TRUE`. Este entorno solo dura el tiempo de la ejecución, y la compatibilidad con el cifrado se limita únicamente a las claves administradas por el sistema.

### <a name="azure-databricks"></a>Azure Databricks

Azure Databricks se puede utilizar en canalizaciones de Azure Machine Learning. De forma predeterminada, el sistema de archivos que usa Azure Databricks se cifra mediante una clave administrada por Microsoft. Para configurar Azure Databricks para que utilice las claves administradas por el cliente, consulte [Configuración de claves administradas por el cliente en DBFS (raíz) predeterminado](/azure/databricks/security/customer-managed-keys-dbfs).

### <a name="microsoft-generated-data"></a>Datos generados por Microsoft

Cuando usa servicios como el aprendizaje automático automatizado, Microsoft puede generar datos transitorios previamente procesados para entrenar varios modelos. Estos datos se guardan en un almacén de datos en el área de trabajo, lo que permite aplicar los controles de acceso y el cifrado de forma adecuada.

También puede que quiera cifrar la [información de diagnóstico registrada desde el punto de conexión implementado](how-to-enable-app-insights.md) en su instancia de Azure Application Insights.

## <a name="encryption-in-transit"></a>Cifrado en tránsito

Azure Machine Learning usa TLS para proteger la comunicación interna entre varios microservicios de Azure Machine Learning. Todo el acceso de Azure Storage se realiza también a través de un canal seguro.

Para proteger las llamadas externas realizadas al punto de conexión de puntuación, Azure Machine Learning usa TLS. Para obtener más información, vea [Uso de TLS para proteger un servicio web mediante Azure Machine Learning](./how-to-secure-web-service.md).

## <a name="data-collection-and-handling"></a>Recopilación y tratamiento de los datos

### <a name="microsoft-collected-data"></a>Datos recopilados por Microsoft

Microsoft puede recopilar información de identificación no relacionada con el usuario, como los nombres de los recursos (por ejemplo, el nombre del conjunto de datos o el nombre del experimento de aprendizaje automático) o las variables de entorno de trabajo con fines de diagnóstico. Todos estos datos se almacenan mediante claves administradas por Microsoft en el almacenamiento hospedado en suscripciones propiedad de Microsoft, y siguen los [estándares de tratamiento de los datos y la directiva de privacidad estándar de Microsoft](https://privacy.microsoft.com/privacystatement).

Microsoft también recomienda no almacenar información confidencial (como secretos de clave de cuenta) en variables de entorno. Nosotros no encargamos de registrar, cifrar y almacenar las variables de entorno. Del mismo modo, al asignar nombres [run_id](/python/api/azureml-core/azureml.core.run%28class%29), evite incluir información confidencial, como nombres de usuario o nombres de proyectos secretos. Esta información puede aparecer en los registros de telemetría a los que pueden acceder los ingenieros de Soporte técnico de Microsoft.

Puede dejar de participar en la recopilación de datos de diagnóstico si establece el parámetro `hbi_workspace` en `TRUE` mientras aprovisiona el área de trabajo. Esta función se admite cuando se usa el SDK de Python de AzureML, la CLI, las API REST o las plantillas de Azure Resource Manager.

## <a name="using-azure-key-vault"></a>Uso de Azure Key Vault

Azure Machine Learning usa la instancia de Azure Key Vault asociada al área de trabajo para almacenar credenciales de varios tipos:

* La cadena de conexión de cuenta de almacenamiento asociada
* Contraseñas para las instancias de Azure Container Repository
* Cadenas de conexión a almacenes de datos

Las contraseñas y las claves SSH para destinos de proceso, como Azure HDInsight y las máquinas virtuales, se almacenan en un almacén de claves independiente asociado a la suscripción de Microsoft. Azure Machine Learning no almacena las contraseñas ni las claves proporcionadas por los usuarios. En cambio, genera, autoriza y almacena sus propias claves SSH para conectarse a las máquinas virtuales y HDInsight y ejecutar los experimentos.

Cada área de trabajo lleva asociada una identidad administrada asignada por el sistema con el mismo nombre que el área de trabajo. Esta identidad administrada tiene acceso a todas las claves, secretos y certificados del almacén de claves.

## <a name="next-steps"></a>Pasos siguientes

* [Conectar a Azure Storage](how-to-access-data.md)
* [Obtención de datos desde un almacén de datos](how-to-create-register-datasets.md)
* [Conectar a datos](how-to-connect-data-ui.md)
* [Entrenamiento con conjuntos de datos](how-to-train-with-datasets.md)
