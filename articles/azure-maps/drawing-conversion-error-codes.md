---
title: Errores y advertencias de conversión de dibujos de Azure Maps
description: Obtenga información sobre los errores y las advertencias de conversión que puede encontrar al utilizar el servicio de conversión de Azure Maps. Lea las recomendaciones sobre cómo resolver los errores y las advertencias, con algunos ejemplos.
author: anastasia-ms
ms.author: v-stharr
ms.date: 05/21/2021
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: philMea
ms.openlocfilehash: fd6c20277098d0a2d1909cfb93243dd8e3bbcca0
ms.sourcegitcommit: 7f59e3b79a12395d37d569c250285a15df7a1077
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/02/2021
ms.locfileid: "110793242"
---
# <a name="drawing-conversion-errors-and-warnings"></a>Errores y advertencias de conversión de dibujos

El servicio de conversión de [Azure Maps](/rest/api/maps/v2/conversion) permite convertir paquetes de dibujo cargados en datos de mapa. Los paquetes de dibujos deben ajustarse a los [requisitos de los paquetes de dibujos](drawing-requirements.md). Si no se cumplen uno o más requisitos, el servicio de conversión devolverá errores o advertencias. En este artículo se enumeran los códigos de error y de advertencia de conversión, con recomendaciones sobre cómo resolverlos. También proporciona algunos ejemplos de dibujos que pueden hacer que el servicio de conversión devuelva estos códigos.

El servicio de conversión realizará la operación correctamente si hay advertencias de conversión. Sin embargo, se recomienda que revise y resuelva todas las advertencias. Una advertencia significa que parte de la conversión se ignoró o corrigió automáticamente. Si no se resuelven las advertencias, podrían producirse errores en procesos posteriores.

## <a name="general-warnings"></a>Advertencias generales

### <a name="geometrywarning"></a>**geometryWarning**

#### <a name="description-for-geometrywarning"></a>*Descripción de geometryWarning*

Se produce una advertencia **geometryWarning** cuando el dibujo contiene una entidad no válida. Una entidad no válida es una entidad que no se ajusta a las restricciones geométricas. Algunos ejemplos de entidades no válidas son un polígono que forma intersección con él mismo o una polilínea sin cerrar en una capa que solo admite geometría cerrada.

El servicio de conversión no puede crear una característica del mapa a partir de una entidad no válida y, en su lugar, la omite.

#### <a name="examples-for-geometrywarning"></a>*Ejemplos de geometryWarning*

* Las dos imágenes siguientes muestran ejemplos de polígonos que forman intersección con ellos mismos.

     ![Ejemplo de un polígono que forma intersección con él mismo, primer ejemplo.](./media/drawing-conversion-error-codes/geometry-warning-1.png)

     ![Ejemplo de un polígono que forma intersección con él mismo, segundo ejemplo.](./media/drawing-conversion-error-codes/geometry-warning-2.png)

* A continuación se muestra una imagen con una polilínea sin cerrar. Supongamos que la capa solo admite geometría cerrada.

    ![Ejemplo de una polilínea sin cerrar](./media/drawing-conversion-error-codes/geometry-warning-3.png)

#### <a name="how-to-fix-geometrywarning"></a>*Cómo corregir geometryWarning*

Inspeccione **geometryWarning** para cada entidad para comprobar que sigue las restricciones geométricas.

### <a name="unexpectedgeometryinlayer"></a>**unexpectedGeometryInLayer**

#### <a name="description-for-unexpectedgeometryinlayer"></a>*Descripción de unexpectedGeometryInLayer*

Se produce una advertencia **unexpectedGeometryInLayer** cuando el dibujo contiene geometría incompatible con el tipo de geometría esperado para una capa determinada. Cuando el servicio de conversión devuelve una advertencia **unexpectedGeometryInLayer**, omitirá esa geometría.

