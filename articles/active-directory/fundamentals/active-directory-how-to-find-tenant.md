---
title: 'Búsqueda del identificador de inquilino: Azure Active Directory'
description: Instrucciones para buscar el identificador de inquilino de Azure Active Directory de una suscripción existente de Azure.
services: active-directory
author: ajburnle
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: fundamentals
ms.topic: how-to
ms.date: 10/30/2020
ms.author: ajburnle
ms.reviewer: jeffsta
ms.custom: it-pro, devx-track-azurepowershell
ms.collection: M365-identity-device-management
ms.openlocfilehash: 6a6bba9a1fc7974d324696e88e059c1d44924dac
ms.sourcegitcommit: 2f322df43fb3854d07a69bcdf56c6b1f7e6f3333
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/27/2021
ms.locfileid: "108016696"
---
# <a name="how-to-find-your-azure-active-directory-tenant-id"></a>Búsqueda del identificador de inquilino de Azure Active Directory

Las suscripciones de Azure tienen una relación de confianza con Azure Active Directory (Azure AD). Se confía en Azure AD para autenticar usuarios, servicios y dispositivos de la suscripción. Cada suscripción tiene un identificador de inquilino asociado y hay varias maneras de encontrarlo.

## <a name="find-tenant-id-through-the-azure-portal"></a>Búsqueda del identificador de inquilino mediante Azure Portal

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
 
1. Seleccione **Azure Active Directory**.

1. Seleccione **Propiedades**.

1. Luego desplácese hacia abajo hasta el campo **Id. de inquilino**. El identificador de inquilino aparece en el cuadro.

:::image type="content" source="media/active-directory-how-to-find-tenant/portal-tenant-id.png" alt-text="Azure Active Directory - Propiedades - Id. de inquilino - Campo Id. de inquilino":::

## <a name="find-tenant-id-with-powershell"></a>Búsqueda del identificador de inquilino con PowerShell

También puede buscar el inquilino mediante programación. Para buscar el identificador de inquilino con Azure PowerShell, use el cmdlet `Get-AzTenant`.

```azurepowershell-interactive
Connect-AzAccount
Get-AzTenant
```
   
Para obtener más información, vea la referencia del cmdlet de Azure PowerShell [Get-AzTenant](/powershell/module/az.accounts/get-aztenant).


## <a name="find-tenant-id-with-cli"></a>Búsqueda del identificador de inquilino con la CLI
Si quiere usar una interfaz de línea de comandos para buscar el identificador de inquilino, puede hacerlo con la [CLI de Azure](/cli/azure/install-azure-cli) o la [CLI de Microsoft 365](https://pnp.github.io/cli-microsoft365/). 

En la CLI de Azure, use uno de los comandos **az login**, **az account list** o **az account tenant list** como se muestra en el ejemplo siguiente. Observe la propiedad **tenantId** de cada una de las suscripciones en la salida de cada comando.

```azurecli-interactive
az login
az account list
az account tenant list
```

Para obtener más información, vea las referencias de los comandos [az login](/cli/azure/reference-index#az_login), [az account](/cli/azure/account) o [az account tenant](/cli/azure/account/tenant).


En la CLI de Microsoft 365, use el cmdlet **tenant id** como se muestra en el ejemplo siguiente:
 
```cli
m365 tenant id get
```

Para obtener más información, vea la referencia del comando [tenant id get](https://pnp.github.io/cli-microsoft365/cmd/tenant/id/id-get/) de Microsoft 365.


## <a name="next-steps"></a>Pasos siguientes

- Para crear un nuevo inquilino de Azure AD, consulte [Inicio rápido: Creación de un inquilino en Azure Active Directory](active-directory-access-create-new-tenant.md).

- Para aprender a asociar o agregar una suscripción a un inquilino, vea [Asociación o incorporación de una suscripción de Azure al inquilino de Azure Active Directory](active-directory-how-subscriptions-associated-directory.md).

- Para aprender a buscar el identificador de objeto, vea [Búsqueda del identificador de objeto de usuario](/partner-center/find-ids-and-domain-names#find-the-user-object-id).
