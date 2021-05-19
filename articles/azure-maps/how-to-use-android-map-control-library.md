---
title: Introducción al control de mapa de Android | Microsoft Azure Maps
description: Familiarícese con el SDK para Android de Azure Maps. Vea cómo crear un proyecto en Android Studio, instalar el SDK y crear un mapa interactivo.
author: rbrundritt
ms.author: richbrun
ms.date: 2/26/2021
ms.topic: how-to
ms.service: azure-maps
services: azure-maps
manager: cpendle
zone_pivot_groups: azure-maps-android
ms.openlocfilehash: a60ee8faf8d19afba59e46c52aaba9395c3a5292
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "105604454"
---
# <a name="getting-started-with-azure-maps-android-sdk"></a>Introducción a Android SDK para Azure Maps

Android SDK para Azure Maps es una biblioteca de mapas vectoriales para Android. En este artículo se ofrece orientación sobre los procesos para instalar Android SDK para Azure Maps y cargar un mapa.

## <a name="prerequisites"></a>Prerrequisitos

Asegúrese de completar los pasos descritos en el documento [Inicio rápido: Creación de una aplicación de Android](quick-android-map.md).

## <a name="localizing-the-map"></a>Localización del mapa

El Android SDK Azure Maps proporciona tres formas diferentes de establecer el idioma y la vista regional del mapa. En el código siguiente se muestra cómo establecer el idioma en francés ("fr-FR") y la vista regional en "Auto".

La primera opción es pasar el lenguaje y ver la información regional en la clase `AzureMaps` mediante los métodos `setLanguage` y `setView` estáticos globalmente. Así se establecerá el idioma y la vista regional predeterminados en todos los controles de Azure Maps cargados en la aplicación.

::: zone pivot="programming-language-java-android"

```java
static {
    //Alternatively use Azure Active Directory authenticate.
    AzureMaps.setAadProperties("<Your aad clientId>", "<Your aad AppId>", "<Your aad Tenant>");

    //Set your Azure Maps Key.
    //AzureMaps.setSubscriptionKey("<Your Azure Maps Key>");   

    //Set the language to be used by Azure Maps.
    AzureMaps.setLanguage("fr-FR");

    //Set the regional view to be used by Azure Maps.
    AzureMaps.setView("Auto");
}
```

::: zone-end

::: zone pivot="programming-language-kotlin"

```kotlin
companion object {
    init {
        //Alternatively use Azure Active Directory authenticate.
        AzureMaps.setAadProperties("<Your aad clientId>", "<Your aad AppId>", "<Your aad Tenant>");

        //Set your Azure Maps Key.
        //AzureMaps.setSubscriptionKey("<Your Azure Maps Key>");
    
        //Set the language to be used by Azure Maps.
        AzureMaps.setLanguage("fr-FR");
    
        //Set the regional view to be used by Azure Maps.
        AzureMaps.setView("Auto");
    }
}
```

::: zone-end

La segunda opción es usar la información de idioma y de la vista en el XML del control de mapa.

```XML
<com.microsoft.azure.maps.mapcontrol.MapControl
    android:id="@+id/myMap"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    app:mapcontrol_language="fr-FR"
    app:mapcontrol_view="Auto"
    />
```

La tercera opción consiste en establecer mediante programación el idioma y la vista regional del mapa mediante el método `setStyle` de Maps. Esta opción puede realizarse en cualquier momento para cambiar el idioma y la vista regional del mapa.

::: zone pivot="programming-language-java-android"

```java
mapControl.onReady(map -> {
    map.setStyle(
        language("fr-FR"),
        view("Auto")
    );
});
```

::: zone-end

::: zone pivot="programming-language-kotlin"

```kotlin
mapControl.onReady(OnReady { map: AzureMap ->
    map.setStyle(
        language("fr-FR"),
        view("Auto")
    )
})
```

::: zone-end

Este es un ejemplo de Azure Maps con el idioma establecido en "fr-FR" y la vista regional establecida en "Auto".

![Imagen del mapa de Azure Maps que muestra las etiquetas en francés](media/how-to-use-android-map-control-library/android-localization.png)

[Aquí](supported-languages.md) encontrará una lista completa de los idiomas y vistas regionales admitidos.

## <a name="navigating-the-map&quot;></a>Navegación por el mapa

Hay varias maneras de ampliar, desplazar lateralmente, girar e inclinar el mapa. A continuación se detallan todas las formas de navegar por el mapa.

**Acercamiento/alejamiento del mapa**

* Toque el mapa con dos dedos y acérquelos para alejarse o aléjelos para acercarse.
* Pulse dos veces el mapa para acercarse un nivel.
* Pulse dos veces con dos dedos para alejar el mapa un nivel.
* Pulse dos veces; en la segunda pulsación, mantenga presionado el dedo en el mapa y arrástrelo hacia arriba o hacia abajo para acercar o alejar la imagen, respectivamente.

**Desplazamiento lateral del mapa**

* Toque el mapa y arrastre en cualquier dirección.

**Giro del mapa**

* Toque el mapa con dos dedos y gírelos.

**Inclinación del mapa**

* Toque el mapa con dos dedos y arrástrelos hacia arriba o hacia abajo.

## <a name=&quot;azure-government-cloud-support&quot;></a>Compatibilidad con la nube de Azure Government

Android SDK de Azure Maps es compatible con la nube de Azure Government. Se accede a Android SDK de Azure Maps desde el mismo repositorio de Maven. Se deben realizar las siguientes tareas para conectarse a la versión en la nube de Azure Government de la plataforma de Azure Maps.

En el mismo lugar donde se especifican los detalles de autenticación de Azure Maps, agregue la línea de código siguiente para indicar al mapa que use el dominio de nube del gobierno de Azure Maps.

::: zone pivot="programming-language-java-android"

```java
AzureMaps.setDomain(&quot;atlas.azure.us");
```

::: zone-end

::: zone pivot="programming-language-kotlin"

```kotlin
AzureMaps.setDomain("atlas.azure.us")
```

::: zone-end

No olvide usar los detalles de autenticación de Azure Maps de la plataforma en la nube de Azure Government cuando autentique el mapa y los servicios.

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre cómo agregar datos de superposición en el mapa:

> [!div class="nextstepaction"]
> [Administración de la autenticación en Azure Maps](how-to-manage-authentication.md)

> [!div class="nextstepaction"]
> [Cambio de estilos de mapa en mapas de Android](set-android-map-styles.md)

> [!div class="nextstepaction"]
> [Adición de una capa de símbolo](how-to-add-symbol-to-android-map.md)

> [!div class="nextstepaction"]
> [Adición de una capa de línea](android-map-add-line-layer.md)

> [!div class="nextstepaction"]
> [Adición de una capa de polígono](how-to-add-shapes-to-android-map.md)
