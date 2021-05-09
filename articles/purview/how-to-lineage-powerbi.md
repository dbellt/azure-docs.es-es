---
title: Metadatos y linaje de Power BI
description: En este artículo se describe la extracción de linaje de datos del origen Power BI.
author: chanuengg
ms.author: csugunan
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: how-to
ms.date: 03/30/2021
ms.openlocfilehash: b7143f7ccb0cb4c91cbb86cadb3cfa78c5c9bec3
ms.sourcegitcommit: b4032c9266effb0bf7eb87379f011c36d7340c2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107912599"
---
# <a name="how-to-get-lineage-from-power-bi-into-azure-purview"></a>Obtención de linaje de Power BI a Azure Purview

En este artículo se detallan los aspectos del linaje de datos de Power BI origen en Azure Purview. El requisito previo para ver el linaje de datos en Purview para Power BI es [examinar su Power BI.](../purview/register-scan-power-bi-tenant.md) 

## <a name="common-scenarios"></a>Escenarios frecuentes

1. Una vez examinado el origen de Power BI, los consumidores de datos pueden realizar el análisis de la causa principal de un informe o panel desde Purview. Para cualquier discrepancia de datos en un informe, los usuarios pueden identificar fácilmente los conjuntos de datos ascendentes y ponerse en contacto con sus propietarios si es necesario.

2. Los productores de datos pueden ver los informes o paneles de bajada que consumen su conjunto de datos. Antes de realizar cambios en sus conjuntos de datos, los propietarios de los datos pueden tomar decisiones informadas.

2. Los usuarios pueden buscar por nombre, estado de aprobación, etiqueta de confidencialidad, propietario, descripción y otras facetas empresariales para devolver los artefactos de Power BI pertinentes.

## <a name="power-bi-artifacts-in-azure-purview"></a>Artefactos de Power BI en Azure Purview

Una vez completado el [examen de Power BI](../purview/register-scan-power-bi-tenant.md), se inventariarán los artefactos de Power BI en Purview.

* Capacity
* Áreas de trabajo
* Flujo de datos
* Dataset 
* Informe
* Panel

Los artefactos del área de trabajo mostrarán el linaje de Flujo de datos -> Conjunto de datos -> Informe -> Panel

:::image type="content" source="./media/how-to-lineage-powerbi/powerbi-overview.png" alt-text="Captura de pantalla que muestra cómo se representa la pestaña Información general para recursos de Power BI." lightbox="./media/how-to-lineage-powerbi/powerbi-overview.png":::

>[!Note]
> * El linaje de columnas y las transformaciones dentro de los conjuntos de datos de Power BI no se admiten actualmente
> * Actualmente se muestra información limitada para los orígenes de datos a partir de los cuales se crea el flujo de datos de Power BI o el conjunto de datos de Power BI. Por ejemplo, para el origen de SQL Server de un conjunto de datos de Power BI, solo se captura el nombre del servidor. 

## <a name="lineage-of-power-bi-artifacts-in-azure-purview"></a>Linaje de artefactos de Power BI en Azure Purview

Los usuarios pueden buscar el artefacto de Power BI artefacto por nombre, descripción u otros detalles para ver los resultados pertinentes. En la pestaña de información general y propiedades de los recursos se muestran los detalles básicos, como la descripción, la clasificación y otra información. En la pestaña de linaje, las relaciones de recursos se muestran con las dependencias ascendentes y descendentes.

:::image type="content" source="./media/how-to-lineage-powerbi/powerbi-lineage.png" alt-text="Captura de pantalla que muestra cómo se representa el linaje para Power BI." lightbox="./media/how-to-lineage-powerbi/powerbi-lineage.png":::

## <a name="next-steps"></a>Pasos siguientes

- [Obtener información sobre el linaje de datos en Azure Purview](catalog-lineage-user-guide.md)
- [Vinculación de Azure Data Factory para insertar linaje automatizado](how-to-link-azure-data-factory.md)
