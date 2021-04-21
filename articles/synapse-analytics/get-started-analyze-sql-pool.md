---
title: 'Tutorial: Introducción al análisis de datos con grupos de SQL dedicados'
description: En este tutorial, usará los datos de ejemplo de NYC Taxi para explorar las funcionalidades de análisis del grupo de SQL.
services: synapse-analytics
author: saveenr
ms.author: saveenr
manager: julieMSFT
ms.reviewer: jrasnick
ms.service: synapse-analytics
ms.subservice: sql
ms.topic: tutorial
ms.date: 03/24/2021
ms.openlocfilehash: 0def1f957842417c3936e3f1c7bb5bc023109818
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107536330"
---
# <a name="analyze-data-with-dedicated-sql-pools"></a>Análisis de datos con grupos de SQL dedicados

En este tutorial usará los datos de NYC Taxi para explorar las funcionalidades de un grupo de SQL dedicado.

## <a name="create-a-dedicated-sql-pool"></a>Creación de un grupo de SQL dedicado

1. En Synapse Studio, en el panel izquierdo, seleccione **Administrar** > **Grupos de SQL**.
1. Seleccione **Nuevo paso**.
1. En **Nombre del grupo de SQL** seleccione **SQLPOOL1**
1. En **Nivel de rendimiento** seleccione **DW100C**
1. Seleccione **Revisar y crear** > **Crear**. El grupo de SQL dedicado estará listo en unos minutos. 

El grupo de SQL dedicado se asocia con una base de datos SQL, que también se denomina **SQLPOOL1**.
1. Vaya a **Datos** > **Área de trabajo**.
1. Debería ver una base de datos denominada **SQLPOOL1**. Si no la ve, haga clic en **Refresh** (Actualizar).

Los grupos de SQL dedicados consumen recursos facturables mientras están activos. Puede pausar los grupos más adelante para reducir los costos.

> [!NOTE] 
> Al crear un nuevo grupo de SQL dedicado (anteriormente SQL DW) en el área de trabajo, se abrirá la página de aprovisionamiento del grupo de SQL dedicado. El aprovisionamiento se llevará a cabo en el servidor SQL lógico.
## <a name="load-the-nyc-taxi-data-into-sqlpool1"></a>Carga de los datos NYC Taxi en SQLPOOL1

1. En Synapse Studio, vaya al centro de **desarrollo**, haga clic en el botón **+** para agregar un nuevo recurso y cree un nuevo script SQL.
1. Seleccione el grupo "SQLPOOL1" (creado en el [paso 1](./get-started-create-workspace.md) de este tutorial) en la sección "Conectar a" del script.
1. Escriba el siguiente código:
    ```
    IF NOT EXISTS (SELECT * FROM sys.objects O JOIN sys.schemas S ON O.schema_id = S.schema_id WHERE O.NAME = 'NYCTaxiTripSmall' AND O.TYPE = 'U' AND S.NAME = 'dbo')
    CREATE TABLE dbo.NYCTaxiTripSmall
        (
         [DateID] int,
         [MedallionID] int,
         [HackneyLicenseID] int,
         [PickupTimeID] int,
         [DropoffTimeID] int,
         [PickupGeographyID] int,
         [DropoffGeographyID] int,
         [PickupLatitude] float,
         [PickupLongitude] float,
         [PickupLatLong] nvarchar(4000),
         [DropoffLatitude] float,
         [DropoffLongitude] float,
         [DropoffLatLong] nvarchar(4000),
         [PassengerCount] int,
         [TripDurationSeconds] int,
         [TripDistanceMiles] float,
         [PaymentType] nvarchar(4000),
         [FareAmount] numeric(19,4),
         [SurchargeAmount] numeric(19,4),
         [TaxAmount] numeric(19,4),
         [TipAmount] numeric(19,4),
         [TollsAmount] numeric(19,4),
         [TotalAmount] numeric(19,4)
        )
    WITH
        (
        DISTRIBUTION = ROUND_ROBIN,
         CLUSTERED COLUMNSTORE INDEX
         -- HEAP
        )
    GO

    COPY INTO dbo.NYCTaxiTripSmall
    (DateID 1, MedallionID 2, HackneyLicenseID 3, PickupTimeID 4, DropoffTimeID 5,
    PickupGeographyID 6, DropoffGeographyID 7, PickupLatitude 8, PickupLongitude 9, 
    PickupLatLong 10, DropoffLatitude 11, DropoffLongitude 12, DropoffLatLong 13, 
    PassengerCount 14, TripDurationSeconds 15, TripDistanceMiles 16, PaymentType 17, 
    FareAmount 18, SurchargeAmount 19, TaxAmount 20, TipAmount 21, TollsAmount 22, 
    TotalAmount 23)
    FROM 'https://contosolake.dfs.core.windows.net/users/NYCTripSmall.parquet'
    WITH
    (
        FILE_TYPE = 'PARQUET'
        ,MAXERRORS = 0
        ,IDENTITY_INSERT = 'OFF'
    )
    ```
1. Haga clic en el botón Ejecutar para ejecutar el script.
1. Este script finalizará en menos de 60 segundos. Carga dos millones filas de datos de NYC Taxi en una tabla denominada **dbo.Trip**.

## <a name="explore-the-nyc-taxi-data-in-the-dedicated-sql-pool"></a>Exploración de los datos de NYC Taxi en el grupo de SQL dedicado

1. En Synapse Studio, vaya al centro **Data** (Datos).
1. Vaya a **SQLPOOL1** > **Tablas**. 
3. Haga clic con el botón derecho en la tabla **dbo.NYCTaxiTripSmall** y seleccione **New SQL Script** > **Select TOP 100 Rows** (Nuevo script SQL > Seleccionar 100 primeras filas).
4. Espere mientras se crea un nuevo script de SQL y se ejecuta.
5. Observe que en la parte superior del script de SQL, en **Connect to** (Conectar a) está seleccionado automáticamente el grupo de SQL llamado **SQLPOOL1**.
6. Reemplace el texto del script de SQL por este código y ejecútelo.

    ```sql
    SELECT PassengerCount,
          SUM(TripDistanceMiles) as SumTripDistance,
          AVG(TripDistanceMiles) as AvgTripDistance
    FROM  dbo.NYCTaxiTripSmall
    WHERE TripDistanceMiles > 0 AND PassengerCount > 0
    GROUP BY PassengerCount
    ORDER BY PassengerCount;
    ```

    En esta consulta se muestra la relación entre las distancias recorridas totales y la distancia media recorrida, y el número de pasajeros.
1. En la ventana de resultados del script de SQL, cambie el valor de **View** (Vista) a **Chart** (Gráfico) para ver los resultados en un gráfico de líneas.
    
## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Análisis de datos en una cuenta de Azure Storage](get-started-analyze-storage.md)
