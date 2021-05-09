---
title: Creación de suscripciones del Contrato Enterprise de Azure mediante programación con las API más recientes
description: Aprenda a crear suscripciones de Contrato Enterprise de Azure mediante programación con las versiones más recientes de la API REST, la CLI de Azure, Azure PowerShell y las plantillas de Azure Resource Manager.
author: bandersmsft
ms.service: cost-management-billing
ms.subservice: billing
ms.topic: how-to
ms.date: 03/29/2021
ms.reviewer: andalmia
ms.author: banders
ms.custom: devx-track-azurepowershell, devx-track-azurecli
ms.openlocfilehash: e57f385dce6446ebb3aa2df0ceb48f97a7e0c2f4
ms.sourcegitcommit: 2aeb2c41fd22a02552ff871479124b567fa4463c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107877916"
---
# <a name="programmatically-create-azure-enterprise-agreement-subscriptions-with-the-latest-apis"></a>Creación de suscripciones de Contrato Enterprise de Azure mediante programación con las API más recientes

Este artículo le ayuda a crear mediante programación suscripciones de Contrato Enterprise de Azure para una cuenta de facturación de EA mediante las versiones más recientes de las API. Si todavía usa la versión preliminar anterior, consulte [Creación de suscripciones de Azure mediante programación con las API de versión preliminar](programmatically-create-subscription-preview.md). 

En este artículo, se ofrece información sobre cómo crear suscripciones mediante programación con Azure Resource Manager.

