---
title: Acceso a Config Server y Service Registry
titleSuffix: Azure Spring Cloud
description: Cómo acceder a los puntos de conexión de Service Registry y Config Server con el control de acceso basado en roles de Azure Active Directory.
author: MikeDodaro
ms.author: brendm
ms.service: spring-cloud
ms.topic: how-to
ms.date: 02/04/2021
ms.custom: devx-track-java, subject-rbac-steps
ms.openlocfilehash: b8474e7f322025609222b4867db988d22aa2bcb9
ms.sourcegitcommit: fc9fd6e72297de6e87c9cf0d58edd632a8fb2552
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108291216"
---
# <a name="access-config-server-and-service-registry"></a>Acceso a Config Server y Service Registry

En este artículo se explica cómo acceder a Spring Cloud Config Server y Spring Cloud Service Registry administrado por Azure Spring Cloud mediante el control de acceso basado en roles (RBAC) de Azure Active Directory (Azure AD).

## <a name="assign-role-to-azure-ad-usergroup-msi-or-service-principal"></a>Asignación de un rol a un usuario o grupo de Azure AD, una MSI o una entidad de servicio

Asigne el rol [azure-spring-cloud-data-reader](../role-based-access-control/built-in-roles.md#azure-spring-cloud-data-reader) a [usuario | grupo | entidad de servicio | identidad administrada] en el ámbito de [grupo de administración | suscripción | grupo de recursos | recurso].

Para acceder a los pasos detallados, consulte [Asignación de roles de Azure mediante Azure Portal](../role-based-access-control/role-assignments-portal.md).

## <a name="access-config-server-and-service-registry-endpoints"></a>Acceso a los puntos de conexión de Config Server y Service Registry

Después de asignar el rol Lector de datos de Azure Spring Cloud, los clientes pueden acceder a los puntos de conexión de Spring Cloud Config Server y Spring Cloud Service Registry. Siga estos procedimientos:

1. Obtener un token de acceso. Una vez que un usuario de Azure AD tenga asignado el rol Lector de datos de Azure Spring Cloud, los clientes podrán usar los siguientes comandos para iniciar sesión en la CLI de Azure con un usuario, entidad de servicio o identidad administrada para obtener un token de acceso. Para obtener más información, consulte [Autenticación en la CLI de Azure](/cli/azure/authenticate-azure-cli). 

    ```azurecli
    az login
    az account get-access-token
    ```
2. Cree el extremo. Se admiten los puntos de conexión predeterminados de Spring Cloud Config Server y Spring Cloud Service Registry administrados por Azure Spring Cloud. Para más información, consulte [Puntos de conexión listos para producción](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#production-ready-endpoints). Los clientes también pueden obtener una lista completa de los puntos de conexión admitidos de Spring Cloud Config Server y Spring Cloud Service Registry administrados por Azure Spring Cloud si acceden a los siguientes puntos de conexión:

    * *'https://SERVICE_NAME.svc.azuremicroservices.io/eureka/actuator/'*
    * *'https://SERVICE_NAME.svc.azuremicroservices.io/config/actuator/'* 

>[!NOTE]
> Si usa Azure China, reemplace `*.azuremicroservices.io` por`*.microservices.azure.cn`, [más información](/azure/china/resources-developer-guide#check-endpoints-in-azure).

3. Acceda al punto de conexión creado con el token de acceso. Coloque el token de acceso en un encabezado para proporcionar autorización.  Solo se admite el método "GET".

    Por ejemplo, acceda a un punto de conexión como *'https://SERVICE_NAME.svc.azuremicroservices.io/eureka/actuator/health '* para ver el estado de mantenimiento de Eureka.

    Si la respuesta es *401 no autorizado*, compruebe si el rol se ha asignado correctamente.  El rol tardará varios minutos en surtir efecto o bien, compruebe que el token de acceso no ha expirado.

## <a name="next-steps"></a>Pasos siguientes
* [Inicio de sesión con la CLI de Azure](/cli/azure/authenticate-azure-cli)
* [Puntos de conexión listos para producción](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#production-ready-endpoints)

## <a name="see-also"></a>Consulte también
* [Creación de roles y permisos](how-to-permissions.md)