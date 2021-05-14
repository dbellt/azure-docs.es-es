---
title: Administración de granjas
description: Describe cómo administrar granjas
author: RiyazPishori
ms.topic: article
ms.date: 11/04/2019
ms.author: riyazp
ms.openlocfilehash: 70d7ae985a56842f2b02c5af5a7c7c1e589b2b56
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108759822"
---
# <a name="manage-farms"></a>Administración de granjas

Puede administrar las granjas en Azure FarmBeats. En este artículo se proporciona información sobre cómo crear granjas, dispositivos, sensores y drones que le ayudarán a administrar las granjas.

## <a name="create-farms"></a>Creación de granjas

Siga estos pasos:

1. Inicie sesión en Farm Accelerator y se mostrará la página **Farms** (Granjas).
    En la página **Farms** (Granjas) se muestra la lista de granjas en caso de que ya se hayan creado en la suscripción.

    Este es la imagen de ejemplo:

    ![Captura de pantalla que muestra la página Farms (Granjas).](./media/create-farms-in-azure-farmbeats/create-farm-main-page-1.png)


2. Seleccione **Create Farm** (Crear granja) y proporcione valores para **Name** (Nombre), **Crops** (Cultivos) y **Address** (Dirección).
3. En **Define Farm Boundary** (Definir límite de la granja), (campo obligatorio) seleccione **Mark on Map** (Marcar en mapa) o **Paste GeoJSON code** (Pegar código GeoJSON).

Estas son las dos formas de definir un límite de granja:

1. **Mark on Map** (Marcar en mapa): Use la herramienta de control de mapa para dibujar y marcar el límite de la granja. Para marcar los límites, ![Captura de pantalla que muestra el icono de lápiz para dibujar los límites en el mapa](./media/create-farms-in-azure-farmbeats/pencil-icon-1.png) y marque los límites exactos.

    ![Captura de pantalla que muestra los límites dibujados en un mapa.](./media/create-farms-in-azure-farmbeats/create-farm-mark-on-map-1.png)

2. **Paste GeoJSON Code** (Pegar código GeoJSON): GeoJSON es un formato de codificación de estructuras de datos geográficos mediante notación de objetos JavaScript (JSON). Con esta opción se muestra un cuadro de texto en el que se puede escribir una cadena GeoJSON para marcar los límites de la granja. También puede crear código GeoJSON desde GeoJSON.io.
Utilice la información sobre herramientas como ayuda para rellenar la información.

    ![Captura de pantalla que resalta la opción para pegar código GeoJson en la pantalla Create Farm (Crear granja).](./media/create-farms-in-azure-farmbeats/create-new-farm-1.png)

3.  Seleccione **Submit** (Enviar) para crear la granja. Se creará una granja y se mostrará en la página **Farms** (Granjas).

## <a name="view-farm"></a>Visualización de la granja

La página de la lista Farms (Granjas) muestra una lista de granjas que se han creado. Seleccione una para ver la lista de:

 - **Device count** (Número de dispositivos): muestra el número de dispositivos implementados en la granja y su estado.
 - **Map** (Mapa): mapa de la granja con los dispositivos implementados en ella.
 - **Telemetry** (Telemetría): muestra la telemetría de los sensores implementados en la granja.
 - **Latest Precision Maps** (Mapas de precisión más recientes): muestra el mapa de índices por satélite (EVI, NDWI), el mapa térmico de humedad del suelo y el mapa de ubicación del sensor más recientes.

## <a name="edit-farm"></a>Edición de la granja

En la página **Farms** (Granjas) se muestra una lista de las granjas que se han creado.

1.  Seleccione una para ver y editar la granja.
2.  Seleccione **Edit Farm** (Editar granja) para editar la información de la granja. En la ventana **Farm Details** (Detalles de la granja) puede editar los campos **Name** (Nombre), **Crops** (Cultivos), **Address** (Dirección) y **Define Farm Boundary** (Definir límite de la granja).

    ![Proyecto de FarmBeats](./media/create-farms-in-azure-farmbeats/edit-farm-1.png)

3. Seleccione **Submit** (Enviar) para guardar los datos editados.

## <a name="delete-farm"></a>Eliminación de la granja

En la página **Farms** (Granjas) se muestra una lista de las granjas que se han creado. Siga estos pasos para eliminar una granja:

1.  Selecciónela en la lista para eliminar sus detalles.
2.  Seleccione **Delete Farm** (Eliminar granja) para eliminar la granja.

    ![Captura de pantalla que muestra la pantalla Delete Farm (Eliminar granja) y resalta el botón Delete (Eliminar).](./media/create-farms-in-azure-farmbeats/delete-farm-1.png)

    > [!NOTE]
    > Al eliminar una granja, los dispositivos y los mapas asociados a la granja no se eliminan. Cualquier información de la granja asociada con el dispositivo y los mapas no será pertinente. Los dispositivos, la telemetría y los mapas se seguirán viendo desde el servicio FarmBeats.


## <a name="next-steps"></a>Pasos siguientes

Ahora que ha creado la granja, aprenda a [obtener datos del sensor](get-sensor-data-from-sensor-partner.md) en la granja.
