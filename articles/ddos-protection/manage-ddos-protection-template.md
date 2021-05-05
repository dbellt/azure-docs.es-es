---
title: Cree y habilite un plan de Azure DDoS Protection mediante una plantilla de Azure Resource Manager (plantilla de ARM).
description: Aprenda a crear y habilitar un plan de Azure DDoS Protection mediante una plantilla de Azure Resource Manager (plantilla de ARM).
services: ddos-protection
documentationcenter: na
author: mumian
ms.service: ddos-protection
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.custom: subject-armqs
ms.author: jgao
ms.date: 04/26/2021
ms.openlocfilehash: 732606c24b116edb3f89f9f60db2bd5621d90e63
ms.sourcegitcommit: 43be2ce9bf6d1186795609c99b6b8f6bb4676f47
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/29/2021
ms.locfileid: "108277415"
---
# <a name="quickstart-create-an-azure-ddos-protection-standard-using-arm-template"></a>Inicio rápido: Creación de una instancia de Azure DDoS Protection Estándar mediante la plantilla de ARM

En este inicio rápido se describe cómo usar una plantilla de Azure Resource Manager (plantilla de ARM) para crear un plan de protección contra la denegación de servicio distribuido (DDoS) y una red virtual y, después, habilita el plan de protección para la red virtual. Un plan de Azure DDoS Protection Estándar define un conjunto de redes virtuales que tiene habilitada la protección contra DDoS en distintas suscripciones. Puede configurar un plan de protección contra DDoS para la organización y vincular redes virtuales de distintas suscripciones al mismo plan.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Si su entorno cumple los requisitos previos y está familiarizado con el uso de plantillas de Resource Manager, seleccione el botón **Implementar en Azure**. La plantilla se abrirá en Azure Portal.

[![Implementación en Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fquickstarts%2Fmicrosoft.network%2Fcreate-and-enable-ddos-protection-plans%2Fazuredeploy.json)

## <a name="prerequisites"></a>Requisitos previos

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="review-the-template"></a>Revisión de la plantilla

La plantilla usada en este inicio rápido forma parte de las [plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/101-create-and-enable-ddos-protection-plans).

:::code language="json" source="~/quickstart-templates/quickstarts/microsoft.network/create-and-enable-ddos-protection-plans/azuredeploy.json":::

La plantilla define dos recursos:

- [Microsoft.Network/ddosProtectionPlans](/azure/templates/microsoft.network/ddosprotectionplans)
- [Microsoft.Network/virtualNetworks](/azure/templates/microsoft.network/virtualnetworks)

## <a name="deploy-the-template"></a>Implementación de la plantilla

En este ejemplo, la plantilla crea un nuevo grupo de recursos, un plan de protección contra DDoS y una red virtual.

1. Para iniciar sesión en Azure y abrir la plantilla, seleccione el botón **Implementar en Azure**.

    [![Implementación en Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fquickstarts%2Fmicrosoft.network%2Fcreate-and-enable-ddos-protection-plans%2Fazuredeploy.json)

1. Escriba los valores para crear un nuevo grupo de recursos, un plan de protección contra DDoS y un nombre de red virtual.

    :::image type="content" source="media/manage-ddos-protection-template/ddos-template.png" alt-text="Plantilla de inicio rápido de DDoS":::

    - **Suscripción**: nombre de la suscripción de Azure en la que se implementarán los recursos.
    - **Grupo de recursos**: Seleccione un grupo de recursos existente o cree uno nuevo.
    - **Región**: la región donde se implementa el grupo de recursos, por ejemplo, Este de EE. UU.
    - **Nombre el plan de DDoS Protection**: el nombre del nuevo plan de DDoS Protection.
    - **Nombre de la red virtual**: crea un nombre para la red virtual nueva.
    - **Ubicación**: función que utiliza la misma región que el grupo de recursos para la implementación de recursos.
    - **Prefijo de dirección de la red virtual**: use el valor predeterminado o escriba la dirección de la red virtual.
    - **Prefijo de la subred**: use el valor predeterminado o escriba la subred de la red virtual.
    - **Plan de DDoS Protection habilitado**: el valor predeterminado es `true` para habilitar el plan de protección contra DDoS.

1. Seleccione **Revisar + crear**.
1. Compruebe que se ha superado la validación de la plantilla y seleccione **Crear** para comenzar la implementación.

## <a name="review-deployed-resources"></a>Revisión de los recursos implementados

Para copiar el comando de la CLI de Azure o Azure PowerShell, seleccione el botón **Copiar**. El botón **Probar** abre Azure Cloud Shell para ejecutar el comando.

# <a name="cli"></a>[CLI](#tab/CLI)

```azurecli-interactive
az network ddos-protection show \
  --resource-group MyResourceGroup \
  --name MyDdosProtectionPlan
```

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

```azurepowershell-interactive
Get-AzDdosProtectionPlan -ResourceGroupName 'MyResourceGroup' -Name 'MyDdosProtectionPlan'
```

---

La salida muestra los nuevos recursos.

# <a name="cli"></a>[CLI](#tab/CLI)

```Output
{
  "etag": "W/\"abcdefgh-1111-2222-bbbb-987654321098\"",
  "id": "/subscriptions/b1111111-2222-3333-aaaa-012345678912/resourceGroups/MyResourceGroup/providers/Microsoft.Network/ddosProtectionPlans/MyDdosProtectionPlan",
  "location": "eastus",
  "name": "MyDdosProtectionPlan",
  "provisioningState": "Succeeded",
  "resourceGroup": "MyResourceGroup",
  "resourceGuid": null,
  "tags": null,
  "type": "Microsoft.Network/ddosProtectionPlans",
  "virtualNetworks": [
    {
      "id": "/subscriptions/b1111111-2222-3333-aaaa-012345678912/resourceGroups/MyResourceGroup/providers/Microsoft.Network/virtualNetworks/MyVNet",
      "resourceGroup": "MyResourceGroup"
    }
  ]
}
```

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

```Output
Name              : MyDdosProtectionPlan
Id                : /subscriptions/b1111111-2222-3333-aaaa-012345678912/resourceGroups/MyResourceGroup/providers/Microsoft.Network/ddosProtectionPlans/MyDdosProtectionPlan
Etag              : W/"abcdefgh-1111-2222-bbbb-987654321098"
ProvisioningState : Succeeded
VirtualNetworks   : [
                      {
                        "Id": "/subscriptions/b1111111-2222-3333-aaaa-012345678912/resourceGroups/MyResourceGroup/providers/Microsoft.Network/virtualNetworks/MyVNet"
                      }
                    ]
```

---

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando haya terminado, puede eliminar los recursos. Este comando elimina el grupo de recursos y todos los recursos que contiene.

# <a name="cli"></a>[CLI](#tab/CLI)

```azurecli-interactive
az group delete --name MyResourceGroup
```

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

```azurepowershell-interactive
Remove-AzResourceGroup -Name 'MyResourceGroup'
```

---

## <a name="next-steps"></a>Pasos siguientes

Para obtener información sobre cómo ver y configurar la telemetría del plan de protección contra DDoS, continúe con los tutoriales.

> [!div class="nextstepaction"]
> [Visualización y configuración de la telemetría de protección contra DDoS](telemetry.md)
