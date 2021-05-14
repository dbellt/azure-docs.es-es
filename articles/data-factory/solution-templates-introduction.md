---
title: Introducción a las plantillas
description: Aprenda a usar una plantilla predefinida para empezar a trabajar rápidamente con Azure Data Factory.
ms.service: data-factory
ms.topic: conceptual
ms.author: susabat
author: ssabat
ms.custom: seo-lt-2019
ms.date: 01/04/2019
ms.openlocfilehash: a95ef6e04f7a49f33f2bb0551110d7aa0f9bd581
ms.sourcegitcommit: b4032c9266effb0bf7eb87379f011c36d7340c2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107905619"
---
# <a name="templates"></a>Plantillas

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

Las plantillas son canalizaciones de Azure Data Factory predefinidas que le permiten empezar a trabajar rápidamente con Data Factory. Las plantillas son útiles cuando no está familiarizado con Data Factory y desea empezar a trabajar rápidamente. Estas plantillas reducen el tiempo de desarrollo para la creación de proyectos de integración de datos, con lo que mejoran la productividad del desarrollador.

## <a name="create-data-factory-pipelines-from-templates"></a>Creación de canalizaciones de Data Factory a partir de plantillas

Puede empezar a crear una canalización de Data Factory a partir de una plantilla de las siguientes dos maneras:

1.  Seleccione **Create pipeline from template** (Creación de una canalización a partir de una plantilla) en la página de información general para abrir la galería de plantillas.

    ![Abrir la galería de plantillas desde la página de información general](media/solution-templates-introduction/templates-intro-image1.png)

1.  En la pestaña de autor en el Explorador de recursos, seleccione **+** , a continuación, **Pipeline from template** (Canalización a partir de la plantilla) para abrir la galería de plantillas.

    ![Abrir la galería de plantillas desde la pestaña de autor](media/solution-templates-introduction/templates-intro-image2.png)

## <a name="template-gallery"></a>Galería de plantillas

![La galería de plantillas](media/solution-templates-introduction/templates-intro-image3.png)

### <a name="out-of-the-box-data-factory-templates"></a>Plantillas listas para usar de Data Factory

Data Factory usa plantillas de Azure Resource Manager para guardar las plantillas de canalización de las factorías de datos. Puede ver todas las plantillas de Resource Manager, junto con el archivo de manifiesto que se usa para las plantillas listas para usar de Data Factory, en el [repositorio de GitHub de Azure Data Factory oficial](https://github.com/Azure/Azure-DataFactory/tree/master/templates). Las plantillas predefinidas proporcionadas por Microsoft incluyen, pero no se limitan, a los siguientes elementos:

-   Plantillas de copia:

    -   [Copia masiva desde base de datos](solution-template-bulk-copy-with-control-table.md)
    
    -   [Copia de archivos nuevos por LastModifiedDate](solution-template-copy-new-files-lastmodifieddate.md)

    -   [Copiar varios contenedores de archivos entre almacenes basados en archivos](solution-template-copy-files-multiple-containers.md)

    -   [Traslado de archivos](solution-template-move-files.md)

    -   [Copia diferencial desde base de datos](solution-template-delta-copy-with-control-table.md)

    -   Copia de \<source\> a \<destination\>

        -   [De Amazon S3 a Azure Data Lake Store Gen 2](solution-template-migration-s3-azure.md)

        -   De Google Big Query a Azure Data Lake Store Gen 2

        -   De HDF a Azure Data Lake Store Gen 2

        -   De Netezza a Azure Data Lake Store Gen 1

        -   De SQL Server local a Azure SQL Database

        -   De SQL Server local a Azure Synapse Analytics

        -   De Oracle local a Azure Synapse Analytics

-   Plantillas de SSIS

    -   Programación de Azure-SSIS Integration Runtime para ejecutar paquetes SSIS

-   Plantillas de transformación

    -   [ETL con Azure Databricks](solution-template-databricks-notebook.md)

### <a name="my-templates"></a>Mis plantillas

También puede guardar una canalización como plantilla seleccionando **Guardar como plantilla** en la pestaña de la canalización.

![Guardar la canalización como plantilla](media/solution-templates-introduction/templates-intro-image4.png)

Puede ver las canalizaciones guardadas como plantillas en la sección **Mis plantillas** de la galería de plantillas. También puede verlas en la sección **Plantillas** del Explorador de recursos.

![Mis plantillas](media/solution-templates-introduction/templates-intro-image5.png)

> [!NOTE]
> Para usar la característica Mis plantillas, tendrá que habilitar la integración de GIT. Se admiten Azure DevOps GIT y GitHub.
