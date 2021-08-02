---
title: Elección de la forma de autorizar el acceso a los datos de blob en Azure Portal
titleSuffix: Azure Storage
description: Cuando se accede a los datos de blob desde Azure Portal, este realiza ciertas solicitudes a Azure Storage en segundo plano. Estas solicitudes a Azure Storage se pueden autenticar y autorizar utilizando bien la cuenta de Azure AD, bien la clave de acceso a la cuenta de almacenamiento.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 06/08/2021
ms.author: tamram
ms.reviewer: ozgun
ms.subservice: blobs
ms.custom: contperf-fy21q1
ms.openlocfilehash: 3d654a84b6e0e5cfb00c7b5d5dd5742649ec7a81
ms.sourcegitcommit: f9e368733d7fca2877d9013ae73a8a63911cb88f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111900780"
---
# <a name="choose-how-to-authorize-access-to-blob-data-in-the-azure-portal"></a>Elección de la forma de autorizar el acceso a los datos de blob en Azure Portal

Cuando se accede a los datos de blob desde [Azure Portal](https://portal.azure.com), este realiza ciertas solicitudes a Azure Storage en segundo plano. Una solicitud a Azure Storage se puede autorizar mediante la cuenta de Azure AD o la clave de acceso a la cuenta de almacenamiento. El portal indica qué método está usando, y le permite alternar entre ambos si tiene los permisos adecuados.  

También puede especificar cómo autorizar una operación de carga de blobs individual en Azure Portal. De forma predeterminada, el portal usa el método que ya está utilizando para autorizar una operación de carga de blobs, pero tiene la opción de cambiar esta configuración al cargar un blob.

## <a name="permissions-needed-to-access-blob-data"></a>Permisos necesarios para acceder a datos de blob

Necesitará permisos específicos según cómo quiera autorizar el acceso a los datos de blob en Azure Portal. En la mayoría de los casos, estos permisos se proporcionan a través del control de acceso basado en rol de Azure (Azure RBAC). Para más información acerca de Azure RBAC, consulte [¿Qué es el control de acceso basado en rol de Azure (Azure RBAC)?](../../role-based-access-control/overview.md)

### <a name="use-the-account-access-key"></a>Uso de la clave de acceso de la cuenta

Para acceder a los datos de blob con la clave de acceso a la cuenta, debe tener asignado un rol de Azure que incluya la acción de Azure RBAC **Microsoft.Storage/storageAccounts/listkeys/action**. Este rol de Azure puede ser un rol integrado o personalizado. Los roles integrados que **Microsoft.Storage/storageAccounts/listkeys/action** admite son los siguientes, en orden de permisos mínimos a máximos:

- Rol [Lector y acceso a los datos](../../role-based-access-control/built-in-roles.md#reader-and-data-access)
- El [rol Colaborador de una cuenta de almacenamiento](../../role-based-access-control/built-in-roles.md#storage-account-contributor)
- El [rol Colaborador](../../role-based-access-control/built-in-roles.md#contributor) de Azure Resource Manager
- El [rol Propietario](../../role-based-access-control/built-in-roles.md#owner) de Azure Resource Manager

Al intentar acceder a los datos de blob en Azure Portal, este comprueba primero si tiene asignado un rol con **Microsoft.Storage/storageAccounts/listkeys/action**. Si se le ha asignado un rol con esta acción, Azure Portal usa la clave de cuenta para tener acceso a los datos de blob. Si no tiene un rol asignado con esta acción, Portal intenta obtener acceso a los datos mediante su cuenta de Azure AD.

> [!IMPORTANT]
> Cuando una cuenta de almacenamiento está bloqueada con un bloqueo **ReadOnly** de Azure Resource Manager, no se permite la operación [Crear lista de claves](/rest/api/storagerp/storageaccounts/listkeys) para esa cuenta de almacenamiento. **Crear lista de claves** es una operación POST y todas las operaciones POST se impiden cuando se configura un bloqueo **ReadOnly** para la cuenta. Por esta razón, cuando la cuenta está bloqueada con un bloqueo **ReadOnly**, los usuarios deben usar las credenciales de Azure AD para acceder a los datos de blob en el portal. Para información sobre cómo acceder a los datos de blob en el portal con Azure AD, consulte [Uso de la cuenta de Azure AD](#use-your-azure-ad-account).

> [!NOTE]
> Los roles clásicos de administrador de suscripciones Administrador del servicio y Coadministrador equivalen al rol [Propietario](../../role-based-access-control/built-in-roles.md#owner) de Azure Resource Manager. El rol **Propietario** engloba todas las acciones (incluida **Microsoft.Storage/storageAccounts/listkeys/action**), por lo que un usuario con uno de estos roles administrativos también puede acceder a datos de blob con la clave de cuenta. Para obtener más información, consulte [Roles de administrador de suscripciones clásico, de Azure y de administrador de Azure AD](../../role-based-access-control/rbac-and-directory-admin-roles.md#classic-subscription-administrator-roles).

### <a name="use-your-azure-ad-account"></a>Uso de la cuenta de Azure AD

Para acceder a datos de blob desde Azure Portal con la cuenta de Azure AD, se deben cumplir estas dos premisas:

- Tiene asignado un rol (ya sea integrado o personalizado) que proporciona acceso a los datos de blob.
- Tiene asignado como mínimo el rol [Lector](../../role-based-access-control/built-in-roles.md#reader) de Azure Resource Manager, con el ámbito establecido en el nivel de la cuenta de almacenamiento o en un nivel superior. El rol **Lector** concede los permisos más restringidos, pero otro rol de Azure Resource Manager que conceda acceso a los recursos de administración de la cuenta de almacenamiento también es aceptable.

El rol **Lector** de Azure Resource Manager permite a los usuarios ver recursos de la cuenta de almacenamiento, pero no modificarlos. No proporciona permisos de lectura en los datos de Azure Storage, sino únicamente en los recursos de administración de la cuenta. El rol **Lector** es necesario para que los usuarios puedan navegar a contenedores de blobs en Azure Portal.

Para más información sobre los roles integrados que admiten el acceso a los datos de blob, consulte [Roles de Azure para blobs](assign-azure-role-data-access.md#azure-roles-for-blobs).

Los roles personalizados pueden admitir diferentes combinaciones de los mismos permisos que proporcionan los roles integrados. Para obtener más información sobre cómo crear roles RBAC de Azure personalizados, consulte el artículo sobre [roles personalizados de Azure](../../role-based-access-control/custom-roles.md) y la [descripción de las definiciones de roles de recursos de Azure](../../role-based-access-control/role-definitions.md).

> [!NOTE]
> La versión preliminar de Explorador de Storage en Azure Portal no admite el uso de credenciales de Azure AD para ver y modificar datos de blob. Explorador de Storage en Azure Portal usa siempre las claves de cuenta para acceder a los datos. Para usar Explorador de Storage en Azure Portal, debe tener asignado un rol que incluya **Microsoft.Storage/storageAccounts/listkeys/action**.

## <a name="navigate-to-blobs-in-the-azure-portal"></a>Desplazamiento a blobs en Azure Portal

Para ver datos de blob en Azure Portal, vaya a la sección **Información general** de la cuenta de almacenamiento y haga clic en los vínculos **Blobs**. También tiene la opción de ir a la sección **Contenedores** del menú.

:::image type="content" source="media/authorize-data-operations-portal/blob-access-portal.png" alt-text="Captura de pantalla que muestra cómo ir a los datos de blob en Azure Portal":::

## <a name="determine-the-current-authentication-method"></a>Determinar el método de autenticación actual

Al ir a un contenedor, Azure Portal indica si lo que se usa actualmente para autenticarse es la clave de acceso a la cuenta o la cuenta de Azure AD.

### <a name="authenticate-with-the-account-access-key"></a>Autenticación con la clave de acceso de la cuenta

Si se autentica mediante la clave de acceso a la cuenta, verá **Clave de acceso** especificado como método de autenticación en Portal:

:::image type="content" source="media/authorize-data-operations-portal/auth-method-access-key.png" alt-text="Captura de pantalla que muestra el usuario que accede actualmente a los contenedores con la clave de cuenta":::

Para cambiar y usar la cuenta de Azure AD, haga clic en el vínculo que aparece resaltado en la imagen. Si posee los permisos adecuados a través de los roles de Azure que tiene asignados, podrá continuar. Pero, si no los tiene, verá un mensaje de error como el siguiente:

:::image type="content" source="media/authorize-data-operations-portal/auth-error-azure-ad.png" alt-text="Error que aparece si la cuenta de Azure AD no admite el acceso":::

Cabe mencionar que la lista no contendrá ningún blob si su cuenta de Azure AD no tiene permisos para verlos. Haga clic en el vínculo **Cambiar a la clave de acceso** para usar la clave de acceso para intentar autenticarse de nuevo.

### <a name="authenticate-with-your-azure-ad-account"></a>Autenticación con la cuenta de Azure AD

Si se autentica utilizando la cuenta de Azure AD, verá **Cuenta de usuario de Azure AD** especificado como método de autenticación en Portal:

:::image type="content" source="media/authorize-data-operations-portal/auth-method-azure-ad.png" alt-text="Captura de pantalla que muestra el usuario que accede actualmente a los contenedores con una cuenta de Azure AD":::

Para cambiar y usar la clave de acceso a la cuenta, haga clic en el vínculo que aparece resaltado en la imagen. Si tiene acceso a la clave de cuenta, podrá continuar. Pero, si no puede acceder a ella, verá un mensaje de error como el siguiente:

:::image type="content" source="media/authorize-data-operations-portal/auth-error-access-key.png" alt-text="Error que aparece si no tiene acceso a la clave de cuenta":::

Cabe mencionar que la lista no contendrá ningún blob si carece de acceso a las claves de cuenta. Haga clic en el vínculo **Cambiar a la cuenta de usuario de Azure AD** para usar la cuenta de Azure AD para intentar autenticarse de nuevo.

## <a name="specify-how-to-authorize-a-blob-upload-operation"></a>Especificación de cómo autorizar una operación de carga de blobs

Al cargar un blob desde Azure Portal, puede especificar si desea autenticar y autorizar esa operación con la clave de acceso a la cuenta o con sus credenciales de Azure AD. De forma predeterminada, el portal usa el método de autenticación actual, como se muestra en [Determinar el método de autenticación actual](#determine-the-current-authentication-method).

Para especificar cómo autorizar una operación de carga de blobs, siga estos pasos:

1. En Azure Portal, vaya al contenedor en el que desea cargar un blob.
1. Seleccione el botón **Cargar**.
1. Expanda la sección **Avanzado** para mostrar las propiedades avanzadas del blob.
1. En el campo **Tipo de autenticación**, indique si desea autorizar la operación de carga mediante su cuenta de Azure AD o con la clave de acceso a la cuenta, tal como se muestra en la siguiente imagen:

    :::image type="content" source="media/authorize-data-operations-portal/auth-blob-upload.png" alt-text="Captura de pantalla que muestra cómo cambiar el método de autorización en la carga de blobs":::

## <a name="next-steps"></a>Pasos siguientes

- [Autenticar el acceso a blobs y colas de Azure con Azure Active Directory](../common/storage-auth-aad.md)
- [Asignación de un rol de Azure el acceso a datos de blob](assign-azure-role-data-access.md).
