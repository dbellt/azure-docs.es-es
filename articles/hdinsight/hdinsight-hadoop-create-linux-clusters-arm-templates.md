---
title: 'Creación de clústeres de Apache Hadoop con plantillas: Azure HDInsight'
description: Aprenda a crear clústeres para HDInsight con plantillas de Resource Manager
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 04/07/2020
ms.openlocfilehash: 7d39163fa2890eb16a455d505baa0a6bbeddd42e
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111950634"
---
# <a name="create-apache-hadoop-clusters-in-hdinsight-by-using-resource-manager-templates"></a>Creación de clústeres de Apache Hadoop en HDInsight con plantillas de Resource Manager

[!INCLUDE [selector](../../includes/hdinsight-create-linux-cluster-selector.md)]

En este artículo aprenderá varias formas de crear clústeres de Azure HDInsight mediante [plantillas de Azure Resource Manager](../azure-resource-manager/templates/deploy-powershell.md). Para obtener información sobre otras herramientas y características de creación de clústeres, haga clic en el selector de pestaña de la parte superior de esta página. Consulte también, [Métodos de creación de clústeres](hdinsight-hadoop-provision-linux-clusters.md#cluster-setup-methods).

[!INCLUDE [delete-cluster-warning](../../includes/hdinsight-delete-cluster-warning.md)]

## <a name="resource-manager-templates"></a>Plantillas de Resource Manager

Una plantilla de Resource Manager facilita la creación de los siguientes recursos de la aplicación en una única operación coordinada:

* Clústeres de HDInsight y sus recursos dependientes (por ejemplo, la cuenta de almacenamiento predeterminada).
* Otros recursos (por ejemplo, Azure SQL Database para usar [Apache Sqoop](https://sqoop.apache.org/)).

En la plantilla, se definen los recursos que son necesarios para la aplicación. También se especifican los parámetros de implementación para introducir los valores para los diferentes entornos. La plantilla consta de JSON y expresiones que puede usar para generar valores para su implementación.

Puede encontrar plantillas de HDInsight de ejemplo en [Plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/?term=hdinsight). Utilice [Visual Studio Code](https://code.visualstudio.com/#alt-downloads) multiplataforma con la [extensión de Resource Manager](https://marketplace.visualstudio.com/items?itemName=msazurermtools.azurerm-vscode-tools) o un editor de texto para guardar la plantilla en un archivo en su estación de trabajo.

Para más información sobre la plantilla de Resource Manager, consulte los artículos y ejemplos siguientes:

* [Creación de plantillas del Administrador de recursos de Azure](../azure-resource-manager/templates/syntax.md)
* [Implementación de una aplicación con las plantillas de Azure Resource Manager](../azure-resource-manager/templates/deploy-powershell.md)
* Referencia de plantilla de [Microsoft.HDInsight/clusters](/azure/templates/microsoft.hdinsight/allversions)
* [Plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Hdinsight&pageNumber=1&sort=Popular)

## <a name="generate-templates"></a>Generación de plantillas

Resource Manager permite exportar una plantilla de Resource Manager a partir de los recursos existentes en una suscripción con el uso de distintas herramientas. Puede usar esa plantilla generada para aprender sobre la sintaxis de plantillas o para automatizar la nueva implementación de su solución según sea necesario. Para obtener más información, consulte [Exportar plantillas](../azure-resource-manager/templates/export-template-portal.md).

## <a name="deploy-using-the-portal"></a>Implementación mediante Azure Portal

Puede implementar una plantilla de Resource Manager mediante Azure Portal. Para más información, vea [Implementación de recursos desde plantilla personalizada](../azure-resource-manager/templates/deploy-portal.md#deploy-resources-from-custom-template).

## <a name="deploy-using-powershell"></a>Implementación mediante PowerShell

Puede implementar una plantilla de Resource Manager mediante Azure PowerShell. Para más información, vea [Implementación de recursos con las plantillas de Resource Manager y Azure PowerShell](../azure-resource-manager/templates/deploy-powershell.md) e [Implementar la plantilla de Resource Manager privada con el token de SAS y Azure PowerShell](../azure-resource-manager/templates/secure-template-with-sas-token.md).

## <a name="deploy-using-azure-cli"></a>Implementación con la CLI de Azure

Puede implementar una plantilla de Resource Manager mediante la CLI de Azure. Para más información, vea [Implementación de recursos con plantillas de Resource Manager y la CLI de Azure](../azure-resource-manager/templates/deploy-cli.md) e [Implementar la plantilla de Resource Manager privada con el token de SAS y la CLI de Azure](../azure-resource-manager/templates/secure-template-with-sas-token.md).

## <a name="deploy-using-the-rest-api"></a>Implementación mediante la API de REST

Puede implementar una plantilla de Resource Manager mediante la API de REST. Para más información, vea [Implementación de recursos con las plantillas de Resource Manager y la API de REST de Resource Manager](../azure-resource-manager/templates/deploy-rest.md).

## <a name="deploy-with-visual-studio"></a>Implementación con Visual Studio

 Use Visual Studio para crear un proyecto del grupo de recursos e implementarlo en Azure a través de la interfaz de usuario. Seleccione el tipo de recursos que va a incluir en su proyecto. Estos recursos se agregan automáticamente a la plantilla de Resource Manager. El proyecto también ofrece un script de PowerShell para implementar la plantilla.

Para ver una introducción sobre el uso de Visual Studio con grupos de recursos, consulte [Creación e implementación de grupos de recursos de Azure mediante Visual Studio](../azure-resource-manager/templates/create-visual-studio-deployment-project.md).

## <a name="troubleshoot"></a>Solución de problemas

Si experimenta problemas con la creación de clústeres de HDInsight, consulte los [requisitos de control de acceso](hdinsight-hadoop-customize-cluster-linux.md#access-control).

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha aprendido varias maneras de crear un clúster de HDInsight. Para más información, vea los siguientes artículos:

* Para ver plantillas relacionadas con HDInsight, consulte [Plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/?term=hdinsight).
* Para ver un ejemplo de cómo implementar los recursos mediante la biblioteca cliente de .NET, consulte [Deploy Azure resources using .NET libraries and a template](/previous-versions/azure/virtual-machines/windows/csharp-template?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)(Implementación de recursos de Azure mediante bibliotecas de .NET y una plantilla).
* Para ver un ejemplo en profundidad de la implementación de una aplicación, consulte [Aprovisionamiento e implementación predecibles de microservicios en Azure](../app-service/deploy-complex-application-predictably.md).
* Para obtener instrucciones sobre cómo implementar la solución en diferentes entornos, vea [Entornos de desarrollo y pruebas en Microsoft Azure](../devtest-labs/devtest-lab-overview.md).
* Para información sobre las secciones de la plantilla de Azure Resource Manager, consulte [Creación de plantillas de Azure Resource Manager](../azure-resource-manager/templates/syntax.md).
* Para ver una lista de las funciones que puede usar en una plantilla de Azure Resource Manager, consulte [Funciones de la plantilla de Azure Resource Manager](../azure-resource-manager/templates/template-functions.md).