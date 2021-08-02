---
title: archivo de inclusión
description: archivo de inclusión
services: storage
author: tamram
ms.service: storage
ms.topic: include
ms.date: 06/07/2021
ms.author: tamram
ms.custom: include file
ms.openlocfilehash: a93e85114c46f6e0ce252bb4059836d62fadf20f
ms.sourcegitcommit: f9e368733d7fca2877d9013ae73a8a63911cb88f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111904590"
---
RBAC de Azure proporciona una serie de roles integrados para autorizar el acceso a datos de cola con Azure AD y OAuth. Entre los roles que proporcionan permisos a los recursos de datos en Azure Storage están, por ejemplo, los siguientes:

- [Colaborador de datos de la cola de Storage](../articles/role-based-access-control/built-in-roles.md#storage-queue-data-contributor): se usa para conceder permisos de lectura, escritura y eliminación a las colas de Azure.
- [Lector de datos de la cola de Storage](../articles/role-based-access-control/built-in-roles.md#storage-queue-data-reader): se usa para conceder permisos de solo lectura a las colas de Azure.
- [Procesador de mensajes de datos de la cola de Storage](../articles/role-based-access-control/built-in-roles.md#storage-queue-data-message-processor): se usa para conceder permisos de inspección, recuperación y eliminación a los mensajes de las colas de Azure Storage.
- [Emisor de mensajes de datos de la cola de Storage](../articles/role-based-access-control/built-in-roles.md#storage-queue-data-message-sender): se usa para conceder permisos de adición a los mensajes de las colas de Azure Storage.

Para obtener información sobre cómo enumerar los roles RBAC de Azure y sus permisos, consulte [Enumeración de las definiciones de roles de Azure](../articles/role-based-access-control/role-definitions-list.md).

Solo los roles definidos explícitamente para el acceso a datos permiten a una entidad de seguridad acceder a los datos de colas. Los roles integrados, como **Propietario**, **Colaborador** y **Colaborador de la cuenta de almacenamiento**, permiten que una entidad de seguridad administre una cuenta de almacenamiento, pero no proporcionan acceso a los datos de colas dentro de esa cuenta a través de Azure AD. Sin embargo, si un rol incluye **Microsoft.Storage/storageAccounts/listKeys/action**, el usuario al que se haya asignado ese rol podrá acceder a los datos de la cuenta de almacenamiento mediante la autorización de clave compartida con las claves de acceso de la cuenta. Para obtener más información, vea [Elección de la forma de autorizar el acceso a los datos de blob en Azure Portal](../articles/storage/queues/authorize-data-operations-portal.md).

Para obtener información detallada sobre los roles integrados de Azure para Azure Storage para los servicios de datos y el servicio de administración, consulte la sección **Almacenamiento** en [Roles integrados de Azure para RBAC de Azure](../articles/role-based-access-control/built-in-roles.md#storage). Además, para obtener información sobre los diferentes tipos de roles que proporcionan permisos en Azure, consulte [Roles de administrador de suscripciones clásico, roles de Azure y roles de Azure AD](../articles/role-based-access-control/rbac-and-directory-admin-roles.md).

> [!IMPORTANT]
> Las asignaciones de roles de Azure pueden tardar hasta 30 minutos en propagarse.
