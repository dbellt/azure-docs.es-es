---
title: Trabajo con planos interiores en Azure Maps Creator
description: En este artículo se presentan los conceptos que se aplican a los servicios de Azure Maps Creator.
author: anastasia-ms
ms.author: v-stharr
ms.date: 05/26/2021
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.openlocfilehash: c3e6632f607ca9168f63538e4c2a6d96d6197d8a
ms.sourcegitcommit: c05e595b9f2dbe78e657fed2eb75c8fe511610e7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112032148"
---
# <a name="creator-for-indoor-maps"></a>Uso de Creator para planos interiores

En este artículo se presentan los conceptos y herramientas que se aplican a los servicios de Azure Maps Creator. Le recomendamos que lea este artículo antes de empezar a usar la API y el SDK de Azure Maps Creator.

Puede usar Creator para desarrollar aplicaciones con características de mapa basadas en datos de mapas de interiores. En este artículo se describe el proceso de carga, conversión, creación y uso de los datos del plano.  Normalmente, el flujo de trabajo lo completan dos roles diferentes con distintas áreas de experiencia y responsabilidad:

- Creador de mapas: responsable de la protección y preparación de los datos del mapa.
- Usuario de datos del mapa de Creator: aprovecha los datos del mapa del cliente en las aplicaciones.

En el siguiente diagrama se muestra todo el flujo de trabajo.

![Flujo de trabajo de datos del plano de Creator](./media/creator-indoor-maps/workflow.png)

## <a name="create-azure-maps-creator"></a>Creación de Azure Maps Creator

Para usar los servicios de Creator, debe crear Azure Maps Creator en una cuenta de Azure Maps. Para obtener información sobre cómo crear Azure Maps Creator en Azure Maps, consulte [Administración de Creator de Azure Maps](how-to-manage-creator.md).

## <a name="creator-authentication"></a>Autenticación de Creator

Creator hereda la configuración de Access Control (IAM) de Azure Maps. Todas las llamadas de API para el acceso a los datos se deben enviar con reglas de autenticación y autorización.

Los datos de uso de Creator se incorporan en los gráficos de uso de Azure Maps y en el registro de actividad. Para obtener más información, consulte [Administrar la autenticación en Azure Maps](./how-to-manage-authentication.md).

