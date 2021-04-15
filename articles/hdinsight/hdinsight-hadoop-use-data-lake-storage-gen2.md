---
title: Uso de Azure Data Lake Storage Gen2 con clústeres de Azure HDInsight
description: Obtenga información sobre cómo usar Azure Data Lake Storage Gen2 con clústeres de Azure HDInsight.
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive,seoapr2020
ms.date: 04/24/2020
ms.openlocfilehash: ea9dc2627d5a6498f69ca8c61a9cac8089816886
ms.sourcegitcommit: 2654d8d7490720a05e5304bc9a7c2b41eb4ae007
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107378591"
---
# <a name="use-azure-data-lake-storage-gen2-with-azure-hdinsight-clusters"></a>Uso de Azure Data Lake Storage Gen2 con clústeres de Azure HDInsight

[Azure Data Lake Storage Gen2](../storage/blobs/data-lake-storage-introduction.md) es un servicio de almacenamiento en la nube dedicado al análisis de macrodatos e integrado en [Azure Blob Storage](../storage/blobs/storage-blobs-introduction.md). Data Lake Storage Gen2 combina las funcionalidades de Azure Blob Storage y Azure Data Lake Storage Gen1. El servicio resultante ofrece características de Azure Data Lake Storage Gen1, entre las que se incluyen la semántica del sistema de archivos, la seguridad de nivel de directorio y de archivo y la adaptabilidad. Junto con las funcionalidades de bajo costo, almacenamiento en capas, alta disponibilidad y recuperación ante desastres de Azure Blob Storage.

Para ver una comparación completa de las opciones de creación de clústeres con Data Lake Storage Gen2, consulte [Comparación de opciones de almacenamiento para los clústeres de Azure HDInsight](hdinsight-hadoop-compare-storage-options.md).

[!INCLUDE [delete-cluster-warning](../../includes/hdinsight-delete-cluster-warning.md)]

## <a name="data-lake-storage-gen2-availability"></a>Disponibilidad de Data Lake Storage Gen2

Data Lake Storage Gen2 está disponible como opción de almacenamiento para casi todos los tipos de clúster de Azure HDInsight como una cuenta de almacenamiento predeterminada o adicional. Aun así, HBase solo puede tener una cuenta con Data Lake Storage Gen2.

> [!Note]  
> Una vez que selecciona Data Lake Storage Gen2 como el **tipo de almacenamiento principal**, no puede seleccionar Data Lake Storage Gen1 como almacenamiento adicional.

## <a name="create-hdinsight-clusters-using-data-lake-storage-gen2"></a>Creación de clústeres de HDInsight con Data Lake Storage Gen2

Use los vínculos siguientes para obtener instrucciones detalladas sobre cómo crear clústeres de HDInsight con acceso a Data Lake Storage Gen2.

* [Uso del portal](../hdinsight/hdinsight-hadoop-use-data-lake-storage-gen2-portal.md)
* [Uso de la CLI de Azure](../hdinsight/hdinsight-hadoop-use-data-lake-storage-gen2-azure-cli.md)
* PowerShell no se admite actualmente para crear un clúster de HDInsight con Azure Data Lake Storage Gen2.

## <a name="access-control-for-data-lake-storage-gen2-in-hdinsight"></a>Control de acceso para Data Lake Storage Gen2 en HDInsight

### <a name="what-kinds-of-permissions-does-data-lake-storage-gen2-support"></a>¿Qué tipos de permisos admite Data Lake Storage Gen2?

Data Lake Storage Gen2 usa un modelo de control de acceso compatible con el control de acceso basado en roles de Azure (Azure RBAC) y las listas de control de acceso (ACL) de tipo POSIX. Data Lake Storage Gen1 admite las listas de control de acceso solo para controlar el acceso a los datos.

Azure RBAC usa las asignaciones de roles para aplicar de forma eficaz conjuntos de permisos a usuarios, grupos y entidades de servicio para los recursos de Azure. Normalmente, los recursos de Azure están limitados a los recursos de nivel superior (por ejemplo, las cuentas de Azure Blob Storage). En el caso de Azure Blob Storage y también de Data Lake Storage Gen2, este mecanismo se ha ampliado al recurso del sistema de archivos.

