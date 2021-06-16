---
title: 'Administración de identidades administradas asignadas por el usuario mediante REST: Azure AD'
description: Instrucciones paso a paso sobre cómo crear, enumerar y eliminar una identidad administrada asignada por el usuario para hacer llamadas a la API de REST.
services: active-directory
documentationcenter: ''
author: barclayn
manager: daveba
editor: ''
ms.service: active-directory
ms.subservice: msi
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 12/02/2020
ms.author: barclayn
ms.collection: M365-identity-device-management
ROBOTS: NOINDEX
ms.openlocfilehash: 7f48d2bb799cb641389d0197fbfb9997c2c773d0
ms.sourcegitcommit: 8bca2d622fdce67b07746a2fb5a40c0c644100c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111749586"
---
# <a name="create-list-or-delete-a-user-assigned-managed-identity-using-rest-api-calls"></a>Creación, enumeración o eliminación de una identidad administrada asignada por el usuario usando llamadas a la API de REST

Las identidades administradas de los recursos de Azure proporcionan a los servicios de Azure la capacidad de autenticar servicios que admitan la autenticación de Azure AD, sin necesidad de tener credenciales en el código. 

En este artículo, aprenderá a crear, enumerar y eliminar una identidad administrada asignada por el usuario mediante CURL para hacer llamadas a la API REST.

## <a name="prerequisites"></a>Requisitos previos

- Si no está familiarizado con las identidades administradas de los recursos de Azure, consulte la [sección de introducción](overview.md). **No olvide revisar la [diferencia entre una identidad administrada asignada por el sistema y una identidad administrada asignada por el usuario](overview.md#managed-identity-types)** .
- Si aún no tiene una cuenta de Azure, [regístrese para una cuenta gratuita](https://azure.microsoft.com/free/) antes de continuar.
- Puede ejecutar todos los comandos de este artículo, ya sea en la nube o localmente:
    - Para ejecutar en la nube, use [Azure Cloud Shell](../../cloud-shell/overview.md).
    - Para ejecutar localmente, instale [curl](https://curl.haxx.se/download.html) y la [CLI de Azure](/cli/azure/install-azure-cli).

## <a name="obtain-a-bearer-access-token"></a>Obtención de un token de acceso de portador

1. Si se ejecuta localmente, inicie sesión en Azure a través de la CLI de Azure:

    ```
    az login
    ```

1. Obtenga un token de acceso mediante [az account get-access-token](/cli/azure/account#az_account_get_access_token)

    ```azurecli-interactive
    az account get-access-token
    ```

## <a name="create-a-user-assigned-managed-identity"></a>Crear una identidad administrada asignada por el usuario 

Para crear una identidad administrada asignada por el usuario, la cuenta requiere la asignación del rol [Colaborador de identidades administradas](../../role-based-access-control/built-in-roles.md#managed-identity-contributor).

[!INCLUDE [ua-character-limit](~/includes/managed-identity-ua-character-limits.md)]

```bash
curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroup
s/<RESOURCE GROUP>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<USER ASSIGNED IDENTITY NAME>?api-version=2015-08-31-preview' -X PUT -d '{"loc
ation": "<LOCATION>"}' -H "Content-Type: application/json" -H "Authorization: Bearer <ACCESS TOKEN>"
```

```HTTP
PUT https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroup
s/<RESOURCE GROUP>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<USER ASSIGNED IDENTITY NAME>?api-version=2015-08-31-preview HTTP/1.1
```

**Encabezados de solicitud**

|Encabezado de solicitud  |Descripción  |
|---------|---------|
|*Content-Type*     | Necesario. Establézcalo en `application/json`.        |
|*Autorización*     | Necesario. Establézcalo en un token de acceso `Bearer` válido.        |

**Cuerpo de la solicitud**

|Nombre  |Descripción  |
|---------|---------|
|ubicación     | Necesario. Ubicación del recurso        |

## <a name="list-user-assigned-managed-identities"></a>Enumerar identidades administradas asignadas por el usuario

Para crear o leer una identidad administrada asignada por el usuario, la cuenta requiere la asignación del rol [Operador de identidades administradas ](../../role-based-access-control/built-in-roles.md#managed-identity-operator) o [Colaborador de identidades administradas](../../role-based-access-control/built-in-roles.md#managed-identity-contributor).

```bash
curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>/providers/Microsoft.ManagedIdentity/userAssignedIdentities?api-version=2015-08-31-preview' -H "Authorization: Bearer <ACCESS TOKEN>"
```

```HTTP
GET https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>/providers/Microsoft.ManagedIdentity/userAssignedIdentities?api-version=2015-08-31-preview HTTP/1.1
```

|Encabezado de solicitud  |Descripción  |
|---------|---------|
|*Content-Type*     | Necesario. Establézcalo en `application/json`.        |
|*Autorización*     | Necesario. Establézcalo en un token de acceso `Bearer` válido.        |

## <a name="delete-a-user-assigned-managed-identity"></a>Eliminar una identidad administrada asignada por el usuario

Para eliminar una identidad administrada asignada por el usuario, la cuenta requiere la asignación del rol [Colaborador de identidades administradas](../../role-based-access-control/built-in-roles.md#managed-identity-contributor).

> [!NOTE]
> La eliminación de una identidad administrada asignada por el usuario no quitará la referencia de ningún recurso al que se haya asignado. Para quitar una identidad administrada asignada por el usuario desde una máquina virtual mediante CURL, consulte [Eliminar una identidad asignada por el usuario de una máquina virtual de Azure](qs-configure-rest-vm.md#remove-a-user-assigned-managed-identity-from-an-azure-vm).

```bash
curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroup
s/<RESOURCE GROUP>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<USER ASSIGNED IDENTITY NAME>?api-version=2015-08-31-preview' -X DELETE -H "Authorization: Bearer <ACCESS TOKEN>"
```

```HTTP
DELETE https://management.azure.com/subscriptions/80c696ff-5efa-4909-a64d-f1b616f423ca/resourceGroups/TestRG/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<USER ASSIGNED IDENTITY NAME>?api-version=2015-08-31-preview HTTP/1.1
```
|Encabezado de solicitud  |Descripción  |
|---------|---------|
|*Content-Type*     | Necesario. Establézcalo en `application/json`.        |
|*Autorización*     | Necesario. Establézcalo en un token de acceso `Bearer` válido.        |

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre cómo asignar una identidad asignada por el usuario a una máquina virtual o a un conjunto de escalado de máquinas virtuales de Azure mediante CURL, consulte cómo [configurar las identidades administradas para los recursos de Azure en una máquina virtual de Azure mediante las llamadas a API REST](qs-configure-rest-vm.md#user-assigned-managed-identity) y cómo [configurar las identidades administradas para los recursos de Azure en un conjunto de escalado de máquinas virtuales mediante las llamadas a API REST](qs-configure-rest-vmss.md#user-assigned-managed-identity).
