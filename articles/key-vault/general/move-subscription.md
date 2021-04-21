---
title: Azure Key Vault traslada un almacén a una suscripción diferente | Microsoft Docs
description: Instrucciones para trasladar un almacén de claves a una suscripción diferente.
services: key-vault
author: msmbaldwin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: general
ms.topic: how-to
ms.date: 05/05/2020
ms.author: mbaldwin
ms.openlocfilehash: 65dc9da03a6b763d419c51e53bf756550e8b56a4
ms.sourcegitcommit: 6686a3d8d8b7c8a582d6c40b60232a33798067be
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107751857"
---
# <a name="moving-an-azure-key-vault-to-another-subscription"></a>Traslado de Azure Key Vault a otra suscripción

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="overview"></a>Información general

> [!IMPORTANT]
> **Trasladar un almacén de claves a otra suscripción, producirá un cambio importante en el entorno.**
> Asegúrese de entender el impacto de este cambio y siga atentamente las instrucciones de este artículo antes de decidir si desea trasladar el almacén de claves a una nueva suscripción.
> Si usa identidades de servicio administradas (MSI), lea las instrucciones posteriores al traslado, al final del documento. 

[Azure Key Vault](overview.md) se asocia automáticamente al identificador de inquilino predeterminado de [Azure Active Directory](../../active-directory/fundamentals/active-directory-whatis.md) para la suscripción en la que se ha creado. Puede encontrar el identificador de inquilino asociado a la suscripción si sigue esta [guía](../../active-directory/fundamentals/active-directory-how-to-find-tenant.md). Todas las entradas de la directiva de acceso y las asignaciones de roles también se asocian con este identificador de inquilino.  Al trasladar una suscripción de Azure del inquilino A al inquilino B, las entidades de servicio (usuarios y aplicaciones) del inquilino B no podrán acceder a los almacenes de claves ya existentes. Para corregir este problema, es necesario:

* Cambiar el identificador de inquilino asociado a todas las instancias de Key Vault existentes en la suscripción al inquilino B.
* Quitar todas las entradas de la directiva de acceso existentes.
* Agregar nuevas entradas de la directiva de acceso asociadas al inquilino B.

Para obtener más información sobre Azure Key Vault y Azure Active Directory, vea
- [Acerca de Azure Key Vault](overview.md)
- [¿Qué es Azure Active Directory?](../../active-directory/fundamentals/active-directory-whatis.md)
- [Búsqueda de un identificador de inquilino](../../active-directory/fundamentals/active-directory-how-to-find-tenant.md)

## <a name="limitations"></a>Limitaciones

> [!IMPORTANT]
> **Los almacenes de claves utilizados para el cifrado de disco no pueden moverse** Si utiliza Key Vault con el cifrado de discos para una máquina virtual, el almacén de claves no puede migrarse a otro grupo de recursos o a una suscripción mientras esté habilitado el cifrado de discos. Antes de mover el almacén de claves a un nuevo grupo de recursos o a una nueva suscripción, debe deshabilitar el cifrado de discos. 

Algunas entidades de servicio (usuarios y aplicaciones) están enlazadas a un inquilino específico. Si traslada el almacén de claves a una suscripción de otro inquilino, existe la posibilidad de que no pueda restaurar el acceso a una entidad de servicio específica. Asegúrese de que todas las entidades de servicio esenciales existen en el inquilino al que va a trasladar el almacén de claves.

## <a name="prerequisites"></a>Requisitos previos

