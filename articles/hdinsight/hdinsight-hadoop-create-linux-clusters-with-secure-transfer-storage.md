---
title: 'Apache Hadoop y el almacenamiento de transferencia segura: Azure HDInsight'
description: Aprenda a crear clústeres de HDInsight con cuentas de almacenamiento de Azure habilitadas para transferencia segura.
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 02/18/2020
ms.openlocfilehash: 22804015ebf0344c00e60c88f780fe22ba440b52
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107774996"
---
# <a name="apache-hadoop-clusters-with-secure-transfer-storage-accounts-in-azure-hdinsight"></a>Clústeres de Apache Hadoop con cuentas de almacenamiento de transferencia segura en Azure HDInsight

La característica [Se requiere transferencia segura](../storage/common/storage-require-secure-transfer.md) mejora la seguridad de su cuenta de Azure Storage al aplicar todas las solicitudes a su cuenta mediante una conexión segura. Esta característica y el esquema wasbs solo son compatibles con la versión de clúster de HDInsight 3.6 o posterior.

> [!IMPORTANT]
> Habilitar la transferencia segura del almacenamiento después de crear un clúster puede producir errores al usar la cuenta de almacenamiento, por lo que no se recomienda. Es mejor crear un clúster mediante una cuenta de almacenamiento que ya tenga habilitada la transferencia segura.

## <a name="storage-accounts"></a>Cuentas de almacenamiento

### <a name="azure-portal"></a>Azure Portal

De forma predeterminada, la propiedad de transferencia segura requerida se habilita cuando se crea una cuenta de almacenamiento en Azure Portal.

Para actualizar una cuenta de almacenamiento existente con Azure Portal, consulte [Requisito de transferencia segura con Azure Portal](../storage/common/storage-require-secure-transfer.md#require-secure-transfer-for-an-existing-storage-account).

### <a name="powershell"></a>PowerShell

En el caso del cmdlet de PowerShell [New-AzStorageAccount](/powershell/module/az.storage/new-azstorageaccount), asegúrese de que el parámetro `-EnableHttpsTrafficOnly` está establecido en `1`.

Para actualizar una cuenta de almacenamiento existente con PowerShell, consulte [Requisito de transferencia segura con PowerShell](../storage/common/storage-require-secure-transfer.md#require-secure-transfer-with-powershell).

### <a name="azure-cli"></a>Azure CLI

Para el comando [az storage account create](/cli/azure/storage/account#az_storage_account_create) de la CLI de Azure, asegúrese de que el parámetro `--https-only` está establecido en `true`.

Para actualizar una cuenta de almacenamiento existente con la CLI de Azure, consulte [Requisito de transferencia segura con la CLI de Azure](../storage/common/storage-require-secure-transfer.md#require-secure-transfer-with-azure-cli).

## <a name="add-additional-storage-accounts"></a>Adición de cuentas de almacenamiento adicionales

Existen varias opciones para agregar cuentas de almacenamiento adicionales habilitadas para transferencia segura:

* Modificar la plantilla de Azure Resource Manager de la última sección.
* Crear un clúster mediante [Azure Portal](https://portal.azure.com) y especificar la cuenta de almacenamiento vinculada.
* Use la acción de script para agregar cuentas de almacenamiento adicionales habilitadas para transferencia segura a un clúster de HDInsight existente. Para más información, consulte [Adición de más cuentas de almacenamiento a HDInsight](hdinsight-hadoop-add-storage.md).

## <a name="next-steps"></a>Pasos siguientes

* El uso de Azure Storage (WASB) en lugar de [Apache Hadoop HDFS](https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/HdfsUserGuide.html) como el almacén de datos predeterminado
* Para más información sobre cómo HDInsight usa Azure Storage, consulte [Uso de Azure Storage con HDInsight](hdinsight-hadoop-use-blob-storage.md).
* Para más información sobre cómo cargar datos en HDInsight, consulte [Carga de datos en HDInsight](hdinsight-upload-data.md).