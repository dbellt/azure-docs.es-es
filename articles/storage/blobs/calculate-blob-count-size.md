---
title: Cálculo del recuento de blobs y su tamaño mediante el inventario de Azure Storage
description: Obtenga información sobre cómo calcular el recuento de blobs y su tamaño total por contenedor.
services: storage
author: normesta
ms.author: normesta
ms.date: 03/10/2021
ms.service: storage
ms.subservice: blobs
ms.topic: how-to
ms.openlocfilehash: 8365c4873165b5a8040bc3def5743eca435cb7e9
ms.sourcegitcommit: 1b698fb8ceb46e75c2ef9ef8fece697852c0356c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110653943"
---
# <a name="calculate-blob-count-and-total-size-per-container-using-azure-storage-inventory"></a>Cálculo del recuento de blobs y su tamaño total por contenedor mediante el inventario de Azure Storage

En este artículo se usa la característica de inventario de Azure Blob Storage y Azure Synapse para calcular el recuento de blobs y su tamaño total por contenedor. Estos valores son útiles al optimizar el uso de blobs por contenedor.

Los metadatos de blob no se incluyen en este método. La característica de inventario de Azure Blob Storage usa la API de REST para [List Blobs](/rest/api/storageservices/list-blobs) con parámetros predeterminados. Así pues, el ejemplo no admite instantáneas, contenedores '$', etc.

> [!IMPORTANT]
> Inventario de blobs está actualmente en **VERSIÓN PRELIMINAR**. Consulte [Términos de uso complementarios para las versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) para conocer los términos legales que se aplican a las características de Azure que se encuentran en la versión beta, en versión preliminar o que todavía no se han publicado para que estén disponibles con carácter general.

## <a name="enable-inventory-reports"></a>Habilitación de informes de inventario

El primer paso de este método es [habilitar informes de inventario](blob-inventory.md#enable-inventory-reports) en la cuenta de almacenamiento. Es posible que tenga que esperar hasta 24 horas después de habilitar los informes de inventario para que se genere el primer informe.

Cuando tenga un informe de inventario para analizarlo, conceda usted mismo acceso de lectura de blobs al contenedor en el que reside el archivo CSV del informe.

1. Vaya al contenedor con el archivo de informe CSV de inventario.
1. Seleccione **Control de acceso (IAM)** y, luego, **Agregar asignación de roles**.

    :::image type="content" source="media/calculate-blob-count-size/access.png" alt-text="Seleccione Agregar asignación de roles":::

1. Seleccione **Lector de datos de blobs de almacenamiento** en la lista desplegable de **Rol**.

    :::image type="content" source="media/calculate-blob-count-size/add-role-assignment.png" alt-text="Adición del rol Lector de datos de blobs de almacenamiento en la lista desplegable":::

1. Escriba la dirección de correo electrónico de la cuenta que usa para ejecutar el informe en el campo **Seleccionar**.

## <a name="create-an-azure-synapse-workspace"></a>Creación de un área de trabajo de Azure Synapse

A continuación, [cree un área de trabajo de Azure Synapse](../../synapse-analytics/get-started-create-workspace.md) donde ejecute una consulta SQL para informar de los resultados del inventario.

## <a name="create-the-sql-query"></a>Creación de la consulta SQL

Después de crear el área de trabajo de Azure Synapse, siga estos pasos.

1. Vaya a [https://web.azuresynapse.net](https://web.azuresynapse.net).
1. Seleccione la pestaña **Desarrollar** del borde izquierdo.
1. Seleccione el signo más (+) grande para agregar un elemento.
1. Seleccione el **script SQL**.

    :::image type="content" source="media/calculate-blob-count-size/synapse-sql-script.png" alt-text="Selección del script SQL para crear una nueva consulta":::

## <a name="run-the-sql-query"></a>Ejecución de la consulta SQL

1. Agregue la siguiente consulta SQL en el área de trabajo de Azure Synapse para [leer el archivo CSV de inventario](../../synapse-analytics/sql/query-single-csv-file.md#read-a-csv-file).

    En el parámetro `bulk`, utilice la dirección URL del archivo CSV de informe de inventario que desea analizar.

    ```sql
    SELECT LEFT([Name], CHARINDEX('/', [Name]) - 1) AS Container, 
            COUNT(*) As TotalBlobCount,
            SUM([Content-Length]) As TotalBlobSize
    FROM OPENROWSET(
        bulk '<URL to your inventory CSV file>',
        format='csv', parser_version='2.0', header_row=true
    ) AS Source
    GROUP BY LEFT([Name], CHARINDEX('/', [Name]) - 1)
    ```

1. Asigne un nombre a la consulta SQL en el panel Propiedades de la derecha.

1. Publique la consulta SQL presionando CTRL+S o seleccionando el botón **Publicar todo**.

1. Seleccione el botón **Ejecutar** para ejecutar la consulta SQL. En el panel **Resultados** se informa del recuento de blobs y su tamaño total por contenedor.

    :::image type="content" source="media/calculate-blob-count-size/output.png" alt-text="Resultado de la ejecución del script para calcular el recuento de blobs y su tamaño total.":::

## <a name="next-steps"></a>Pasos siguientes

- [Uso del inventario de blobs de Azure Storage para administrar datos de blobs (versión preliminar)](blob-inventory.md)
- [Cálculo del tamaño total de facturación de un contenedor de blobs](../scripts/storage-blobs-container-calculate-billing-size-powershell.md)