---
title: Administración de servicios en el portal
titleSuffix: Azure Cognitive Search
description: Administrar un servicio de Azure Cognitive Search, un servicio hospedado de búsqueda en la nube de Microsoft Azure, mediante Azure Portal.
manager: nitinme
author: HeidiSteen
ms.author: heidist
tags: azure-portal
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 04/06/2021
ms.openlocfilehash: 98e0137c8e48696737cd5d8d1fd4d3de925b9f7f
ms.sourcegitcommit: d63f15674f74d908f4017176f8eddf0283f3fac8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "106579796"
---
# <a name="service-administration-for-azure-cognitive-search-in-the-azure-portal"></a>Administración de los servicios de Azure Cognitive Search en Azure Portal

> [!div class="op_single_selector"]
>
> * [PowerShell](search-manage-powershell.md)
> * [CLI de Azure](search-manage-azure-cli.md)
> * [REST API](/rest/api/searchmanagement/)
> * [SDK de .NET](/dotnet/api/microsoft.azure.management.search)
> * [Portal](search-manage.md)
> * [Python](https://pypi.python.org/pypi/azure-mgmt-search/0.1.0)> 

Azure Cognitive Search es un servicio de búsqueda basado en la nube totalmente administrado que se utiliza para la creación de una experiencia de búsqueda enriquecida en aplicaciones personalizadas. En este artículo se tratan las tareas de administración que se pueden realizar en [Azure Portal](https://portal.azure.com) para un servicio de búsqueda que ya se haya creado. El portal permite realizar todas las [tareas de administración](#management-tasks) relacionadas con el servicio, así como la exploración y la administración de contenido. Por lo tanto, el portal proporciona acceso amplio a todos los aspectos de las operaciones del servicio de búsqueda.

Cada servicio de búsqueda se administra como un recurso independiente. En la imagen siguiente se muestran las páginas del portal para un único servicio de búsqueda gratuito denominado "demo-search-svc". Aunque es posible que esté acostumbrado a usar Azure PowerShell o la CLI de Azure para la administración de servicios, tiene sentido familiarizarse con las herramientas y funcionalidades que proporcionan las páginas del portal. Algunas tareas son más fáciles y rápidas de realizar en el portal que a través del código. 

## <a name="overview-home-page"></a>Página de información general (Inicio)

La página de información general es la página "Inicio" de cada servicio. A continuación, las áreas de la pantalla entre cuadros rojos indican tareas, herramientas e iconos que puede usar con frecuencia, especialmente si es nuevo en el servicio.

:::image type="content" source="media/search-manage/search-portal-overview-page.png" alt-text="Páginas del portal para un servicio de búsqueda" border="true":::

| Área | Descripción |
|------|-------------|
| 1  | En la sección **Información esencial** se muestran las propiedades del servicio, incluido el punto de conexión que se usa al configurar las conexiones. También se muestran los recuentos de niveles, réplicas y particiones de un vistazo. |
| 2 | En la parte superior de la página hay una serie de comandos para invocar herramientas interactivas o administrar el servicio. Tanto la [Importación de datos](search-get-started-portal.md) como el [Explorador de búsqueda](search-explorer.md) se usan normalmente para la creación de prototipos y la exploración. |
| 3 | Debajo de la sección **Información esencial**, hay una serie de subpáginas con pestañas para obtener acceso rápido a las estadísticas de uso, las métricas de estado del servicio y el acceso a todos los índices, indexadores, orígenes de datos y conjuntos de aptitudes existentes en el servicio. Si selecciona un índice u otro objeto, hay páginas adicionales disponibles para mostrar la composición, la configuración y el estado de los objetos (si procede). |
| 4 | A la izquierda, puede acceder a vínculos que abren páginas adicionales para acceder a las claves de API que se usan en las conexiones, configurar el servicio, configurar la seguridad, supervisar las operaciones, automatizar tareas y obtener soporte técnico. |

### <a name="read-only-service-properties"></a>Propiedades de servicio de solo lectura

Se determinan varios aspectos de un servicio de búsqueda cuando se aprovisiona el servicio y no se pueden cambiar:

* Nombre del servicio (no se puede cambiar el nombre de un servicio de búsqueda).
* Ubicación del servicio (no se puede trasladar fácilmente un servicio de búsqueda intacto a otra región. Aunque hay una plantilla, el traslado del contenido es un proceso manual).
* Nivel de servicio (no se puede cambiar de S1 a S2, por ejemplo, sin crear un nuevo servicio).

## <a name="management-tasks"></a>Tareas administrativas

La administración de servicios es ligera por diseño y a menudo se define mediante las tareas que puede realizar en relación con el propio servicio:

* [Ajuste de la capacidad](search-capacity-planning.md) mediante la adición o eliminación de réplicas y particiones.
* [Administración de claves de API](search-security-api-keys.md) usadas para operaciones de administración y consulta.
* [Control del acceso a las operaciones de administración](search-security-rbac.md) a través de la seguridad basada en roles.
* [Configuración de reglas de firewall de IP](service-configure-firewall.md) para restringir el acceso por dirección IP.
* [Configuración de un punto de conexión privado](service-create-private-endpoint.md) mediante Azure Private Link y una red virtual privada.
* [Supervisión del estado del servicio](search-monitor-usage.md): almacenamiento, volúmenes de consultas y latencia.

También puede enumerar todos los objetos creados en el servicio: índices, indexadores, orígenes de datos, conjuntos de aptitudes, etc. En la página de información general del portal se muestra todo el contenido que existe en el servicio. La gran mayoría de las operaciones en un servicio de búsqueda están relacionadas con el contenido.

Las mismas tareas de administración realizadas en el portal también se pueden controlar mediante programación a través de las [API REST](/rest/api/searchmanagement/)de administración, el [módulo Az.Search PowerShell](search-manage-powershell.md), el [módulo az search de la CLI de Azure](search-manage-azure-cli.md) y los SDK de Azure para .NET, Python, Java y JavaScript. Las tareas administrativas se representan completamente en todas las interfaces programáticas y en el portal. No hay ninguna tarea administrativa específica que esté disponible solo en una modalidad.

Cognitive Search aprovecha otros servicios de Azure para realizar una supervisión y administración más profundas. Por sí solos, los únicos datos almacenados en un servicio de búsqueda son contenido del objeto (índices, indexador y definiciones de orígenes de datos, además de otros objetos). Las métricas que se indican en las páginas del portal se extraen de los registros internos en un ciclo continuo de 30 días. Para la retención de registros controlada por el usuario y eventos adicionales, necesitará [Azure Monitor](../azure-monitor/index.yml). Para obtener más información sobre cómo configurar el registro de diagnóstico para un servicio de búsqueda, consulte [Recopilación y análisis de datos de registro](search-monitor-logs.md).

## <a name="administrator-permissions"></a>Permisos de administrador

Al abrir la página de información general del servicio de búsqueda, los permisos asignados a su cuenta determinan qué páginas están disponibles. La página de información general al principio del artículo muestra las páginas del portal que verá un administrador o colaborador.

En el recurso de Azure, los derechos administrativos se conceden a través de asignaciones de roles. En el contexto de Azure Cognitive Search, las [asignaciones de roles](search-security-rbac.md) determinarán quién puede asignar réplicas y particiones o administrar claves de API, independientemente de si usan el portal, [PowerShell](search-manage-powershell.md), la [CLI de Azure](search-manage-azure-cli.md) o las [API REST de administración](/rest/api/searchmanagement/search-howto-management-rest-api):

> [!TIP]
> El aprovisionamiento o la retirada del propio servicio los puede realizar un administrador o coadministrador de una suscripción de Azure. Mediante el uso de mecanismos de aplicación en todo el sistema de Azure, puede bloquear una suscripción o un recurso para evitar la eliminación accidental o no autorizada del servicio de búsqueda por parte de usuarios con derechos de administrador. Para más información, consulte [Bloqueo de recursos para impedir eliminación inesperada](../azure-resource-manager/management/lock-resources.md).

## <a name="next-steps"></a>Pasos siguientes

* Revise las [funcionalidades de supervisión](search-monitor-usage.md) disponibles en el portal.
* Automatice con [PowerShell](search-manage-powershell.md) o la [CLI de Azure](search-manage-azure-cli.md).
* Revisión de las [características de seguridad](search-security-overview.md) para proteger contenido y operaciones
* Habilitación de [registro de diagnóstico](search-monitor-logs.md) para supervisar las cargas de trabajo de consulta e indexación