#### <a name="example-for-unexpectedgeometryinlayer"></a>*Ejemplo de unexpectedGeometryInLayer*

En la imagen siguiente se muestra una polilínea sin cerrar. Supongamos que la capa solo admite geometría cerrada.

![Ejemplo de una polilínea sin cerrar](./media/drawing-conversion-error-codes/geometry-warning-3.png)

#### <a name="how-to-fix-unexpectedgeometryinlayer"></a>*Cómo corregir unexpectedGeometryInLayer*

Inspeccione cada advertencia **unexpectedGeometryInLayer** y mueva la geometría incompatible a una capa compatible. Si no es compatible con ninguna de las demás capas, se debe quitar.

### <a name="unsupportedfeaturerepresentation"></a>**unsupportedFeatureRepresentation**

#### <a name="description-for-unsupportedfeaturerepresentation"></a>*Descripción de unsupportedFeatureRepresentation*

La advertencia **unsupportedFeatureRepresentation** se produce cuando el dibujo contiene un tipo de entidad no admitido.

#### <a name="example-for-unsupportedfeaturerepresentation"></a>*Ejemplo de unsupportedFeatureRepresentation*

La imagen siguiente muestra un tipo de entidad no admitido como un objeto de texto de varias líneas en una capa de etiquetas.
  
![Ejemplo de un objeto de texto de varias líneas en la capa de etiquetas](./media/drawing-conversion-error-codes/multi-line.png)

#### <a name="how-to-fix-unsupportedfeaturerepresentation"></a>*Cómo corregir unsupportedFeatureRepresentation*

