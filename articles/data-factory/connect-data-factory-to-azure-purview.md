---
title: Conexión de Data Factory a Azure Purview
description: Información sobre la conexión de Data Factory a Azure Purview
ms.author: jingwang
author: linda33wj
ms.service: data-factory
ms.topic: conceptual
ms.custom:
- seo-lt-2019
- references_regions
ms.date: 12/3/2020
ms.openlocfilehash: fee7f240903f1a812b69fcc359225ea197c56293
ms.sourcegitcommit: 1fbd591a67e6422edb6de8fc901ac7063172f49e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109482424"
---
# <a name="connect-data-factory-to-azure-purview-preview"></a>Conexión de Data Factory a Azure Purview (versión preliminar)
[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

En este artículo se explica cómo conectar Data Factory a Azure Purview y cómo notificar el linaje de datos de las actividades de Azure Data Factory Copiar datos, Flujo de datos y Ejecutar paquete SSIS.


## <a name="connect-data-factory-to-azure-purview"></a>Conexión de Data Factory a Azure Purview
Azure Purview es un nuevo servicio en la nube para los usuarios de datos que administran de forma centralizada la gobernanza de datos en su patrimonio de datos distribuido en entornos locales y de nube. Puede conectar su instancia de Data Factory a Azure Purview. La conexión le permite aprovechar Azure Purview para capturar datos de linaje de Copiar, Flujo de datos y Ejecutar paquete SSIS. Tiene dos maneras de conectar Data Factory a Azure Purview:
### <a name="register-azure-purview-account-to-data-factory"></a>Registro de la cuenta de Azure Purview en Data Factory
1. En el portal de ADF, vaya a **Administrar** -> **Azure Purview**. Seleccione **Connect to a Purview account** (Conectarse a una cuenta de Purview). 

:::image type="content" source="./media/data-factory-purview/register-purview-account.png" alt-text="Captura de pantalla para registrar una cuenta de Purview.":::
2. Puede elegir **A partir de una suscripción de Azure** o **Especificar manualmente**. En **A partir de una suscripción de Azure**, puede seleccionar la cuenta a la que tiene acceso. 
3. Una vez conectado, debería poder ver el nombre de la cuenta de Purview en la pestaña **Cuenta de Purview**. 
4. Puede usar la barra de búsqueda de la parte superior central del portal de Azure Data Factory para buscar datos. 

Si ve una advertencia en el portal de Azure Data Factory después de registrar la cuenta de Azure Purview en Data Factory, siga los pasos que se indican a continuación para corregir el problema:

:::image type="content" source="./media/data-factory-purview/register-purview-account-warning.png" alt-text="Captura de pantalla de la advertencia de registro de una cuenta de Purview.":::

1. Vaya a Azure Portal y busque su instancia de Data Factory. Elija la sección "Etiquetas" y compruebe si hay una etiqueta denominada **catalogUri**. Si no es así, desconéctese y vuelva a conectar la cuenta de Azure Purview en el portal de ADF.

:::image type="content" source="./media/data-factory-purview/register-purview-account-tag.png" alt-text="Captura de pantalla para etiquetas de registro de una cuenta de Purview.":::

2. Compruebe si se ha concedido el permiso para registrar una cuenta de Azure Purview en Data Factory. Consulte [Cómo conectar Azure Data Factory y Azure Purview](../purview/how-to-link-azure-data-factory.md#create-new-data-factory-connection).

### <a name="register-data-factory-in-azure-purview"></a>Registro de Data Factory en Azure Purview
Para obtener información sobre cómo registrar Data Factory en Azure Purview, consulte [Cómo conectar Azure Data Factory y Azure Purview](../purview/how-to-link-azure-data-factory.md). 

## <a name="report-lineage-data-to-azure-purview"></a>Notificación de datos de linaje a Azure Purview
Cuando los clientes ejecutan las actividades Copiar, Flujo de datos o Ejecutar paquetes SSIS en Azure Data Factory, pueden obtener la relación de dependencia y tener una introducción de alto nivel del proceso de flujo de trabajo completo entre los orígenes de datos y el destino.
Para obtener información sobre cómo recopilar el linaje de Azure Data Factory, consulte el artículo sobre el [linaje de Data Factory](../purview/how-to-link-azure-data-factory.md#supported-azure-data-factory-activities).

## <a name="next-steps"></a>Pasos siguientes
[Guía de usuario del linaje de Data Catalog](../purview/catalog-lineage-user-guide.md)

[Tutorial: Inserción de datos de linaje de Data Factory en Azure Purview](turorial-push-lineage-to-purview.md)