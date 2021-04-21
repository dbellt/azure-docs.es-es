---
title: Uso de Azure Stream Analytics en un grupo de SQL dedicado
description: Sugerencias para usar Azure Stream Analytics con un grupo de SQL dedicado en Azure Synapse para el desarrollo de soluciones en tiempo real.
services: synapse-analytics
author: julieMSFT
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 9/25/2020
ms.author: jrasnick
ms.reviewer: igorstan
ms.custom: azure-synapse
ms.openlocfilehash: 0c7f139b50cd43e3e8862fda3f5401a853ced8d0
ms.sourcegitcommit: 590f14d35e831a2dbb803fc12ebbd3ed2046abff
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107566586"
---
# <a name="use-azure-stream-analytics-with-dedicated-sql-pool-in-azure-synapse-analytics"></a>Uso de Azure Stream Analytics con un grupo de SQL dedicado en Azure Synapse Analytics

Azure Stream Analytics es un servicio totalmente administrado que proporciona un procesamiento completo de eventos de baja latencia, alta disponibilidad y escalable a través el streaming de datos en la nube. Para aprender los conceptos básicos, lea [Introducción a Azure Stream Analytics](../../stream-analytics/stream-analytics-introduction.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json). Después puede aprender a crear una solución de extremo a extremo siguiendo el tutorial [Introducción al uso de Azure Stream Analytics](../../stream-analytics/stream-analytics-real-time-fraud-detection.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json) .

En este artículo, aprenderá a usar el grupo de SQL dedicado como receptor de salida para la ingesta de datos de alto rendimiento con trabajos de Azure Stream Analytics.

## <a name="prerequisites"></a>Prerrequisitos

* Trabajo de Azure Stream Analytics: para crear un trabajo de Azure Stream Analytics, siga los pasos descritos en el tutorial [Introducción al uso de Azure Stream Analytics](../../stream-analytics/stream-analytics-real-time-fraud-detection.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json) relativos a lo siguiente:  

    1. Creación de una entrada de Centro de eventos
    2. Configuración e inicio de la aplicación del generador de eventos
    3. Aprovisionamiento de un trabajo de Stream Analytics
    4. Especificación de una consulta y entrada de trabajo
* Grupo de SQL dedicado: para crear un nuevo grupo de SQL dedicado, siga los pasos descritos en [Inicio rápido: creación de un grupo de SQL dedicado](../quickstart-create-sql-pool-portal.md).

## <a name="specify-streaming-output-to-point-to-your-dedicated-sql-pool"></a>Especificación de la salida de streaming para que apunte al grupo de SQL dedicado

### <a name="step-1"></a>Paso 1

En Azure Portal, vaya a su trabajo de Stream Analytics y haga clic en **Salidas** en el menú **Topología de trabajo**.

### <a name="step-2"></a>Paso 2

Haga clic en el botón **Agregar** y elija **Azure Synapse Analytics** en el menú desplegable.

![Selección de Azure Synapse Analytics](./media/sql-data-warehouse-integrate-azure-stream-analytics/sql-pool-azure-stream-analytics-output.png)

### <a name="step-3"></a>Paso 3

Escriba los siguientes valores:

* *Alias de salida*: escriba un nombre descriptivo para esta salida de trabajo.
* *Suscripción*:
  * Si el grupo de SQL dedicado está en la misma suscripción que el trabajo de Stream Analytics, haga clic en ***Seleccionar Azure Synapse Analytics de las suscripciones***.
  * Si el grupo de SQL dedicado está en otra suscripción, haga clic en Proporcionar configuración de Azure Synapse Analytics de forma manual.
* *Base de datos*: Seleccione la base de datos de destino en la lista desplegable.
* *Nombre de usuario*: especifique el nombre de usuario de una cuenta que tenga permisos de escritura para la base de datos.
* *Contraseña*: proporcione la contraseña de la cuenta de usuario especificada.
* *Tabla*: especifique el nombre de la tabla de destino en la base de datos.
* Haga clic en el botón **Guardar**.

![Formulario completado de Azure Synapse Analytics](./media/sql-data-warehouse-integrate-azure-stream-analytics/sql-pool-azure-stream-analytics-output-db-settings.png)

### <a name="step-4"></a>Paso 4

Antes de poder ejecutar una prueba, deberá crear la tabla en el grupo de SQL dedicado.  Ejecute el siguiente script de creación de tabla con SQL Server Management Studio (SSMS) o la herramienta de consulta que prefiera.

```sql
CREATE TABLE SensorLog
(
    RecordType VARCHAR(2)
    , SystemIdentity VARCHAR(2)
    , FileNum INT
    , SwitchNum VARCHAR(50)
    , CallingNum VARCHAR(25)
    , CallingIMSI VARCHAR(25)
    , CalledNum VARCHAR(25)
    , CalledIMSI VARCHAR(25)
    , DateS VARCHAR(25)
    , TimeS VARCHAR(25)
    , TimeType INT
    , CallPeriod INT
    , CallingCellID VARCHAR(25)
    , CalledCellID VARCHAR(25)
    , ServiceType VARCHAR(25)
    , [Transfer] INT
    , IncomingTrunk VARCHAR(25)
    , OutgoingTrunk VARCHAR(25)
    , MSRN VARCHAR(25)
    , CalledNum2 VARCHAR(25)
    , FCIFlag VARCHAR(25)
    , callrecTime VARCHAR(50)
    , EventProcessedUtcTime VARCHAR(50)
    , PartitionId int
    , EventEnqueuedUtcTime VARCHAR(50)
    )
WITH (DISTRIBUTION = ROUND_ROBIN)
```

### <a name="step-5"></a>Paso 5

En Azure Portal para su trabajo de Stream Analytics, haga clic en el nombre del trabajo.  Haga clic en el botón ***Probar** _ en el panel _ *_Detalles de salida_**.

![Botón Probar en Detalles de salida](./media/sql-data-warehouse-integrate-azure-stream-analytics/sqlpool-asatest.png) Cuando la conexión a la base de datos se realice correctamente, verá una notificación en el portal.

### <a name="step-6"></a>Paso 6

Haga clic en el menú ***Consulta** _ en _*_Topología de trabajo_*_ y cambie la consulta para insertar datos en la salida de flujo que ha creado.  Haga clic en el botón _*_Probar consulta seleccionada_*_ para probar la consulta.  Haga clic en el botón _ *_Guardar consulta_** cuando la prueba de la consulta sea correcta.

![Guardar consulta](./media/sql-data-warehouse-integrate-azure-stream-analytics/sqlpool-asaquery.png)

### <a name="step-7"></a>Paso 7

Inicie el trabajo de Azure Stream Analytics.  Haga clic en el botón ***Iniciar** en el menú _ *_Información general_**.

![Inicio del trabajo de Stream Analytics](./media/sql-data-warehouse-integrate-azure-stream-analytics/sqlpool-asastart.png)

Haga clic en el botón ***Iniciar*** en el panel Iniciar trabajo.

![Hacer clic en Iniciar](./media/sql-data-warehouse-integrate-azure-stream-analytics/sqlpool-asastartconfirm.png)

## <a name="next-steps"></a>Pasos siguientes

Para obtener información general sobre la integración, consulte [Integrar otros servicios](sql-data-warehouse-overview-integrate.md).
Para más consejos de desarrollo, consulte [Decisiones de diseño y técnicas de codificación para el grupo de SQL dedicado](sql-data-warehouse-overview-develop.md).
