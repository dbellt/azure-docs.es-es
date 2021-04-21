---
title: Protección de un área de trabajo de Azure Machine Learning con redes virtuales
titleSuffix: Azure Machine Learning
description: Use una instancia aislada de Azure Virtual Network para proteger el área de trabajo de Azure Machine Learning y los recursos asociados.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.reviewer: larryfr
ms.author: peterlu
author: peterclu
ms.date: 03/17/2021
ms.topic: conceptual
ms.custom: how-to, contperf-fy20q4, tracking-python, contperf-fy21q1
ms.openlocfilehash: 20f75580c425cee9128f9c94123dcf902642eac4
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107531022"
---
# <a name="secure-an-azure-machine-learning-workspace-with-virtual-networks"></a>Protección de un área de trabajo de Azure Machine Learning con redes virtuales

En este artículo, aprenderá a proteger un área de trabajo de Azure Machine Learning y sus recursos asociados en una red virtual.

Este artículo es la segunda parte de una serie de cinco capítulos que le guía a través de la protección de un flujo de trabajo de Azure Machine Learning. Le recomendamos encarecidamente que lea [Parte uno: Introducción a las redes virtuales](how-to-network-security-overview.md) para comprender la arquitectura general en primer lugar. 

Consulte los demás artículos de esta serie:

[1. Introducción a las redes virtuales](how-to-network-security-overview.md) > **2. Protección del área de trabajo**  > [3. Protección del entorno de entrenamiento](how-to-secure-training-vnet.md) > [4. Protección del entorno de inferencia](how-to-secure-inferencing-vnet.md) > [5. Habilitación de la funcionalidad de Studio](how-to-enable-studio-virtual-network.md)

En este artículo aprenderá a habilitar los siguientes recursos de áreas de trabajo en una red virtual:
> [!div class="checklist"]
> - Área de trabajo de Azure Machine Learning
> - Cuentas de Azure Storage
> - Conjuntos de datos y almacenes de datos de Azure Machine Learning
> - Azure Key Vault
> - Azure Container Registry

## <a name="prerequisites"></a>Requisitos previos

+ Lea el artículo [Introducción a la seguridad de red](how-to-network-security-overview.md) para comprender los escenarios comunes de redes virtuales y la arquitectura de red virtual general.

+ Una red virtual y una subred existentes que se usarán con los recursos de proceso.

