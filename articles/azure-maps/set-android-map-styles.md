---
title: Establecimiento de un estilo de mapa en los mapas de Android | Microsoft Azure Maps
description: Obtenga información sobre dos maneras de establecer el estilo de un mapa. Vea cómo usar el SDK para Android de Azure Maps en el archivo de diseño o en la clase de actividad para ajustar el estilo.
author: rbrundritt
ms.author: richbrun
ms.date: 02/26/2021
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: cpendle
zone_pivot_groups: azure-maps-android
ms.openlocfilehash: 4d96d116bc4350a3326722c87f65e887d5a54791
ms.sourcegitcommit: f6b76df4c22f1c605682418f3f2385131512508d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108326550"
---
# <a name="set-map-style-android-sdk"></a>Establecimiento del estilo de mapa (Android SDK)

En este artículo se muestran dos maneras de establecer estilos de mapa mediante Android SDK de Azure Maps. Azure Maps tiene cuatro estilos de mapas entre los que puede elegir. Para obtener más información sobre los estilos de mapa compatibles, consulte [Estilos de mapa admitidos en Azure Maps](supported-map-styles.md).

## <a name="prerequisites"></a>Prerrequisitos

Asegúrese de completar los pasos descritos en el documento [Inicio rápido: Creación de una aplicación de Android](quick-android-map.md).

>[!important]
>El procedimiento descrito en esta sección requiere una cuenta de Azure Maps en el plan de tarifa Gen 1 o Gen 2. Para más información sobre los planes de tarifa, consulte [Elección del plan de tarifa adecuado de Azure Maps](choose-pricing-tier.md).


## <a name="set-map-style-in-the-layout"></a>Establecimiento del estilo del mapa en el diseño

Puede establecer un estilo de mapa en el archivo de diseño de la clase de actividad al agregar el control de mapa. En el código siguiente se establece la ubicación central, el nivel de zoom y el estilo de mapa.

```XML
<com.microsoft.azure.maps.mapcontrol.MapControl
    android:id="@+id/mapcontrol"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    app:mapcontrol_centerLat="47.602806"
    app:mapcontrol_centerLng="-122.329330"
    app:mapcontrol_zoom="12"
    app:mapcontrol_style="grayscale_dark"
    />
```

En la siguiente captura de pantalla se muestra cómo el código anterior presenta un mapa de carreteras con el estilo oscuro en escala de grises.

![Mapa con estilo de mapa de carreteras oscuro en escala de grises](media/set-android-map-styles/android-grayscale-dark.png)

## <a name="set-map-style-in-code"></a>Establecimiento del estilo de mapa en el código

El estilo de mapa se puede establecer mediante programación en código con el método `setStyle` del mapa. En el siguiente código se establece la ubicación central y el nivel de zoom mediante el método `setCamera` de los mapas y el estilo de mapa en `SATELLITE_ROAD_LABELS`.

::: zone pivot="programming-language-java-android"

```java
mapControl.onReady(map -> {

    //Set the camera of the map.
    map.setCamera(center(Point.fromLngLat(-122.33, 47.64)), zoom(14));

    //Set the style of the map.
    map.setStyle(style(MapStyle.SATELLITE_ROAD_LABELS));
});
```

::: zone-end

::: zone pivot="programming-language-kotlin"

```kotlin
mapControl!!.onReady { map: AzureMap ->
    //Set the camera of the map.
    map.setCamera(center(Point.fromLngLat(-122.33, 47.64)), zoom(14))

    //Set the style of the map.
    map.setStyle(style(MapStyle.SATELLITE_ROAD_LABELS))
}
```

::: zone-end

En la siguiente captura de pantalla se muestra cómo el código anterior presenta un mapa con el estilo de etiquetas de carretera de satélite.

![Mapa con estilo de etiquetas de carretera de satélite](media/set-android-map-styles/android-satellite-road-labels.png)

## <a name="setting-the-map-camera"></a>Establecimiento de la cámara del mapa

