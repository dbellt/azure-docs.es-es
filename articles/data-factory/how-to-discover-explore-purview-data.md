---
title: Detección y exploración de datos en ADF mediante Purview
description: Aprenda a detectar y explorar datos en Azure Data Factory mediante Purview
ms.service: data-factory
ms.topic: conceptual
author: linda33wj
ms.author: jingwang
ms.custom: seo-lt-2019
ms.date: 01/15/2021
ms.openlocfilehash: f37d2461a0e9c2db0cb3c7aa911dfb0ff5e049af
ms.sourcegitcommit: 1fbd591a67e6422edb6de8fc901ac7063172f49e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109486492"
---
# <a name="discover-and-explore-data-in-adf-using-purview"></a>Detección y exploración de datos en ADF mediante Purview

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

En este artículo, registrará una cuenta de Azure Purview en una instancia de Data Factory. Esa conexión le permite detectar recursos de Azure Purview e interactuar con ellos mediante las funcionalidades de ADF. 

Puede realizar las tareas siguientes en ADF: 
- Usar el cuadro de búsqueda de la parte superior para buscar recursos de Purview basados en palabras clave. 
- Comprender los datos en función de los metadatos, linaje y anotaciones. 
- Conectar esos datos a su factoría de datos con conjuntos de datos o servicios vinculados. 

## <a name="prerequisites"></a>Requisitos previos 
- [Cuenta de Azure Purview](../purview/create-catalog-portal.md) 
- [Data Factory](./quickstart-create-data-factory-portal.md) 
- [Conexión de una cuenta de Azure Purview en Data Factory](./connect-data-factory-to-azure-purview.md) 

## <a name="using-azure-purview-in-data-factory"></a>Uso de Azure Purview en Data Factory 

Para usar Azure Purview en Data Factory, es necesario acceder a la cuenta de Purview. Data Factory pasa su permiso de Purview. Como ejemplo, si tiene un rol de permiso de administrador provisional, podrá editar los metadatos examinados por Azure Purview. 

### <a name="data-discovery-search-datasets"></a>Detección de datos: conjuntos de datos de búsqueda 

Para detectar los datos que Azure Purview ha registrado y examinado, puede usar la barra de búsqueda de la parte superior central del portal de Data Factory. Asegúrese de seleccionar Azure Purview para buscar todos los datos de su organización. 

:::image type="content" source="./media/data-factory-purview/search-dataset.png" alt-text="Captura de pantalla de acciones que se realizan en conjuntos de datos.":::

### <a name="actions-that-you-can-perform-over-datasets-with-data-factory-resources"></a>Acciones que puede realizar en los conjuntos de datos con recursos de Data Factory 
Puede crear directamente un servicio vinculado, un conjunto de datos o un flujo de datos en los datos que busca mediante Azure Purview.

:::image type="content" source="./media/data-factory-purview/actions-over-purview-data.png" alt-text="Captura de pantalla que muestra cómo puede crear directamente un servicio vinculado, un conjunto de datos o un flujo de datos en los datos que busca mediante Azure Purview.":::

##  <a name="nextsteps"></a>Pasos siguientes 

- [Registro y análisis de recursos de Azure Data Factory en Azure Purview](../purview/register-scan-azure-synapse-analytics.md)
- [Búsqueda de datos en Data Catalog de Azure Purview](../purview/how-to-search-catalog.md).