---
title: Administración de orígenes de datos en Azure Purview (versión preliminar)
description: Obtenga información sobre cómo registrar nuevos orígenes de datos, administrar colecciones de orígenes de datos y ver orígenes en Azure Purview (versión preliminar).
author: viseshag
ms.author: viseshag
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: how-to
ms.date: 11/25/2020
ms.openlocfilehash: b8eefda763765fd17fbf86bcf3205e854459a9f6
ms.sourcegitcommit: 3de22db010c5efa9e11cffd44a3715723c36696a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109656258"
---
# <a name="manage-data-sources-in-azure-purview-preview"></a>Administración de orígenes de datos en Azure Purview (versión preliminar)

En este artículo va a aprender a registrar nuevos orígenes de datos, a administrar colecciones de orígenes de datos y a ver orígenes en Azure Purview (versión preliminar).

## <a name="register-a-new-source"></a>Registro de un dispositivo nuevo

Para registrar un nuevo origen, siga estos pasos.

1. Abra Purview Studio y seleccione el icono de **Register sources** (Registrar orígenes).

   :::image type="content" source="media/manage-data-sources/purview-studio.png" alt-text="Azure Purview Studio":::

1. Seleccione **Register** (Registrar) y, a continuación, seleccione un tipo de origen. En este ejemplo se utiliza Azure Blob Storage. Seleccione **Continuar**.

   :::image type="content" source="media/manage-data-sources/select-source-type.png" alt-text="Selección de un tipo de origen de datos en la página Register sources (Registrar orígenes)":::

2. Rellene el formulario en la página **Register sources** (Registrar orígenes). Seleccione un nombre para el origen y escriba la información pertinente. Si eligió **From Azure subscription** (Desde la suscripción de Azure) como método de selección de cuenta, los orígenes de su suscripción aparecen en una lista desplegable. 

   :::image type="content" source="media/manage-data-sources/register-sources-form.png" alt-text="Formulario de información de orígenes de datos":::

3. Seleccione **Registrar**.

## <a name="view-sources"></a>Visualización de orígenes

Puede ver todos los orígenes registrados en la pestaña **Sources** (Orígenes) de Azure Purview Studio. Hay dos tipos de vistas: vista de mapa y vista de lista.

### <a name="map-view"></a>Vista del mapa

En la vista de mapa, puede ver todos los orígenes y colecciones. En la siguiente imagen, hay un origen de Azure Blob Storage. Desde cada icono de origen, puede editar el origen, iniciar un nuevo examen o ver los detalles del origen.

:::image type="content" source="media/manage-data-sources/map-view.png" alt-text="Vista de mapa de orígenes de datos de Azure Purview":::

### <a name="list-view"></a>Vista de lista

En la vista de lista, puede ver una lista de orígenes que se pueden ordenar. Mantenga el mouse sobre el origen para ver opciones de edición, inicio de un nuevo examen o eliminación.

:::image type="content" source="media/manage-data-sources/list-view.png" alt-text="Vista de lista de orígenes de datos de Azure Purview":::

## <a name="manage-collections"></a>Administración de recopilaciones

Puede agrupar los orígenes de datos en colecciones. Para crear una colección, seleccione **+ New collection** (+ Nueva colección) en la página *Sources* (Orígenes) de Azure Purview Studio. Asigne un nombre a la colección y seleccione *None* (Ninguna) como Parent (Primaria). La nueva colección aparece en la vista de mapa.

Para agregar orígenes a una colección, seleccione el lápiz **Edit** (Editar) en el origen y elija una colección en el menú desplegable **Select a collection** (Seleccionar una colección).

Para crear una jerarquía de colecciones, asigne colecciones de nivel superior como elemento primario a las colecciones de nivel inferior. En la imagen siguiente, *Fabrikam* es un elemento primario de la colección *Finance*, que contiene un origen de datos de Azure Blob Storage. Para contraer o expandir las colecciones, haga clic en el círculo asociado a la flecha entre niveles.

:::image type="content" source="media/manage-data-sources/collections.png" alt-text="Una jerarquía de colecciones en Azure Purview Studio":::

Puede quitar orígenes de una jerarquía seleccionando *None* (Ninguno) en el elemento primario. Los orígenes no primarios se agrupan en un cuadro punteado en la vista de mapa sin ninguna flecha que los vincule a elementos primarios.

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre cómo registrar y examinar varios orígenes de datos:

* [Azure Data Lake Storage Gen 2](register-scan-adls-gen2.md)
* [Inquilino de Power BI](register-scan-power-bi-tenant.md)
* [Azure SQL Database](register-scan-azure-sql-database.md)
