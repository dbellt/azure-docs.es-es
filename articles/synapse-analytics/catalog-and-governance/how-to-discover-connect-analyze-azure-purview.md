---
title: Detección, conexión y exploración de datos en Synapse con Azure Purview
description: Guía sobre cómo detectar datos, conectarlos y explorarlos en Synapse
author: Rodrigossz
ms.service: synapse-analytics
ms.subservice: purview
ms.topic: how-to
ms.date: 12/16/2020
ms.author: rosouz
ms.reviewer: jrasnick
ms.openlocfilehash: 1f326c1caaa019222854741dcc5d8a8e087885f7
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108138236"
---
# <a name="discover-connect-and-explore-data-in-synapse-using-azure-purview"></a>Detección, conexión y exploración de datos en Synapse con Azure Purview 

En este documento, conocerá el tipo de interacciones que puede realizar al registrar una cuenta de Azure Purview en Synapse. 

## <a name="prerequisites"></a>Requisitos previos 

- [Cuenta de Azure Purview](../../purview/create-catalog-portal.md) 
- [Área de trabajo de Synapse](../quickstart-create-workspace.md) 
- [Conexión de una cuenta de Azure Purview en Synapse](quickstart-connect-azure-purview.md) 

## <a name="using-azure-purview-in-synapse"></a>Uso de Azure Purview en Synapse 

Para usar Azure Purview en Synapse, es necesario acceder a la cuenta de Purview. Synapse pasa su permiso de Purview. Como ejemplo, si tiene un rol de permiso de administrador provisional, podrá editar los metadatos examinados por Azure Purview. 

### <a name="data-discovery-search-datasets"></a>Detección de datos: conjuntos de datos de búsqueda 

Para detectar los datos que Azure Purview ha registrado y examinado, puede usar la barra de búsqueda de la parte superior central del área de trabajo de Synapse. Asegúrese de seleccionar Azure Purview para buscar todos los datos de su organización. 

[![Búsqueda de recursos de Azure Purview](./media/purview-access.png)](./media/purview-access.png#lightbox)

## <a name="azure-purview-actions"></a>Acciones de Azure Purview 

Esta es una lista de las características de Azure Purview que están disponibles en Synapse: 
- **Información general** de los metadatos. 
- Vista y edición de **esquemas** de los metadatos con clasificaciones, términos de glosario, tipos de datos y descripciones. 
- Vista de **linajes** para comprender las dependencias y realizar análisis de los efectos. Para más información, consulte [Linajes](../../purview/catalog-lineage-user-guide.md).
- Vista y edición de **contactos** para saber quién es propietario o experto en un conjunto de datos. 
- **Relacionadas** para comprender las dependencias jerárquicas de un conjunto de datos concreto. Esta experiencia es útil para examinar la jerarquía de datos.

## <a name="actions-that-you-can-perform-over-datasets-with-synapse-resources"></a>Acciones que puede realizar en los conjuntos de datos con recursos de Synapse 

### <a name="connect-data-to-synapse"></a>Conexión de los datos a Synapse 

- Puede crear un **servicio vinculado** a Synapse. Esa acción será necesaria para copiar datos en Synapse o tenerlos en el centro de datos (en el caso de orígenes de datos admitidos, como ADLSg2). 
- En el caso de objetos, como archivos, carpetas o tablas, puede crear directamente un **conjunto de datos de integración** y aprovechar un servicio vinculado existente, si ya se ha creado. 

Todavía no se puede deducir si hay un servicio vinculado existente o un conjunto de datos de integración. 

###  <a name="develop-in-synapse"></a>Desarrollo en Synapse 

Hay tres acciones que puede realizar: **New SQL Script** (Nuevo script de SQL), **Nuevo cuaderno** y **New Data Flow** (Nuevo flujo de datos). 

Con **New SQL Script** (Nuevo script de SQL), en función del tipo de compatibilidad, puede hacer lo siguiente: 
- Ver las 100 primeras filas para comprender la forma de los datos. 
- Crear una tabla externa a partir de una base de datos de Synapse SQL. 
- Cargar los datos en una base de datos de Synapse SQL. 
 
Con **Nuevo cuaderno**, puede hacer lo siguiente: 
- Cargar datos en una instancia de DataFrame de Spark. 
- Crear una tabla de Spark (si lo hace con el formato Parquet, también se crea una tabla de grupo de SQL sin servidor). 
 
Con **New data flow** (Nuevo flujo de datos), puede crear un conjunto de datos de integración que se puede usar como origen en una canalización de flujo de datos. El flujo de datos es una funcionalidad de desarrollador sin código para realizar la transformación de datos. Más información sobre el [uso de flujos de datos en Synapse](../quickstart-data-flow.md).

##  <a name="nextsteps"></a>Pasos siguientes 

- [Registro y análisis de recursos de Azure Synapse en Azure Purview](../../purview/register-scan-azure-synapse-analytics.md)
- [Búsqueda de datos en Data Catalog de Azure Purview](../../purview/how-to-search-catalog.md).
