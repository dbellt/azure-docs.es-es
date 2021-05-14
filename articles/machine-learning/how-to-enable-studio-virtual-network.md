---
title: Habilitación de Azure Machine Learning Studio en una red virtual
titleSuffix: Azure Machine Learning
description: Aprenda a configurar Estudio de Azure Machine Learning para tener acceso a los datos almacenados en una red virtual.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: how-to
ms.reviewer: larryfr
ms.author: aashishb
author: aashishb
ms.date: 10/21/2020
ms.custom: contperf-fy20q4, tracking-python
ms.openlocfilehash: 13becdf8c49d9affe8c2946d6147707fbe954437
ms.sourcegitcommit: 5ce88326f2b02fda54dad05df94cf0b440da284b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107889329"
---
# <a name="use-azure-machine-learning-studio-in-an-azure-virtual-network"></a>Habilitación de Azure Machine Learning Studio en una Azure Virtual Network

En este artículo, aprenderá a usar Azure Machine Learning Studio en una red virtual. Studio incluye características como AutoML, el diseñador y el etiquetado de datos. Para poder usar estas características en una red virtual, debe seguir los pasos de este artículo.

En este artículo aprenderá a:

> [!div class="checklist"]
> - Proporcionar a Studio acceso a los datos almacenados dentro de una red virtual.
> - Obtener acceso a Studio desde un recurso dentro de una red virtual.
> - Comprender cómo Studio afecta a la seguridad del almacenamiento.

Este artículo es la quinta parte de una serie de cinco capítulos que le guía a través de la protección de un flujo de trabajo de Azure Machine Learning. Le recomendamos encarecidamente que lea las partes anteriores para configurar un entorno de red virtual.

Consulte los demás artículos de esta serie:

[1. Introducción a las redes virtuales](how-to-network-security-overview.md) > [2. Protección del área de trabajo](how-to-secure-workspace-vnet.md) > [3. Protección del entorno de entrenamiento](how-to-secure-training-vnet.md) > [4. Protección del entorno de inferencia](how-to-secure-inferencing-vnet.md) > **5. Habilitación de la funcionalidad de Studio**