Asegúrese de que los archivos DWG contengan solo los tipos de entidad admitidos. Los tipos admitidos se enumeran en la [sección Requisitos de los archivos de dibujos en el artículo Requisitos de los paquetes de dibujos](drawing-requirements.md#drawing-package-requirements).

### <a name="automaticrepairperformed"></a>**automaticRepairPerformed**

#### <a name="description-for-automaticrepairperformed"></a>*Descripción de automaticRepairPerformed*

La advertencia **automaticRepairPerformed** se produce cuando el servicio de conversión repara automáticamente geometría no válida.

#### <a name="examples-for-automaticrepairperformed"></a>*Ejemplos de automaticRepairPerformed*

* En la imagen siguiente se muestra cómo el servicio de conversión reparó un polígono que formaba intersección con él mismo y generó una geometría válida.

    ![Ejemplo de un polígono que forma intersección con él mismo reparado](./media/drawing-conversion-error-codes/automatic-repair-1.png)

* En la imagen siguiente se muestra cómo el servicio de conversión ha ajustado el primero y el último vértice de una polilínea sin cerrar para crear una polilínea cerrada, en la que el primero y el último vértice tenían menos de 1 mm de distancia.  

    ![Ejemplo de polilínea partida](./media/drawing-conversion-error-codes/automatic-repair-2.png)

* En la imagen siguiente se muestra cómo, en una capa que solo admite polilíneas cerradas, el servicio de conversión reparó varias polilíneas sin cerrar. Para evitar descartar las polilíneas sin cerrar, el servicio las combinó en una única polilínea cerrada.

    ![Ejemplo de polilíneas sin cerrar combinadas en una única polilínea cerrada](./media/drawing-conversion-error-codes/automatic-repair-3.png)

#### <a name="how-to-fix-automaticrepairperformed"></a>*Cómo corregir automaticRepairPerformed*

Para corregir una advertencia **automaticRepairPerformed**, realice las siguientes acciones:

1. Inspeccione la geometría de cada advertencia y el texto de advertencia específico.
2. Determine si la reparación automatizada es correcta.
3. Si la reparación es correcta, continúe. En caso contrario, vaya al archivo de diseño y resuelva la advertencia manualmente.

>[!TIP]
>Para suprimir una advertencia en el futuro, realice cambios en el dibujo original para que el dibujo original coincida con el dibujo reparado.

## <a name="manifest-warnings"></a>Advertencias del manifiesto

### <a name="redundantattribution"></a>**redundantAttribution**

#### <a name="description-for-redundantattribution"></a>*Descripción de redundantAttribution*

La advertencia **redundantAttribution** se produce cuando el manifiesto contiene propiedades de objeto redundantes o en conflicto.

#### <a name="examples-for-redundantattribution"></a>*Ejemplos de redundantAttribution*

* El siguiente fragmento de JSON contiene dos o más objetos `unitProperties` con el mismo `name`.

    ```json
    "unitProperties": [
        {
            "unitName": "L1-100",
            "categoryName": "room.office"
        },
        {
            "unitName": "L1-101",
            "categoryName": "room.office"
        },
        {
            "unitName": "L1-101",
            "categoryName": "room.office"
        }
    ]
    ```

* En el siguiente fragmento de JSON, dos o más objetos `zoneProperties` tienen el mismo `name`.

    ```json
     "zoneProperties": [
        {
            "zoneName": "Assembly Area 1",
            "categoryName": "zone.assembly"
        },
        {
            "zoneName": "Assembly Area 2",
            "categoryName": "zone.assembly"
        },
        {
            "zoneName": "Assembly Area 2",
            "categoryName": "zone.assembly"
        }
    ```

#### <a name="how-to-fix-redundantattribution"></a>*Cómo corregir redundantAttribution*

Para corregir una advertencia **redundantAttribution*, quite las propiedades de objeto redundantes o en conflicto.

### <a name="manifestwarning"></a>**manifestWarning**

#### <a name="description-for-manifestwarning"></a>*Descripción de manifestWarning*

Se produce una advertencia **manifestWarning** cuando el manifiesto contiene objetos unitProperties o zoneProperties que no se usan durante la conversión.

#### <a name="examples-for-manifestwarning"></a>*Ejemplos de manifestWarning*

* El manifiesto contiene un objeto `unitProperties` con un elemento `unitName` que no tiene ninguna etiqueta coincidente en una capa `unitLabel`.

* El manifiesto contiene un objeto `zoneProperties` con un elemento `zoneName` que no tiene ninguna etiqueta coincidente en una capa `zoneLabel`.

#### <a name="how-to-fix-manifestwarning"></a>*Cómo corregir manifestWarning*

Para corregir una advertencia **manifestWarning**, quite el objeto `unitProperties` o `zoneProperties` sin usar del manifiesto, o agregue una etiqueta de unidad o zona al dibujo para que el objeto de propiedades se use durante la conversión.

## <a name="wall-warnings"></a>Advertencias de paredes

### <a name="walloutsidelevel"></a>**wallOutsideLevel**

#### <a name="description-for-walloutsidelevel"></a>*Descripción de wallOutsideLevel*

La advertencia **wallOutsideLevel** se produce cuando el dibujo contiene una geometría de pared fuera de los límites del esquema de un nivel.

#### <a name="example-for-walloutsidelevel"></a>*Ejemplo de wallOutsideLevel*

* La imagen siguiente muestra una pared interior, en rojo, fuera del límite del nivel amarillo.

    ![Ejemplo de pared interior fuera del límite del nivel](./media/drawing-conversion-error-codes/wall-outside-level.png)

* En la imagen siguiente se muestra una pared exterior, en rojo, fuera del límite del nivel amarillo.

    ![Ejemplo de pared exterior fuera del límite del nivel](./media/drawing-conversion-error-codes/wall-outside-level-exterior.png)

#### <a name="how-to-fix-walloutsidelevel"></a>*Cómo corregir wallOutsideLevel*

Para corregir una advertencia **wallOutsideLevel**, amplíe la geometría del nivel para incluir todas las paredes. O bien, modifique los límites de la pared para que quepan dentro del límite del nivel.

## <a name="unit-warnings"></a>Advertencias de unidades

### <a name="unitoutsidelevel"></a>**unitOutsideLevel**

#### <a name="description-for-unitoutsidelevel"></a>*Descripción de unitOutsideLevel*

Se produce una advertencia **unitOutsideLevel** cuando el dibujo contiene una geometría de unidad fuera de los límites del esquema del nivel.

#### <a name="example-for-unitoutsidelevel"></a>*Ejemplo de unitOutsideLevel*

 En la imagen siguiente, la geometría de la unidad, en rojo, supera los límites del límite del nivel amarillo.

 ![Ejemplo de unidad que supera el límite del nivel](./media/drawing-conversion-error-codes/unit-outside-level.png)

#### <a name="how-to-fix-unitoutsidelevel"></a>*Cómo corregir unitOutsideLevel*

Para corregir una advertencia **unitOutsideLevel**, amplíe el límite del nivel para incluir todas las unidades. O bien, modifique la geometría de la unidad para que quepa dentro del límite del nivel.

### <a name="partiallyoverlappingunit"></a>**partiallyOverlappingUnit**

#### <a name="description-for-partiallyoverlappingunit"></a>*Descripción de partiallyOverlappingUnit*

Se produce una advertencia **partiallyOverlappingUnit** cuando el dibujo contiene una geometría de unidad parcialmente superpuesta en otra geometría de unidad. El servicio de conversión descarta las unidades superpuestas.

#### <a name="example-scenarios-partiallyoverlappingunit"></a>*Escenarios de ejemplo con partiallyOverlappingUnit*

En la imagen siguiente, la unidad superpuesta se resalta en rojo. `UNIT110` y `HALLWAY` se descartan.

![Ejemplo de unidades superpuestas](./media/drawing-conversion-error-codes/partially-overlapping-unit.png)

#### <a name="how-to-fix-partiallyoverlappingunit"></a>*Cómo corregir partiallyOverlappingUnit*

Para corregir una advertencia **partiallyOverlappingUnit**, vuelva a dibujar cada unidad parcialmente superpuesta para que no se superponga a otras unidades.

## <a name="door-warnings"></a>Advertencias de puertas

### <a name="dooroutsidelevel"></a>**doorOutsideLevel**

#### <a name="description-for-dooroutsidelevel"></a>*Descripción de doorOutsideLevel*

Se produce una advertencia **doorOutsideLevel** cuando el dibujo contiene una geometría de puerta fuera de los límites de la geometría del nivel.

#### <a name="example-for-dooroutsidelevel"></a>*Ejemplo de doorOutsideLevel*

En la imagen siguiente, la geometría de la puerta, resaltada en rojo, se superpone con el límite del nivel amarillo.

![Ejemplo de una puerta que se superpone a un límite del nivel](./media/drawing-conversion-error-codes/door-outside-level.png)

#### <a name="how-to-fix-dooroutsidelevel"></a>*Cómo corregir doorOutsideLevel*

Para corregir una advertencia **doorOutsideLevel**, vuelva a dibujar la geometría de la puerta para que esté dentro de los límites del nivel.

## <a name="zone-warnings"></a>Advertencias de zonas

### <a name="zonewarning"></a>**zoneWarning**

#### <a name="description-for-zonewarning"></a>*Descripción de zoneWarning*

La advertencia **zoneWarning** se produce cuando una zona no contiene una etiqueta. El servicio de conversión descarta una zona sin etiqueta.

#### <a name="example-for-zonewarning"></a>*Ejemplo de zoneWarning*

En la imagen siguiente se muestra una zona que no contiene ninguna etiqueta.

![Ejemplo de zona que no contiene ninguna etiqueta](./media/drawing-conversion-error-codes/zone-warning.png)

#### <a name="how-to-fix-zonewarning"></a>*Cómo corregir zoneWarning*

Para corregir una advertencia **zoneWarning**, compruebe que cada zona tenga una sola etiqueta.

## <a name="label-warnings"></a>Advertencias de etiquetas

### <a name="labelwarning"></a>*labelWarning*

#### <a name="description-for-labelwarning"></a>*Descripción de labelWarning*

La advertencia **labelWarning** se produce cuando el dibujo contiene una característica de etiquetas ambigua o contradictoria.

Se produce una advertencia **labelWarning** debido a uno o más de los siguientes motivos:

* Una etiqueta de unidad no está en ninguna unidad.
* Una etiqueta de zona no está en ninguna zona.
* Una etiqueta de zona está dentro de dos o más zonas.

#### <a name="example-for-labelwarning"></a>*Ejemplo de labelWarning*

En la imagen siguiente se muestra una etiqueta que se encuentra dentro de dos zonas.

![Ejemplo de una etiqueta dentro de dos zonas ](./media/drawing-conversion-error-codes/label-warning.png)

#### <a name="how-to-fix-labelwarning"></a>*Cómo corregir labelWarning*

Para corregir una advertencia **labelWarning**, asegúrese de lo siguiente:

* Todas las etiquetas de unidad están dentro de unidades.
* Todas las etiquetas de zona están dentro de zonas.
* Todas las etiquetas de zona se encuentran en una sola zona.

## <a name="drawing-package-errors"></a>Errores de paquetes de dibujos

### <a name="invalidarchiveformat"></a>**invalidArchiveFormat**

#### <a name="description-for-invalidarchiveformat"></a>*Descripción de invalidArchiveFormat*

Se produce un error **invalidArchiveFormat** cuando el paquete de dibujos está en un formato de archivo no válido, como GZIP o 7-Zip. Solo se admite el formato de archivo ZIP.

También se producirá un error **invalidArchiveFormat** si el archivo ZIP está vacío.

#### <a name="how-to-fix-invalidarchiveformat"></a>*Cómo corregir invalidArchiveFormat*

Para corregir un error **invalidArchiveFormat**, compruebe lo siguiente:

* El nombre del archivo de almacenamiento termina en _.zip_.
* El archivo ZIP contiene datos.
* Puede abrir el archivo ZIP.

### <a name="invaliduserdata"></a>**invalidUserData**

#### <a name="description-for-invaliduserdata"></a>*Descripción de invalidUserData*

Se produce un error **invalidUserData** cuando el servicio de conversión no puede leer un objeto de datos de usuario del almacenamiento.

#### <a name="example-scenario-for-invaliduserdata"></a>*Escenario de ejemplo de invalidUserData*

Ha intentado cargar un paquete de dibujos con un parámetro `udid` incorrecto.

#### <a name="how-to-fix-invaliduserdata"></a>*Cómo corregir invalidUserData*

Para corregir un error **invalidUserData**, compruebe lo siguiente:

* Ha proporcionado un `udid` correcto para el paquete cargado.
* Azure Maps Creator se ha habilitado para la cuenta de Azure Maps que usó para cargar el paquete de dibujos.
* La solicitud de la API al servicio de conversión contiene la clave de suscripción a la cuenta de Azure Maps que usó para cargar el paquete de dibujos.

### <a name="dwgerror"></a>**dwgError**

#### <a name="description-for-dwgerror"></a>*Descripción de dwgError*

Se produce un error **dwgError** cuando el paquete de dibujos tiene un problema con uno o más archivos DWG en el archivo ZIP cargado.

El error **dwgError** se produce cuando el paquete de dibujos contiene un archivo DWG que no se puede abrir porque no es válido o está dañado.

* Un archivo DWG no es un dibujo en el formato de archivo DWG de AutoCAD válido.
* Un archivo DWG está dañado.
* Un archivo DWG aparece en el archivo _manifest.json_, pero no está en el archivo ZIP.

#### <a name="how-to-fix-dwgerror"></a>*Cómo corregir dwgError*

Para corregir un error **dwgError**, inspeccione el archivo _manifest.json_ para confirmar lo siguiente:

* Todos los archivos DWG del archivo ZIP son dibujos válidos en el formato DWG de AutoCAD; abra cada uno de ellos en AutoCAD. Quite o corrija todos los dibujos no válidos.
* La lista de archivos DWG en el archivo _manifest.json_ coincide con los archivos DWG del archivo ZIP.

## <a name="manifest-errors"></a>Errores del manifiesto

### <a name="invalidjsonformat"></a>**invalidJsonFormat**

#### <a name="description-for-invalidjsonformat"></a>Descripción de invalidJsonFormat

Se produce un error **invalidJsonFormat** cuando no se puede leer el archivo _manifest.json_.

No se puede leer el archivo _manifest.json_ debido a errores de formato o sintaxis JSON. Para obtener más información sobre el formato y la sintaxis JSON, consulte [Formato de intercambio de datos de notación de objetos JavaScript (JSON)](https://tools.ietf.org/html/rfc7159)

#### <a name="how-to-fix-invalidjsonformat"></a>*Cómo corregir invalidJsonFormat*

Para corregir un error **invalidJsonFormat**, use un linter de JSON para detectar y resolver errores de JSON.

### <a name="missingrequiredfield"></a>**missingRequiredField**

#### <a name="description-for-missingrequiredfield"></a>*Descripción de missingRequiredField*

Se produce un error **missingRequiredField** cuando el archivo _manifest.json_ no contiene datos obligatorios.

#### <a name="how-to-fix-missingrequiredfield"></a>*Cómo corregir missingRequiredField*

Para corregir un error **missingRequiredField**, compruebe que el manifiesto contiene todas las propiedades obligatorias. Para obtener una lista completa de los objetos del manifiesto obligatorios, consulte la [sección sobre el manifiesto en los requisitos del paquete de dibujos](drawing-requirements.md#manifest-file-requirements)  

### <a name="missingmanifest"></a>**missingManifest**

#### <a name="description-for-missingmanifest"></a>*Descripción de missingManifest*

El error **missingManifest** se produce cuando falta el archivo _manifest.json_ en el archivo ZIP.

El error **labelWarning** se produce debido a uno o más de los siguientes motivos:

* El nombre del archivo _manifest.json_ está mal escrito.
* Falta el archivo _manifest.json_.
* El archivo _manifest.json_ no está dentro del directorio raíz del archivo ZIP.

#### <a name="how-to-fix-missingmanifest"></a>*Cómo corregir missingManifest*

Para corregir un error **missingManifest**, confirme que el archivo tiene un archivo denominado _manifest.json_ en el nivel raíz del archivo ZIP.

### <a name="conflict"></a>**conflict**

#### <a name="description-for-conflict"></a>*Descripción de conflict*

El error **conflict** se produce cuando el archivo _manifest.json_ contiene información en conflicto.

#### <a name="example-scenario-for-conflict"></a>*Escenario de ejemplo de conflict*

El servicio de conversión devolverá un error **conflict** cuando se defina más de un nivel con el mismo ordinal de nivel. El siguiente fragmento de JSON muestra dos niveles definidos con el mismo ordinal.

```JSON
"buildingLevels":
{
    "levels": [
        {
            "levelName": "Ground",
            "ordinal": 0,
            "filename": "./Level_0.dwg"
        },
        {
            "levelName": "Parking",
            "ordinal": 0,
            "filename": "./Level_P.dwg"
        }
    ]
}
```

#### <a name="how-to-fix-conflict"></a>*Cómo corregir conflict*

Para corregir un error **conflict**, inspeccione el archivo _manifest.json_ y quite cualquier información en conflicto.

### <a name="invalidgeoreference"></a>**invalidGeoreference**

#### <a name="description-for-invalidgeoreference"></a>*Descripción de invalidGeoreference*

Se produce el error **invalidGeoreference** cuando un archivo _manifest.json_ contiene una referencia geográfica no válida.

El error **invalidGeoreference** se produce debido a uno o más de los siguientes motivos:

* El usuario está realizando una referencia geográfica de un valor de latitud o longitud que está fuera del intervalo.
* El usuario está realizando una referencia geográfica de un valor de rotación que está fuera del intervalo.

#### <a name="example-scenario-for-invalidgeoreference"></a>*Escenario de ejemplo de invalidGeoreference*

En el siguiente fragmento de JSON, la latitud está por encima del límite superior.

```json
"georeference"
{
    "lat": 88.0,
    "lon": -122.132600,
    "angle": 0
},
```

#### <a name="how-to-fix-invalidgeoreference"></a>*Cómo corregir invalidGeoreference*

Para corregir un error **invalidGeoreference**, compruebe que los valores de la referencia geográfica se encuentran dentro del intervalo.

>[!IMPORTANT]
>En GeoJSON, el orden de las coordenadas es longitud y latitud. Si no usa el orden correcto, puede que haga referencia accidentalmente a un valor de latitud o longitud que esté fuera del intervalo.

## <a name="wall-errors"></a>Errores de paredes

### <a name="wallerror"></a>**wallError**

#### <a name="description-for-wallerror"></a>*Descripción de wallError*

El error **wallError** se produce cuando el dibujo contiene un error al intentar crear una característica de pared.

#### <a name="example-scenario-for-wallerror"></a>*Escenario de ejemplo de wallError*

En la imagen siguiente se muestra una característica de pared que no se superpone a ninguna unidad.

![Ejemplo de característica de pared que no se superpone a ninguna unidad](./media/drawing-conversion-error-codes/wall-error.png)

#### <a name="how-to-fix-wallerror"></a>*Cómo corregir wallError*

Para corregir un error **wallError**, vuelva a dibujar la pared para que se superponga al menos a una unidad. O bien, cree una nueva unidad que se superponga a la pared.

## <a name="vertical-penetration-errors"></a>Errores de penetraciones verticales

### <a name="verticalpenetrationerror"></a>**verticalPenetrationError**

#### <a name="description-for-verticalpenetrationerror"></a>*Descripción de verticalPenetrationError*

El error **verticalPenetrationError** se produce cuando el dibujo contiene una característica de penetración vertical ambigua.

El error **verticalPenetrationError** se produce debido a uno o más de los siguientes motivos:

* El dibujo contiene un área de penetración vertical sin áreas de penetración vertical superpuestas en ningún nivel por encima o por debajo.
* El paquete de dibujos contiene un nivel con dos o más características de penetración vertical que se superponen a una única característica de penetración vertical en otro nivel directamente por encima o por debajo.

#### <a name="example-scenario-for-verticalpenetrationerror"></a>*Escenario de ejemplo de verticalPenetrationError*

En la imagen siguiente se muestra un área de penetración vertical sin áreas de penetración vertical superpuestas en los niveles por encima o por debajo.

![Ejemplo de penetración vertical 1](./media/drawing-conversion-error-codes/vrt-2.png)

En la imagen siguiente se muestra un área de penetración vertical que se superpone a más de un área de penetración vertical en un nivel adyacente.

![Ejemplo de penetración vertical 2](./media/drawing-conversion-error-codes/vrt-1.png)

#### <a name="how-to-fix-verticalpenetrationerror"></a>Cómo corregir verticalPenetrationError

Para corregir un error **verticalPenetrationError**, consulte cómo usar una característica de penetración vertical en el artículo [Requisitos de los paquetes de dibujos](drawing-requirements.md).

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Cómo usar el visualizador de errores de dibujo de Azure Maps](drawing-error-visualizer.md)

> [!div class="nextstepaction"]
> [Guía de paquetes de dibujos](drawing-package-guide.md)

> [!div class="nextstepaction"]
> [Uso de Creator para mapas interiores](creator-indoor-maps.md)