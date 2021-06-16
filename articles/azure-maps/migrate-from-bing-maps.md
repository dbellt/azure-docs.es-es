---
title: 'Tutorial: Migración de Mapas de Bing a Azure Maps | Microsoft Azure Maps'
description: Tutorial sobre cómo migrar de Mapas de Bing a Microsoft Azure Maps. Este documento le guiará por los pasos para cambiar a las API y los SDK de Azure Maps.
author: rbrundritt
ms.author: richbrun
ms.date: 12/17/2020
ms.topic: tutorial
ms.service: azure-maps
services: azure-maps
manager: cpendle
ms.custom: ''
ms.openlocfilehash: 4be7bc555578c4abeaa65c6433da3e285b9235b2
ms.sourcegitcommit: 7f59e3b79a12395d37d569c250285a15df7a1077
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/02/2021
ms.locfileid: "110786572"
---
# <a name="tutorial-migrate-from-bing-maps-to-azure-maps"></a>Tutorial: Migración de Mapas de Bing a Azure Maps

En este tutorial se proporciona información sobre cómo migrar aplicaciones web, móviles y basadas en servidor de Mapas de Bing a la plataforma Azure Maps. El tutorial incluye ejemplos de código comparativos, sugerencias de migración y procedimientos recomendados para migrar a Azure Maps. 

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Hacer una comparación de alto nivel para características equivalentes de Bing Maps disponibles en Azure Maps.
> * Identificar las diferencias en las licencias que se deben tener en cuenta.
> * Planear la migración.
> * Encontrar recursos técnicos y soporte técnico

## <a name="prerequisites"></a>Requisitos previos

