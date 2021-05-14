---
title: Generación de un mapa térmico de humedad del suelo
description: Describe cómo generar un mapa térmico de humedad del suelo en Azure FarmBeats
author: RiyazPishori
ms.topic: article
ms.date: 11/04/2019
ms.author: riyazp
ms.openlocfilehash: 4b796af22604a2f09482bcb9a321159ebaafea11
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108756780"
---
# <a name="generate-soil-moisture-heatmap"></a>Generación de un mapa térmico de humedad del suelo

La humedad del suelo es el agua que se retiene en los espacios entre las partículas del suelo. El mapa térmico de humedad del suelo ayuda a comprender los datos de humedad a cualquier profundidad y a alta resolución en las granjas. Para generar un mapa térmico de humedad del suelo preciso y útil, se requiere una implementación uniforme de sensores del mismo proveedor. Los diferentes proveedores tendrán diferencias en la forma en la que se mide la humedad del suelo, así como diferencias de calibración. El mapa térmico se genera para una profundidad determinada mediante los sensores implementados a esa profundidad.

En este artículo se describe el proceso de generación de un mapa térmico de humedad del suelo para la granja mediante el acelerador de Azure FarmBeats. En este artículo, aprenderá a:

- [Crear granjas](#create-a-farm)
- [Asignar sensores a las granjas](#get-soil-moisture-sensor-data-from-partner)
- [Generar un mapa térmico de humedad del suelo](#generate-soil-moisture-heatmap)

## <a name="before-you-begin"></a>Antes de empezar

Asegúrese de lo siguiente:  

- Suscripción a Azure.
- Una instancia en ejecución de Azure FarmBeats.
- Hay disponible un mínimo de tres sensores de humedad del suelo para la granja.

## <a name="create-a-farm"></a>Creación de una granja

Una granja es una zona geográfica de interés para la que desea crear una mapa térmico de humedad del suelo. Puede crear una granja mediante [Farms API](https://aka.ms/FarmBeatsDatahubSwagger) o en la [interfaz de usuario del acelerador de FarmsBeats](manage-farms-in-azure-farmbeats.md#create-farms).

## <a name="deploy-sensors"></a>Implementación de sensores

Debe implementar físicamente sensores de humedad del suelo en la granja. Puede adquirir sensores de humedad del suelo de nuestros asociados aprobados: [Davis Instruments](https://www.davisinstruments.com/product/enviromonitor-gateway/) y [Teralytic](https://teralytic.com/). Debe coordinarse con el proveedor de sensores para realizar la instalación física en la granja.

## <a name="get-soil-moisture-sensor-data-from-partner"></a>Obtención de datos del sensor de humedad del suelo del asociado

A medida que los sensores inician la transmisión de los datos al panel de datos del asociado, habilitan los datos en Azure FarmBeats. Esto puede hacerse desde la aplicación asociada.

Por ejemplo, si ha comprado sensores de Davis, iniciará sesión en su cuenta de vínculo meteorológico y proporcionará las credenciales necesarias para habilitar la transmisión de datos a Azure FarmBeats. Para obtener las credenciales necesarias, siga las instrucciones de [Obtención de datos del sensor](get-sensor-data-from-sensor-partner.md#get-sensor-data-from-sensor-partners).

Cuando haya escrito sus credenciales y seleccionado **Submit** (Enviar) en la aplicación asociada, puede hacer que los datos fluyan a Azure FarmBeats.

### <a name="assign-soil-moisture-sensors-to-the-farm"></a>Asignación de sensores de humedad de suelo a la granja

Una vez que haya vinculado su cuenta de sensor a Azure FarmBeats, deberá asignar los sensores de humedad del suelo a la granja en cuestión.

1.  En la página principal, seleccione **Farms** (Granjas) en el menú y aparecerá la página de la lista **Farms** (Granjas).
2.  Seleccione **MyFarm** > **Add Devices** (Mi granja > Agregar dispositivos).
3.  Aparece la ventana **Add Devices** (Agregar dispositivos). Seleccione cualquier dispositivo vinculado a los sensores de humedad del suelo de la granja.

    ![Captura de pantalla que muestra la pantalla Agregar dispositivos.](./media/get-sensor-data-from-sensor-partner/add-devices-1.png)

4. Seleccione **Add Devices** (Agregar dispositivos).     

## <a name="generate-soil-moisture-heatmap"></a>Generación de un mapa térmico de humedad del suelo

Este paso consiste en crear un trabajo o una operación de larga duración que generará un mapa térmico de humedad del suelo para la granja.

1.  En la página principal, vaya a **Farms** (Granjas) en el menú de navegación izquierdo para ver la página de granjas.
2.  Seleccione **MyFarm** (Mi granja).
3.  En la página **Farm Details** (Detalles de la granja), seleccione **Generate Precision Map** (Generar mapa de precisión).
4.  En el menú desplegable, seleccione **Soil Moisture** (Humedad del suelo).
5.  En la ventana **Soil Moisture** (Humedad del suelo), seleccione **This Week** (Esta semana).
6.  En **Select Soil Moisture** **Sensor Measure** (Seleccionar medida del sensor de humedad del suelo), escriba la medida que desea utilizar para el mapa.
    Para buscar la medida del sensor, en **Sensors** (Sensores), seleccione cualquier sensor de humedad del suelo. En **Sensor Properties** (Propiedades del sensor), use el valor de **Measure Name** (Nombre de medida).

    ![Captura de pantalla que muestra la pantalla Humedad del suelo.](./media/get-sensor-data-from-sensor-partner/soil-moisture-1.png)


7.  Seleccione **Generate Maps** (Generar mapas).
    Aparecerá un mensaje de confirmación con los detalles del trabajo. Para más información, consulte Job Status (Estado del trabajo) en Jobs (Trabajos).

    >[!NOTE]
    > El trabajo tarda entre tres y cuatro horas en completarse.

### <a name="download-the-soil-moisture-heatmap"></a>Descarga del mapa térmico de humedad del suelo

Siga estos pasos:

1. En la página **Jobs** (Trabajos), compruebe **Job Status** (Estado del trabajo) para el trabajo que creó en el último procedimiento.
2. Cuando el estado del trabajo muestre **Succeeded** (Correcto), haga clic en **Maps** (Mapas) en el menú.
3. Busque el mapa por el día en que se creó con el formato <soil-moisture_miGranja_AAAA-MM-DD>.
4. Seleccione un mapa de la columna **Name** (Nombre) y aparecerá una ventana emergente con la vista previa del mapa seleccionado.
5. Seleccione **Descargar**. El mapa se descarga y se almacena en la carpeta local del equipo.

    ![Proyecto de FarmBeats](./media/get-sensor-data-from-sensor-partner/download-soil-moisture-map-1.png)

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha generado correctamente un mapa térmico de humedad del suelo, aprenda a [generar mapas de colocación de sensores](generate-maps-in-azure-farmbeats.md#sensor-placement-map) y a [ingerir datos históricos de telemetría](ingest-historical-telemetry-data-in-azure-farmbeats.md). 
