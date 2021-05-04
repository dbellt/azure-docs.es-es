---
title: 'Tutorial: Configuración de filtros de ruta para el emparejamiento de Microsoft: Azure Portal'
description: En este tutorial se describe cómo configurar los filtros de ruta para el emparejamiento de Microsoft mediante Azure Portal.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: tutorial
ms.date: 10/15/2020
ms.author: duau
ms.custom: seodec18
ms.openlocfilehash: 64a721514783c0cda1d4c2a859e25ed6589653b8
ms.sourcegitcommit: bd1a4e4df613ff24e954eb3876aebff533b317ae
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/23/2021
ms.locfileid: "107931026"
---
# <a name="tutorial-configure-route-filters-for-microsoft-peering-using-the-azure-portal"></a>Tutorial: Configuración de filtros de ruta para el emparejamiento de Microsoft: Azure Portal

> [!div class="op_single_selector"]
> * [Azure Portal](how-to-routefilter-portal.md)
> * [Azure PowerShell](how-to-routefilter-powershell.md)
> * [CLI de Azure](how-to-routefilter-cli.md)
> 

Los filtros de ruta son una forma de consumir un subconjunto de servicios admitidos mediante el emparejamiento de Microsoft. Los pasos que se describen en este artículo le ayudan a configurar y administrar filtros de ruta para circuitos ExpressRoute.