1. Inicie sesión en [Azure Portal](https://portal.azure.com). Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.
2. [Cree una cuenta de Azure Maps](quick-demo-map-app.md#create-an-azure-maps-account).
3. [Obtenga una clave de suscripción principal](quick-demo-map-app.md#get-the-primary-key-for-your-account), también conocida como clave principal o clave de suscripción. Para más información sobre la autenticación en Azure Maps, consulte [Administración de la autenticación en Azure Maps](how-to-manage-authentication.md).

## <a name="azure-maps-platform-overview"></a>Introducción a la plataforma Azure Maps

Azure Maps ofrece a desarrolladores de todos los sectores una potente funcionalidad geoespacial, con los datos de asignación más recientes disponibles a fin de proporcionar contexto geográfico para las aplicaciones web y móviles. Azure Maps es un conjunto de API REST compatible con Azure One API para mapas, búsqueda, rutas, tráfico, zonas horarias, geolocalización, geovalla, datos de mapas, datos meteorológicos y muchos otros servicios, junto con SDK web y de Android, para facilitar el desarrollo, la flexibilidad y la portabilidad entre varias plataformas. [Azure Maps también está disponible en Power BI](power-bi-visual-getting-started.md).

## <a name="high-level-platform-comparison"></a>Comparación general de las plataformas

En la tabla siguiente se proporciona una lista general de las características de Mapas de Bing y la compatibilidad relativa con esas características en Azure Maps. Esta lista no incluye características adicionales de Azure Maps como la accesibilidad, las API de geovallas, los servicios de tráfico, las operaciones espaciales, el acceso directo a iconos de mapa y los servicios de lote.

| Característica de Mapas de Bing                     | Compatibilidad con Azure Maps |
|---------------------------------------|:------------------:|
| SDK web                               | ✓                  |
| SDK de Android                           | ✓                  |
| SDK de iOS                               | Planeado            |
| SDK de UWP                               | N/D                 |
| SDK de WPF                               | N/D                 |
| API REST del servicio                     | ✓                  |
| Autosuggest                           | ✓                  |
| Direcciones (incluido el camión)          | ✓                  |
| Matriz de distancia                       | ✓                  |
| Elevaciones                            | ✓      |
| Imágenes: mapa estático                  | ✓                  |
| Metadatos de imágenes                      | ✓                  |
| Isócronas                            | ✓                  |
| Local Insights                        | ✓                  |
| Búsqueda local                          | ✓                  |
| Reconocimiento de ubicación                  | ✓                  |
| Ubicaciones (codificación geográfica directa/inversa) | ✓                  |
| Rutas de itinerario optimizadas            | Planeado            |
| Ajustar a carretera                         | ✓                  |
| Servicios de datos espaciales (SDS)           | Parcial            |
| Zona horaria                             | ✓                  |
| Incidentes de tráfico                     | ✓                  |
| Mapas controlados por la configuración             | N/D                |

Mapas de Bing proporciona autenticación básica basada en claves. Azure Maps proporciona autenticación básica basada en claves y autenticación de alta seguridad de Azure Active Directory.

## <a name="licensing-considerations"></a>Consideraciones acerca de las licencias

Al migrar de Mapas de Bing a Azure Maps, se debe tener en cuenta la información siguiente relativa a las licencias.

* En Azure Maps se cobra por el uso de mapas interactivos en función del número de iconos de mapa cargados, mientras que en Mapas de Bing se cobra por cargar el control de mapa (sesiones). Para reducir los costos para los desarrolladores, Azure Maps almacena automáticamente en caché los iconos de mapa. Se genera una transacción de Azure Maps por cada 15 iconos de mapa que se cargan. Los SDK interactivos de Azure Maps usan iconos de 512 píxeles y, de media, generan una o menos transacciones por cada vista de página.

* Azure Maps permite almacenar los datos de su plataforma en Azure. También se pueden almacenar en caché en otro lugar durante un máximo de seis meses, según las [condiciones de uso](https://www.microsoftvolumelicensing.com/DocumentSearch.aspx?Mode=3&DocumentTypeId=31).

A continuación se incluyen algunos recursos relacionados con las licencias para Azure Maps:

-   [Página de precios de Azure Maps](https://azure.microsoft.com/pricing/details/azure-maps/)
-   [Calculadora de precios de Azure](https://azure.microsoft.com/pricing/calculator/?service=azure-maps)
-   [Condiciones de uso de Azure Maps](https://www.microsoftvolumelicensing.com/DocumentSearch.aspx?Mode=3&DocumentTypeId=31) (incluido en los términos de Microsoft Online Services)
-   [Elección del plan de tarifa adecuado de Azure Maps](./choose-pricing-tier.md)

## <a name="suggested-migration-plan"></a>Plan de migración sugerido

A continuación se muestra un ejemplo de plan de migración de alto nivel.

1.  Recopile los SDK y servicios de Mapas de Bing que se usan en la aplicación y compruebe que Azure Maps proporciona SDK y servicios alternativos a los que puede realizar la migración.
2.  Cree una suscripción de Azure (si todavía no tiene una) en <https://azure.com>.
3.  Cree una cuenta de Azure Maps ([documentación](./how-to-manage-account-keys.md))y una clave de autenticación o de Azure Active Directory ([documentación](./how-to-manage-authentication.md)).
4.  Migre el código de la aplicación.
5.  Pruebe la aplicación migrada.
6.  Implemente la aplicación migrada en producción.

## <a name="create-an-azure-maps-account"></a>Crear una cuenta de Azure Maps

Para crear una cuenta de Azure Maps y acceder a la plataforma Azure Maps, siga estos pasos:

1. Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.
2. Inicie sesión en [Azure Portal](https://portal.azure.com/).
3. Cree una [cuenta de Azure Maps](./how-to-manage-account-keys.md).
4. [Obtenga su clave de suscripción de Azure Maps](./how-to-manage-authentication.md#view-authentication-details) o configure la autenticación de Azure Active Directory para reforzar la seguridad.

## <a name="azure-maps-technical-resources"></a>Recursos técnicos para Azure Maps

Esta es una lista de recursos técnicos útiles para Azure Maps.

* Introducción: <https://azure.com/maps>
* Documentación: <https://aka.ms/AzureMapsDocs>
* Ejemplos de código del SDK web: <https://aka.ms/AzureMapsSamples>
* Foros para desarrolladores: <https://aka.ms/AzureMapsForums>
* Vídeos: <https://aka.ms/AzureMapsVideos>
* Blog: <https://aka.ms/AzureMapsBlog>
* Comentarios de Azure Maps (UserVoice): <https://aka.ms/AzureMapsFeedback>

## <a name="migration-support"></a>Compatibilidad con la migración

Los desarrolladores pueden buscar compatibilidad con la migración a través de los [foros](/answers/topics/azure-maps.html) o de una de las muchas opciones de soporte técnico de Azure: <https://azure.microsoft.com/support/options/>

## <a name="new-terminology"></a>Nueva terminología

La lista siguiente contiene términos comunes de Mapas de Bing y sus correspondientes en Azure Maps.

| Término del servicio Mapas de Bing                    | Término de Azure Maps                                                |
|-----------------------------------|----------------------------------------------------------------|
| Aéreo                            | De satélite o aéreo                                            |
| Direcciones                        | También se puede denominar ruta                             |
| Entidades                          | Geometrías o características                                         |
| `EntityCollection`                | Origen de datos o capa                                           |
| `Geopoint`                        | Posición                                                       |
| `GeoXML`                          | Archivos XML en el módulo de E/S espacial                             |
| Superposición de suelo                    | Capa de imagen                                                    |
| Híbrido (en referencia al tipo de mapa) | Satélite con carreteras                                           |
| InfoBox                           | Popup                                                          |
| Location                          | Posición                                                       |
| `LocationRect`                    | Rectángulo de selección                                                   |
| Tipo de asignación                          | Estilo del mapa                                                      |
| Barra de navegación                    | Selector de estilo de mapa, control de zoom, control de inclinación, control de brújula |
| Marcador                           | Capa de burbujas, capa de símbolos o marcador HTML                      |

## <a name="clean-up-resources"></a>Limpieza de recursos

No hay recursos que requieran limpieza.

## <a name="next-steps"></a>Pasos siguientes

Aprenda cómo migrar la aplicación de Mapas de Bing en estos artículos:

> [!div class="nextstepaction"]
> [Migración de una aplicación web](migrate-from-bing-maps-web-app.md)