+ Para implementar recursos en una red virtual o subred, la cuenta de usuario debe tener permisos para realizar las siguientes acciones en los controles de acceso basados en roles de Azure (Azure RBAC):

    - "Microsoft.Network/virtualNetworks/join/action" en el recurso de red virtual.
    - "Microsoft.Network/virtualNetworks/subnet/join/action" en el recurso de subred.

    Para obtener más información sobre Azure RBAC con redes, consulte los [roles integrados de redes](../role-based-access-control/built-in-roles.md#networking).


## <a name="secure-the-workspace-with-private-endpoint"></a>Protección del área de trabajo con punto de conexión privado

Azure Private Link le permite conectarse a su área de trabajo mediante un punto de conexión privado. El punto de conexión privado es un conjunto de direcciones IP privadas dentro de la red virtual. Después, puede limitar el acceso al área de trabajo para que solo se produzca en las direcciones IP privadas. Private Link ayuda a reducir el riesgo de una filtración de datos.

Para obtener más información sobre la configuración de un área de trabajo de Private Link, consulte [Configuración de Private Link](how-to-configure-private-link.md).

> [!Warning]
> La protección de un área de trabajo con puntos de conexión privados no garantiza la seguridad de un extremo a otro por sí misma. Debe seguir los pasos que se describen en el resto de este artículo y la serie de redes virtuales para proteger componentes individuales de la solución.

## <a name="secure-azure-storage-accounts-with-service-endpoints"></a>Protección de cuentas de almacenamiento de Azure con puntos de conexión de servicio

Azure Machine Learning admite cuentas de almacenamiento configuradas para usar puntos de conexión de servicio o puntos de conexión privados. En esta sección, aprenderá a proteger una cuenta de Azure Storage mediante puntos de conexión de servicio. En el caso de los puntos de conexión privados, consulte la sección siguiente.

Para usar una cuenta de Azure Storage para el área de trabajo en una red virtual, siga estos pasos:

1. En Azure Portal, vaya al servicio de almacenamiento que quiera usar en el área de trabajo.

   [![Almacenamiento asociado al área de trabajo de Azure Machine Learning](./media/how-to-enable-virtual-network/workspace-storage.png)](./media/how-to-enable-virtual-network/workspace-storage.png#lightbox)

1. En la página de la cuenta de servicio de almacenamiento, seleccione __Redes__.

   ![Área de redes de la página de Azure Storage en Azure Portal](./media/how-to-enable-virtual-network/storage-firewalls-and-virtual-networks.png)

1. En la pestaña __Firewalls y redes virtuales__, realice las acciones siguientes:
    1. Seleccione __Redes seleccionadas__.
    1. En __Redes virtuales__, seleccione el vínculo __Agregar red virtual existente__. Esta acción agrega la red virtual en la que reside el proceso (vea el paso 1).

        > [!IMPORTANT]
        > La cuenta de almacenamiento debe estar en la misma red virtual y subred que las instancias de proceso o clústeres usados para entrenamiento o inferencia.

    1. Seleccione la casilla __Permitir que los servicios de Microsoft de confianza accedan a esta cuenta de almacenamiento__. Este cambio no concede a todos los servicios de Azure acceso a su cuenta de almacenamiento.
    
        * Los recursos de algunos servicios, cuando **están registrados en la suscripción**, pueden obtener acceso a la cuenta de almacenamiento **de la misma suscripción** para ciertas operaciones. Por ejemplo, operaciones de escritura de registros o creación de copias de seguridad.
        * Los recursos de algunos servicios pueden conceder acceso explícito a su cuenta de almacenamiento. Para ello, __asignan un rol de Azure__ a su identidad administrada asignada por el sistema.

        Para más información, vea [Configuración de Firewalls y redes virtuales de Azure Storage](../storage/common/storage-network-security.md#trusted-microsoft-services).

    > [!IMPORTANT]
    > Al trabajar con el SDK de Azure Machine Learning, el entorno de desarrollo debe poder conectarse a la cuenta de Azure Storage. Si la cuenta de almacenamiento se encuentra dentro de una red virtual, el firewall debe permitir el acceso desde la dirección IP del entorno de desarrollo.
    >
    > Para habilitar el acceso a la cuenta de almacenamiento, visite __Firewalls y redes virtuales__ de la cuenta de almacenamiento *desde un explorador web en el cliente de desarrollo*. A continuación, use la casilla __Agregar la dirección IP del cliente__ para agregar la dirección IP del cliente al campo __INTERVALO DE DIRECCIONES__. También puede usar el campo __INTERVALO DE DIRECCIONES__ para especificar manualmente la dirección IP del entorno de desarrollo. Una vez agregada la dirección IP del cliente, este puede acceder a la cuenta de almacenamiento mediante el SDK.

   [![Panel "Firewalls y redes virtuales" de Azure Portal](./media/how-to-enable-virtual-network/storage-firewalls-and-virtual-networks-page.png)](./media/how-to-enable-virtual-network/storage-firewalls-and-virtual-networks-page.png#lightbox)

## <a name="secure-azure-storage-accounts-with-private-endpoints"></a>Protección de cuentas de almacenamiento de Azure con puntos de conexión privados

Azure Machine Learning admite cuentas de almacenamiento configuradas para usar puntos de conexión de servicio o puntos de conexión privados. Si la cuenta de almacenamiento usa puntos de conexión privados, debe configurar dos para la cuenta de almacenamiento predeterminada:
1. Un punto de conexión privado con un recurso secundario de destino de **blob**.
1. Un punto de conexión privado con un recurso secundario de destino de **archivo** (recurso compartido de archivos).

![Captura de pantalla que muestra la página de configuración de los puntos de conexión privados con opciones de blob y archivo](./media/how-to-enable-studio-virtual-network/configure-storage-private-endpoint.png)

Para configurar un punto de conexión privado para una cuenta de almacenamiento que **no** sea el almacenamiento predeterminado, seleccione el tipo **Subrecurso de destino** correspondiente a la cuenta de almacenamiento que quiere agregar.

Para más información, consulte [Uso de puntos de conexión privados para Azure Storage](../storage/common/storage-private-endpoints.md).

## <a name="secure-datastores-and-datasets"></a>Protección de almacenes de datos y conjuntos de datos

En esta sección, aprenderá a usar el almacén de datos y los conjuntos de datos de la experiencia del SDK con una red virtual. Para obtener más información sobre la experiencia de Studio, consulte [Uso de Azure Machine Learning Studio en una red virtual](how-to-enable-studio-virtual-network.md).

Para acceder a los datos mediante el SDK, debe usar el método de autenticación requerido por el servicio en el que se almacenan los datos. Por ejemplo, si registra un almacén de datos para tener acceso a Azure Data Lake Store Gen2, debe usar una entidad de servicio como se documenta en [Conexión con los servicios de Azure Storage](how-to-access-data.md#azure-data-lake-storage-generation-2).

### <a name="disable-data-validation"></a>Deshabilitación de la validación de datos

De forma predeterminada, Azure Machine Learning realiza comprobaciones de las credenciales y la validez de los datos al intentar acceder a ellos mediante el SDK. Si los datos están detrás de una red virtual, Azure Machine Learning no puede completar estas comprobaciones. Para omitir esta comprobación, debe crear almacenes de datos y conjuntos de datos que omitan la validación.

### <a name="use-datastores"></a>Uso de almacenes de datos

 En Azure Data Lake Store Gen1 y Azure Data Lake Store Gen2 la validación se omite de forma predeterminada, por lo que no es necesario realizar ninguna otra acción. Pero para los servicios siguientes puede usar una sintaxis similar a fin de omitir la validación del almacén de datos:

- Azure Blob Storage
- Recurso compartido de archivos de Azure
- PostgreSQL
- Azure SQL Database

En el ejemplo de código siguiente se crea un almacén de datos de blobs de Azure y se establece `skip_validation=True`.

```python
blob_datastore = Datastore.register_azure_blob_container(workspace=ws,  

                                                         datastore_name=blob_datastore_name,  

                                                         container_name=container_name,  

                                                         account_name=account_name, 

                                                         account_key=account_key, 

                                                         skip_validation=True ) // Set skip_validation to true
```

### <a name="use-datasets"></a>Uso de conjuntos de datos

La sintaxis para omitir la validación de conjuntos de datos es similar para los siguientes tipos de datos:
- Archivo delimitado
- JSON 
- Parquet
- SQL
- Archivo

En el código siguiente se crea un conjunto de datos JSON y se establece `validate=False`.

```python
json_ds = Dataset.Tabular.from_json_lines_files(path=datastore_paths, 

validate=False) 

```

## <a name="secure-azure-key-vault"></a>Protección de Azure Key Vault

Azure Machine Learning usa una instancia de Key Vault asociada para almacenar las siguientes credenciales:
* La cadena de conexión de cuenta de almacenamiento asociada
* Contraseñas para las instancias de Azure Container Repository
* Cadenas de conexión a almacenes de datos

Para usar las funcionalidades de experimentación de Azure Machine Learning con Azure Key Vault detrás de una red virtual, siga los pasos siguientes:

1. Vaya a la instancia de Key Vault asociada al área de trabajo.

1. En la página de __Key Vault__, en el panel izquierdo, seleccione __Redes__.

1. En la pestaña __Firewalls y redes virtuales__, realice las acciones siguientes:
    1. En __Permitir el acceso desde__, seleccione __Punto de conexión privado y redes seleccionadas__.
    1. En __Redes virtuales__, seleccione __Agregar redes virtuales existentes__ para agregar la red virtual donde reside el proceso de experimentación.
    1. En __¿Quiere permitir que los servicios de confianza de Microsoft puedan omitir este firewall?__ , seleccione __Sí__.

   [![Sección "Firewalls y redes virtuales" del panel de Key Vault](./media/how-to-enable-virtual-network/key-vault-firewalls-and-virtual-networks-page.png)](./media/how-to-enable-virtual-network/key-vault-firewalls-and-virtual-networks-page.png#lightbox)

## <a name="enable-azure-container-registry-acr"></a>Habilitación de un registro de Azure Container Registry (ACR)

Para usar Azure Container Registry en una red virtual, debe cumplir los siguientes requisitos:

* La instancia de Azure Container Registry debe tener la versión Prémium. Para más información sobre la actualización, vea [Cambio de SKU](../container-registry/container-registry-skus.md#changing-tiers).

* La instancia de Azure Container Registry debe estar en la misma red virtual y subred que la cuenta de almacenamiento y los destinos de proceso utilizados para entrenamiento o inferencia.

* El área de trabajo de Azure Machine Learning debe contener un [clúster de proceso de Azure Machine Learning](how-to-create-attach-compute-cluster.md).

    Cuando la instancia de ACR está detrás de una red virtual, Azure Machine Learning no puede usarla para compilar imágenes de Docker directamente. En su lugar, se usa el clúster de proceso para compilar las imágenes.

    > [!IMPORTANT]
    > El clúster de proceso que se usa para compilar imágenes de Docker debe poder acceder a los repositorios de paquetes que se usan para entrenar e implementar los modelos. Es posible que tenga que agregar reglas de seguridad de red que permitan el acceso a repositorios públicos, [usar paquetes de Python privados](how-to-use-private-python-packages.md) o usar [imágenes de Docker personalizadas](how-to-train-with-custom-image.md) que ya incluyan los paquetes.

Una vez cumplidos estos requisitos, siga los pasos que se indican a continuación para habilitar Azure Container Registry.

> [!TIP]
> Si no usó una instancia de Azure Container Registry existente al crear el área de trabajo, es posible que no exista ninguna. De forma predeterminada, el área de trabajo no creará ninguna instancia de ACR hasta que necesite una. Para forzar la creación de una, entrene o implemente un modelo mediante el área de trabajo antes de seguir los pasos de esta sección.

1. Busque el nombre de la instancia de Azure Container Registry para el área de trabajo mediante alguno de los métodos siguientes:

    __Azure Portal__

    En la sección de información general del área de trabajo, el valor __Registro__ se vincula a la instancia de Azure Container Registry.

    :::image type="content" source="./media/how-to-enable-virtual-network/azure-machine-learning-container-registry.png" alt-text="Azure Container Registry para el área de trabajo" border="true":::

    __CLI de Azure__

    Si ha [instalado la extensión de Machine Learning para la CLI de Azure](reference-azure-machine-learning-cli.md), puede usar el comando `az ml workspace show` para mostrar la información del área de trabajo.

    ```azurecli-interactive
    az ml workspace show -w yourworkspacename -g resourcegroupname --query 'containerRegistry'
    ```

    Este comando devuelve un valor similar a `"/subscriptions/{GUID}/resourceGroups/{resourcegroupname}/providers/Microsoft.ContainerRegistry/registries/{ACRname}"`. La última parte de la cadena es el nombre de Azure Container Registry para el área de trabajo.

1. Limite el acceso a la red virtual mediante los pasos descritos en [Configuración del acceso de red al registro](../container-registry/container-registry-vnet.md#configure-network-access-for-registry). Al agregar la red virtual, seleccione la red virtual y la subred para los recursos de Azure Machine Learning.

1. Configure la instancia de ACR del área de trabajo para [permitir el acceso de los servicios de confianza](../container-registry/allow-access-trusted-services.md).

1. Use el SDK de Python para Azure Machine Learning para configurar un clúster de proceso para compilar imágenes de Docker. El siguiente fragmento de código muestra cómo hacerlo:

    ```python
    from azureml.core import Workspace
    # Load workspace from an existing config file
    ws = Workspace.from_config()
    # Update the workspace to use an existing compute cluster
    ws.update(image_build_compute = 'mycomputecluster')
    # To switch back to using ACR to build (if ACR is not in the VNet):
    # ws.update(image_build_compute = '')
    ```

    > [!IMPORTANT]
    > La cuenta de almacenamiento, el clúster de proceso y Azure Container Registry deben estar en la misma subred de la red virtual.
    
    Para más información, vea la referencia del método [update()](/python/api/azureml-core/azureml.core.workspace.workspace#update-friendly-name-none--description-none--tags-none--image-build-compute-none--enable-data-actions-none-).

## <a name="next-steps"></a>Pasos siguientes

Este artículo es la segunda parte de una serie de cinco capítulos sobre redes virtuales. Vea el resto de los artículos para obtener información sobre cómo proteger una red virtual:

* [Parte 1: Introducción a las redes virtuales](how-to-network-security-overview.md)
* [Parte 3: Protección del entorno de entrenamiento](how-to-secure-training-vnet.md)
* [Parte 4: Protección del entorno de inferencia](how-to-secure-inferencing-vnet.md)
* [Parte 5: Habilitación de la funcionalidad de Studio](how-to-enable-studio-virtual-network.md)

Consulte también el artículo sobre el uso de [DNS personalizado](how-to-custom-dns.md) para la resolución de nombres.
