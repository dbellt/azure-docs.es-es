---
title: Autorización del acceso a través de roles de Azure
titleSuffix: Azure Cognitive Search
description: Control del acceso basado en rol de Azure (RBAC de Azure) en Azure Portal para controlar y delegar tareas administrativas para la administración de Azure Cognitive Search.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 05/28/2021
ms.openlocfilehash: 4e79af03f4545e4af799c660314212d18dbc91ff
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110459970"
---
# <a name="authorize-access-through-azure-roles-in-azure-cognitive-search"></a>Autorización del acceso a través de roles de Azure en Azure Cognitive Search

Azure proporciona un [modelo de autorización global basado en roles](../role-based-access-control/role-assignments-portal.md) para todos los servicios que se administran del portal o de las API de Resource Manager. El modelo de autorización proporciona los roles de Propietario, Colaborador y Lector, que determinan el nivel de *administración de servicio* de los usuarios, grupos y entidades de seguridad de Active Directory asignados a cada rol. Cognitive Search usa estos tres roles para autorizar el acceso a la administración del servicio de búsqueda.

Cognitive Search no admite lo siguiente:

+ [Roles personalizados](../role-based-access-control/custom-roles.md).
+ Control de acceso basado en rol (RBAC de Azure) sobre operaciones relacionadas con el contenido, como la creación o consulta de un índice, o cualquier otro objeto en el servicio.

  La autorización para realizar operaciones de contenido requiere una [clave de API de administrador o una clave de API de consulta](search-security-api-keys.md).

> [!Note]
> Para el acceso basado en la identidad a través de los resultados de búsqueda (lo que a veces se conoce como seguridad de nivel de fila), puede crear filtros de seguridad para recortar los resultados según la identidad y quitar los documentos para los que el solicitante no debe tener acceso. Para más información, consulte [Filtros de seguridad](search-security-trimming-for-azure-search.md).

## <a name="roles-used-in-cognitive-search"></a>Roles usados en Cognitive Search

Para Azure Cognitive Search, los roles están asociados a los niveles de permisos que admiten las siguientes tareas de administración:

| Role | Tarea |
| --- | --- |
| Propietario |Crear o eliminar el servicio. Crear, actualizar o eliminar cualquier objeto en el servicio: claves de API, índices, mapas de sinónimos, indexadores, orígenes de datos de indexador y conjuntos de aptitudes. </br></br>Acceso total a toda la información del servicio expuesta en el portal o a través de la API de REST de administración, Azure PowerShell o la CLI de Azure. </br></br>Asignar pertenencia a roles.</br></br>Los administradores de suscripciones y los propietarios de servicios tienen pertenencia automática al rol Propietario. |
| Colaborador | El mismo nivel de acceso que Propietario, menos la asignación de roles. [Colaborador del servicio Search](../role-based-access-control/built-in-roles.md#search-service-contributor) equivale al rol integrado Colaborador genérico. |
| Lector | Acceso limitado a la información parcial del servicio. En el portal, el rol Lector puede acceder a la información de la página Información general del servicio, en la sección Essentials y en la pestaña Supervisión. Todas las demás pestañas y páginas están fuera de los límites. </br></br>En la sección Essentials: grupo de recursos, estado, ubicación, nombre e identificador de la suscripción, etiquetas, dirección URL, plan de tarifa, réplicas, particiones y unidades de búsqueda. </br></br>En la pestaña Supervisión, ver las métricas del servicio: latencia de búsqueda, porcentaje de solicitudes limitadas y promedio de consultas por segundo. </br></br>No hay acceso a la pestaña Uso (almacenamiento, recuentos de índices o indexadores creados en el servicio) ni a ninguna información en las pestañas Índices, Indexadores, Orígenes de datos, Conjuntos de aptitudes o Sesiones de depuración. |

Los roles no otorgan derechos de acceso al punto de conexión de servicio. Las operaciones del servicio Search, como la administración de índices, el relleno del índice y las consultas sobre datos de búsqueda se controlan mediante claves de API, no a través de roles. Para más información, consulte [Administración de claves de API](search-security-api-keys.md).

## <a name="tasks-and-permission-requirements"></a>Tareas y requisitos de permisos

En la tabla siguiente se resumen las operaciones permitidas en Azure Cognitive Search y el rol o la clave que desbloquea el acceso a una operación determinada.

+ La pertenencia a RBAC de Azure concede acceso a las páginas del portal y a las tareas administración de servicios (crear, eliminar o cambiar un servicio o sus claves de API).

+ Las claves de API se crean después de que un servicio exista y se aplican a las operaciones de contenido del servicio.

Además, para las operaciones relacionadas con el contenido en el portal, como la creación o eliminación de objetos, se admite el acceso total a todas las operaciones y a la información mediante la pertenencia explícita a roles (Propietario o Colaborador), además del uso interno del portal de una clave de administrador. En otras palabras, si va a crear o cargar un índice en el portal, la pertenencia a RBAC le proporciona acceso a las páginas, pero el propio portal usa una clave de administrador para autenticar la operación en el servicio.

| Operación | Controlado por |
|-----------|-------------------------|
| Creación o eliminación de un servicio | Permisos de RBAC de Azure: Propietario o Colaborador |
| Configurar la seguridad de red (firewall de IP) | Permisos de RBAC de Azure: Propietario o Colaborador |
| Crear o eliminar un punto de conexión privado | Permisos de RBAC de Azure: Propietario o Colaborador |
| Implementar claves administradas por el cliente | Permisos de RBAC de Azure: Propietario o Colaborador |
| Ajustar réplicas o particiones | Permisos de RBAC de Azure: Propietario o Colaborador|
| Administrar claves de administrador o de consulta | Permisos de RBAC de Azure: Propietario o Colaborador|
| Ver la información del servicio en el portal o en una API de administración | Permisos de RBAC de Azure: Propietario, Colaborador o Lector  |
| Ver métricas e información del objeto en el portal o en una API de administración | Permisos de RBAC de Azure: Propietario o Colaborador |
| Crear, modificar y eliminar objetos en el servicio: <br>Índices y partes de componentes (incluidas las definiciones del analizador, los perfiles de puntuación y las opciones de CORS), indexadores, orígenes de datos, conjuntos de aptitudes, sinónimos, proveedores de sugerencias | Clave de administrador si se usa una API, más roles Propietario o Colaborador de RBAC de Azure si se usa el portal |
| Consultar un índice | Clave de administrador o de consulta si se usa una API, más roles Propietario o Colaborador de RBAC de Azure si se usa el portal |
| Consultar la información del sistema sobre los objetos, como devolver estadísticas, recuentos y listas de objetos | Clave de administrador si se usa una API, más roles Propietario o Colaborador de RBAC de Azure si se usa el portal |

## <a name="next-steps"></a>Pasos siguientes

+ [Administración mediante PowerShell](search-manage-powershell.md) 
+ [Consideraciones sobre el rendimiento y la optimización de Azure Cognitive Search](search-performance-optimization.md)
+ [¿Qué es el control de acceso basado en rol de Azure (RBAC de Azure)?](../role-based-access-control/overview.md)
