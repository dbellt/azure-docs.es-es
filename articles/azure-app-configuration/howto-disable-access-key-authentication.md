---
title: Deshabilitación de la autenticación mediante clave de acceso para una instancia de Azure App Configuration (versión preliminar)
titleSuffix: Azure App Configuration
description: Obtenga información sobre cómo deshabilitar la autenticación mediante clave de acceso para una instancia de Azure App Configuration (versión preliminar)
ms.service: azure-app-configuration
author: jimmyca15
ms.author: jimmyca
ms.topic: how-to
ms.date: 5/14/2021
ms.openlocfilehash: 451d9701b62cf46fe81bdd17f4eded63a38d1dce
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110483491"
---
# <a name="disable-access-key-authentication-for-an-azure-app-configuration-instance-preview"></a>Deshabilitación de la autenticación mediante clave de acceso para una instancia de Azure App Configuration (versión preliminar)

Todas las solicitudes a un recurso de Azure App Configuration deben autenticarse. De manera predeterminada, las solicitudes se pueden autenticar con credenciales de Azure Active Directory (Azure AD) o mediante una clave de acceso. De estos dos tipos de esquema de autenticación, Azure AD proporciona mayor seguridad y facilidad de uso a través de las claves de acceso y es el esquema que Microsoft recomienda. Para requerir que los clientes usen Azure AD para autenticar solicitudes, puede deshabilitar el uso de claves de acceso para un recurso de Azure App Configuration.

Al deshabilitar la autenticación mediante clave de acceso para un recurso de Azure App Configuration, se eliminan las claves de acceso existentes para ese recurso. Se rechazarán las solicitudes posteriores al recurso que usen las claves de acceso que existían anteriormente. Solo se realizarán correctamente las solicitudes que estén autenticadas con Azure AD. Para más información sobre el uso de Azure AD, consulte [Autorización del acceso a Azure App Configuration con Azure Active Directory](./concept-enable-rbac.md).

## <a name="disable-access-key-authentication"></a>Deshabilitación de la autenticación mediante claves de acceso

Al deshabilitar la autenticación mediante clave de acceso, se eliminarán todas las claves de acceso. Si alguna aplicación en ejecución usa claves de acceso para la autenticación, comenzará a generar errores una vez que se deshabilite la autenticación mediante clave de acceso. Si se vuelve a habilitar la autenticación mediante clave de acceso, se generará un nuevo conjunto de claves de acceso y las aplicaciones que intenten usar las claves de acceso antiguas seguirán generando errores.

