---
title: Configuración de identidades administradas con Azure AD para su cuenta de Azure Cosmos DB
description: Aprenda a configurar las identidades administradas con Azure Active Directory para la cuenta de Azure Cosmos DB.
author: ThomasWeiss
ms.service: cosmos-db
ms.topic: how-to
ms.date: 04/02/2021
ms.author: thweiss
ms.openlocfilehash: 30efaed09a400611861bdd3adeae1f650054b405
ms.sourcegitcommit: 3f684a803cd0ccd6f0fb1b87744644a45ace750d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/02/2021
ms.locfileid: "106230928"
---
# <a name="configure-managed-identities-with-azure-active-directory-for-your-azure-cosmos-db-account"></a>Configuración de identidades administradas con Azure Active Directory para la cuenta de Azure Cosmos DB.
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

Las identidades administradas para los recursos de Azure proporcionan a los servicios de Azure una identidad administrada automáticamente en Azure Active Directory. En este artículo se muestra cómo crear una identidad administrada para cuentas de Azure Cosmos DB.

> [!NOTE]
> Actualmente Azure Cosmos DB solo admite identidades administradas asignadas por el sistema.

## <a name="prerequisites"></a>Prerrequisitos

- Si no está familiarizado con las identidades administradas para los recursos de Azure, vea [¿Qué son las identidades administradas para recursos de Azure?](../active-directory/managed-identities-azure-resources/overview.md). Para obtener información sobre los tipos de identidades administradas, vea [Tipos de identidad administrada](../active-directory/managed-identities-azure-resources/overview.md#managed-identity-types).
- Para configurar identidades administradas, la cuenta debe tener el [rol Colaborador de la cuenta de DocumentDB](../role-based-access-control/built-in-roles.md#documentdb-account-contributor).

## <a name="add-a-system-assigned-identity"></a>Adición de una identidad asignada por el sistema

### <a name="using-an-azure-resource-manager-arm-template"></a>Uso de una plantilla de Azure Resource Manager (ARM)

> [!IMPORTANT]
> Asegúrese de usar una `apiVersion` de `2021-03-15` o una versión superior al trabajar con identidades administradas.

Para habilitar una identidad asignada por el sistema en una cuenta de Azure Cosmos DB nueva o existente, incluya la siguiente propiedad en la definición de recursos:

```json
"identity": {
    "type": "SystemAssigned"
}
```

La sección `resources` de la plantilla de ARM debería tener un aspecto similar al siguiente:

```json
"resources": [
    {
        "type": " Microsoft.DocumentDB/databaseAccounts",
        "identity": {
            "type": "SystemAssigned"
        },
        // ...
    },
    // ...
 ]
```

Una vez creada o actualizada la cuenta de Azure Cosmos DB, se mostrará la siguiente propiedad:

```json
"identity": {
    "type": "SystemAssigned",
    "tenantId": "<azure-ad-tenant-id>",
    "principalId": "<azure-ad-principal-id>"
}
```

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre las [identidades administradas para recursos de Azure](../active-directory/managed-identities-azure-resources/overview.md)
- Obtenga más información sobre las [claves administradas por el cliente en Azure Cosmos DB](how-to-setup-cmk.md).