La cámara del mapa controla qué parte del mundo se muestra en la ventanilla de mapa. La cámara puede estar en el diseño o establecerse mediante programación en código. Cuando se establece en el código, hay dos métodos principales para determinar la posición del mapa: usar el centro y el zoom o indicar un rectángulo delimitador. En el siguiente código se muestra cómo establecer todas las opciones posibles de la cámara cuando se usan `center` y `zoom`.

::: zone pivot="programming-language-java-android"

```java
//Set the camera of the map using center and zoom.
map.setCamera(
    center(Point.fromLngLat(-122.33, 47.64)), 

    //The zoom level. Typically a value between 0 and 22.
    zoom(14),

    //The amount of tilt in degrees the map where 0 is looking straight down.
    pitch(45),

    //Direction the top of the map is pointing in degrees. 0 = North, 90 = East, 180 = South, 270 = West
    bearing(90),

    //The minimum zoom level the map will zoom-out to when animating from one location to another on the map.
    minZoom(10),
    
    //The maximum zoom level the map will zoom-in to when animating from one location to another on the map.
    maxZoom(14)
);
```

::: zone-end

::: zone pivot="programming-language-kotlin"

```kotlin
//Set the camera of the map using center and zoom.
map.setCamera(
    center(Point.fromLngLat(-122.33, 47.64)), 

    //The zoom level. Typically a value between 0 and 22.
    zoom(14),

    //The amount of tilt in degrees the map where 0 is looking straight down.
    pitch(45),

    //Direction the top of the map is pointing in degrees. 0 = North, 90 = East, 180 = South, 270 = West
    bearing(90),

    //The minimum zoom level the map will zoom-out to when animating from one location to another on the map.
    minZoom(10),
    
    //The maximum zoom level the map will zoom-in to when animating from one location to another on the map.
    maxZoom(14)
)
```

::: zone-end

A menudo conviene centrar el mapa en un conjunto de datos. Se puede calcular un rectángulo delimitador a partir de las características mediante el método `MapMath.fromData`, que después se puede pasar en la opción `bounds` de la cámara del mapa. Al establecer una vista de mapa en función de un rectángulo delimitador, a menudo resulta útil especificar un valor `padding` para tener en cuenta el tamaño de píxel de los puntos que se representan como burbujas o símbolos. En el siguiente código se muestra cómo establecer todas las opciones posibles de la cámara cuando se usa un rectángulo delimitador para establecer la posición de la cámara.

::: zone pivot="programming-language-java-android"

```java
//Set the camera of the map using a bounding box.
map.setCamera(
    //The area to focus the map on.
    bounds(BoundingBox.fromLngLats(
        //West
        -122.4594,

        //South
        47.4333,
        
        //East
        -122.21866,
        
        //North
        47.75758
    )),

    //Amount of pixel buffer around the bounding box to provide extra space around the bounding box.
    padding(20),

    //The maximum zoom level the map will zoom-in to when animating from one location to another on the map.
    maxZoom(14)
);
```

::: zone-end

::: zone pivot="programming-language-kotlin"

```kotlin
//Set the camera of the map using a bounding box.
map.setCamera(
    //The area to focus the map on.
    bounds(BoundingBox.fromLngLats(
        //West
        -122.4594,

        //South
        47.4333,
        
        //East
        -122.21866,
        
        //North
        47.75758
    )),

    //Amount of pixel buffer around the bounding box to provide extra space around the bounding box.
    padding(20),

    //The maximum zoom level the map will zoom-in to when animating from one location to another on the map.
    maxZoom(14)
)
```

::: zone-end

Tenga en cuenta que la relación de aspecto de un rectángulo delimitador puede ser diferente de la del mapa, ya que el mapa mostrará a menudo toda el área del rectángulo delimitador, pero a menudo solo se ajustará vertical u horizontalmente.

## <a name="next-steps"></a>Pasos siguientes

Para obtener más ejemplos de código para agregar a los mapas:

> [!div class="nextstepaction"]
> [Adición de una capa de símbolo](how-to-add-symbol-to-android-map.md)

> [!div class="nextstepaction"]
> [Adición de una capa de burbuja](map-add-bubble-layer-android.md)
