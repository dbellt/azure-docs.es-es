---
title: Creación de un entorno de ejecución de integración de Azure en Azure Data Factory
description: Aprenda a crear una instancia de Azure Integration Runtime en Azure Data Factory, que se usa para copiar datos y distribuir actividades de transformación.
ms.service: data-factory
ms.topic: conceptual
ms.date: 06/09/2020
author: lrtoyou1223
ms.author: lle
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: b98f95c0cd3013af055b85e4dfe1405b2eaf3032
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110681106"
---
# <a name="how-to-create-and-configure-azure-integration-runtime"></a>Cómo crear y configurar una instancia de Azure Integration Runtime
[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Integration Runtime (IR) es la infraestructura de proceso que usa Azure Data Factory para proporcionar capacidades de integración de datos en distintos entornos de red. Para obtener más información sobre IR, vea [Integration Runtime](concepts-integration-runtime.md).

Azure IR proporciona un proceso totalmente administrado para llevar a cabo movimientos de datos de forma nativa y trasladar actividades de transformación de datos para procesar servicios como HDInsight. Se hospeda en el entorno de Azure y es compatible con la conexión a recursos en el entorno de red pública con puntos de conexión accesibles públicos.

En este documento se muestra cómo crear y configurar Azure Integration Runtime. 

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="default-azure-ir"></a>Azure IR predeterminado
De forma predeterminada, cada factoría de datos tiene una instancia de Azure IR en el back-end que admite la ejecución de operaciones en almacenes de datos en la nube y servicios de proceso en la red pública. La ubicación de esa instancia de Azure IR se resuelve automáticamente. Si la propiedad **connectVia** no se especifica en la definición del servicio vinculado, se usa la instancia de Azure IR predeterminada. Basta con crear una instancia de Azure IR de forma explícita cuando quiera definir de forma explícita la ubicación del IR o cuando quiera agrupar de forma virtual las ejecuciones de actividades en distintos IR con fines de administración. 

## <a name="create-azure-ir"></a>Crear una instancia de Azure IR

Para crear y configurar una instancia de Azure IR, puede usar los procedimientos siguientes.

### <a name="create-an-azure-ir-via-azure-powershell"></a>Creación de una instancia de Azure IR mediante Azure PowerShell
Integration Runtime se puede crear con el cmdlet de PowerShell **Set-AzDataFactoryV2IntegrationRuntime**. Para crear una instancia de Azure IR, especifique el nombre, la ubicación y el tipo en el comando. A continuación se muestra un comando de ejemplo para crear una instancia de Azure IRcon la ubicación establecida en "Europa Occidental":

```powershell
Set-AzDataFactoryV2IntegrationRuntime -DataFactoryName "SampleV2DataFactory1" -Name "MySampleAzureIR" -ResourceGroupName "ADFV2SampleRG" -Type Managed -Location "West Europe"
```  
Para Azure IR, el tipo se debe establecer en **Administrado**. No es necesario especificar los detalles de proceso, porque están totalmente administrados de forma elástica en la nube. Especifique detalles de proceso, como el tamaño y el recuento de los nodos, cuando quiera crear una instancia de IR de Azure SSIS. Para más información, vea [Crear y configurar un IR de Azure SSIS](create-azure-ssis-integration-runtime.md).

Puede configurar una instancia de Azure IR existente para cambiar su ubicación mediante el cmdlet de PowerShell Set-AzDataFactoryV2IntegrationRuntime. Para más información sobre la ubicación de una instancia de Azure IR, vea [Introducción a Integration Runtime](concepts-integration-runtime.md).

### <a name="create-an-azure-ir-via-azure-data-factory-ui"></a>Creación de una instancia de Azure IR mediante la interfaz de usuario de Azure Data Factory
Use los pasos siguientes para crear una instancia de Azure IR mediante la interfaz de usuario de Azure Data Factory.

1. En la página **Comencemos** de la interfaz de usuario de Azure Data Factory, seleccione la [pestaña Administrar](./author-management-hub.md) en el panel izquierdo.

   ![Botón Administrar de la página principal](media/doc-common-process/get-started-page-manage-button.png)

1. Seleccione **Entornos de ejecución de integración** en el panel izquierdo y, a continuación, seleccione **+ Nuevo**.

   ![Captura de pantalla que resalta los entornos de ejecución de integración en el panel izquierdo y el botón +Nuevo.](media/doc-common-process/manage-new-integration-runtime.png)

1. En la página **Configuración de Integration Runtime**, seleccione **Azure, Self-Hosted** (Azure, autohospedado) y, luego, seleccione **Continuar**. 

1. En la página siguiente, seleccione **Azure** para crear una instancia de Azure IR y, luego, seleccione **Continuar**.
   ![Creación de un entorno de ejecución de integración](media/create-azure-integration-runtime/new-azure-integration-runtime.png)

1. Escriba un nombre para la instancia de Azure IR y seleccione **Crear**.
   ![Creación de una instancia de Azure IR](media/create-azure-integration-runtime/create-azure-integration-runtime.png)

1. Verá una notificación emergente cuando se complete la creación. En la página **Entornos de ejecución de integración**, asegúrese de que ve el entorno de ejecución de integración recién creado en la lista.

## <a name="use-azure-ir"></a>Usar Azure IR

Una vez creada la instancia de Azure IR, puede hacer referencia a ella en la definición del servicio vinculado. A continuación se muestra un ejemplo de cómo hacer referencia a la instancia de Azure Integration Runtime creada anteriormente a partir de un servicio vinculado de Azure Storage:

```json
{
    "name": "MyStorageLinkedService",
    "properties": {
      "type": "AzureStorage",
      "typeProperties": {
        "connectionString": "DefaultEndpointsProtocol=https;AccountName=myaccountname;AccountKey=..."
      },
      "connectVia": {
        "referenceName": "MySampleAzureIR",
        "type": "IntegrationRuntimeReference"
      }   
    }
}

```

## <a name="next-steps"></a>Pasos siguientes
Vea los siguientes artículos sobre cómo crear otros tipos de instancias de Integration Runtime:

- [Crear Integration Runtime autohospedado](create-self-hosted-integration-runtime.md)
- [Crear una instancia de Integration Runtime de Azure SSIS](create-azure-ssis-integration-runtime.md)
