---
title: Transformación de datos en la nube con PowerShell
description: Este script de PowerShell transforma los datos en la nube mediante la ejecución del programa de Spark en un clúster de Spark de Azure HDInsight.
author: dcstwh
ms.author: weetok
ms.reviewer: jburchel
ms.service: data-factory
ms.topic: conceptual
ms.custom: seo-lt-2019, devx-track-azurepowershell
ms.date: 09/12/2017
ms.openlocfilehash: 4cb84b437097ba0d1953782004369c0b3a1fcb6d
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110665406"
---
# <a name="powershell-script---transform-data-in-cloud-using-azure-data-factory"></a>Script de PowerShell: transformación de datos en la nube con Azure Data Factory

Este script de PowerShell de ejemplo crea una canalización que transforma los datos en la nube mediante la ejecución del programa de Spark en un clúster de Spark de Azure HDInsight. 

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

[!INCLUDE [sample-powershell-install](../../../includes/sample-powershell-install-no-ssh-az.md)]

## <a name="prerequisites"></a>Requisitos previos
* **Cuenta de Azure Storage**. Cree un script de Python y un archivo de entrada y cárguelos en Azure Storage. La salida del programa Spark se almacena en esta cuenta de almacenamiento. El clúster de Spark a petición usa la misma cuenta de almacenamiento que el almacenamiento principal.  

### <a name="upload-python-script-to-your-blob-storage-account"></a>Carga del script de Python en la cuenta de Blob Storage
1. Cree un archivo de Python denominado **WordCount_Spark.py** con el siguiente contenido: 

    ```python
    import sys
    from operator import add
    
    from pyspark.sql import SparkSession
    
    def main():
        spark = SparkSession\
            .builder\
            .appName("PythonWordCount")\
            .getOrCreate()
            
        lines = spark.read.text("wasbs://adftutorial@<storageaccountname>.blob.core.windows.net/spark/inputfiles/minecraftstory.txt").rdd.map(lambda r: r[0])
        counts = lines.flatMap(lambda x: x.split(' ')) \
            .map(lambda x: (x, 1)) \
            .reduceByKey(add)
        counts.saveAsTextFile("wasbs://adftutorial@<storageaccountname>.blob.core.windows.net/spark/outputfiles/wordcount")
        
        spark.stop()
    
    if __name__ == "__main__":
        main()
    ```
2. Reemplace **&lt;storageaccountname&gt;** por el nombre de la cuenta de Azure Storage. A continuación, guarde el archivo. 
3. En Azure Blob Storage, cree un contenedor denominado **adftutorial** si no existe. 
4. Cree una carpeta llamada **spark**.
5. Cree una subcarpeta denominada **script** en la carpeta **spark**. 
6. Cargue el archivo **WordCount_Spark.py** a la subcarpeta **script**. 


### <a name="upload-the-input-file"></a>Carga del archivo de entrada
1. Cree un archivo denominado **minecraftstory.txt** con algo de texto. El programa Spark contará el número de palabras de este texto. 
2. Cree una subcarpeta denominada `inputfiles` en la carpeta `spark` del contenedor de blobs. 
3. Cargue `minecraftstory.txt` a la subcarpeta `inputfiles`. 

## <a name="sample-script"></a>Script de ejemplo
> [!IMPORTANT]
> Este script crea archivos JSON que definen entidades de Data Factory (servicio vinculado, conjunto de datos y canalización) en la carpeta c:\ del disco duro.

[!code-powershell[main](../../../powershell_scripts/data-factory/transform-data-using-spark/transform-data-using-spark.ps1 "Transform data using Spark")]

## <a name="clean-up-deployment"></a>Limpieza de la implementación

Después de ejecutar el script de ejemplo, puede usar el comando siguiente para quitar el grupo de recursos y todos los recursos asociados a él:

```powershell
Remove-AzResourceGroup -ResourceGroupName $resourceGroupName
```
Para eliminar la factoría de datos del grupo de recursos, ejecute el siguiente comando: 

```powershell
Remove-AzDataFactoryV2 -Name $dataFactoryName -ResourceGroupName $resourceGroupName
```

## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos:

| Get-Help | Notas |
|---|---|
| [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) | Crea un grupo de recursos en el que se almacenan todos los recursos. |
| [Set-AzDataFactoryV2](/powershell/module/az.datafactory/set-Azdatafactoryv2) | Creación de una factoría de datos. |
| [Set-AzDataFactoryV2LinkedService](/powershell/module/az.datafactory/set-Azdatafactoryv2linkedservice) | Crea un servicio vinculado en la factoría de datos. Un servicio vinculado enlaza un almacén de datos o proceso a una factoría de datos. |
| [Set-AzDataFactoryV2Pipeline](/powershell/module/az.datafactory/set-Azdatafactoryv2pipeline) | Crea una canalización en la factoría de datos. Una canalización contiene una o varias actividades que realizan una operación determinada. En esta canalización, una actividad de Spark transforma los datos mediante la ejecución de un programa en un clúster de Spark de Azure HDInsight. |
| [Invoke-AzDataFactoryV2Pipeline](/powershell/module/az.datafactory/invoke-Azdatafactoryv2pipeline) | Crea una ejecución para la canalización. En otras palabras, ejecuta la canalización. |
| [Get-AzDataFactoryV2ActivityRun](/powershell/module/az.datafactory/get-Azdatafactoryv2activityrun) | Obtiene información detallada sobre la ejecución de la actividad (actividad ejecutar) en la canalización. 
| [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) | Elimina un grupo de recursos, incluidos todos los recursos anidados. |
|||

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre Azure PowerShell, consulte la [documentación de Azure PowerShell](/powershell/).

Encontrará más ejemplos de scripts de PowerShell para Azure Data Factory en el artículo [Ejemplos de Azure PowerShell para Azure Data Factory](../samples-powershell.md).
