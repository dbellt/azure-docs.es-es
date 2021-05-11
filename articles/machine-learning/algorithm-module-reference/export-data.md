---
title: 'Exportación de datos: referencia para los módulos'
titleSuffix: Azure Machine Learning
description: Use el módulo de exportación de datos en el diseñador de Azure Machine Learning para guardar los resultados y los datos intermedios fuera de Azure Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 03/19/2021
ms.openlocfilehash: 82821b29669139f378d4dd24e4a96ab66f3d56e1
ms.sourcegitcommit: 52491b361b1cd51c4785c91e6f4acb2f3c76f0d5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108321940"
---
# <a name="export-data-module"></a>Módulo Exportación de datos

En este artículo se describe un módulo del diseñador de Azure Machine Learning.

Utilice este módulo para guardar los resultados, los datos intermedios y los datos de trabajo de las canalizaciones en los destinos de almacenamiento en la nube. 

Este módulo admite la exportación de los datos a los siguientes servicios de datos en la nube:

- Azure Blob Container
- Recurso compartido de archivos de Azure
- Azure Data Lake Storage Gen1
- Azure Data Lake Storage Gen2
- Azure SQL Database

Antes de exportar los datos, debe registrar un almacén de datos en el área de trabajo de Azure Machine Learning. Para más información, consulte [Acceso a los datos en los servicios de almacenamiento de Azure](../how-to-access-data.md).

## <a name="how-to-configure-export-data"></a>Procedimiento para configurar la exportación de datos

1. Agregue el módulo **Exportación de datos** a la canalización en el diseñador. Puede encontrar este módulo en la categoría **Entrada y salida**.

1. Conecte **Exportación de datos** al módulo que contiene los datos que desea exportar.

1. Seleccione **Exportación de datos** para abrir el panel **Propiedades**.

1. En **Almacén de datos**, seleccione un almacén de datos existente en la lista desplegable. También puede crear un nuevo almacén de datos. Para ver cómo, consulte [Acceso a los datos en los servicios de almacenamiento de Azure](../how-to-access-data.md).

    > [!NOTE]
    > No se admite la exportación de datos de un tipo determinado a una columna de SQL Database especificada como otro tipo de datos. No es necesario que la tabla de destino exista primero.

1. La casilla **Regenerate output** (Regenerar salida), decide si se debe ejecutar el módulo para regenerar la salida en tiempo de ejecución. 

    De forma predeterminada, no está seleccionada, lo que significa que si el módulo se ha ejecutado con los mismos parámetros anteriormente, el sistema reutilizará la salida de la última ejecución para reducir el tiempo de ejecución. 

    Si se selecciona, el sistema volverá a ejecutar el módulo para regenerar la salida.

1. Defina la ruta del almacén de datos en el que se encuentran los datos. La ruta de acceso es una ruta de acceso relativa. Tomemos `data/testoutput` como ejemplo, que significa que los datos introducidos en **Exportar datos** se exportarán a `data/testoutput` en el almacén de datos establecido en **Configuración de salida** del módulo.

    > [!NOTE]
    > No se permiten las rutas de acceso vacías ni de **dirección URL**.


1. Para **Formato de archivo**, seleccione el formato en el que se deben almacenar los datos.
 
1. Envíe la canalización.

## <a name="limitations"></a>Limitaciones

Debido a la limitación del acceso al almacén de datos, si la canalización de inferencia contiene el módulo **Exportar datos**, se quitará automáticamente cuando se implemente en el punto de conexión en tiempo real.

## <a name="next-steps"></a>Pasos siguientes

Consulte el [conjunto de módulos disponibles](module-reference.md) para Azure Machine Learning. 
