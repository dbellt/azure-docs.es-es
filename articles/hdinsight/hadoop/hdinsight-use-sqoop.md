---
title: Ejecutar trabajos de Apache Sqoop con Azure HDInsight (Apache Hadoop)
description: Aprenda a utilizar Azure PowerShell desde una estación de trabajo para ejecutar la importación y la exportación en Sqoop entre un clúster de Hadoop y una base de datos de Azure SQL.
ms.service: hdinsight
ms.topic: how-to
ms.date: 12/06/2019
ms.openlocfilehash: 165660a0141c50cf1c6dc24e3ba8df0299f885f2
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110069011"
---
# <a name="use-apache-sqoop-with-hadoop-in-hdinsight"></a>Uso de Apache Sqoop con Hadoop en HDInsight

[!INCLUDE [sqoop-selector](../../../includes/hdinsight-selector-use-sqoop.md)]

Aprenda a utilizar Apache Sqoop en HDInsight para importar y exportar datos entre un clúster de HDInsight y Azure SQL Database.

A pesar de que Apache Hadoop es una opción natural para procesar datos no estructurados y datos semiestructurados, como registros y archivos, es posible que también sea necesario procesar datos estructurados almacenados en bases de datos relacionales.

[Apache Sqoop](https://sqoop.apache.org/docs/1.99.7/user.html) es una herramienta diseñada para transferir datos entre clústeres de Hadoop y bases de datos relacionales. Puede usarla para importar datos desde un sistema de administración de bases de datos relacionales (RDBMS) como SQL Server, MySQL u Oracle en el sistema de archivos distribuidos Hadoop (HDFS), transformar los datos de Hadoop con MapReduce o Apache Hive y, a continuación, exportar los datos en un RDBMS. En este artículo, usará Azure SQL Database como base de datos relacional.

> [!IMPORTANT]  
> En este artículo se configura un entorno de prueba para realizar la transferencia de datos. A continuación, se elige un método de transferencia de datos para este entorno entre los métodos de la sección [Ejecución de trabajos de Sqoop](#run-sqoop-jobs), más adelante.

Para ver las versiones de Sqoop compatibles con los clústeres de HDInsight, consulte [Novedades en las versiones de clústeres proporcionadas por HDInsight](../hdinsight-component-versioning.md).

## <a name="understand-the-scenario"></a>Descripción del escenario

El clúster de HDInsight incluye algunos datos de ejemplo. Utilice los dos ejemplos siguientes:

* Un archivo de registro de Apache Log4j, ubicado en `/example/data/sample.log`. Los registros siguientes se extraen del archivo:

```text
2012-02-03 18:35:34 SampleClass6 [INFO] everything normal for id 577725851
2012-02-03 18:35:34 SampleClass4 [FATAL] system problem at id 1991281254
2012-02-03 18:35:34 SampleClass3 [DEBUG] detail for id 1304807656
...
```

* Una tabla de Hive denominada `hivesampletable`, que hace referencia al archivo de datos ubicado en `/hive/warehouse/hivesampletable`. La tabla contiene algunos datos del dispositivo móvil.
  
  | Campo | Tipo de datos |
  | --- | --- |
  | clientid |string |
  | querytime |string |
  | market |string |
  | deviceplatform |string |
  | devicemake |string |
  | devicemodel |string |
  | state |string |
  | country |string |
  | querydwelltime |double |
  | sessionid |bigint |
  | sessionpagevieworder |bigint |

En este artículo, usará estos dos conjuntos de datos para probar los procesos de importación y exportación de Sqoop.

## <a name="set-up-test-environment"></a><a name="create-cluster-and-sql-database"></a>Configuración del entorno de prueba

El clúster, la instancia de SQL Database y otros objetos se crean a través de Azure Portal mediante una plantilla de Azure Resource Manager. La plantilla se puede encontrar en [Plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/101-hdinsight-linux-with-sql-database/). La plantilla de Resource Manager llama a un paquete de bacpac para que implemente los esquemas de tabla en SQL Database.  El paquete bacpac se encuentra en un contenedor de blobs público, https://hditutorialdata.blob.core.windows.net/usesqoop/SqoopTutorial-2016-2-23-11-2.bacpac. Si desea usar un contenedor privado para los archivos bacpac, utilice los siguientes valores en la plantilla:

```json
"storageKeyType": "Primary",
"storageKey": "<TheAzureStorageAccountKey>",
```

> [!NOTE]  
> La importación mediante Azure Portal solo admite la importación de un archivo BACPAC desde Azure Blob Storage.

1. Seleccione en la imagen siguiente para abrir una plantilla de Resource Manager en Azure Portal.

    <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fquickstarts%2Fmicrosoft.hdinsight%2Fhdinsight-linux-with-sql-database%2Fazuredeploy.json" target="_blank"><img src="./media/hdinsight-use-sqoop/hdi-deploy-to-azure1.png" alt="Deploy to Azure button for new cluster"></a>

2. Especifique las siguientes propiedades:

    |Campo |Valor |
    |---|---|
    |Suscripción |Seleccione la suscripción de Azure en la lista desplegable.|
    |Resource group |Seleccione el grupo de recursos en la lista desplegable o cree uno nuevo.|
    |Location |Seleccione una región en la lista desplegable.|
    |Cluster Name |Escriba el nombre del clúster de Hadoop. Utilice solo letras minúsculas.|
    |Cluster Login User Name (Nombre de usuario de inicio de sesión del clúster) |Mantenga el valor rellenado previamente `admin`.|
    |Cluster Login Password (Contraseña de inicio de sesión del clúster) |Escriba una contraseña.|
    |Nombre de usuario de SSH |Mantenga el valor rellenado previamente `sshuser`.|
    |Contraseña de SSH |Escriba una contraseña.|
    |Inicio de sesión de administrador de SQL |Mantenga el valor rellenado previamente `sqluser`.|
    |Contraseña del administrador de SQL |Escriba una contraseña.|
    |_artifacts Location (Ubicación de _artefactos) | Use el valor predeterminado a no ser que quiera usar su propio archivo .bacpac en una ubicación diferente.|
    |_artifacts Location Sas Token (Token de Sas de ubicación de _artefactos) |déjelo en blanco.|
    |Bacpac File Name (Nombre de archivo .bacpac) |Use el valor predeterminado a no ser que quiera usar su propio archivo .bacpac.|
    |Location |Use el valor predeterminado.|

    El nombre del [servidor lógico de SQL Server](../../azure-sql/database/logical-servers.md) será `<ClusterName>dbserver`. El nombre de la base de datos será `<ClusterName>db`. El nombre predeterminado de la cuenta de almacenamiento será `e6qhezrh2pdqu`.

3. Seleccione **Acepto los términos y condiciones indicados anteriormente**.

4. Seleccione **Comprar**. Verá un icono nuevo llamado Envío de implementación para la implementación de plantilla. Tarda aproximadamente 20 minutos en crear un clúster y la base de datos SQL.

## <a name="run-sqoop-jobs"></a>Ejecución de trabajos de Sqoop

HDInsight puede ejecutar trabajos de Sqoop mediante una variedad de métodos. Use la tabla siguiente para decidir cuál es el método adecuado para usted luego siga el vínculo para ver un tutorial.

| **Utilícelo** si desea... | ...un shell **interactivo** | ...procesamiento por **lotes** | ...desde este **sistema operativo de cliente** |
|:--- |:---:|:---:|:--- |:--- |
| [SSH](apache-hadoop-use-sqoop-mac-linux.md) |? |? |Linux, Unix, Mac OS X o Windows |
| [.NET SDK para Hadoop](apache-hadoop-use-sqoop-dotnet-sdk.md) |&nbsp; |?  |Windows (por ahora) |
| [Azure PowerShell](apache-hadoop-use-sqoop-powershell.md) |&nbsp; |? |Windows |

## <a name="limitations"></a>Limitaciones

* Exportación masiva: con HDInsight basado en Linux, el conector Sqoop que se utiliza para exportar datos a Microsoft SQL Server o SQL Database no es compatible actualmente con las inserciones masivas.
* Procesamiento por lotes: con HDInsight basado en Linux, cuando se usa `-batch` al realizar inserciones, Sqoop realiza varias inserciones en lugar de procesar por lotes las operaciones de inserción.

## <a name="next-steps"></a>Pasos siguientes

Ahora ya ha aprendido a usar Sqoop. Para obtener más información, consulte:

* [Uso de Apache Hive con HDInsight](./hdinsight-use-hive.md)
* [Carga de datos en HDInsight](../hdinsight-upload-data.md): busque otros métodos para cargar datos en HDInsight o Azure Blob Storage.
* [Usar Apache Sqoop para importar y exportar datos entre Apache Hadoop en HDInsight y SQL Database](./apache-hadoop-use-sqoop-mac-linux.md)