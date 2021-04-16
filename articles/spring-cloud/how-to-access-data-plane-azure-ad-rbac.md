---
title: Acceso a Config Server y Service Registry
titleSuffix: Azure Spring Cloud
description: Cómo acceder a los puntos de conexión de Service Registry y Config Server con el control de acceso basado en roles de Azure Active Directory.
author: MikeDodaro
ms.author: brendm
ms.service: spring-cloud
ms.topic: how-to
ms.date: 02/04/2021
ms.custom: devx-track-java
ms.openlocfilehash: 16433d5b148d7bc441e375591c64af497cd7b8de
ms.sourcegitcommit: c2a41648315a95aa6340e67e600a52801af69ec7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/06/2021
ms.locfileid: "106505340"
---
# <a name="access-config-server-and-service-registry"></a>Acceso a Config Server y Service Registry

En este artículo se explica cómo acceder a Spring Cloud Config Server y Spring Cloud Service Registry administrado por Azure Spring Cloud mediante el control de acceso basado en roles (RBAC) de Azure Active Directory (Azure AD).

## <a name="assign-role-to-azure-ad-usergroup-msi-or-service-principal"></a>Asignación de un rol a un usuario o grupo de Azure AD, una MSI o una entidad de servicio

Para usar Azure AD y RBAC, debe asignar el rol *Lector de datos de Azure Spring Cloud* a un usuario, un grupo o una entidad de servicio mediante el siguiente procedimiento:

1. Vaya a la página de información general del servicio de la instancia de servicio.

2. Haga clic en **Control de acceso (IAM)** para abrir la hoja de control de acceso.

3. Haga clic en el botón **Agregar** y en **Agregar asignación de roles** (es posible que se necesite autorización para poder agregar).

4. Busque y seleccione *Lector de datos de Azure Spring Cloud* en **Rol**.
5. Asigne el acceso a `User, group, or service principal` o `User assigned managed identity` según el tipo de usuario. Busque y seleccione el usuario.  
6. Haga clic en `Save`

   ![Asignación de roles](media/access-data-plane-aad-rbac/assign-data-reader-role.png)

## <a name="access-config-server-and-service-registry-endpoints"></a>Acceso a los puntos de conexión de Config Server y Service Registry

Después de asignar el rol Lector de datos de Azure Spring Cloud, los clientes pueden acceder a los puntos de conexión de Spring Cloud Config Server y Spring Cloud Service Registry. Siga estos procedimientos:

1. Obtener un token de acceso. Una vez que un usuario de Azure AD tenga asignado el rol Lector de datos de Azure Spring Cloud, los clientes podrán usar los siguientes comandos para iniciar sesión en la CLI de Azure con un usuario, entidad de servicio o identidad administrada para obtener un token de acceso. Para obtener más información, consulte [Autenticación en la CLI de Azure](https://docs.microsoft.com/cli/azure/authenticate-azure-cli). 

    ```azurecli
    az login
    az account get-access-token
    ```
2. Cree el extremo. Se admiten los puntos de conexión predeterminados de Spring Cloud Config Server y Spring Cloud Service Registry administrados por Azure Spring Cloud. Para más información, consulte [Puntos de conexión listos para producción](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#production-ready-endpoints). Los clientes también pueden obtener una lista completa de los puntos de conexión admitidos de Spring Cloud Config Server y Spring Cloud Service Registry administrados por Azure Spring Cloud si acceden a los siguientes puntos de conexión:

    * *https://SERVICE_NAME.svc.azuremicroservices.io/eureka/actuator/*
    * *https://SERVICE_NAME.svc.azuremicroservices.io/config/actuator/* 

>[!NOTE]
> Si usa Azure China, reemplace `*.azuremicroservices.io` por`*.microservices.azure.cn`, [más información](https://docs.microsoft.com/azure/china/resources-developer-guide#check-endpoints-in-azure).

3. Acceda al punto de conexión creado con el token de acceso. Coloque el token de acceso en un encabezado para proporcionar autorización.  Solo se admite el método "GET".

    Por ejemplo, acceda a un punto de conexión como *https://SERVICE_NAME.svc.azuremicroservices.io/eureka/actuator/health* para ver el estado de mantenimiento de Eureka.

    Si la respuesta es *401 no autorizado*, compruebe si el rol se ha asignado correctamente.  El rol tardará varios minutos en surtir efecto o bien, compruebe que el token de acceso no ha expirado.

## <a name="next-steps"></a>Pasos siguientes
* [Inicio de sesión con la CLI de Azure](https://docs.microsoft.com/cli/azure/authenticate-azure-cli)
* [Puntos de conexión listos para producción](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#production-ready-endpoints)

## <a name="see-also"></a>Consulte también
* [Creación de roles y permisos](how-to-permissions.md)
