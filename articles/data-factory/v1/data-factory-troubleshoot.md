---
title: Solución de problemas de Azure Data Factory
description: Obtenga información acerca de la solución de problemas relacionados con Azure Data Factory.
ms.service: data-factory
ms.topic: conceptual
ms.date: 01/10/2018
author: dcstwh
ms.author: weetok
ms.reviewer: jburchel
ms.custom: devx-track-azurepowershell
robots: noindex
ms.openlocfilehash: fbc21ea65e410c71641d81794cbfe2ff256e18af
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110674926"
---
# <a name="troubleshoot-data-factory-issues"></a>Solución de problemas de la factoría de datos
> [!NOTE]
> Este artículo se aplica a la versión 1 de Azure Data Factory. 

En este artículo se proporcionan consejos para solución de problemas surgidos al usar Azure Data Factory. Este artículo no incluye todos los posibles problemas que pueden aparecer al usar el servicio, pero trata algunos de ellos, así como consejos para solución de problemas generales.   

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="troubleshooting-tips"></a>Sugerencias de solución de problemas
### <a name="error-the-subscription-is-not-registered-to-use-namespace-microsoftdatafactory"></a>Error: la suscripción no está registrada para usar el espacio de nombres "Microsoft.DataFactory".
Si recibe este error, el proveedor de recursos de Azure Data Factory no se ha registrado en su equipo. Haga lo siguiente:

1. Inicie Azure PowerShell.
2. Inicie sesión en la cuenta de Azure mediante el siguiente comando.

   ```powershell
   Connect-AzAccount
   ```

3. Ejecute el siguiente comando para registrar el proveedor de Azure Data Factory.

   ```powershell
   Register-AzResourceProvider -ProviderNamespace Microsoft.DataFactory
   ```

### <a name="problem-unauthorized-error-when-running-a-data-factory-cmdlet"></a>Problema: error no autorizado al ejecutar un cmdlet de Data Factory.
Probablemente no está usando la cuenta o suscripción de Azure correctas con Azure PowerShell. Use los cmdlets siguientes para seleccionar la cuenta y la suscripción de Azure correctas que se usarán con Azure PowerShell.

1. Connect-AzAccount: use el id. de usuario y la contraseña correctos
2. Get-AzureRmSubscription: consulte todas las suscripciones de la cuenta.
3. Select-AzSubscription &lt;nombre de la suscripción&gt;: seleccione la suscripción correcta. Use la misma que usó para crear una factoría de datos en el Portal de Azure.

### <a name="problem-fail-to-launch-data-management-gateway-express-setup-from-azure-portal"></a>Problema: no se pudo iniciar la configuración rápida de la puerta de enlace de administración de datos desde Azure Portal.
La configuración rápida de Data Management Gateway requiere Internet Explorer o un explorador web compatible con Microsoft ClickOnce. Si no se puede iniciar la configuración rápida, realice una de las siguientes acciones:

* Utilice Internet Explorer o un explorador web compatible con Microsoft ClickOnce.

    Si usa Chrome, vaya a [Chrome Web Store](https://chrome.google.com/webstore/), busque la palabra clave "ClickOnce", elija una de las extensiones de ClickOnce e instálela.

    Debe hacer lo mismo para Firefox (instale un complemento). Haga clic en el botón Abrir menú de la barra de herramientas (tres líneas horizontales en la esquina superior derecha), haga clic en Complementos, busque la palabra clave "ClickOnce", elija una de las extensiones de ClickOnce e instálela.
* Haga clic en el vínculo **Instalación manual** que aparece en la misma hoja del portal. Utilice este enfoque para descargar el archivo de instalación y ejecutarlo manualmente. Después de realizarse correctamente la instalación, verá el cuadro de diálogo Configuración de Data Management Gateway. Copie la **clave** desde la pantalla del portal y úsela en el administrador de configuración para registrar manualmente la puerta de enlace con el servicio.  

### <a name="problem-fail-to-connect-to-sql-server"></a>Problema: No se puede conectar a SQL Server
Inicie **Administrador de configuración de Data Management Gateway** en el equipo de la puerta de enlace y use la pestaña **Solución de problemas** para probar la conexión a SQL Server desde el equipo de la puerta de enlace. Consulte [Solución de problemas de la puerta de enlace](data-factory-data-management-gateway.md#troubleshooting-gateway-issues) para obtener sugerencias para solucionar problemas de conexión o puerta de enlace.   

### <a name="problem-input-slices-are-in-waiting-state-forever"></a>Problema: Los segmentos de entrada están en el estado En espera de forma permanente
Los intervalos pueden tener el estado **En espera** por diversos motivos. Uno de los motivos comunes es que la propiedad **external** no está establecida en **True**. Cualquier conjunto de datos que se produce fuera del ámbito de Azure Data Factory debe marcarse con la propiedad **external** . Esto indica que los datos son externos y no están respaldados por ninguna canalización dentro de la factoría de datos. Los segmentos de datos se marcan con el estado **Listo** una vez que están disponibles en el almacén correspondiente.

Consulte el ejemplo siguiente para el uso de la propiedad **external** . Opcionalmente, puede especificar **externalData*** al establecer external en true.

Consulte el artículo [Conjuntos de datos](data-factory-create-datasets.md) para obtener más información sobre esta propiedad.

```json
{
  "name": "CustomerTable",
  "properties": {
    "type": "AzureBlob",
    "linkedServiceName": "MyLinkedService",
    "typeProperties": {
      "folderPath": "MyContainer/MySubFolder/",
      "format": {
        "type": "TextFormat",
        "columnDelimiter": ",",
        "rowDelimiter": ";"
      }
    },
    "external": true,
    "availability": {
      "frequency": "Hour",
      "interval": 1
    },
    "policy": {
      }
    }
  }
}
```

Para resolver el error, agregue la propiedad **external** y la sección **externalData** opcional a la definición de JSON de la tabla de entrada y vuelva a crear la tabla.

### <a name="problem-hybrid-copy-operation-fails"></a>Problema: la operación de copia híbrida produce un error.
Consulte [Solución de problemas de la puerta de enlace](data-factory-data-management-gateway.md#troubleshooting-gateway-issues) si quiere ver los pasos para solucionar problemas con la copia en un almacén de datos local como origen o destino usando Data Management Gateway.

### <a name="problem-on-demand-hdinsight-provisioning-fails"></a>Problema: el aprovisionamiento de HDInsight a petición produce un error.
Al usar un servicio vinculado de tipo HDInsightOnDemand, debe especificar linkedServiceName que apunte a Azure Blob Storage. El servicio Data Factory usa este almacenamiento para almacenar registros y archivos auxiliares para el clúster de HDInsight a petición.  A veces, el aprovisionamiento de un clúster de HDInsight a petición produce el siguiente error:

```
Failed to create cluster. Exception: Unable to complete the cluster create operation. Operation failed with code '400'. Cluster left behind state: 'Error'. Message: 'StorageAccountNotColocated'.
```

Este error suele indicar que la ubicación de la cuenta de almacenamiento especificada en linkedServiceName no está en la misma ubicación del centro de datos que donde se produce el aprovisionamiento de HDInsight. Ejemplo: si la factoría de datos está en el oeste de EE. UU. y el almacenamiento de Azure en el este de EE. UU., el aprovisionamiento a petición no se realizará correctamente en el oeste de EE. UU.

Además, hay una segunda propiedad de JSON additionalLinkedServiceNames, donde puede se especifiquen cuentas de almacenamiento adicionales en HDInsight a petición. Las cuentas de almacenamiento vinculado adicionales deben estar en la misma ubicación que el clúster de HDInsight; de lo contrario, se producirá el mismo error.

### <a name="problem-custom-net-activity-fails"></a>Problema: Se produce un error en la actividad personalizada de .NET
Consulte [Debug a pipeline with custom activity](data-factory-use-custom-activities.md#troubleshoot-failures) (Depurar una canalización con una actividad personalizada) para obtener pasos detallados.

## <a name="use-azure-portal-to-troubleshoot"></a>Uso del Portal de Azure para solucionar problemas
### <a name="using-portal-blades"></a>Uso de hojas del Portal
Consulte [Supervisión de la canalización](data-factory-monitor-manage-pipelines.md) para ver los pasos.

### <a name="using-monitor-and-manage-app"></a>Uso de la aplicación de supervisión y administración
Consulte [Monitor and manage data factory pipelines using Monitor and Manage App](data-factory-monitor-manage-app.md) (Supervisar y administrar canalizaciones de Data Factory con una aplicación de supervisión y administración) para obtener más información.

## <a name="use-azure-powershell-to-troubleshoot"></a>Usar Azure PowerShell para solucionar problemas
### <a name="use-azure-powershell-to-troubleshoot-an-error"></a>Usar Azure PowerShell para solucionar un error
Consulte [Monitor Data Factory pipelines using Azure PowerShell](data-factory-monitor-manage-pipelines.md) (Supervisar canalizaciones de Data Factory con Azure PowerShell) para obtener más información.

[adfgetstarted]: data-factory-copy-data-from-azure-blob-storage-to-sql-database.md
[use-custom-activities]: data-factory-use-custom-activities.md
[troubleshoot]: data-factory-troubleshoot.md
[developer-reference]: /previous-versions/azure/dn834987(v=azure.100)
[cmdlet-reference]: https://go.microsoft.com/fwlink/?LinkId=517456
[json-scripting-reference]: /previous-versions/azure/dn835050(v=azure.100)

[azure-portal]: https://portal.azure.com/

[image-data-factory-troubleshoot-with-error-link]: ./media/data-factory-troubleshoot/DataFactoryWithErrorLink.png

[image-data-factory-troubleshoot-datasets-with-errors-blade]: ./media/data-factory-troubleshoot/DatasetsWithErrorsBlade.png

[image-data-factory-troubleshoot-table-blade-with-problem-slices]: ./media/data-factory-troubleshoot/TableBladeWithProblemSlices.png

[image-data-factory-troubleshoot-activity-run-with-error]: ./media/data-factory-troubleshoot/ActivityRunDetailsWithError.png

[image-data-factory-troubleshoot-dataslice-blade-with-active-runs]: ./media/data-factory-troubleshoot/DataSliceBladeWithActivityRuns.png

[image-data-factory-troubleshoot-walkthrough2-with-errors-link]: ./media/data-factory-troubleshoot/Walkthrough2WithErrorsLink.png

[image-data-factory-troubleshoot-walkthrough2-datasets-with-errors]: ./media/data-factory-troubleshoot/Walkthrough2DataSetsWithErrors.png

[image-data-factory-troubleshoot-walkthrough2-table-with-problem-slices]: ./media/data-factory-troubleshoot/Walkthrough2TableProblemSlices.png

[image-data-factory-troubleshoot-walkthrough2-slice-activity-runs]: ./media/data-factory-troubleshoot/Walkthrough2DataSliceActivityRuns.png

[image-data-factory-troubleshoot-activity-run-details]: ./media/data-factory-troubleshoot/Walkthrough2ActivityRunDetails.png