> [!WARNING]
> Si algún cliente está accediendo actualmente a datos en el recurso de Azure App Configuration mediante claves de acceso, Microsoft recomienda migrar esos clientes a [Azure AD](./concept-enable-rbac.md) antes de deshabilitar la autenticación mediante clave de acceso.
> Además, se recomienda leer la siguiente sección sobre [limitaciones](#limitations) para comprobar que las limitaciones no afectarán al uso previsto del recurso.

# <a name="azure-portal"></a>[Azure Portal](#tab/portal)

A fin de no permitir la autenticación mediante clave de acceso para un recurso de Azure App Configuration en Azure Portal, siga estos pasos:

1. Vaya al recurso de Azure App Configuration en Azure Portal.
2. Busque la opción **Claves de acceso** en **Configuración**.

    :::image type="content" border="true" source="./media/access-keys-blade.png" alt-text="Captura de pantalla en la que se muestra cómo acceder a la hoja de las claves de acceso de un recurso de Azure App Configuration":::

3. Establezca la opción **Habilitar claves de acceso** en **Deshabilitado**.

    :::image type="content" border="true" source="./media/disable-access-keys.png" alt-text="Captura de pantalla que muestra cómo deshabilitar la autenticación mediante clave de acceso para Azure App Configuration":::

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

La funcionalidad para deshabilitar la autenticación mediante clave de acceso con la CLI de Azure está en desarrollo.

---

### <a name="verify-that-access-key-authentication-is-disabled"></a>Comprobación de que la autenticación mediante clave de acceso está deshabilitada

Para comprobar que ya no se permite la autenticación mediante clave de acceso, se puede realizar una solicitud para enumerar las claves de acceso del recurso de Azure App Configuration. Si la autenticación mediante clave de acceso está deshabilitada, no habrá ninguna clave de acceso y la operación de enumeración devolverá una lista vacía.

# <a name="azure-portal"></a>[Azure Portal](#tab/portal)

A fin de comprobar que la autenticación mediante clave de acceso para un recurso de Azure App Configuration está deshabilitada en Azure Portal, siga estos pasos:

1. Vaya al recurso de Azure App Configuration en Azure Portal.
2. Busque la opción **Claves de acceso** en **Configuración**.

    :::image type="content" border="true" source="./media/access-keys-blade.png" alt-text="Captura de pantalla en la que se muestra cómo acceder a la hoja de las claves de acceso de un recurso de Azure App Configuration":::

3. Compruebe que no se muestre ninguna clave de acceso y que la opción **Habilitar claves de acceso** esté establecida en **Deshabilitado**.

    :::image type="content" border="true" source="./media/access-keys-disabled-portal.png" alt-text="Captura de pantalla en la que se muestran las claves de acceso deshabilitadas para un recurso de Azure App Configuration":::

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

A fin de comprobar que la autenticación mediante clave de acceso para un recurso de Azure App Configuration está deshabilitada en Azure Portal, use el siguiente comando. El comando enumerará las claves de acceso de un recurso de Azure App Configuration y, si la autenticación mediante clave de acceso está deshabilitada, la lista estará vacía.

```azurecli-interactive
az appconfig credential list \
    --name <app-configuration-name> \
    --resource-group <resource-group>
```

Si la autenticación mediante clave de acceso está deshabilitada, se devolverá una lista vacía.

```
C:\Users\User>az appconfig credential list -g <resource-group> -n <app-configuration-name>
[]
```

---

## <a name="permissions-for-allowing-or-disallowing-access-key-authentication"></a>Permisos para admitir o denegar la autenticación mediante clave de acceso

A fin de modificar el estado de la autenticación mediante clave de acceso para un recurso de Azure App Configuration, un usuario debe tener permisos para crear y administrar recursos de Azure App Configuration. Los roles de control de acceso basado en rol de Azure (Azure RBAC) que proporcionan estos permisos incluyen la acción **Microsoft.AppConfiguration/configurationStores/write** o **Microsoft.AppConfiguration/configurationStores/\*** . Los roles integrados con esta acción incluyen:

- El rol [Propietario](../role-based-access-control/built-in-roles.md#owner) de Azure Resource Manager
- El rol [Colaborador](../role-based-access-control/built-in-roles.md#contributor) de Azure Resource Manager

Estos roles no proporcionan acceso a los datos de un recurso de Azure App Configuration a través de Azure Active Directory (Azure AD). Sin embargo, incluyen el permiso de la acción **Microsoft.AppConfiguration/configurationStores/listKeys/action**, que concede acceso a las claves de acceso del recurso. Con este permiso, un usuario puede usar las claves de acceso para acceder a todos los datos del recurso.

Las asignaciones de roles deben tener como ámbito |el nivel del recurso de Azure App Configuration o uno superior para permitir que un usuario permita o deniegue la autenticación mediante clave de acceso para el recurso. Para obtener más información sobre el ámbito de los roles, vea [Comprensión del ámbito para RBAC de Azure](../role-based-access-control/scope-overview.md).

Tenga cuidado de restringir la asignación de estos roles solo a aquellos usuarios que requieran la capacidad de crear un recurso de Azure App Configuration o actualizar sus propiedades. Use el principio de privilegios mínimos para asegurarse de que los usuarios tienen los permisos mínimos que necesitan para realizar sus tareas. Para más información sobre la administración del acceso con RBAC de Azure, consulte [Procedimientos recomendados para RBAC de Azure](../role-based-access-control/best-practices.md).

> [!NOTE]
> Los roles clásicos de administrador de suscripciones Administrador del servicio y Coadministrador equivalen al rol [Propietario](../role-based-access-control/built-in-roles.md#owner) de Azure Resource Manager. El rol **Propietario** incluye todas las acciones, por lo que un usuario con uno de estos roles administrativos también puede crear y administrar recursos de Azure App Configuration. Para más información, consulte [Roles de administrador de suscripciones clásico, de Azure y de administrador de Azure AD](../role-based-access-control/rbac-and-directory-admin-roles.md#classic-subscription-administrator-roles).

## <a name="limitations"></a>Limitaciones

La funcionalidad para deshabilitar la autenticación mediante clave de acceso está disponible como una versión preliminar. Actualmente están en vigor las siguientes limitaciones.

### <a name="arm-template-access"></a>Acceso a plantillas de Resource Manager

Cuando la autenticación mediante clave de acceso esté deshabilitada, también se deshabilitará la funcionalidad para leer y escribir valores de clave en una [plantilla de ARM](./quickstart-resource-manager.md). Esto se debe a que el acceso al recurso Microsoft.AppConfiguration/configurationStores/keyValues usado en las plantillas de ARM requiere un rol de Azure Resource Manager, ya sea como colaborador o propietario. Cuando la autenticación mediante clave de acceso está deshabilitada, el acceso al recurso requiere uno de los [roles del plano de datos](concept-enable-rbac.md) de Azure App Configuration, por lo que se rechaza el acceso a la plantilla de ARM.

## <a name="next-steps"></a>Pasos siguientes

- [Uso de claves administradas por el cliente para cifrar los datos de configuración de App Configuration](concept-customer-managed-keys.md)
- [Uso de puntos de conexión privados en Azure App Configuration](concept-private-endpoint.md)