Los servicios de Microsoft 365, como Exchange Online, SharePoint Online y Skype Empresarial, están accesibles mediante el emparejamiento de Microsoft. Cuando se configura el emparejamiento de Microsoft en un circuito ExpressRoute, todos los prefijos relacionados con estos servicios se anuncian mediante las sesiones BGP establecidas. A cada prefijo se asocia un valor de la comunidad de BGP para identificar el servicio que se ofrece mediante el prefijo en cuestión. Para ver una lista de valores de la comunidad de BGP y los servicios a los que se asignan, consulte [Comunidades de BGP](expressroute-routing.md#bgp).

La conectividad a todos los servicios de Azure y Microsoft 365 genera un gran número de prefijos que se anuncian a través de BGP. Como consecuencia, aumenta considerablemente el tamaño de las tablas de rutas que mantienen los enrutadores dentro de la red. Si tiene previsto consumir solo un subconjunto de los servicios ofrecidos mediante el emparejamiento de Microsoft, puede reducir el tamaño de las tablas de rutas de dos maneras. Puede:

* Filtrar los prefijos no deseados mediante la aplicación de filtros de ruta en las comunidades de BGP. Los filtros de ruta son un procedimiento de red estándar y se usa normalmente en muchas redes.

* Definir filtros de ruta y aplicarlos al circuito ExpressRoute. Un filtro de ruta es un nuevo recurso que le permite seleccionar la lista de servicios que se va a consumir mediante el emparejamiento de Microsoft. Los enrutadores de ExpressRoute solo envían la lista de prefijos que pertenecen a los servicios identificados en el filtro de ruta.

En este tutorial, obtendrá información sobre cómo:
> [!div class="checklist"]
> - Conseguir valores de comunidad de BGP.
> - Creación de un filtro de ruta y una regla de filtro.
> - Asociación del filtro de ruta a un circuito ExpressRoute.

### <a name="about-route-filters"></a><a name="about"></a>Acerca de los filtros de ruta

Cuando se configura el emparejamiento de Microsoft en el circuito ExpressRoute, los enrutadores perimetrales de Microsoft establecen un par de sesiones BGP con los enrutadores perimetrales suyos o del proveedor de conectividad. No se anuncia ningún enrutador en la red. Para habilitar los anuncios de rutas en la red, debe asociar un filtro de ruta.

Un filtro de ruta permite identificar los servicios que desea consumir mediante el emparejamiento de Microsoft del circuito ExpressRoute. Se trata básicamente de una lista de todos los valores permitidos de la comunidad de BGP. Una vez que se define un recurso de filtro de ruta y se asocia a un circuito ExpressRoute, todos los prefijos que se asignan a valores de la comunidad de BGP se anuncian en la red.

Para asociar filtros de ruta con servicios de Microsoft 365, debe tener autorización para consumir tales servicios mediante ExpressRoute. Si no la tiene, la operación para asociar filtros de ruta producirá un error. Para obtener más información sobre el proceso de autorización, consulte [Azure ExpressRoute para Microsoft 365](/microsoft-365/enterprise/azure-expressroute).

> [!IMPORTANT]
> Se anunciarán todos los prefijos de servicio de Microsoft Office para el emparejamiento de Microsoft de los circuitos ExpressRoute que se configuraron antes del 1 de agosto de 2017, incluso si no se definen filtros de ruta. No se anunciará ningún prefijo para el emparejamiento de Microsoft de los circuitos ExpressRoute que se configuraron el 1 de agosto de 2017 o con posterioridad, hasta que se asocie un filtro de ruta al circuito.
> 

## <a name="prerequisites"></a>Requisitos previos

- Revise los [Requisitos previos y lista de comprobación de ExpressRoute](expressroute-prerequisites.md) y los [Flujos de trabajo de ExpressRoute para aprovisionamiento de circuitos y estados de circuitos de ExpressRoute](expressroute-workflows.md) antes de comenzar la configuración.

- Debe tener un circuito ExpressRoute activo que tenga el emparejamiento de Microsoft aprovisionado. Para realizar estas tareas, puede usar las siguientes instrucciones:
  - [Cree un circuito ExpressRoute](expressroute-howto-circuit-portal-resource-manager.md) y pida a su proveedor de conectividad que se lo habilite antes de proceder. El circuito ExpressRoute debe estar en estado habilitado y aprovisionado.
  - Si administra la sesión BGP directamente, [cree el emparejamiento de Microsoft](expressroute-howto-routing-portal-resource-manager.md). O bien, pida a su proveedor de conectividad que aprovisione el emparejamiento de Microsoft para su circuito.

## <a name="get-a-list-of-prefixes-and-bgp-community-values"></a><a name="prefixes"></a>Obtención de una lista de prefijos y valores de la comunidad de BGP

### <a name="get-a-list-of-bgp-community-values"></a>Obtención de una lista de valores de la comunidad de BGP

Los valores de la comunidad de BGP asociados con los servicios accesibles a través de emparejamiento de Microsoft están disponibles en la página [Requisitos de enrutamiento de ExpressRoute](expressroute-routing.md).

### <a name="make-a-list-of-the-values-that-you-want-to-use"></a>Creación de una lista de los valores que quiera usar

Cree una lista de [valores de la comunidad de BGP](expressroute-routing.md#bgp) que quiera usar en el filtro de ruta. 

## <a name="create-a-route-filter-and-a-filter-rule"></a><a name="filter"></a>Creación de un filtro de ruta y una regla de filtro

Un filtro de ruta puede tener una única regla y la regla debe ser de tipo "Permitir". Esta regla puede tener una lista de valores de la comunidad de BGP asociados a ella.

1. Seleccione **Crear un recurso** y, a continuación, busque el *filtro de ruta* tal y como se muestra en la siguiente imagen:

    :::image type="content" source="./media/how-to-routefilter-portal/create-route-filter.png" alt-text="Captura de pantalla que muestra la página Filtro de ruta":::

1. Coloque el filtro de ruta en un grupo de recursos. Asegúrese de que la ubicación es la misma que la del circuito ExpressRoute. Seleccione **Revisar y crear** y, a continuación, **Crear**.

    :::image type="content" source="./media/how-to-routefilter-portal/create-route-filter-basic.png" alt-text="Captura de pantalla que muestra la página Crear filtro de ruta con los valores de ejemplo especificados":::

### <a name="create-a-filter-rule"></a>Creación de una regla de filtro

1. Para agregar y actualizar las reglas, seleccione la pestaña de administración de reglas del filtro de ruta.

    :::image type="content" source="./media/how-to-routefilter-portal/manage-route-filter.png" alt-text="Captura de pantalla que muestra la página Información general con la opción Administrar acción de regla resaltada":::

1. Seleccione los servicios a los que desea conectarse desde la lista desplegable y guardar la regla cuando haya finalizado.

    :::image type="content" source="./media/how-to-routefilter-portal/add-route-filter-rule.png" alt-text="Captura de pantalla que muestra la ventana Administrar regla con servicios seleccionados en la lista desplegable Comunidades de servicio permitidas":::

## <a name="attach-the-route-filter-to-an-expressroute-circuit"></a><a name="attach"></a>Asociación del filtro de ruta a un circuito ExpressRoute

Adjunte el filtro de ruta a un circuito seleccionando el botón **Agregar circuito** y seleccionando el circuito de ExpressRoute en la lista desplegable.

:::image type="content" source="./media/how-to-routefilter-portal/add-circuit-to-route-filter.png" alt-text="Captura de pantalla que muestra la página de información general con la acción Agregar circuito seleccionada":::

Si el proveedor de conectividad configura emparejamiento para su circuito de ExpressRoute, actualice el circuito desde la página de circuito de ExpressRoute antes de seleccionar el botón **Agregar circuito**.

:::image type="content" source="./media/how-to-routefilter-portal/refresh-express-route-circuit.png" alt-text="Captura de pantalla que muestra la página de información general con la acción Actualizar seleccionada.":::

## <a name="common-tasks"></a><a name="tasks"></a>Tareas comunes

### <a name="to-get-the-properties-of-a-route-filter"></a><a name="getproperties"></a>Obtención de las propiedades de un filtro de ruta

Puede ver propiedades de un filtro de ruta cuando abra el recurso en el portal.

:::image type="content" source="./media/how-to-routefilter-portal/view-route-filter.png" alt-text="Captura de pantalla en la que se muestra la página Información general":::

### <a name="to-update-the-properties-of-a-route-filter"></a><a name="updateproperties"></a>Actualización de las propiedades de un filtro de ruta

1. Puede actualizar la lista de valores de la comunidad de BGP adjunta a un circuito seleccionando el botón **Administrar regla**.

    :::image type="content" source="./media/how-to-routefilter-portal/update-route-filter.png" alt-text="Actualice los filtros de ruta con la acción Administrar regla":::

1. Seleccione las comunidades de servicio que desee y, a continuación, seleccione **Guardar**.

    :::image type="content" source="./media/how-to-routefilter-portal/add-route-filter-rule.png" alt-text="Captura de pantalla que muestra la ventana Administrar regla con servicios seleccionados":::

### <a name="to-detach-a-route-filter-from-an-expressroute-circuit"></a><a name="detach"></a>Desasociación de un filtro de ruta de un circuito ExpressRoute

Para desasociar un circuito del filtro de ruta, haga clic con el botón derecho en el circuito y haga clic en **Desasociar**.

:::image type="content" source="./media/how-to-routefilter-portal/detach-route-filter.png" alt-text="Captura de pantalla que muestra la página Información general con la acción Desasociar resaltada":::


## <a name="clean-up-resources"></a>Limpieza de recursos

Para eliminar un filtro de ruta, haga clic en el botón **Eliminar**. Asegúrese de que el filtro de ruta no está asociado a ningún circuito antes de hacerlo.

:::image type="content" source="./media/how-to-routefilter-portal/delete-route-filter.png" alt-text="Eliminación de un filtro de ruta":::

## <a name="next-steps"></a>Pasos siguientes

Para más información acerca de las muestras de configuración de ruta, vea:

> [!div class="nextstepaction"]
> [Ejemplos de configuración de enrutadores para configurar y administrar enrutamiento](expressroute-config-samples-routing.md)