> [!IMPORTANT]
> Si el área de trabajo está en una __nube soberana__, como Azure Government o Azure China 21Vianet, los cuadernos integrados _no_ admiten el uso de almacenamiento que se encuentra en una red virtual. En su lugar, puede usar cuadernos de Jupyter Notebook en una instancia de Compute. Para obtener más información, consulte la sección [Acceso a los datos en un cuaderno de instancia de Compute](how-to-secure-training-vnet.md#access-data-in-a-compute-instance-notebook).

## <a name="prerequisites"></a>Requisitos previos

+ Lea el artículo [Introducción a la seguridad de red](how-to-network-security-overview.md) para comprender la arquitectura y los escenarios comunes de redes virtuales.

+ Una red virtual y una subred preexistentes que se usarán.

+ Un [área de trabajo existente de Azure Machine Learning con Private Link habilitado](how-to-secure-workspace-vnet.md#secure-the-workspace-with-private-endpoint).

+ Una [cuenta de Azure Storage existente agregada a la red virtual](how-to-secure-workspace-vnet.md#secure-azure-storage-accounts-with-service-endpoints).

## <a name="configure-data-access-in-the-studio"></a>Configuración del acceso a datos en Studio

Algunas de las características de Studio están deshabilitadas de forma predeterminada en una red virtual. Para volver a habilitarlas, debe habilitar la identidad administrada para las cuentas de almacenamiento que desea usar en Studio. 

Las siguientes operaciones están deshabilitadas de forma predeterminada en una red virtual:

* Vista previa de los datos en Studio.
* Visualización de los datos en el diseñador.
* Implementación de un modelo en el diseñador ([cuenta de almacenamiento predeterminada](#enable-managed-identity-authentication-for-default-storage-accounts)).
* Envío de experimentos de AutoML ([cuenta de almacenamiento predeterminada](#enable-managed-identity-authentication-for-default-storage-accounts)).
* Inicio de un proyecto de etiquetado.

Studio admite la lectura de datos de los siguientes tipos de almacén de datos en una red virtual:

* Blob de Azure
* Azure Data Lake Storage Gen1
* Azure Data Lake Storage Gen2
* Azure SQL Database

### <a name="configure-datastores-to-use-workspace-managed-identity"></a>Configuración de almacenes de datos para usar una identidad administrada del área de trabajo

Después de agregar una cuenta de Azure Storage a la red virtual con un [punto de conexión de servicio](how-to-secure-workspace-vnet.md#secure-azure-storage-accounts-with-service-endpoints) o un [punto de conexión privado](how-to-secure-workspace-vnet.md#secure-azure-storage-accounts-with-private-endpoints), debe configurar el almacén de datos para usar la autenticación de [identidad administrada](../active-directory/managed-identities-azure-resources/overview.md). Esto permite que Studio tenga acceso a los datos de la cuenta de almacenamiento.

Azure Machine Learning usa [almacenes de datos](concept-data.md#datastores) para conectarse a las cuentas de almacenamiento. Siga los pasos que se indican a continuación para configurar un almacén de datos para que use una identidad administrada:

1. En Studio, seleccione __Almacenes de datos__.

1. Para actualizar un almacén de datos existente, seleccione el almacén de datos y después __Actualizar credenciales__.

    Para crear un almacén de datos, seleccione __+ Nuevo almacén de datos__.

1. En la configuración del almacén de datos, seleccione __Sí__ para __Usar la identidad administrada del área de trabajo para la vista previa de datos y la generación de perfiles en Azure Machine Learning Studio__.

    ![Captura de pantalla que muestra cómo habilitar la identidad administrada del área de trabajo](./media/how-to-enable-studio-virtual-network/enable-managed-identity.png)

En estos pasos se agrega la identidad administrada del área de trabajo como __Lector__ al servicio de almacenamiento mediante el control de acceso basado en roles de Azure RBAC. El acceso __Lector__ permite que el área de trabajo recupere la configuración del firewall para asegurarse de que los datos no salgan de la red virtual. Los cambios pueden tardar hasta 10 minutos en surtir efecto.

### <a name="enable-managed-identity-authentication-for-default-storage-accounts"></a>Habilitación de la autenticación de identidad administrada para cuentas de almacenamiento predeterminadas

Cada área de trabajo de Azure Machine Learning tiene dos cuentas de almacenamiento, una cuenta de almacenamiento de blobs y una cuenta de almacenamiento de archivos predeterminadas que se definen al crear el área de trabajo. También puede establecer nuevos valores predeterminados en la página de administración del **almacén de datos**.

![Captura de pantalla que muestra dónde se pueden encontrar los almacenes de datos predeterminados](./media/how-to-enable-studio-virtual-network/default-datastores.png)

En la tabla siguiente se describe por qué debe habilitar la autenticación de identidad administrada para las cuentas de almacenamiento predeterminadas del área de trabajo.

|Cuenta de almacenamiento  | Notas  |
|---------|---------|
|Almacenamiento de blobs predeterminado del área de trabajo| Almacena recursos del modelo desde el diseñador. Debe habilitar la autenticación de identidad administrada en esta cuenta de almacenamiento para implementar modelos en el diseñador. <br> <br> Puede visualizar y ejecutar una canalización del diseñador si usa un almacén de datos no predeterminado que se ha configurado para utilizar una identidad administrada. Sin embargo, si intenta implementar un modelo entrenado sin la identidad administrada habilitada en el almacén de datos predeterminado, se producirá un error en la implementación independientemente de que se usen otros almacenes de datos.|
|Almacén de archivos predeterminado del área de trabajo| Almacena los recursos de experimentos de AutoML. Debe habilitar la autenticación de identidad administrada en esta cuenta de almacenamiento para enviar experimentos de AutoML. |

> [!WARNING]
> Hay un problema conocido en el que el almacén de archivos predeterminado no crea automáticamente la carpeta `azureml-filestore` necesaria para enviar experimentos de aprendizaje automático automatizado. Esto sucede cuando los usuarios seleccionan un almacén de archivos existente para establecerlo como predeterminado durante la creación del área de trabajo.
> 
> Tiene dos opciones para evitar este problema: 1) Use el almacén de archivos predeterminado que se crea automáticamente durante la creación del área de trabajo. 2) Para seleccionar su propio almacén de archivos, asegúrese de que este está fuera de la red virtual durante la creación del área de trabajo. Una vez creada el área de trabajo, agregue la cuenta de almacenamiento a la red virtual.
>
> Para resolver este problema, elimine la cuenta del almacén de archivos de la red virtual y, a continuación, agréguela de nuevo.

### <a name="grant-workspace-managed-identity-__reader__-access-to-storage-private-link"></a>Concesión de acceso __Lector__ de identidad administrada del área de trabajo al vínculo privado de almacenamiento

Si la cuenta de almacenamiento de Azure usa un punto de conexión privado, debe conceder a la identidad administrada del área de trabajo acceso **Lector** al vínculo privado. Para más información, consulte el rol integrado [Lector](../role-based-access-control/built-in-roles.md#reader). 

Si la cuenta de almacenamiento usa un punto de conexión de servicio, puede omitir este paso.

## <a name="access-the-studio-from-a-resource-inside-the-vnet"></a>Acceso a Studio desde un recurso dentro de una red virtual

Si accede a Studio desde un recurso dentro de una red virtual (por ejemplo, una instancia de proceso o una máquina virtual), tendrá que permitir el tráfico de salida desde la red virtual a Studio. 

Por ejemplo, si usa grupos de seguridad de red (NSG) para restringir el tráfico de salida, agregue una regla a un destino de __etiqueta de servicio__ de __AzureFrontDoor.Frontend__.

## <a name="technical-notes-for-managed-identity"></a>Notas técnicas de identidad administrada

El uso de identidad administrada para tener acceso a los servicios de almacenamiento afecta a las consideraciones de seguridad. En esta sección se describen los cambios para cada tipo de cuenta de almacenamiento. 

Estas consideraciones son únicas en cuanto al __tipo de cuenta de almacenamiento__ al que se obtiene acceso.

### <a name="azure-blob-storage"></a>Azure Blob Storage

Para __Azure Blob Storage__, la identidad administrada del área de trabajo también se agrega como un [lector de datos de blob](../role-based-access-control/built-in-roles.md#storage-blob-data-reader) para que pueda leer datos del almacenamiento de blobs.

### <a name="azure-data-lake-storage-gen2-access-control"></a>Control de acceso de Azure Data Lake Storage Gen2

Puede usar RBAC de Azure y listas de control de acceso (ACL) de tipo POSIX para controlar el acceso a los datos dentro de una red virtual.

Para usar RBAC de Azure, agregue la identidad administrada del área de trabajo al rol [Lector de datos de blob](../role-based-access-control/built-in-roles.md#storage-blob-data-reader). Para obtener más información, consulte [Control de acceso basado en roles de Azure](../storage/blobs/data-lake-storage-access-control-model.md#role-based-access-control).

Para usar ACL, se puede asignar el acceso de la identidad administrada del área de trabajo como cualquier otra entidad de seguridad. Para obtener más información, vea [Listas de control de acceso en archivos y directorios](../storage/blobs/data-lake-storage-access-control.md#access-control-lists-on-files-and-directories).

### <a name="azure-data-lake-storage-gen1-access-control"></a>Control de acceso de Azure Data Lake Storage Gen1

Azure Data Lake Storage Gen1 solo admite listas de control de acceso de estilo POSIX. Puede asignar el acceso de la identidad administrada del área de trabajo a los recursos como cualquier otra entidad de seguridad. Para obtener más información, vea [Control de acceso en Azure Data Lake Storage Gen1](../data-lake-store/data-lake-store-access-control.md).

### <a name="azure-sql-database-contained-user"></a>Usuario independiente de Azure SQL Database

Para acceder a los datos almacenados en una instancia de Azure SQL Database mediante la identidad administrada, debe crear un usuario independiente de SQL que se asigne a la identidad administrada. Para obtener más información sobre cómo crear un usuario desde un proveedor externo, vea [Creación de usuarios independientes asignados a identidades de Azure AD](../azure-sql/database/authentication-aad-configure.md#create-contained-users-mapped-to-azure-ad-identities).

Después de crear un usuario independiente de SQL, utilice el [comando GRANT de T-SQL](/sql/t-sql/statements/grant-object-permissions-transact-sql) para concederle permisos.

### <a name="azure-machine-learning-designer-intermediate-module-output"></a>Salida del módulo intermedio del diseñador de Azure Machine Learning

Puede especificar la ubicación de salida de cualquier módulo en el diseñador. Úselo para almacenar conjuntos de datos intermedios en una ubicación independiente para la seguridad, el registro o la auditoría. Para especificar la salida:

1. Seleccione el módulo cuya salida quiere especificar.
1. En el panel de configuración del módulo que aparece a la derecha, seleccione **Configuración de salida**.
1. Especifique el almacén de datos que desea usar para cada salida del módulo.
 
Asegúrese de que tiene acceso a las cuentas de almacenamiento intermedias en la red virtual. De lo contrario, se producirá un error en la canalización.

También debe [habilitar la autenticación de identidad administrada](#configure-datastores-to-use-workspace-managed-identity) para las cuentas de almacenamiento intermedias para visualizar los datos de salida.

## <a name="next-steps"></a>Pasos siguientes

Este artículo es la quinta parte de una serie de cinco capítulos sobre redes virtuales. Vea el resto de los artículos para obtener información sobre cómo proteger una red virtual:

* [Parte 1: Introducción a las redes virtuales](how-to-network-security-overview.md)
* [Parte 2: Protección de los recursos de un área de trabajo](how-to-secure-workspace-vnet.md)
* [Parte 3: Protección del entorno de entrenamiento](how-to-secure-training-vnet.md)
* [Parte 4: Protección del entorno de inferencia](how-to-secure-inferencing-vnet.md)

Consulte también el artículo sobre el uso de [DNS personalizado](how-to-custom-dns.md) para la resolución de nombres.