Para obtener más información sobre los permisos de archivo con Azure RBAC, consulte el [Control de acceso basado en roles de Azure (Azure RBAC)](../storage/blobs/data-lake-storage-access-control-model.md#role-based-access-control).

Para más información sobre los permisos de archivo con las listas de control de acceso, consulte [Listas de control de acceso en archivos y directorios](../storage/blobs/data-lake-storage-access-control.md).

### <a name="how-do-i-control-access-to-my-data-in-data-lake-storage-gen2"></a>¿Cómo se controla el acceso a mis datos en Data Lake Storage Gen2?

La capacidad que el clúster de HDInsight tiene de acceder a los archivos en Data Lake Storage Gen2 se controla mediante las identidades administradas. Una identidad administrada es una identidad registrada en Azure Active Directory (Azure AD) cuyas credenciales son administradas por Azure. Con identidades administradas, no es necesario registrar entidades de servicio en Azure AD. Tampoco mantener credenciales como certificados.

Los servicios de Azure tienen dos tipos de identidades administradas: asignadas por el sistema y asignadas por el usuario. HDInsight usa las identidades administradas asignadas por el usuario para acceder a Data Lake Storage Gen2. Una `user-assigned managed identity` se crea como un recurso de Azure independiente. Mediante un proceso de creación, Azure crea una identidad en el inquilino de Azure AD de confianza para la suscripción que se utiliza. Una vez creada la identidad, esta puede asignarse a una o varias instancias de servicio de Azure.

El ciclo de vida de una identidad asignada por el usuario no se administra junto con el ciclo de vida de las instancias de servicio de Azure a las que se asigna. Para más información sobre las identidades administradas, consulte el artículo sobre [¿Qué son las identidades administradas para recursos de Azure?](../active-directory/managed-identities-azure-resources/overview.md).

### <a name="how-do-i-set-permissions-for-azure-ad-users-to-query-data-in-data-lake-storage-gen2-by-using-hive-or-other-services"></a>¿Cómo puedo establecer los permisos para que los usuarios de Azure AD consulten los datos en Data Lake Storage Gen2 mediante Hive u otros servicios?

Para establecer los permisos para que los usuarios consulten los datos, utilice grupos de seguridad de Azure AD como la entidad de seguridad asignada en las ACL. No asigne directamente permisos de acceso a archivos a usuarios individuales ni a entidades de servicio. Si se usan los grupos de seguridad de Azure AD para controlar el flujo de los permisos, puede agregar y quitar usuarios o entidades de servicio sin volver a aplicar las ACL a la estructura de todo un directorio. Solo tiene que agregar o quitar los usuarios del grupo de seguridad de Azure AD adecuado. Las ACL no se heredan y, por ello, para volver a aplicar las ACL es necesario actualizarlas en cada archivo y subdirectorio.

## <a name="access-files-from-the-cluster"></a>Acceso a los archivos desde el clúster

Existen varias maneras de acceder a los archivos de Data Lake Storage Gen2 desde un clúster de HDInsight.

* **Con el nombre completo**. Con este enfoque, proporciona la ruta de acceso completa al archivo al que quiere acceder.

    ```
    abfs://<containername>@<accountname>.dfs.core.windows.net/<file.path>/
    ```

* **Con el formato abreviado de la ruta de acceso**. Con este enfoque, reemplazará la ruta de acceso a la raíz del clúster por:

    ```
    abfs:///<file.path>/
    ```

* **Con la ruta de acceso relativa**. Con este enfoque, solo proporciona la ruta de acceso relativa al archivo al que quiere acceder.

    ```
    /<file.path>/
    ```

### <a name="data-access-examples"></a>Ejemplos de acceso a datos

Los ejemplos se basan en una [conexión SSH](./hdinsight-hadoop-linux-use-ssh-unix.md) al nodo principal del clúster. En los ejemplos se usan los tres esquemas de URI. Reemplace `CONTAINERNAME` y `STORAGEACCOUNT` por los valores pertinentes.

#### <a name="a-few-hdfs-commands"></a>Algunos comandos HDFS

1. Crear un archivo en el almacenamiento local.

    ```bash
    touch testFile.txt
    ```

1. Crear directorios en el almacenamiento de clúster.

    ```bash
    hdfs dfs -mkdir abfs://CONTAINERNAME@STORAGEACCOUNT.dfs.core.windows.net/sampledata1/
    hdfs dfs -mkdir abfs:///sampledata2/
    hdfs dfs -mkdir /sampledata3/
    ```

1. Copiar datos del almacenamiento local al almacenamiento de clúster.

    ```bash
    hdfs dfs -copyFromLocal testFile.txt  abfs://CONTAINERNAME@STORAGEACCOUNT.dfs.core.windows.net/sampledata1/
    hdfs dfs -copyFromLocal testFile.txt  abfs:///sampledata2/
    hdfs dfs -copyFromLocal testFile.txt  /sampledata3/
    ```

1. Mostrar el contenido del directorio en el almacenamiento de clúster.

    ```bash
    hdfs dfs -ls abfs://CONTAINERNAME@STORAGEACCOUNT.dfs.core.windows.net/sampledata1/
    hdfs dfs -ls abfs:///sampledata2/
    hdfs dfs -ls /sampledata3/
    ```

#### <a name="creating-a-hive-table"></a>Creación de una tabla de Hive

A efectos ilustrativos, se muestran tres ubicaciones de archivos. En la ejecución real, use solo una de las entradas `LOCATION`.

```hql
DROP TABLE myTable;
CREATE EXTERNAL TABLE myTable (
    t1 string,
    t2 string,
    t3 string,
    t4 string,
    t5 string,
    t6 string,
    t7 string)
ROW FORMAT DELIMITED FIELDS TERMINATED BY ' '
STORED AS TEXTFILE
LOCATION 'abfs://CONTAINERNAME@STORAGEACCOUNT.dfs.core.windows.net/example/data/';
LOCATION 'abfs:///example/data/';
LOCATION '/example/data/';
```

## <a name="next-steps"></a>Pasos siguientes

* [Azure HDInsight integration with Data Lake Storage Gen2 preview - ACL and security update](https://azure.microsoft.com/blog/azure-hdinsight-integration-with-data-lake-storage-gen-2-preview-acl-and-security-update/) (Integración de Azure HDInsight con Data Lake Storage Gen2 [versión preliminar]: ACL y actualización de seguridad)
* [Introducción a Azure Data Lake Storage Gen2](../storage/blobs/data-lake-storage-introduction.md)
* [Tutorial: Extracción, transformación y carga de datos mediante Interactive Query en Azure HDInsight](./interactive-query/interactive-query-tutorial-analyze-flight-data.md)