>[!Important]
>Recomendamos usar lo siguiente:
>
> - Azure Active Directory (Azure AD) en todas las soluciones que se han creado con una cuenta de Azure Maps mediante los servicios de Creator. Para obtener más información sobre Azure AD, consulte [Autenticación de Azure AD](azure-maps-authentication.md#azure-ad-authentication).
>
>- Configuración del control de acceso basado en roles. Con esta configuración, los creadores de mapas pueden actuar como si tuvieran el rol Colaborador de datos de Azure Maps, mientras que los usuarios de datos de mapas de Creator pueden actuar con el rol Lector de datos de Azure Maps. Para obtener más información, vea [Autorización con el control de acceso basado en rol](azure-maps-authentication.md#authorization-with-role-based-access-control).

## <a name="creator-data-item-types"></a>Tipos de elementos de datos de Creator

Los servicios de Creator crean, almacenan y usan varios tipos de datos que se definen y se analizan en las secciones siguientes. Un elemento de datos de Creator puede ser de los siguientes tipos:

- Datos convertidos
- Dataset
- Conjunto de mosaicos
- Conjunto de estados de características

## <a name="upload-a-drawing-package"></a>Carga de un paquete de dibujo

La instancia de Creator recopila datos de planos interiores mediante la conversión de un paquete de dibujo que esté cargado. El paquete de dibujo representa una instalación construida o remodelada. Para obtener información sobre los requisitos de los paquetes de dibujo, consulte el artículo [que trata sobre este tema](drawing-requirements.md).

Use la [API de carga de datos de Azure Maps](/rest/api/maps/data-v2/update-preview) para cargar un paquete de dibujo. Una vez cargado el paquete de dibujo, la API de carga de datos devuelve un identificador de datos de usuario (`udid`). A continuación, `udid` se puede usar para convertir el paquete cargado en datos de mapa de interiores.

## <a name="convert-a-drawing-package"></a>Conversión de un paquete de dibujo

El [servicio de conversión de Azure Maps](/rest/api/maps/v2/conversion) convierte un paquete de dibujo cargado en datos de un plano interior. El servicio de conversión también valida el paquete. Los problemas de validación se clasifican en dos tipos:

- Errores: si se detecta algún error, se produce un error en el proceso de conversión. Cuando se produce un error, el servicio de conversión proporciona un vínculo a la aplicación web independiente [Visualizador de errores de dibujo de Azure Maps](drawing-error-visualizer.md). Puede usar el Visualizador de errores de dibujo para inspeccionar los [errores y advertencias del paquete de dibujo](drawing-conversion-error-codes.md) que se produjeron durante el proceso de conversión. Después de corregir los errores, puede intentar cargar y convertir el paquete.
- Advertencias: si se detecta alguna advertencia, la conversión se realiza correctamente. Sin embargo, recomendamos que revise y resuelva todas las advertencias. Una advertencia significa que parte de la conversión se ignoró o corrigió automáticamente. Si no se resuelven las advertencias, podrían producirse errores en procesos posteriores.
Para obtener más información, consulte [Advertencias y errores del paquete de dibujo](drawing-conversion-error-codes.md).

## <a name="create-indoor-map-data"></a>Creación de datos de los planos interiores

Azure Maps Creator proporciona los siguientes servicios que admiten la creación de mapas:

- [Servicio de conjunto de datos](/rest/api/maps/v2/dataset).
- [Servicio de conjunto de mosaicos](/rest/api/maps/v2/tileset).
Use el servicio de conjunto de mosaicos para crear una representación basada en vectores de un conjunto de datos. Las aplicaciones pueden utilizar un conjunto de mosaicos para presentar una vista visual basada en mosaicos del conjunto de datos.
- [Servicio Feature State](/rest/api/maps/v2/feature-state). Use el servicio Feature State para admitir el estilo de mapa dinámico. Las aplicaciones pueden usar estilos de mapas dinámicos para reflejar eventos en tiempo real en espacios que proporciona el sistema de IoT.

### <a name="datasets"></a>Conjuntos de datos

Un conjunto de datos es una colección de características del plano interior. Las características de los mapas de interiores representan las instalaciones definidas en un paquete de dibujo convertido. Después de crear un conjunto de datos con el [servicio de conjunto de datos](/rest/api/maps/v2/dataset), puede crear cualquier número de [conjuntos de mosaicos](#tilesets) o [conjuntos de estados de características](#feature-statesets).

En cualquier momento, los desarrolladores pueden utilizar el [servicio de conjunto de datos](/rest/api/maps/v2/dataset) para agregar o eliminar instalaciones en un conjunto de datos existente. Para obtener más información sobre cómo actualizar un conjunto de datos existente mediante la API, consulte las opciones para anexar contenido en el [servicio de conjunto de datos](/rest/api/maps/v2/dataset). Para obtener un ejemplo de cómo actualizar un conjunto de datos, consulte [Mantenimiento de datos](#data-maintenance).

### <a name="tilesets"></a>Conjunto de mosaicos

Un conjunto de mosaicos es una colección de datos vectoriales que representa un conjunto de mosaicos de cuadrícula uniformes. Los desarrolladores pueden usar el [servicio de conjunto de mosaicos](/rest/api/maps/v2/tileset) para crear conjuntos de mosaicos desde un conjunto de datos.

Para reflejar diferentes fases de contenido, puede crear varios conjuntos de mosaicos desde el mismo conjunto de datos. Por ejemplo, puede crear un conjunto de mosaicos con mobiliario y equipamiento, y otro conjunto de mosaicos sin mobiliario ni equipamiento. Puede generar un conjunto de mosaicos con las actualizaciones de datos más recientes y otro sin las actualizaciones de datos más recientes.

Además de los datos vectoriales, el conjunto de mosaicos proporciona metadatos para la optimización de la representación de planos. Por ejemplo, los metadatos del conjunto de mosaicos contienen un nivel de zoom mínimo y máximo para ese conjunto de mosaicos. Asimismo, los metadatos proporcionan un cuadro de límite que define la extensión geográfica del conjunto de mosaicos. Una aplicación puede utilizar un cuadro de límite para establecer mediante programación el punto central correcto. Para obtener más información sobre los metadatos del conjunto de mosaicos, consulte [Tileset List API](/rest/api/maps/v2/tileset/list).

Una vez que se ha creado un conjunto de mosaicos, el [servicio Render V2](#render-v2-get-map-tile-api) puede recuperarlo.

Si un conjunto de mosaicos se queda obsoleto y ya no es útil, puede eliminar ese conjunto de mosaicos. Para obtener información sobre cómo eliminar conjuntos de mosaicos, consulte [Mantenimiento de datos](#data-maintenance).

>[!NOTE]
>Un conjunto de mosaicos es independiente del conjunto de datos del que se creó. Si crea conjuntos de mosaicos a partir de un conjunto de datos y, después, actualiza ese conjunto de datos, los conjuntos de mosaicos no se actualizarán. 
>
>Para reflejar los cambios en un conjunto de datos, debe crear nuevos conjuntos de mosaicos. Del mismo modo, si elimina un conjunto de mosaicos, el conjunto de datos no se verá afectado.

### <a name="feature-statesets"></a>Conjuntos de estados de características

Los conjuntos de estados de características son colecciones de propiedades dinámicas (*estados*) asignadas a las características del conjunto de datos, como salas o equipamiento. Un ejemplo de un *estado* podría ser la temperatura o la ocupación. Cada *estado* es un par clave-valor que contiene el nombre de la propiedad, el valor y la marca de tiempo de la última actualización.

Puede usar el [servicio de estado de característica](/rest/api/maps/v2/feature-state/create-stateset) para crear y administrar un conjunto de estados de características para un conjunto de datos. El conjunto de estados se define en uno o varios *estados*. Cada característica, como una sala, puede tener un *estado* adjunto.

El valor de cada *estado* en un conjunto de estados se puede actualizar o recuperar en dispositivos IoT u otras aplicaciones.  Por ejemplo, al usar [Feature State Update API](/rest/api/maps/v2/feature-state/update-states), los dispositivos que miden la ocupación de espacio pueden publicar sistemáticamente el cambio de estado de una habitación.

Una aplicación puede usar un conjunto de estados de características para representar dinámicamente características en una instalación según su estado actual y su estilo de plano respectivo. Para obtener más información sobre el uso de conjuntos de estados de características para dar estilo a las características en un mapa de representación, consulte el [módulo de Indoor Maps](#indoor-maps-module).

>[!NOTE]
>Al igual que sucede con los conjuntos de mosaicos, cambiar un conjunto de datos no afecta al conjunto de estados de características existente; igualmente, eliminar un conjunto de estados de características no afecta al conjunto de datos al que esté asociado.

## <a name="using-indoor-maps"></a>Uso de planos interiores

### <a name="render-v2-get-map-tile-api"></a>Get Map Tile API de Render V2

[Get Map Tile API de Render V2](/rest/api/maps/renderv2/getmaptilepreview) de Azure Maps se ha ampliado para admitir los conjuntos de mosaicos de Creator.

Las aplicaciones pueden usar Get Map Tile API de Render V2 para solicitar conjuntos de mosaicos. El conjunto de mosaicos se puede integrar en un control de plano o un SDK. Para obtener un ejemplo de un control de mapa que use el servicio Render V2, consulte el [módulo de Indoor Maps](#indoor-maps-module).

### <a name="web-feature-service-api"></a>API del servicio de características web

Puede usar la [API Web Feature Service (WFS)](/rest/api/maps/v2/wfs) para consultar conjuntos de datos. WFS sigue las [características de Open Geospatial Consortium API](http://docs.opengeospatial.org/DRAFTS/17-069r1.html). Puede usar la API WFS para consultar características en el propio conjunto de datos. Por ejemplo, puede usar WFS para buscar todas las salas de reuniones de tamaño medio de una instalación y una planta determinadas.

### <a name="alias-api"></a>Alias API

Los servicios de Creator, como la conversión, el conjunto de datos, el conjunto de mosaicos y el estado de características, devuelven un identificador para cada recurso que se crea a partir de las API. [Alias API](/rest/api/maps/v2/alias) le permite asignar un alias para hacer referencia a un identificador de recurso.

### <a name="indoor-maps-module"></a>Módulo de Indoor Maps

El [SDK web de Azure Maps](./index.yml) incluye el módulo Indoor Maps. Este módulo ofrece funcionalidades extendidas para la biblioteca del *control de mapa* de Azure Maps. El módulo Indoor Maps representa los planos interiores creados en Creator. Integra widgets, como el *selector de planta*, que ayuda a los usuarios a visualizar las distintas plantas.

Puede usar el módulo de Indoor Maps para crear aplicaciones web que integren datos de mapas de interiores con otros [servicios de Azure Maps](./index.yml). Las configuraciones de aplicaciones más comunes incluyen la adición de conocimiento a mapas de interiores desde otros mapas, como carreteras, imágenes, el tiempo y el tránsito.

El módulo Indoor Maps también admite el estilo dinámico del plano. Para ver un tutorial paso a paso sobre cómo implementar el estilo dinámico del conjunto de estados de características en una aplicación, consulte [Cómo usar el módulo de mapa de interiores](how-to-use-indoor-module.md).

### <a name="azure-maps-integration"></a>Integración de Azure Maps

Cuando empiece a desarrollar soluciones para planos interiores, puede descubrir diferentes maneras de integrar las capacidades de Azure Maps existentes. Por ejemplo, puede implementar escenarios de seguridad o de seguimiento de recursos mediante el uso de [Geofence API de Azure Maps](/rest/api/maps/spatial/postgeofence) con los mapas de interiores de Creator. Por ejemplo, puede usar Geofence API para determinar si un trabajador entra o sale de determinadas áreas interiores. Para obtener más información sobre cómo conectar Azure Maps con la telemetría de IoT, consulte [este tutorial de análisis espacial de IoT](tutorial-iot-hub-maps.md).

### <a name="data-maintenance"></a>Mantenimiento de datos

 Puede usar Creator List, Update, y Delete API de Azure Maps para enumerar, actualizar y eliminar conjuntos de datos, conjuntos de mosaicos y conjuntos de estados de características.

>[!NOTE]
>Cuando revise una lista de elementos para determinar si debe eliminarlos, considere el impacto que esa eliminación tendrá en todas las API o aplicaciones dependientes. Por ejemplo, si elimina un conjunto de mosaicos que usa una aplicación mediante [Get Map Tile API de Render V2](/rest/api/maps/renderv2/getmaptilepreview), la aplicación no podrá representar ese conjunto de mosaicos.

### <a name="example-updating-a-dataset"></a>Ejemplo: actualización de un conjunto de datos

En el ejemplo siguiente se muestra cómo actualizar un conjunto de datos, crear un nuevo conjunto de mosaicos y eliminar un conjunto de mosaicos antiguo:

1. Siga los pasos de las secciones [Carga de un paquete de dibujo](#upload-a-drawing-package) y [Conversión de un paquete de dibujo](#convert-a-drawing-package) para cargar y convertir el nuevo paquete de dibujo.
2. Use [Dataset Create API](/rest/api/maps/v2/dataset/create) para anexar los datos convertidos al conjunto de datos existente.
3. Use [Tileset Create API](/rest/api/maps/v2/tileset/create) para generar un nuevo conjunto de mosaicos fuera del conjunto de datos actualizado.
4. Guarde el nuevo valor de **tilesetId** para el paso siguiente.
5. Para habilitar la visualización del conjunto de datos actualizado del campus, actualice el identificador del conjunto de mosaicos en la aplicación. Si el antiguo conjunto de mosaicos ya no está en uso, puede eliminarlo.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Tutorial: creación de un plano interior de Creator](tutorial-creator-indoor-maps.md)