* Acceso de nivel de [colaborador](../../role-based-access-control/built-in-roles.md#contributor) o superior a la suscripción actual en la que existe el almacén de claves. Puede asignar el rol mediante [Azure Portal](../../role-based-access-control/role-assignments-portal.md), la [CLI de Azure](../../role-based-access-control/role-assignments-cli.md) o [PowerShell](../../role-based-access-control/role-assignments-powershell.md).
* Acceso de nivel de [colaborador](../../role-based-access-control/built-in-roles.md#contributor) o superior a la suscripción actual a la que quiera cambiar el almacén de claves. Puede asignar el rol mediante [Azure Portal](../../role-based-access-control/role-assignments-portal.md), la [CLI de Azure](../../role-based-access-control/role-assignments-cli.md) o [PowerShell](../../role-based-access-control/role-assignments-powershell.md).
* Un grupo de recursos en la nueva suscripción. Puede crear uno mediante [Azure Portal](../../azure-resource-manager/management/manage-resource-groups-portal.md), [PowerShell](../../azure-resource-manager/management/manage-resource-groups-powershell.md) o la [CLI de Azure](../../azure-resource-manager/management/manage-resource-groups-cli.md).

Puede comprobar los roles existentes mediante [Azure Portal](../../role-based-access-control/role-assignments-list-portal.md), [PowerShell](../../role-based-access-control/role-assignments-list-powershell.md), la [CLI de Azure](../../role-based-access-control/role-assignments-list-cli.md) o la [API REST](../../role-based-access-control/role-assignments-list-rest.md).


## <a name="moving-a-key-vault-to-a-new-subscription"></a>Traslado de un almacén de claves a una nueva suscripción

1. Inicie sesión en Azure Portal en https://portal.azure.com.
2. Vaya al [almacén de claves](overview.md).
3. Haga clic en la pestaña "Introducción".
4. Seleccione el botón «Mover»
5. Seleccione «Mover a otra suscripción» de las opciones de la lista desplegable
6. Seleccione el grupo de recursos a donde quiere trasladar el almacén de claves
7. Confirme la advertencia relacionada con el traslado de recursos.
8. Seleccione Aceptar.

## <a name="additional-steps-when-subscription-is-in-a-new-tenant"></a>Pasos adicionales cuando la suscripción se encuentra en un nuevo inquilino

Si ha migrado el almacén de claves a una suscripción en un nuevo inquilino, tiene que actualizar manualmente el identificador de inquilino y quitar las directivas de acceso anteriores y las asignaciones de roles. Estos son los tutoriales de estos pasos en PowerShell y la CLI de Azure. Si usa PowerShell, es posible que tenga que ejecutar el comando Clear-AzContext que se describe a continuación para poder ver recursos fuera del ámbito seleccionado actualmente. 

### <a name="update-tenant-id-in-a-key-vault"></a>Actualización del identificador de inquilino en un almacén de claves

```azurepowershell
Select-AzSubscription -SubscriptionId <your-subscriptionId>                # Select your Azure Subscription
$vaultResourceId = (Get-AzKeyVault -VaultName myvault).ResourceId          # Get your key vault's Resource ID 
$vault = Get-AzResource -ResourceId $vaultResourceId -ExpandProperties     # Get the properties for your key vault
$vault.Properties.TenantId = (Get-AzContext).Tenant.TenantId               # Change the Tenant that your key vault resides in
$vault.Properties.AccessPolicies = @()                                     # Access policies can be updated with real
                                                                           # applications/users/rights so that it does not need to be                             # done after this whole activity. Here we are not setting 
                                                                           # any access policies. 
Set-AzResource -ResourceId $vaultResourceId -Properties $vault.Properties  # Modifies the key vault's properties.

Clear-AzContext                                                            #Clear the context from PowerShell
Connect-AzAccount                                                          #Log in again to confirm you have the correct tenant id
````

```azurecli
az account set -s <your-subscriptionId>                                    # Select your Azure Subscription
tenantId=$(az account show --query tenantId)                               # Get your tenantId
az keyvault update -n myvault --remove Properties.accessPolicies           # Remove the access policies
az keyvault update -n myvault --set Properties.tenantId=$tenantId          # Update the key vault tenantId
```
### <a name="update-access-policies-and-role-assignments"></a>Actualización de las directivas de acceso y las asignaciones de roles

> [!NOTE]
> Si Key Vault usa el modelo de permisos [RBAC de Azure](../../role-based-access-control/overview.md). También tendrá que quitar las asignaciones de roles del almacén de claves. Puede quitar las asignaciones de roles [Azure Portal](../../role-based-access-control/role-assignments-portal.md), la [CLI de Azure](../../role-based-access-control/role-assignments-cli.md) o [PowerShell](../../role-based-access-control/role-assignments-powershell.md). 

Ahora que el almacén está asociado al identificador de inquilino correcto y que se han quitado las entradas de la directiva de acceso antiguas o las asignaciones de roles, establezca nuevas entradas o asignaciones.

Para asignar directivas, vea lo siguiente:
- [Asignación de directivas de acceso mediante el portal](assign-access-policy-portal.md)
- [Asignación de directivas de acceso mediante la CLI de Azure](assign-access-policy-cli.md)
- [Asignación de directivas de acceso mediante PowerShell](assign-access-policy-powershell.md)

Para agregar asignaciones de roles, vea lo siguiente:
- [Asignación de roles de Azure mediante Azure Portal](../../role-based-access-control/role-assignments-portal.md)
- [Asignación de roles de Azure mediante la CLI de Azure](../../role-based-access-control/role-assignments-cli.md)
- [Asignación de roles de Azure mediante PowerShell](../../role-based-access-control/role-assignments-powershell.md)


### <a name="update-managed-identities"></a>Actualización de identidades administradas

Si va a transferir la suscripción completa y usa una identidad administrada para los recursos de Azure, tendrá que actualizarla también al nuevo inquilino de Azure Active Directory. Para más información sobre las identidades administradas, consulte [Identidades administradas para recursos de Azure](../../active-directory/managed-identities-azure-resources/overview.md).

Si usa identidad administrada, también tendrá que actualizar la identidad, ya que la anterior ya no estará en el inquilino de Azure Active Directory correcto. Consulte los siguientes documentos para ayudar a resolver este problema. 

* [Actualización de MSI](../../active-directory/managed-identities-azure-resources/known-issues.md#transferring-a-subscription-between-azure-ad-directories)
* [Transferencia de una suscripción a un nuevo directorio](../../role-based-access-control/transfer-subscription.md)

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre [claves, secretos y certificados](about-keys-secrets-certificates.md)
- Para obtener información conceptual, incluido cómo interpretar los registros de Key Vault, consulte [Registro de Azure Key Vault](logging.md).
- [Guía del desarrollador de Key Vault](../general/developers-guide.md)
- [Protección de Key Vault](security-overview.md)
- [Configuración de firewalls y redes virtuales de Azure Key Vault](network-security.md)