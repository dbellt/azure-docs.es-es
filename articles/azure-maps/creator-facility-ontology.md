---
title: Ontología de instalaciones en Creator de Microsoft Azure Maps
description: Ontología de instalaciones que describe las definiciones de clases de características para Creator de Azure Maps
author: anastasia-ms
ms.author: v-stharr
ms.date: 06/14/2021
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: philMea
zone_pivot_groups: facility-ontology-schema
ms.openlocfilehash: 63e9702f8bebb449518002e18a824c65d81fab80
ms.sourcegitcommit: 3bb9f8cee51e3b9c711679b460ab7b7363a62e6b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112076463"
---
# <a name="facility-ontology"></a>Ontología de instalaciones

La ontología de instalaciones define cómo Creator de Azure Maps almacena internamente los datos de las instalaciones en un conjunto de datos de Creator.  Además de definir la estructura de datos interna de las instalaciones, la ontología de instalaciones también se expone externamente a través de la API de WFS. Cuando la API de WFS se usa para consultar los datos de las instalaciones de un conjunto de datos, el formato de respuesta se define mediante la ontología proporcionada a ese conjunto de datos.

En un nivel alto, la ontología de instalaciones divide el conjunto de datos en clases de características. Todas las clases de características comparten un conjunto común de propiedades, como `ID` y `Geometry` .  Además del conjunto de propiedades común, cada clase de característica define un conjunto de propiedades. Cada propiedad se define por su tipo de datos y restricciones. Algunas clases de características tienen propiedades que dependen de otras clases de características. Las propiedades dependientes se evalúan como `ID` de otra clase de característica.  

## <a name="changes-and-revisions"></a>Cambios y revisiones

:::zone pivot="facility-ontology-v1"