Al crear una suscripción a Azure mediante programación, dicha suscripción se rige por el contrato en cuyo marco ha obtenido los servicios de Azure de Microsoft o de un distribuidor autorizado. Para más información, consulte [Información legal de Microsoft Azure](https://azure.microsoft.com/support/legal/).

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Requisitos previos

Debe tener un rol Propietario en una cuenta de inscripción para crear una suscripción. Existen dos formas de obtener el rol:

* El administrador de empresa de la inscripción puede [convertirle en propietario de una cuenta](https://ea.azure.com/helpdocs/addNewAccount) (inicio de sesión requerido), lo que le hace propietario de la cuenta de inscripción.
* Un propietario existente de la cuenta de inscripción puede [concederle acceso](/rest/api/billing/2019-10-01-preview/enrollmentaccountroleassignments/put). De forma similar, para usar una entidad de servicio con el fin de crear una suscripción a Contrato Enterprise, debe [conceder a la entidad de servicio la capacidad de crear suscripciones](/rest/api/billing/2019-10-01-preview/enrollmentaccountroleassignments/put).  
    Si usa un nombre de entidad de seguridad de servicio para crear suscripciones, use el ObjectId del registro de aplicación de Azure AD como objectId de la entidad de servicio mediante [Azure Active Directory PowerShell](/powershell/module/azuread/get-azureadserviceprincipal?view=azureadps-2.0) o la [CLI de Azure](/cli/azure/ad/sp?view=azure-cli-latest#az_ad_sp_list).
  > [!NOTE]
  > Asegúrese de usar la versión correcta de API para conceder permisos de propietario a la cuenta de inscripción. A los efectos de este artículo y de las API que se documentan en él, use la API [2019-10-01-Preview](/rest/api/billing/2019-10-01-preview/enrollmentaccountroleassignments/put). Si va a efectuar una migración para usar las API más recientes, tendrá que conceder nuevamente permiso de propietario mediante la API [2019-10-01-Preview](/rest/api/billing/2019-10-01-preview/enrollmentaccountroleassignments/put). La configuración anterior realizada con la versión [2015-07-01](grant-access-to-create-subscription.md) no se puede usar automáticamente con las API más recientes.

## <a name="find-accounts-you-have-access-to"></a>Búsqueda de cuentas a las que tiene acceso

Cuando haya sido agregado a una cuenta de inscripción asociada a un propietario de cuenta, Azure usa la relación cuenta-inscripción para determinar dónde se cobra la suscripción. Todas las suscripciones creadas en la cuenta se facturan a la inscripción de EA en la que se encuentra la cuenta. Para crear suscripciones, debe pasar valores sobre la cuenta de inscripción y las entidades de seguridad de usuario al propietario de la suscripción.

Para ejecutar los comandos siguientes, debe iniciar sesión en el *directorio particular* del propietario de cuenta, que es el directorio en el que las suscripciones se crean de manera predeterminada.

### <a name="rest"></a>[REST](#tab/rest)

Solicite mostrar todas las cuentas de inscripción a las que tiene acceso:

```json
GET https://management.azure.com/providers/Microsoft.Billing/billingaccounts/?api-version=2020-05-01
```

La respuesta de la API muestra todas las cuentas de inscripción a las que tiene acceso:

```json
{
  "value": [
    {
      "id": "/providers/Microsoft.Billing/billingAccounts/1234567",
      "name": "1234567",
      "properties": {
        "accountStatus": "Unknown",
        "accountType": "Enterprise",
        "agreementType": "EnterpriseAgreement",
        "soldTo": {
          "companyName": "Contoso",
          "country": "US "
        },
        "billingProfiles": {
          "hasMoreResults": false
        },
        "displayName": "Contoso",
        "enrollmentAccounts": [
          {
            "id": "/providers/Microsoft.Billing/billingAccounts/1234567/enrollmentAccounts/7654321",
            "name": "7654321",
            "type": "Microsoft.Billing/enrollmentAccounts",
            "properties": {
              "accountName": "Contoso",
              "accountOwnerEmail": "kenny@contoso.onmicrosoft.com",
              "costCenter": "Test",
              "isDevTest": false
            }
          }
        ],
        "hasReadAccess": false
      },
      "type": "Microsoft.Billing/billingAccounts"
    }
  ]
}

```

Los valores para un ámbito de facturación y `id` son lo mismo. El `id` de la cuenta de inscripción es el ámbito de facturación en el que se inicia la solicitud de suscripción. Es importante conocer el identificador porque es un parámetro necesario que se usará más adelante en el artículo para crear una suscripción.

### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Use la CLI de Azure o la API REST o para obtener este valor.

### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Solicite mostrar todas las cuentas de inscripción a las que tiene acceso:

```azurecli-interactive
> az billing account list
```

La respuesta enumera todas las cuentas de inscripción a las que tiene acceso.

```json
[
  {
    "accountStatus": "Unknown",
    "accountType": "Enterprise",
    "agreementType": "EnterpriseAgreement",
    "billingProfiles": {
      "hasMoreResults": false,
      "value": null
    },
    "departments": null,
    "displayName": "Contoso",
    "enrollmentAccounts": [
      {
        "accountName": "Contoso",
        "accountOwner": "",
        "costCenter": "Test",
        "department": null,
        "endDate": null,
        "id": "/providers/Microsoft.Billing/billingAccounts/1234567/enrollmentAccounts/7654321",
        "name": "7654321",
        "startDate": null,
        "status": null,
        "type": "Microsoft.Billing/enrollmentAccounts"
      }
    ],
    "enrollmentDetails": null,
    "hasReadAccess": false,
    "id": "/providers/Microsoft.Billing/billingAccounts/1234567",
    "name": "1234567",
    "soldTo": {
      "addressLine1": null,
      "addressLine2": null,
      "addressLine3": null,
      "city": null,
      "companyName": "Contoso",
      "country": "US ",
      "district": null,
      "email": null,
      "firstName": null,
      "lastName": null,
      "phoneNumber": null,
      "postalCode": null,
      "region": null
    },
    "type": "Microsoft.Billing/billingAccounts"
  },
```

Los valores para un ámbito de facturación y `id` son lo mismo. El `id` de la cuenta de inscripción es el ámbito de facturación en el que se inicia la solicitud de suscripción. Es importante conocer el identificador porque es un parámetro necesario que se usará más adelante en el artículo para crear una suscripción.

---

## <a name="create-subscriptions-under-a-specific-enrollment-account"></a>Creación de suscripciones con una cuenta de inscripción concreta

En el ejemplo siguiente se crea una suscripción denominada *Dev Team Subscription* en la cuenta de inscripción seleccionada en el paso anterior. 

### <a name="rest"></a>[REST](#tab/rest)

Llame a PUT API para crear una solicitud de creación de suscripción o un alias.

```json
PUT  https://management.azure.com/providers/Microsoft.Subscription/aliases/sampleAlias?api-version=2020-09-01 
```

En el cuerpo de la solicitud, proporcione el `billingScope` como el `id` de una de las `enrollmentAccounts`.

```json 
{
  "properties": {
        "billingScope": "/providers/Microsoft.Billing/BillingAccounts/1234567/enrollmentAccounts/7654321",
        "DisplayName": "Dev Team Subscription", //Subscription Display Name
        "Workload": "Production"
  }
}
```

Los valores permitidos de `Workload` son `Production` y `DevTest`.

#### <a name="response"></a>Response

```json
{
  "id": "/providers/Microsoft.Subscription/aliases/sampleAlias",
  "name": "sampleAlias",
  "type": "Microsoft.Subscription/aliases",
  "properties": {
    "subscriptionId": "b5bab918-e8a9-4c34-a2e2-ebc1b75b9d74",
    "provisioningState": "Accepted"
  }
}
```

Puede realizar una operación GET en la misma dirección URL para obtener el estado de la solicitud.

### <a name="request"></a>Solicitud

```json
GET https://management.azure.com/providers/Microsoft.Subscription/aliases/sampleAlias?api-version=2020-09-01
```

### <a name="response"></a>Response

```json
{
  "id": "/providers/Microsoft.Subscription/aliases/sampleAlias",
  "name": "sampleAlias",
  "type": "Microsoft.Subscription/aliases",
  "properties": {
    "subscriptionId": "b5bab918-e8a9-4c34-a2e2-ebc1b75b9d74",
    "provisioningState": "Succeeded"
  }
}
```

Un estado en curso se devuelve como estado `Accepted` en `provisioningState`.

### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Para instalar la versión más reciente del módulo que contiene el cmdlet `New-AzSubscriptionAlias`, ejecute `Install-Module Az.Subscription`. Para instalar una versión reciente de PowerShellGet, consulte [Obtención del módulo PowerShellGet](/powershell/scripting/gallery/installing-psget).

Ejecute el siguiente comando [New-AzSubscriptionAlias](/powershell/module/az.subscription/new-azsubscription) con el ámbito de facturación `"/providers/Microsoft.Billing/BillingAccounts/1234567/enrollmentAccounts/7654321"`. 

```azurepowershell-interactive
New-AzSubscriptionAlias -AliasName "sampleAlias" -SubscriptionName "Dev Team Subscription" -BillingScope "/providers/Microsoft.Billing/BillingAccounts/1234567/enrollmentAccounts/7654321" -Workload "Production"
```

Se obtiene subscriptionId como parte de la respuesta del comando.

```azurepowershell
{
  "id": "/providers/Microsoft.Subscription/aliases/sampleAlias",
  "name": "sampleAlias",
  "type": "Microsoft.Subscription/aliases",
  "properties": {
    "provisioningState": "Succeeded",
    "subscriptionId": "4921139b-ef1e-4370-a331-dd2229f4f510"
  }
}
```

### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

En primer lugar, instale la extensión mediante la ejecución de `az extension add --name account` y `az extension add --name alias`.

Ejecute el comando siguiente [az account alias create](/cli/azure/account/alias#az_account_alias_create) y proporcione `billing-scope` y `id` de una de las `enrollmentAccounts`. 

```azurecli-interactive
az account alias create --name "sampleAlias" --billing-scope "/providers/Microsoft.Billing/billingAccounts/1234567/enrollmentAccounts/654321" --display-name "Dev Team Subscription" --workload "Production"
```

Se obtiene subscriptionId como parte de la respuesta del comando.

```azurecli
{
  "id": "/providers/Microsoft.Subscription/aliases/sampleAlias",
  "name": "sampleAlias",
  "properties": {
    "provisioningState": "Succeeded",
    "subscriptionId": "4921139b-ef1e-4370-a331-dd2229f4f510"
  },
  "type": "Microsoft.Subscription/aliases"
}
```

---

## <a name="use-arm-template"></a>Uso de una plantilla de Resource Manager

En la sección anterior se mostró cómo crear una suscripción con PowerShell, la CLI o la API REST. Si necesita automatizar la creación de suscripciones, considere la posibilidad de usar una plantilla de Azure Resource Manager (plantilla de ARM).

El ejemplo siguiente crea una suscripción. Para `billingScope`, proporcione el identificador de la cuenta de inscripción. Para `targetManagementGroup`, proporcione el grupo de administración donde desea crear la suscripción.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-08-01/managementGroupDeploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "subscriptionAliasName": {
            "type": "string",
            "metadata": {
                "description": "Provide a name for the alias. This name will also be the display name of the subscription."
            }
        },
        "billingScope": {
            "type": "string",
            "metadata": {
                "description": "Provide the full resource ID of billing scope to use for subscription creation."
            }
        },
        "targetManagementGroup": {
            "type": "string",
            "metadata": {
                "description": "Provide the ID of the target management group to place the subscription."
            }
        }
    },
    "resources": [
        {
            "scope": "/", 
            "name": "[parameters('subscriptionAliasName')]",
            "type": "Microsoft.Subscription/aliases",
            "apiVersion": "2020-09-01",
            "properties": {
                "workLoad": "Production",
                "displayName": "[parameters('subscriptionAliasName')]",
                "billingScope": "[parameters('billingScope')]",
                "managementGroupId": "[tenantResourceId('Microsoft.Management/managementGroups/', parameters('targetManagementGroup'))]"
            }
        }
    ],
    "outputs": {}
}
```

Implemente la plantilla en el [nivel de grupo de administración](../../azure-resource-manager/templates/deploy-to-management-group.md).

### <a name="rest"></a>[REST](#tab/rest)

```json
PUT https://management.azure.com/providers/Microsoft.Management/managementGroups/mg1/providers/Microsoft.Resources/deployments/exampledeployment?api-version=2020-06-01
```

Con un cuerpo de la solicitud:

```json
{
  "location": "eastus",
  "properties": {
    "templateLink": {
      "uri": "http://mystorageaccount.blob.core.windows.net/templates/template.json"
    },
    "parameters": {
      "subscriptionAliasName": {
        "value": "sampleAlias"
      },
      "billingScope": {
        "value": "/providers/Microsoft.Billing/BillingAccounts/1234567/enrollmentAccounts/7654321"
      },
      "targetManagementGroup": {
        "value": "mg2"
      }
    },
    "mode": "Incremental"
  }
}
```

### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
New-AzManagementGroupDeployment `
  -Name exampledeployment `
  -Location eastus `
  -ManagementGroupId mg1 `
  -TemplateFile azuredeploy.json `
  -subscriptionAliasName sampleAlias `
  -billingScope "/providers/Microsoft.Billing/BillingAccounts/1234567/enrollmentAccounts/7654321" `
  -targetManagementGroup mg2
```

### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli-interactive
az deployment mg create \
  --name exampledeployment \
  --location eastus \
  --management-group-id mg1 \
  --template-file azuredeploy.json \
  --parameters subscriptionAliasName='sampleAlias' billingScope='/providers/Microsoft.Billing/BillingAccounts/1234567/enrollmentAccounts/7654321' targetManagementGroup=mg2
```

---

## <a name="limitations-of-azure-enterprise-subscription-creation-api"></a>Limitaciones de la API de creación de suscripciones de Azure Enterprise

- Con esta API solo pueden crearse suscripciones de Azure Enterprise.
- Hay un límite de 2000 suscripciones por cuenta de inscripción. Después, solo se pueden crear más suscripciones para la cuenta en Azure Portal. Para crear más suscripciones mediante la API, cree otra cuenta de inscripción. Las suscripciones canceladas, eliminadas y transferidas cuentan para el límite de 2000.
- Los usuarios que no son propietarios de cuenta, pero se han agregado a una cuenta de inscripción a través de Azure RBAC, no pueden crear suscripciones en Azure Portal.
- No puede seleccionar el inquilino para la suscripción en que se va a crear. La suscripción siempre se crea en el inquilino principal del propietario de cuenta. Para mover la suscripción a otro inquilino, vea cómo [cambiar un inquilino de la suscripción](../../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md).


## <a name="next-steps"></a>Pasos siguientes

* Ahora que ha creado una suscripción, puede conceder dicha capacidad a otros usuarios y entidades de servicio. Para más información, vea [Concesión de acceso para crear suscripciones de EA (versión preliminar)](grant-access-to-create-subscription.md).
* Para más información sobre la administración de grandes cantidades de suscripciones mediante grupos de administración, consulte [Organización de los recursos con grupos de administración de Azure](../../governance/management-groups/overview.md).
* Para cambiar el grupo de administración de una suscripción, consulte [Movimiento de suscripciones](../../governance/management-groups/manage.md#move-subscriptions).
