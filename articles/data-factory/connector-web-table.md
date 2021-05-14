---
title: Copia de datos de tabla web mediante Azure Data Factory
description: Obtenga información sobre el conector de tabla web de Azure Data Factory que permite copiar datos desde una tabla web a almacenes de datos compatibles con Data Factory como receptores.
author: jianleishen
ms.service: data-factory
ms.topic: conceptual
ms.date: 08/01/2019
ms.author: jianleishen
ms.openlocfilehash: d3035220f719bb4afde4dec8ff8417114b6569ee
ms.sourcegitcommit: 1fbd591a67e6422edb6de8fc901ac7063172f49e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109484764"
---
# <a name="copy-data-from-web-table-by-using-azure-data-factory"></a>Copia de datos de una tabla web mediante Azure Data Factory
> [!div class="op_single_selector" title1="Seleccione la versión del servicio Data Factory que usa:"]
> * [Versión 1](v1/data-factory-web-table-connector.md)
> * [Versión actual](connector-web-table.md)
[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

En este artículo se explica el uso de la actividad de copia de Azure Data Factory para copiar datos desde una base de datos de tabla web. El documento se basa en el artículo de [introducción a la actividad de copia](copy-activity-overview.md) que describe información general de la actividad de copia.

Las diferencias entre este conector de tabla web, el [conector REST](connector-rest.md) y el [conector HTTP](connector-http.md) son:

- El **conector de tabla web** extrae contenido de la tabla de una página web HTML.
- El **conector REST** admite específicamente la copia de datos desde API RESTful.
- El **conector HTTP** es genérico y puede recuperar datos desde cualquier punto de conexión HTTP, por ejemplo, para descargar archivos. 

## <a name="supported-capabilities"></a>Funcionalidades admitidas

Este conector de tabla web es compatible con las actividades siguientes:

- [Actividad de copia](copy-activity-overview.md) con [matriz de origen o receptor compatible](copy-activity-overview.md)
- [Actividad de búsqueda](control-flow-lookup-activity.md)

Puede copiar datos desde la base de datos de la tabla web en cualquier almacén de datos receptor compatible. Consulte la tabla de [almacenes de datos compatibles](copy-activity-overview.md#supported-data-stores-and-formats) para ver una lista de almacenes de datos que la actividad de copia admite como orígenes o receptores.

En concreto, este conector de tabla web permite **extraer contenido de tablas de una página HTML**.

## <a name="prerequisites"></a>Prerrequisitos

Para usar este conector de tabla web, tiene que configurar una instancia de Integration Runtime autohospedada. Consulte el artículo sobre [Integration Runtime autohospedado](create-self-hosted-integration-runtime.md) para más información.

## <a name="getting-started"></a>Introducción

[!INCLUDE [data-factory-v2-connector-get-started](includes/data-factory-v2-connector-get-started.md)]

Las secciones siguientes proporcionan detalles sobre las propiedades que se usan para definir entidades de Data Factory específicas del conector de tabla web.

## <a name="linked-service-properties"></a>Propiedades del servicio vinculado

Las siguientes propiedades son compatibles con el servicio vinculado de tabla web:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type debe establecerse en: **Web** |Sí |
| url | Dirección URL para el origen de Web |Sí |
| authenticationType | El valor permitido es: **Anonymous**. |Sí |
| connectVia | El entorno [Integration Runtime](concepts-integration-runtime.md) que se usará para conectarse al almacén de datos. Tal y como se mencionó en los [requisitos previos](#prerequisites), se requiere un entorno Integration Runtime autohospedado. |Sí |

**Ejemplo**:

```json
{
    "name": "WebLinkedService",
    "properties": {
        "type": "Web",
        "typeProperties": {
            "url" : "https://en.wikipedia.org/wiki/",
            "authenticationType": "Anonymous"
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

## <a name="dataset-properties"></a>Propiedades del conjunto de datos

Si desea ver una lista completa de las secciones y propiedades disponibles para definir conjuntos de datos, consulte el artículo sobre [conjuntos de datos](concepts-datasets-linked-services.md). En esta sección se proporciona una lista de las propiedades que admite el conjunto de datos de tabla web.

Para copiar datos desde una tabla web, establezca la propiedad type del conjunto de datos en **WebTable**. Se admiten las siguientes propiedades:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type del conjunto de datos debe establecerse en: **WebTable** | Sí |
| path |Dirección URL relativa al recurso que contiene la tabla. |No. Cuando no se especifica la ruta de acceso, se solo se usa la dirección URL especificada en la definición de servicio vinculado. |
| índice |Índice de la tabla en el recurso. Consulte la sección [Obtención de índice de una tabla en una página HTML](#get-index-of-a-table-in-an-html-page) para saber los pasos necesarios para obtener el índice de una tabla en una página HTML. |Sí |

**Ejemplo**:

```json
{
    "name": "WebTableInput",
    "properties": {
        "type": "WebTable",
        "typeProperties": {
            "index": 1,
            "path": "AFI's_100_Years...100_Movies"
        },
        "schema": [],
        "linkedServiceName": {
            "referenceName": "<Web linked service name>",
            "type": "LinkedServiceReference"
        }
    }
}
```

## <a name="copy-activity-properties"></a>Propiedades de la actividad de copia

Si desea ver una lista completa de las secciones y propiedades disponibles para definir actividades, consulte el artículo sobre [canalizaciones](concepts-pipelines-activities.md). En esta sección se proporciona una lista de las propiedades que admite el origen de datos de tabla web.

### <a name="web-table-as-source"></a>Tabla web como origen

Para copiar datos desde una tabla web, establezca el tipo de origen de la actividad de copia en **WebSource**. No se admite ninguna otra propiedad.

**Ejemplo**:

```json
"activities":[
    {
        "name": "CopyFromWebTable",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Web table input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "WebSource"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

## <a name="get-index-of-a-table-in-an-html-page"></a>Obtención de índice de una tabla en una página HTML

Para obtener el índice de una tabla que necesita configurar en [propiedades del conjunto de datos ](#dataset-properties), puede utilizar, por ejemplo, Excel 2016 como herramienta, como se indica a continuación:

1. Inicie **Excel 2016** y cambie a la pestaña **Datos**.
2. Haga clic en **Nueva consulta** en la barra de herramientas, elija **De otros orígenes** y haga clic en **Desde Web**.

    ![Menú de Power Query](./media/copy-data-from-web-table/PowerQuery-Menu.png)
3. En el cuadro de diálogo **Desde Web**, escriba la **dirección URL** que usaría en el objeto JSON del servicio vinculado (por ejemplo: https://en.wikipedia.org/wiki/) ) junto con la ruta de acceso que especificaría para el conjunto de datos (por ejemplo: AFI%27s_100_Years...100_Movies) y haga clic en **Aceptar**.

    ![Cuadro de diálogo Desde Web](./media/copy-data-from-web-table/FromWeb-DialogBox.png)

    Dirección URL que se usa en este ejemplo: https://en.wikipedia.org/wiki/AFI%27s_100_Years...100_Movies
4. Si ve el cuadro de diálogo **Acceso a contenido web**, seleccione la **dirección URL** correcta, la **autenticación** y haga clic en **Conectar**.

   ![Cuadro de diálogo Acceso a contenido web](./media/copy-data-from-web-table/AccessWebContentDialog.png)
5. Haga clic en un elemento de **tabla** en la vista de árbol para ver el contenido de la tabla y después en el botón **Editar** ubicado en la parte inferior.  

   ![Cuadro de diálogo Navegador](./media/copy-data-from-web-table/Navigator-DialogBox.png)
6. En la ventana **Editor de consultas**, haga clic en el botón **Editor avanzado** de la barra de herramientas.

    ![Botón Editor avanzado](./media/copy-data-from-web-table/QueryEditor-AdvancedEditorButton.png)
7. En el cuadro de diálogo Editor avanzado, el número que aparece junto a "Origen" es el índice.

    ![Editor avanzado - Índice](./media/copy-data-from-web-table/AdvancedEditor-Index.png)

Si usa Excel 2013, use [Microsoft Power Query para Excel](https://www.microsoft.com/download/details.aspx?id=39379) para obtener el índice. Consulte el artículo [Conectarse a una página web](https://support.office.com/article/Connect-to-a-web-page-Power-Query-b2725d67-c9e8-43e6-a590-c0a175bd64d8) para más información. Los pasos son similares si usa [Microsoft Power BI Desktop](https://powerbi.microsoft.com/desktop/).


## <a name="lookup-activity-properties"></a>Propiedades de la actividad de búsqueda

Para obtener información detallada sobre las propiedades, consulte [Actividad de búsqueda](control-flow-lookup-activity.md).

## <a name="next-steps"></a>Pasos siguientes
Consulte los [almacenes de datos compatibles](copy-activity-overview.md#supported-data-stores-and-formats) para ver la lista de almacenes de datos que la actividad de copia de Azure Data Factory admite como orígenes y receptores.
