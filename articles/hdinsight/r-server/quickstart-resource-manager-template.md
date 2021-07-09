---
title: 'Inicio rápido: Creación de un clúster de ML Services mediante una plantilla: Azure HDInsight'
description: Esta guía de inicio rápido muestra cómo usar la plantilla de Resource Manager para crear un clúster de ML Services en Azure HDInsight.
ms.service: hdinsight
ms.topic: quickstart
ms.custom: subject-armqs
ms.date: 03/13/2020
ROBOTS: NOINDEX
ms.openlocfilehash: 86d43a09e4c739cc588848429dcaf6276f68dbf2
ms.sourcegitcommit: 91fdedcb190c0753180be8dc7db4b1d6da9854a1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/17/2021
ms.locfileid: "112299400"
---
# <a name="quickstart-create-ml-services-cluster-in-azure-hdinsight-using-arm-template"></a>Inicio rápido: Creación de un clúster de ML Services en Azure HDInsight mediante una plantilla de Resource Manager

[!INCLUDE [retirement banner](../includes/ml-services-retirement.md)]

En este inicio rápido, se usa una plantilla de Azure Resource Manager para crear un clúster de [ML Services](./r-server-overview.md) en Azure HDInsight. Microsoft Machine Learning Server está disponible como opción de implementación al crear clústeres de HDInsight en Azure. El tipo de clúster que proporciona esta opción se denomina ML Services. Esta funcionalidad proporciona a los científicos de datos, a los estadísticos y a los programadores de R acceso a petición a métodos escalables y distribuidos para realizar análisis en HDInsight.

[!INCLUDE [About Azure Resource Manager](../../../includes/resource-manager-quickstart-introduction.md)]

Si su entorno cumple los requisitos previos y está familiarizado con el uso de plantillas de Resource Manager, seleccione el botón **Implementar en Azure**. La plantilla se abrirá en Azure Portal.

[:::image type="icon" source="../../media/template-deployments/deploy-to-azure.svg" alt-text="Implementación en Azure":::](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fquickstarts%2Fmicrosoft.hdinsight%2Fhdinsight-rserver%2Fazuredeploy.json)

## <a name="prerequisites"></a>Requisitos previos

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="review-the-template"></a>Revisión de la plantilla

La plantilla usada en este inicio rápido forma parte de las [plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/hdinsight-rserver/).

:::code language="json" source="~/quickstart-templates/quickstarts/microsoft.hdinsight/hdinsight-rserver/azuredeploy.json":::

En la plantilla se definen dos recursos de Azure:

* [Microsoft.Storage/storageAccounts](/azure/templates/microsoft.storage/storageaccounts): permite crear una cuenta de Azure Storage.
* [Microsoft.HDInsight/cluster](/azure/templates/microsoft.hdinsight/clusters): permite crear un clúster de HDInsight.

## <a name="deploy-the-template"></a>Implementación de la plantilla

1. Seleccione el botón **Implementar en Azure** siguiente para iniciar sesión en Azure y abrir la plantilla de ARM.

    [:::image type="icon" source="../../media/template-deployments/deploy-to-azure.svg" alt-text="Implementación en Azure":::](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fquickstarts%2Fmicrosoft.hdinsight%2Fhdinsight-rserver%2Fazuredeploy.json)

1. Escriba o seleccione los siguientes valores:

    |Propiedad |Descripción |
    |---|---|
    |Subscription|En la lista desplegable, seleccione la suscripción de Azure que se usa para el clúster.|
    |Resource group|En la lista desplegable, seleccione el grupo de recursos existente o seleccione **Crear**.|
    |Location|El valor se rellenará automáticamente con la ubicación usada para el grupo de recursos.|
    |Cluster Name|Escriba un nombre único global. Use solo letras minúsculas y números para esta plantilla.|
    |Cluster Login User Name (Nombre de usuario de inicio de sesión del clúster)|Proporcione el nombre de usuario; el valor predeterminado es **admin**.|
    |Cluster Login Password (Contraseña de inicio de sesión del clúster)|Indique una contraseña. La contraseña debe tener un mínimo de 10 caracteres y contener al menos un dígito, una letra mayúscula y una letra minúscula, y un carácter no alfanumérico (excepto los caracteres ' " y `). |
    |Nombre de usuario de SSH|Proporcione el nombre de usuario; el valor predeterminado es sshuser.|
    |Contraseña de SSH|Proporcione la contraseña.|

    :::image type="content" source="./media/quickstart-resource-manager-template/resource-manager-template-rserver.png" alt-text="Implementación de HBase de plantilla de Resource Manager" border="true":::

1. Revise los **TÉRMINOS Y CONDICIONES**. A continuación, seleccione **Acepto los términos y condiciones indicados anteriormente** y, después, seleccione **Comprar**. Recibirá una notificación de que la implementación está en curso. Se tarda aproximadamente 20 minutos en crear un clúster.

## <a name="review-deployed-resources"></a>Revisión de los recursos implementados

Una vez que se cree el clúster, recibirá una notificación de **Implementación correcta** con el vínculo **Ir al recurso**. La página Grupo de recursos mostrará el nuevo clúster de HDInsight y el almacenamiento predeterminado asociado con el clúster. Todos los clústeres tienen una cuenta de [Azure Blob Storage](../hdinsight-hadoop-use-blob-storage.md), [Azure Data Lake Storage Gen1](../hdinsight-hadoop-use-data-lake-storage-gen1.md) o una dependencia de [`Azure Data Lake Storage Gen2`](../hdinsight-hadoop-use-data-lake-storage-gen2.md). Se conoce como cuenta de almacenamiento predeterminada. El clúster de HDInsight y su cuenta de almacenamiento predeterminada deben estar en la misma región de Azure. Al eliminar los clústeres no se elimina la cuenta de almacenamiento.

## <a name="clean-up-resources"></a>Limpieza de recursos

Después de completar el inicio rápido, puede ser conveniente eliminar el clúster. Con HDInsight, los datos se almacenan en Azure Storage, por lo que puede eliminar un clúster de forma segura cuando no se esté usando. Los clústeres de HDInsight se cobran aunque no se estén usando. Como en muchas ocasiones los cargos por el clúster son mucho más elevados que los cargos por el almacenamiento, desde el punto de vista económico tiene sentido eliminar clústeres cuando no se usen.

En Azure Portal, vaya al clúster y seleccione **Eliminar**.

[Eliminar HBase de plantilla de Resource Manager](./media/quickstart-resource-manager-template/azure-portal-delete-rserver.png)

También puede seleccionar el nombre del grupo de recursos para abrir la página del grupo de recursos y, a continuación, seleccionar **Eliminar grupo de recursos**. Al eliminar el grupo de recursos, se eliminan tanto el clúster de HDInsight como la cuenta de almacenamiento predeterminada.

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha aprendido a crear un clúster de ML Services en HDInsight con una plantilla de Resource Manager. En el siguiente artículo, aprenderá a ejecutar un script de R con RStudio Server que muestra el uso de Spark para cálculos de R distribuidos.

> [!div class="nextstepaction"]
> [Ejecución de un script de R en un clúster de Machine Learning Services en Azure HDInsight con RStudio Server](./machine-learning-services-quickstart-job-rstudio.md)