Facility 1.0 contiene revisiones para las definiciones de clase de característica de instalaciones para [Azure Maps Services](https://aka.ms/AzureMaps).

:::zone-end

:::zone pivot="facility-ontology-v2"

Facility 2.0 contiene revisiones para las definiciones de clase de característica de instalaciones para [Azure Maps Services](https://aka.ms/AzureMaps).

:::zone-end

### <a name="major-changes"></a>Cambios importantes

:::zone pivot="facility-ontology-v1"

Se han corregido las siguientes comprobaciones de validación de restricciones:

* Comprobación de validación de restricciones para la exclusividad de `isObstruction = true` *o* la presencia de `obstructionArea` para las clases de características `lineElement` y `areaElement`.

* Comprobación de validación de restricciones para la exclusividad de `isRoutable = true` *o* la presencia de `routeThroughBehavior` para la clases de características `category`.
:::zone-end

:::zone pivot="facility-ontology-v2"

* Se ha agregado una clase de característica de estructura para contener paredes, columnas, etc.
* Se han limpiado los atributos diseñados para enriquecer escenarios de enrutamiento. El motor de enrutamiento actual no los admite.

:::zone-end

## <a name="unit"></a>unit

La clase de característica `unit` define un área física y no superpuesta que un agente de navegación puede ocupar y recorrer. Un elemento `unit` puede ser un salón, una sala, un patio, etc.

**Tipo de geometría**: polígono

:::zone pivot="facility-ontology-v1"

| Propiedad           | Tipo                        | Requerido | Descripción                                                  |
|--------------------|-----------------------------|----------|--------------------------------------------------------------|
|`originalId`        | string     |true      | Identificador derivado de los datos de cliente. La longitud máxima permitida es de 1000 caracteres.|
|`externalId`        | string     |true      | Identificador que utiliza el cliente para asociar la característica a otra característica de un conjunto de datos diferente, como en una base de datos interna. La longitud máxima permitida es de 1000 caracteres.|
|`categoryId`        | [category.Id](#category)     |true      | Identificador de una característica [`category`](#category).|
|`isOpenArea`        | Booleano (el valor predeterminado es `null`).                    |false     | Representa si la unidad es un área abierta. Si se establece en `true`, las [estructuras](#structure) no rodean el límite de la unidad y un agente de navegación puede escribir el elemento `unit` sin necesidad de [`opening`](#opening). De forma predeterminada, las unidades están rodeadas de barreras físicas y solo están abiertas cuando se coloca una característica de apertura en el límite de la unidad. Si se necesitan paredes en una unidad de área abierta, se pueden representar como [`lineElement`](#lineelement) o [`areaElement`](#areaelement) con una propiedad `isObstruction` igual a `true`.|
|`navigableBy`        | enum ["pedestrian", "wheelchair", "machine", "bicycle", "automobile", "hiredAuto", "bus", "railcar", "emergency", "ferry", "boat"]  | false      |Indica los tipos de agentes de navegación que pueden atravesar la unidad. Si no se especifica, se supone que cualquier agente de navegación puede recorrer la unidad. |
|`isRoutable`        | Booleano (el valor predeterminado es `null`).                      | false    |  Determina si la unidad forma parte del grafo de enrutamiento. Si se establece en `true`, la unidad se puede usar como origen o destino o nodo intermedio en la experiencia de enrutamiento. |
|`routeThroughBehavior`        | enum ["disallowed", "allowed", "preferred"] |  false     | Determina si se permite navegar por la unidad. Si no se especifica, hereda su valor de la característica de categoría a la que se hace referencia en la propiedad `categoryId`. Si se especifica, invalida el valor determinado en su característica de categoría. |
|`nonPublic`        |  boolean| false       | Si es `true`, solo pueden navegar por la unidad los usuarios con privilegios.  El valor predeterminado es `false`. |
| `levelId`          | [level.Id](#level)        | true     | Identificador de una característica de nivel. |
|`occupants`         |  Matriz de [directoryInfo.Id](#directoryinfo) |    false |    Identificadores de las características [directoryInfo](#directoryinfo). Se usa para representar uno o varios ocupantes de la característica. |
|`addressId`         | [directoryInfo.Id](#directoryinfo) | true     | Identificador de una característica [directoryInfo](#directoryinfo). Se usa para representar la dirección de la característica.|
|`addressRoomNumber`         |  [directoryInfo.Id](#directoryinfo) | true     | Número de habitación/unidad/apartamento/suite de la unidad.|
|`name` |    string |    false |    Nombre de la característica en el idioma local. La longitud máxima permitida es de 1000 caracteres. |
|`nameSubtitle` |    string |    false |   Subtítulo que aparece debajo del elemento `name` de la característica. Se puede usar para mostrar el nombre en otro idioma, etc.  La longitud máxima permitida es de 1000 caracteres.|
|`nameAlt` |    string |    false |   Nombre alternativo que se usa para la característica. La longitud máxima permitida es de 1000 caracteres. |
|`anchorPoint` |   [Point](/rest/api/maps/wfs/get-feature-preview#featuregeojson) | false | [Geometría de punto GeoJSON](/rest/api/maps/wfs/get-feature-preview#featuregeojson) que representa la característica como un punto. Se puede usar para colocar la etiqueta de la característica.|

:::zone-end

:::zone pivot="facility-ontology-v2"

| Propiedad           | Tipo                        | Requerido | Descripción                                                  |
|--------------------|-----------------------------|----------|--------------------------------------------------------------|
|`originalId`        | string     |true      | Identificador derivado de los datos de cliente. La longitud máxima permitida es de 1000 caracteres.|
|`externalId`        | string     |true      | Identificador que utiliza el cliente para asociar la característica a otra característica de un conjunto de datos diferente, como en una base de datos interna. La longitud máxima permitida es de 1000 caracteres.|
|`categoryId`        | [category.Id](#category)     |true      | Identificador de una característica [`category`](#category).|
|`isOpenArea`        | Booleano (el valor predeterminado es `null`).                    |false     | Representa si la unidad es un área abierta. Si se establece en `true`, las [estructuras](#structure) no rodean el límite de la unidad y un agente de navegación puede escribir el elemento `unit` sin necesidad de [`opening`](#opening). De forma predeterminada, las unidades están rodeadas de barreras físicas y solo están abiertas cuando se coloca una característica de apertura en el límite de la unidad. Si se necesitan paredes en una unidad de área abierta, se pueden representar como [`lineElement`](#lineelement) o [`areaElement`](#areaelement) con una propiedad `isObstruction` igual a `true`.|
|`isRoutable`        | Booleano (el valor predeterminado es `null`).                     | false    |  Determina si la unidad forma parte del grafo de enrutamiento. Si se establece en `true`, la unidad se puede usar como origen o destino o nodo intermedio en la experiencia de enrutamiento. |
| `levelId`          | [level.Id](#level)        | true     | Identificador de una característica de nivel. |
|`occupants`         |  Matriz de [directoryInfo.Id](#directoryinfo) |    false |    Identificadores de las características [directoryInfo](#directoryinfo). Se usa para representar uno o varios ocupantes de la característica. |
|`addressId`         | [directoryInfo.Id](#directoryinfo) | true     | Identificador de una característica [directoryInfo](#directoryinfo). Se usa para representar la dirección de la característica.|
|`addressRoomNumber`         |  [directoryInfo.Id](#directoryinfo) | true     | Número de habitación/unidad/apartamento/suite de la unidad.|
|`name` |    string |    false |    Nombre de la característica en el idioma local.  La longitud máxima permitida es de 1000 caracteres.|
|`nameSubtitle` |    string |    false |   Subtítulo que aparece debajo del elemento `name` de la característica. Se puede usar para mostrar el nombre en otro idioma, etc.  La longitud máxima permitida es de 1000 caracteres.|
|`nameAlt` |    string |    false |   Nombre alternativo que se usa para la característica.  La longitud máxima permitida es de 1000 caracteres.|
|`anchorPoint` |   [Point](/rest/api/maps/wfs/get-feature-preview#featuregeojson) | false | [Geometría de punto GeoJSON](/rest/api/maps/wfs/get-feature-preview#featuregeojson) que representa la característica como un punto. Se puede usar para colocar la etiqueta de la característica.|

:::zone-end

:::zone pivot="facility-ontology-v2"

## <a name="structure"></a>structure

La clase de característica `structure` define un área física y no superpuesta en la que no se puede navegar. Puede ser una pared, una columna, etc.

**Tipo de geometría**: polígono

| Propiedad  | Tipo | Requerido | Descripción |
|-----------|------|----------|-------------|
|`originalId`        | string     |true      | Identificador derivado de los datos de cliente. La longitud máxima permitida es de 1000 caracteres.|
|`externalId`        | string     |true      | Identificador que utiliza el cliente para asociar la característica a otra característica de un conjunto de datos diferente, como en una base de datos interna. La longitud máxima permitida es de 1000 caracteres.|
|`categoryId`        | [category.Id](#category)      |true      | Identificador de una característica [`category`](#category).|
| `levelId`          |  [level.Id](#level)            | true     | Identificador de una característica [`level`](#level). |
|`name` |    string |    false |    Nombre de la característica en el idioma local. La longitud máxima permitida es de 1000 caracteres. |
|`nameSubtitle` |    string |    false |   Subtítulo que aparece debajo del elemento `name` de la característica. Se puede usar para mostrar el nombre en otro idioma, etc. La longitud máxima permitida es de 1000 caracteres. |
|`nameAlt` |    string |    false |   Nombre alternativo que se usa para la característica.  La longitud máxima permitida es de 1000 caracteres.|
|`anchorPoint` |   [Point](/rest/api/maps/wfs/get-feature-preview#featuregeojson) | false | [Geometría de punto GeoJSON](/rest/api/maps/wfs/get-feature-preview#featuregeojson) que representa la característica como un punto. Se puede usar para colocar la etiqueta de la característica.|

:::zone-end

## <a name="zone"></a>zona

La clase de característica `zone` define un área virtual, como una zona Wi-Fi o un área de ensamblado de emergencia. Las zonas se pueden usar como destinos, pero no están pensadas para el tráfico de paso.

**Tipo de geometría**: polígono

| Propiedad  | Tipo | Requerido | Descripción |
|-----------|------|----------|-------------|
|`originalId`        | string     |true      | Identificador derivado de los datos de cliente. La longitud máxima permitida es de 1000 caracteres.|
|`externalId`        | string     |true      | Identificador que utiliza el cliente para asociar la característica a otra característica de un conjunto de datos diferente, como en una base de datos interna. La longitud máxima permitida es de 1000 caracteres.|
|`categoryId`        | [category.Id](#category)      |true      | Identificador de una característica [`category`](#category).|
| `setId`          | string         | true     |Se requiere para las características de zona que representan zonas de varios niveles. `setId` es el identificador único de una zona que abarca varios niveles. `setId` permite que una zona con cobertura variable en diferentes plantas se represente con geometría diferente en distintos niveles. `setId` puede ser cualquier cadena y distingue mayúsculas de minúsculas. Se recomienda que `setId` sea un GUID.  La longitud máxima permitida es de 1000 caracteres.|
| `levelId`          |  [level.Id](#level)             | true     | Identificador de una característica [`level`](#level). |
|`name` |    string |    false |    Nombre de la característica en el idioma local.  La longitud máxima permitida es de 1000 caracteres.|
|`nameSubtitle` |    string |    false |   Subtítulo que aparece debajo del elemento `name` de la característica. Se puede usar para mostrar el nombre en otro idioma, etc.  La longitud máxima permitida es de 1000 caracteres.|
|`nameAlt` |    string |    false |   Nombre alternativo que se usa para la característica. La longitud máxima permitida es de 1000 caracteres. |
|`anchorPoint` |  [Point](/rest/api/maps/wfs/get-feature-preview#featuregeojson) | false | [Geometría de punto GeoJSON](/rest/api/maps/wfs/get-feature-preview#featuregeojson) que representa la característica como un punto. Se puede usar para colocar la etiqueta de la característica.|

## <a name="level"></a>Nivel

La característica de clase `level` define un área de un edificio en una elevación establecida. Por ejemplo, la planta de un edificio, que contiene un conjunto de características, como [`units`](#unit).  

**Tipo de geometría**: MultiPolygon

| Propiedad  | Tipo | Requerido | Descripción |
|-----------|------|----------|-------------|
|`originalId`        | string     |true      | Identificador derivado de los datos de cliente. La longitud máxima permitida es de 1000 caracteres.|
|`externalId`        | string     |true      | Identificador que utiliza el cliente para asociar la característica a otra característica de un conjunto de datos diferente, como en una base de datos interna. La longitud máxima permitida es de 1000 caracteres.|
|`categoryId`        | [category.Id](#category)    |true      | Identificador de una característica [`category`](#category).|
| `ordinal`          | integer        | true     | El número de nivel. Lo usa la característica [`verticalPenetration`](#verticalpenetration) para determinar el orden relativo de las plantas a fin de proporcionar ayuda con la dirección de desplazamiento. La práctica común es comenzar con 0 para la planta baja. Agregue +1 para cada planta superior y -1 para cada planta inferior. Se puede modelar con cualquier número, siempre y cuando los suelos físicos superiores estén representados por valores ordinales superiores. |
| `abbreviatedName`          | string        | false     | Nombre de nivel abreviado de cuatro caracteres, como el que se encontraría en un botón de ascensor.  La longitud máxima permitida es de 1000 caracteres.|
| `heightAboveFacilityAnchor`          | double         | false     | Distancia vertical de la planta del nivel por encima de [`facility.anchorHeightAboveSeaLevel`](#facility), en metros. |
| `verticalExtent`          | double         | false     | Extensión vertical del nivel, en metros. Si no se proporciona, el valor predeterminado es [`facility.defaultLevelVerticalExtent`](#facility).|
|`name` |    string |    false |    Nombre de la característica en el idioma local.  La longitud máxima permitida es de 1000 caracteres.|
|`nameSubtitle` |    string |    false |   Subtítulo que aparece debajo del elemento `name` de la característica. Se puede usar para mostrar el nombre en otro idioma, etc.  La longitud máxima permitida es de 1000 caracteres.|
|`nameAlt` |    string |    false |   Nombre alternativo que se usa para la característica.  La longitud máxima permitida es de 1000 caracteres.|
|`anchorPoint` |   [Point](/rest/api/maps/wfs/get-feature-preview#featuregeojson) | false | [Geometría de punto GeoJSON](/rest/api/maps/wfs/get-feature-preview#featuregeojson) que representa la característica como un punto. Se puede usar para colocar la etiqueta de la característica.|

## <a name="facility"></a>dispositivo

La clase de característica `facility` define el área del sitio, la superficie del edificio, etc.

**Tipo de geometría**: MultiPolygon

| Propiedad  | Tipo | Requerido | Descripción |
|-----------|------|----------|-------------|
|`originalId`        | string     |true      | Identificador derivado de los datos de cliente. La longitud máxima permitida es de 1000 caracteres.|
|`externalId`        | string     |true      | Identificador que utiliza el cliente para asociar la característica a otra característica de un conjunto de datos diferente, como en una base de datos interna. La longitud máxima permitida es de 1000 caracteres.|
|`categoryId`        | [category.Id](#category)      |true      | Identificador de una característica [`category`](#category).|
|`occupants`         | Matriz de [directoryInfo.Id](#directoryinfo) |    false |    Identificadores de las características [directoryInfo](#directoryinfo). Se usa para representar uno o varios ocupantes de la característica. |
|`addressId`         | [directoryInfo.Id](#directoryinfo)  | true     | Identificador de una característica [directoryInfo](#directoryinfo). Se usa para representar la dirección de la característica.|
|`addressRoomNumber`         |  [directoryInfo.Id](#directoryinfo)| true     | Número de habitación/unidad/apartamento/suite de la unidad.|
|`name` |    string |    false |    Nombre de la característica en el idioma local. La longitud máxima permitida es de 1000 caracteres. |
|`nameSubtitle` |    string |    false |   Subtítulo que aparece debajo del elemento `name` de la característica. Se puede usar para mostrar el nombre en otro idioma, etc. La longitud máxima permitida es de 1000 caracteres. |
|`nameAlt` |    string |    false |   Nombre alternativo que se usa para la característica.  La longitud máxima permitida es de 1000 caracteres.|
|`anchorPoint` |  [Point](/rest/api/maps/wfs/get-feature-preview#featuregeojson) | false | [Geometría de punto GeoJSON](/rest/api/maps/wfs/get-feature-preview#featuregeojson) que representa la característica como un punto. Se puede usar para colocar la etiqueta de la característica.|
|`anchorHeightAboveSeaLevel` |  double | false | Alto del punto de anclaje sobre el nivel del mar, en metros. El nivel del mar se define mediante EGM 2008.|
|`defaultLevelVerticalExtent` |  double| false | Valor predeterminado para la extensión vertical de niveles, en metros.|

## <a name="verticalpenetration"></a>verticalPenetration

La característica de clase `verticalPenetration` define un área que, cuando se usa en un conjunto, representa un método para navegar verticalmente entre niveles. Se puede usar para modelar las escaleras, los ascensores, etc. La geometría puede superponer unidades y otras características de penetración vertical.

**Tipo de geometría**: polígono

:::zone pivot="facility-ontology-v1"

| Propiedad  | Tipo | Requerido | Descripción |
|-----------|------|----------|-------------|
|`originalId`        | string     |true      | Identificador derivado de los datos de cliente. La longitud máxima permitida es de 1000 caracteres.|
|`externalId`        | string     |true      | Identificador que utiliza el cliente para asociar la característica a otra característica de un conjunto de datos diferente, como en una base de datos interna. La longitud máxima permitida es de 1000 caracteres.|
|`categoryId`        | [category.Id](#category)      |true      | Identificador de una característica [`category`](#category).|
| `setId`          | string       | true     | Las características de penetración vertical deben usarse en conjuntos para conectar varios niveles. Las características de penetración vertical del mismo conjunto se consideran iguales. `setId` puede ser cualquier cadena y distingue mayúsculas de minúsculas. Se recomienda usar un GUID como `setId`.  La longitud máxima permitida es de 1000 caracteres.|
| `levelId`          | [level.Id](#level)         | true     | Identificador de una característica de nivel. |
|`direction`         |  string enum [ "both", "lowToHigh", "highToLow", "closed" ]| false     | Dirección de desplazamiento permitida en esta característica. El atributo ordinal de la característica [`level`](#level) se usa para determinar el orden bajo y alto.|
|`navigableBy`        | enum ["pedestrian", "wheelchair", "machine", "bicycle", "automobile", "hiredAuto", "bus", "railcar", "emergency", "ferry", "boat"]  | false      |Indica los tipos de agentes de navegación que pueden atravesar la unidad. Si no se especifica, cualquier agente de navegación puede recorrer la unidad. |
|`nonPublic`        |  boolean| false       | Si es `true`, solo pueden navegar por la unidad los usuarios con privilegios.  El valor predeterminado es `false`. |
|`name` |    string |    false |    Nombre de la característica en el idioma local.  La longitud máxima permitida es de 1000 caracteres.|
|`nameSubtitle` |    string |    false |   Subtítulo que aparece debajo del elemento `name` de la característica. Se puede usar para mostrar el nombre en otro idioma, etc.  La longitud máxima permitida es de 1000 caracteres.|
|`nameAlt` |    string |    false |   Nombre alternativo que se usa para la característica. La longitud máxima permitida es de 1000 caracteres. |
|`anchorPoint` |  [Point](/rest/api/maps/wfs/get-feature-preview#featuregeojson) | false | [Geometría de punto GeoJSON](/rest/api/maps/wfs/get-feature-preview#featuregeojson) que representa la característica como un punto. Se puede usar para colocar la etiqueta de la característica.|

:::zone-end

:::zone pivot="facility-ontology-v2"

| Propiedad  | Tipo | Requerido | Descripción |
|-----------|------|----------|-------------|
|`originalId`        | string     |true      | Identificador derivado de los datos de cliente. La longitud máxima permitida es de 1000 caracteres.|
|`externalId`        | string     |true      | Identificador que utiliza el cliente para asociar la característica a otra característica de un conjunto de datos diferente, como en una base de datos interna. La longitud máxima permitida es de 1000 caracteres.|
|`categoryId`        | [category.Id](#category)      |true      | Identificador de una característica [`category`](#category).|
| `setId`          | string       | true     | Las características de penetración vertical deben usarse en conjuntos para conectar varios niveles. Las características de penetración vertical del mismo conjunto están conectadas. `setId` puede ser cualquier cadena y distingue mayúsculas de minúsculas. Se recomienda usar un GUID como `setId`. La longitud máxima permitida es de 1000 caracteres. |
| `levelId`          | [level.Id](#level)         | true     | Identificador de una característica de nivel. |
|`direction`         |  string enum [ "both", "lowToHigh", "highToLow", "closed" ]| false     | Dirección de desplazamiento permitida en esta característica. El atributo ordinal de la característica [`level`](#level) se usa para determinar el orden bajo y alto.|
|`name` |    string |    false |    Nombre de la característica en el idioma local.  La longitud máxima permitida es de 1000 caracteres.|
|`nameSubtitle` |    string |    false |   Subtítulo que aparece debajo del elemento `name` de la característica. Se puede usar para mostrar el nombre en otro idioma, etc.  La longitud máxima permitida es de 1000 caracteres.|
|`nameAlt` |    string |    false |   Nombre alternativo que se usa para la característica. La longitud máxima permitida es de 1000 caracteres. |
|`anchorPoint` |  [Point](/rest/api/maps/wfs/get-feature-preview#featuregeojson) | false | [Geometría de punto GeoJSON](/rest/api/maps/wfs/get-feature-preview#featuregeojson) que representa la característica como un punto. Se puede usar para colocar la etiqueta de la característica.|

:::zone-end

## <a name="opening"></a>Abriendo

La característica de clase `opening` define un límite que se puede recorrer entre dos unidades o `unit` y `verticalPenetration`.

**Tipo de geometría:** LineString

:::zone pivot="facility-ontology-v1"

| Propiedad  | Tipo | Requerido | Descripción |
|-----------|------|----------|-------------|
|`originalId`        | string     |true      | Identificador derivado de los datos de cliente. La longitud máxima permitida es de 1000 caracteres.|
|`externalId`        | string     |true      | Identificador que utiliza el cliente para asociar la característica a otra característica de un conjunto de datos diferente, como en una base de datos interna. La longitud máxima permitida es de 1000 caracteres.|
|`categoryId`        |[category.Id](#category)     |true      | Identificador de una característica de categoría.|
| `levelId`          | [level.Id](#level)        | true     | Identificador de una característica de nivel. |
| `isConnectedToVerticalPenetration` | boolean | false | Indica si esta característica está conectada o no a una característica `verticalPenetration` en uno de sus lados. El valor predeterminado es `false`. |
|`navigableBy`        | enum ["pedestrian", "wheelchair", "machine", "bicycle", "automobile", "hiredAuto", "bus", "railcar", "emergency", "ferry", "boat"]  | false      |Indica los tipos de agentes de navegación que pueden atravesar la unidad. Si no se especifica, cualquier agente de navegación puede recorrer la unidad. |
| `accessRightToLeft`| enum [ "prohibited", "digitalKey", "physicalKey", "keyPad", "guard", "ticket", "fingerprint", "retina", "voice", "face", "palm", "iris", "signature", "handGeometry", "time", "ticketChecker", "other"] | false | Método de acceso al pasar por la apertura de derecha a izquierda. Los vértices de la geometría de la característica determinan la izquierda y la derecha, que se encuentran en el primer vértice y miran al segundo. Omitir esta propiedad significa que no hay restricciones de acceso.|
| `accessLeftToRight`| enum [ "prohibited", "digitalKey", "physicalKey", "keyPad", "guard", "ticket", "fingerprint", "retina", "voice", "face", "palm", "iris", "signature", "handGeometry", "time", "ticketChecker", "other"] | false | Método de acceso al pasar por la apertura de izquierda a derecha. Los vértices de la geometría de la característica determinan la izquierda y la derecha, que se encuentran en el primer vértice y miran al segundo. Omitir esta propiedad significa que no hay restricciones de acceso.|
| `isEmergency` | boolean | false | Si es `true`, la apertura solo se puede usar durante las emergencias. El valor predeterminado es `false`. |
|`anchorPoint` | [Point](/rest/api/maps/wfs/get-feature-preview#featuregeojson) | false | [Geometría de punto GeoJSON](/rest/api/maps/wfs/get-feature-preview#featuregeojson) y que representa la característica como un punto. Se puede usar para colocar la etiqueta de la característica.|

:::zone-end

:::zone pivot="facility-ontology-v2"

| Propiedad  | Tipo | Requerido | Descripción |
|-----------|------|----------|-------------|
|`originalId`        | string     |true      | Identificador derivado de los datos de cliente. La longitud máxima permitida es de 1000 caracteres.|
|`externalId`        | string     |true      | Identificador que utiliza el cliente para asociar la característica a otra característica de un conjunto de datos diferente, como en una base de datos interna. La longitud máxima permitida es de 1000 caracteres.|
|`categoryId`        |[category.Id](#category)     |true      | Identificador de una característica de categoría.|
| `levelId`          | [level.Id](#level)        | true     | Identificador de una característica de nivel. |
|`anchorPoint` | [Point](/rest/api/maps/wfs/get-feature-preview#featuregeojson) | false | [Geometría de punto GeoJSON](/rest/api/maps/wfs/get-feature-preview#featuregeojson) y que representa la característica como un punto. Se puede usar para colocar la etiqueta de la característica.|

:::zone-end

## <a name="directoryinfo"></a>directoryInfo

La característica de clase de objeto `directoryInfo` define el nombre, la dirección, el número de teléfono, el sitio web y las horas de funcionamiento de una unidad, una instalación o un ocupante de una unidad o instalación.

**Tipo de geometría**: ninguno

| Propiedad  | Tipo | Requerido | Descripción |
|-----------|------|----------|-------------|
|`originalId`        | string     |true      | Identificador derivado de los datos de cliente. La longitud máxima permitida es de 1000 caracteres.|
|`externalId`        | string     |true      | Identificador que utiliza el cliente para asociar la característica a otra característica de un conjunto de datos diferente, como en una base de datos interna. La longitud máxima permitida es de 1000 caracteres.|
|`streetAddress`        |string    |false    |Parte de la dirección postal de la dirección.  La longitud máxima permitida es de 1000 caracteres. |
|`unit`        |string    |false    |Parte del número de unidad de la dirección.  La longitud máxima permitida es de 1000 caracteres. |
|`locality`|    string|    false    |Localidad de la dirección. Por ejemplo: ciudad, localidad, pueblo. La longitud máxima permitida es de 1000 caracteres.|
|`adminDivisions`|    string|    false    |Parte de la división administrativa de la dirección, de menor a mayor (condado, estado, país). Por ejemplo: ["King", "Washington", "USA" ] or ["West Godavari", "Andhra Pradesh", "IND" ]. La longitud máxima permitida es de 1000 caracteres.|
|`postalCode`|    string |    false    |Parte del código postal de la dirección. La longitud máxima permitida es de 1000 caracteres.|
|`name` |    string |    false |    Nombre de la característica en el idioma local.  La longitud máxima permitida es de 1000 caracteres.|
|`nameSubtitle` |    string |    false |   Subtítulo que aparece debajo del elemento `name` de la característica. Se puede usar para mostrar el nombre en otro idioma, etc. La longitud máxima permitida es de 1000 caracteres. |
|`nameAlt` |    string |    false |   Nombre alternativo que se usa para la característica. La longitud máxima permitida es de 1000 caracteres. |
|`phoneNumber` |    string |    false |    Número de teléfono. |
|`website` |    string |    false |  URL del sitio web. La longitud máxima permitida es de 1000 caracteres. |
|`hoursOfOperation` |    string |    false |   Horas de funcionamiento como texto, siguiendo la [especificación Open Street Map](https://wiki.openstreetmap.org/wiki/Key:opening_hours/specification). La longitud máxima permitida es de 1000 caracteres. |

## <a name="pointelement"></a>pointElement

`pointElement` es una característica de clase que define una característica de punto en una unidad, como un kit de primeros auxilios o un aspersor.

**Tipo de geometría**: MultiPolygon

| Propiedad  | Tipo | Requerido | Descripción |
|-----------|------|----------|-------------|
|`originalId`        | string     |true      | Identificador derivado de los datos de cliente. La longitud máxima permitida es de 1000 caracteres.|
|`externalId`        | string     |true      | Identificador que utiliza el cliente para asociar la característica a otra característica de un conjunto de datos diferente, como en una base de datos interna. La longitud máxima permitida es de 1000 caracteres.|
|`categoryId`        |[category.Id](#category)      |true      | Identificador de una característica [`category`](#category).|
| `unitId`          | string     | true     | Identificador de una característica [`unit`](#unit) que contiene esta característica.  La longitud máxima permitida es de 1000 caracteres.|
| `isObstruction`          | Booleano (el valor predeterminado es `null`). | false     | Si es `true`, esta característica representa un obstáculo que se debe evitar al realizar el enrutamiento a través de la característica de la unidad contenedora. |
|`name` |    string |    false |    Nombre de la característica en el idioma local.  La longitud máxima permitida es de 1000 caracteres.|
|`nameSubtitle` |    string |    false |   Subtítulo que aparece debajo del elemento `name` de la característica. Se puede usar para mostrar el nombre en otro idioma, etc. La longitud máxima permitida es de 1000 caracteres. |
|`nameAlt` |    string |    false |   Nombre alternativo que se usa para la característica.  La longitud máxima permitida es de 1000 caracteres.|

## <a name="lineelement"></a>lineElement

`lineElement` es una característica de clase que define una característica de línea en una unidad, como una pared o una ventana divisora.

**Tipo de geometría**: LinearMultiString

| Propiedad  | Tipo | Requerido | Descripción |
|-----------|------|----------|-------------|
|`originalId`        | string     |true      | Identificador derivado de los datos de cliente. La longitud máxima permitida es de 1000 caracteres.|
|`externalId`        | string     |true      | Identificador que utiliza el cliente para asociar la característica a otra característica de un conjunto de datos diferente, como en una base de datos interna. La longitud máxima permitida es de 1000 caracteres.|
|`categoryId`        |[category.Id](#category)      |true      | Identificador de una característica [`category`](#category).|
| `unitId`          | string     | true     | Identificador de una característica [`unit`](#unit) que contiene esta característica. La longitud máxima permitida es de 1000 caracteres. |
| `isObstruction`          | Booleano (el valor predeterminado es `null`).| false     | Si es `true`, esta característica representa un obstáculo que se debe evitar al realizar el enrutamiento a través de la característica de la unidad contenedora. |
|`name` |    string |    false |    Nombre de la característica en el idioma local. La longitud máxima permitida es de 1000 caracteres. |
|`nameSubtitle` |    string |    false |   Subtítulo que aparece debajo del elemento `name` de la característica. Se puede usar para mostrar el nombre en otro idioma, etc. La longitud máxima permitida es de 1000 caracteres. |
|`nameAlt` |    string |    false |   Nombre alternativo que se usa para la característica. La longitud máxima permitida es de 1000 caracteres. |
|`anchorPoint` |  [Point](/rest/api/maps/wfs/get-feature-preview#featuregeojson) | false | [Geometría de punto GeoJSON](/rest/api/maps/wfs/get-feature-preview#featuregeojson) que representa la característica como un punto. Se puede usar para colocar la etiqueta de la característica.|
|`obstructionArea` |   [Polygon](/rest/api/maps/wfs/get-feature-preview#featuregeojson)| false | Geometría simplificada (cuando la geometría de línea es complicada) de la característica que se debe evitar durante el enrutamiento. Requiere que `isObstruction` esté establecido en true.|

## <a name="areaelement"></a>areaElement

`areaElement` es una característica de clase que define una característica de polígono en una unidad, como un área abierta hacia abajo, o un obstáculo, como una isla en una unidad.

**Tipo de geometría**: MultiPolygon

| Propiedad  | Tipo | Requerido | Descripción |
|-----------|------|----------|-------------|
|`originalId`        | string     |true      | Identificador derivado de los datos de cliente. La longitud máxima permitida es de 1000 caracteres.|
|`externalId`        | string     |true      | Identificador que utiliza el cliente para asociar la característica a otra característica de un conjunto de datos diferente, como en una base de datos interna. La longitud máxima permitida es de 1000 caracteres.|
|`categoryId`        |[category.Id](#category)      |true      | Identificador de una característica [`category`](#category).|
| `unitId`          | string     | true     | Identificador de una característica [`unit`](#unit) que contiene esta característica. La longitud máxima permitida es de 1000 caracteres. |
| `isObstruction`          | boolean | false     | Si es `true`, esta característica representa un obstáculo que se debe evitar al realizar el enrutamiento a través de la característica de la unidad contenedora. |
|`obstructionArea` |  Geometría: ["Polygon","MultiPolygon"]| false | Geometría simplificada (cuando la geometría de línea es complicada) de la característica que se debe evitar durante el enrutamiento. Requiere que `isObstruction` esté establecido en true.|
|`name` |    string |    false |    Nombre de la característica en el idioma local. La longitud máxima permitida es de 1000 caracteres. |
|`nameSubtitle` |    string |    false |   Subtítulo que aparece debajo del elemento `name` de la característica. Se puede usar para mostrar el nombre en otro idioma, etc.  La longitud máxima permitida es de 1000 caracteres.|
|`nameAlt` |    string |    false |   Nombre alternativo que se usa para la característica.  La longitud máxima permitida es de 1000 caracteres.|
|`anchorPoint` |  [Point](/rest/api/maps/wfs/get-feature-preview#featuregeojson) | false | [Geometría de punto GeoJSON](/rest/api/maps/wfs/get-feature-preview#featuregeojson) que representa la característica como un punto. Se puede usar para colocar la etiqueta de la característica.|

## <a name="category"></a>category

La característica de clase `category` define los nombres de categoría. Por ejemplo: "room.conference".

**Tipo de geometría**: ninguno

:::zone pivot="facility-ontology-v1"

| Propiedad  | Tipo | Requerido | Descripción |
|-----------|------|----------|-------------|
|`originalId`        | string     |true      | Identificador original de la categoría derivado de los datos del cliente. La longitud máxima permitida es de 1000 caracteres.|
|`externalId`        | string     |true      | Identificador que utiliza el cliente para asociar la categoría a otra categoría de un conjunto de datos diferente, como en una base de datos interna. La longitud máxima permitida es de 1000 caracteres.|
|`name` |    string |    true |   Nombre de la categoría Se recomienda usar "." para representar la jerarquía de categorías. Por ejemplo: "room.conference", "room.privateoffice". La longitud máxima permitida es de 1000 caracteres. |
| `routeThroughBehavior` | boolean | false | Determina si se puede usar una característica para el tráfico de paso.|
|`isRoutable`        | Booleano (el valor predeterminado es `null`).                  | false    |  Determina si una característica debe formar parte del grafo de enrutamiento. Si se establece en `true`, la unidad se puede usar como origen o destino o nodo intermedio en la experiencia de enrutamiento. |

:::zone-end

:::zone pivot="facility-ontology-v2"

| Propiedad  | Tipo | Requerido | Descripción |
|-----------|------|----------|-------------|
|`originalId`        | string     |true      | Identificador original de la categoría derivado de los datos del cliente.  La longitud máxima permitida es de 1000 caracteres.|
|`externalId`        | string     |true      | Identificador que utiliza el cliente para asociar la categoría a otra categoría de un conjunto de datos diferente, como en una base de datos interna. La longitud máxima permitida es de 1000 caracteres.|
|`name` |    string |    true |   Nombre de la categoría Se recomienda usar "." para representar la jerarquía de categorías. Por ejemplo: "room.conference", "room.privateoffice". La longitud máxima permitida es de 1000 caracteres. |

:::zone-end
