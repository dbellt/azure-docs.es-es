---
title: 'Inicio rápido: Introducción a la creación de un área de trabajo de Synapse'
description: En este tutorial, aprenderá a crear un área de trabajo de Synapse, un grupo de SQL dedicado y un grupo de Apache Spark sin servidor.
services: synapse-analytics
author: saveenr
ms.author: saveenr
manager: julieMSFT
ms.reviewer: jrasnick
ms.service: synapse-analytics
ms.subservice: workspace
ms.topic: tutorial
ms.date: 03/17/2021
ms.openlocfilehash: 4b7251be220c012ca51970863ac2eed55d46d711
ms.sourcegitcommit: 6686a3d8d8b7c8a582d6c40b60232a33798067be
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107751155"
---
# <a name="creating-a-synapse-workspace"></a>Creación de un área de trabajo de Synapse

En este tutorial, aprenderá a crear un área de trabajo de Synapse, un grupo de SQL dedicado y un grupo de Apache Spark sin servidor. 

## <a name="prerequisites"></a>Prerrequisitos

Para completar los pasos de este tutorial, es preciso tener acceso a un grupo de recursos para el que tenga asignado el rol de **propietario**. Cree el área de trabajo de Synapse en este grupo de recursos.

## <a name="create-a-synapse-workspace-in-the-azure-portal"></a>Creación de un área de trabajo de Synapse en Azure Portal

### <a name="start-the-process"></a>Inicio del proceso
1. Abra [Azure Portal](https://portal.azure.com) y escriba **Synapse** en la barra de búsqueda sin pulsar entrar.
1. En los resultados de la búsqueda, en **Servicios**, seleccione **Azure Synapse Analytics**.
1. Seleccione **Agregar** para crear un área de trabajo.

## <a name="basics-tab--project-details"></a>Pestaña Información básica > Detalles del proyecto
Rellene los campos siguientes:

1. **Suscripción**: seleccione cualquier suscripción.
1. **Grupo de recursos**: use cualquier grupo de recursos.
1. **Grupo de recursos administrados**: déjelo en blanco.

## <a name="basics-tab--workspace-details"></a>Pestaña Información básica > Detalles del área de trabajo
Rellene los campos siguientes:

1. **Nombre del área de trabajo**: seleccione cualquier nombre que sea globalmente único. Para este tutorial usaremos **myworkspace**.
1. **Región**: elija la región en la que ha colocado las aplicaciones y los servicios del cliente (por ejemplo, Azure Virtual Machines, Power BI, Azure Analysis Service) y los almacenamientos que contienen datos (por ejemplo, Azure Data Lake Storage, almacenamiento analítico de Azure Cosmos DB).

> [!NOTE]
> Un área de trabajo que no se encuentre en la misma ubicación que las aplicaciones cliente o el almacenamiento puede ser la causa principal de muchos problemas de rendimiento. Si los datos o los clientes están colocados en varias regiones, puede crear áreas de trabajo independientes en distintas regiones que se encuentran en la misma ubicación que los datos y los clientes.

En **Seleccionar Data Lake Storage Gen2**:

1. Junto a **Nombre de cuenta**, seleccione **Crear nuevo** y asigne a la nueva cuenta de almacenamiento el nombre **contosolake** o uno similar, ya que el nombre debe ser único.
1. Junto a **Nombre del sistema de archivos**, seleccione **Crear nuevo** y asigne el nombre **users**. Se creará un contenedor de almacenamiento llamado **users**. El área de trabajo usará esta cuenta de almacenamiento como cuenta de almacenamiento "principal" para las tablas de Spark y los registros de aplicaciones de Spark.
1. Active la casilla "Es necesario asignarme el rol Colaborador de datos de Storage Blob en la cuenta de Data Lake Storage Gen2". 

## <a name="completing-the-process"></a>Finalización del proceso
Seleccione **Revisar y crear** > **Crear**. El área de trabajo estará lista en unos minutos.

> [!NOTE]
> Para habilitar las características del área de trabajo de un grupo de SQL dedicado existente (anteriormente SQL DW), consulte [Habilitación de un área de trabajo para el grupo de SQL dedicado (anteriormente SQL DW)](./sql-data-warehouse/workspace-connected-create.md).


## <a name="open-synapse-studio"></a>Abrir Synapse Studio

Una vez creada el área de trabajo de Azure Synapse, hay dos maneras de abrir Synapse Studio:

* Abra el área de trabajo de Synapse en [Azure Portal](https://portal.azure.com) y, en la sección **Información general** del área de trabajo de Synapse, seleccione **Abrir** en el cuadro Abrir Synapse Studio.
* Vaya a `https://web.azuresynapse.net` e inicie sesión en el área de trabajo.

## <a name="place-sample-data-into-the-primary-storage-account"></a>Colocación de datos de ejemplo en la cuenta de almacenamiento principal
Vamos a usar un pequeño conjunto de datos de ejemplo de 100 000 filas de datos de taxis de Nueva York para muchos ejemplos de esta guía de introducción. Comenzaremos por colocarlo en la cuenta de almacenamiento principal que creó para el área de trabajo.

* Descargue este archivo en el equipo: https://azuresynapsestorage.blob.core.windows.net/sampledata/NYCTaxiSmall/NYCTripSmall.parquet 
* En Synapse Studio, vaya al centro de datos. 
* Seleccione **Vinculado**.
* En la categoría **Azure Data Lake Storage Gen2**, verá un elemento con el nombre **myworkspace ( Primary - contosolake )** .
* Seleccione el contenedor denominado **users (Primary)** (usuarios [Principal]).
* Seleccione **Cargar** y seleccione el archivo `NYCTripSmall.parquet` que ha descargado.

Se carga uno de los archivos en formato parquet y está disponible a través de dos identificadores URI equivalentes:
* `https://contosolake.dfs.core.windows.net/users/NYCTripSmall.parquet` 
* `abfss://users@contosolake.dfs.core.windows.net/NYCTripSmall.parquet`

En los ejemplos siguientes de este tutorial, asegúrese de reemplazar en la interfaz de usuario **contosolake** por el nombre de la cuenta de almacenamiento principal que seleccionó para el área de trabajo.



## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Análisis de datos con un grupo de SQL sin servidor](get-started-analyze-sql-on-demand.md)
