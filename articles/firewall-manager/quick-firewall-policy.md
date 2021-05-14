---
title: 'Inicio rápido: Creación de una instancia de Azure Firewall y una directiva de firewall: plantilla de Resource Manager'
description: En este inicio rápido, se implementan una instancia de Azure Firewall y una directiva de firewall.
services: firewall-manager
author: vhorne
ms.author: victorh
ms.date: 02/17/2021
ms.topic: quickstart
ms.service: firewall-manager
ms.custom:
- subject-armqs
- mode-arm
ms.openlocfilehash: 65a1e966fcc6082d4034cd9fe4ee45c057496825
ms.sourcegitcommit: 32ee8da1440a2d81c49ff25c5922f786e85109b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109786220"
---
# <a name="quickstart-create-an-azure-firewall-and-a-firewall-policy---arm-template"></a>Inicio rápido: Creación de una instancia de Azure Firewall y una directiva de firewall: plantilla de Azure Resource Manager

En este inicio rápido, se usa una plantilla de Azure Resource Manager para crear una instancia de Azure Firewall y una directiva de firewall. La directiva de firewall tiene una regla de aplicación que permite establecer conexiones con `www.microsoft.com` y una regla que permite establecer conexiones con Windows Update mediante la etiqueta del nombre de dominio completo **WindowsUpdate**. Una regla de red permite realizar conexiones UDP a un servidor horario en 13.86.101.172.

Además, se usan grupos de direcciones IP en las reglas para definir las direcciones IP de **origen**.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Para obtener información sobre Azure Firewall Manager, consulte [¿Qué es Azure Firewall Manager?](overview.md)

Para obtener información sobre Azure Firewall, consulte [¿Qué es Azure Firewall?](../firewall/overview.md)

Para obtener información sobre los grupos de IP, consulte [Grupos de IP Groups en Azure Firewall](../firewall/ip-groups.md).

Si su entorno cumple los requisitos previos y está familiarizado con el uso de plantillas de Resource Manager, seleccione el botón **Implementar en Azure**. La plantilla se abrirá en Azure Portal.

[![Implementación en Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-azurefirewall-create-with-firewallpolicy-apprule-netrule-ipgroups%2Fazuredeploy.json)

## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

## <a name="review-the-template"></a>Revisión de la plantilla

Esta plantilla crea una red virtual de centro, junto con los recursos necesarios para admitir el escenario.

La plantilla usada en este inicio rápido forma parte de las [plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/101-azurefirewall-create-with-firewallpolicy-apprule-netrule-ipgroups/).

:::code language="json" source="~/quickstart-templates/quickstarts/microsoft.network/azurefirewall-create-with-firewallpolicy-apprule-netrule-ipgroups/azuredeploy.json":::

En la plantilla se definen varios recursos de Azure:

- [**Microsoft.Network/ipGroups**](/azure/templates/microsoft.network/ipGroups)
- [**Microsoft.Network/firewallPolicies**](/azure/templates/microsoft.network/firewallPolicies)
- [**Microsoft.Network/firewallPolicies/ruleCollectionGroups**](/azure/templates/microsoft.network/firewallPolicies/ruleCollectionGroups)
- [**Microsoft.Network/azureFirewalls**](/azure/templates/microsoft.network/azureFirewalls)
- [**Microsoft.Network/virtualNetworks**](/azure/templates/microsoft.network/virtualnetworks)
- [**Microsoft.Network/publicIPAddresses**](/azure/templates/microsoft.network/publicipaddresses)

## <a name="deploy-the-template"></a>Implementación de la plantilla

Implementación de la plantilla de Resource Manager en Azure:

1. Seleccione **Implementar en Azure** para iniciar sesión en Azure y abrir la plantilla. La plantilla crea una instancia de Azure Firewall, una WAN virtual y un centro virtual, la infraestructura de red y dos máquinas virtuales.

   [![Implementación en Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-azurefirewall-create-with-firewallpolicy-apprule-netrule-ipgroups%2Fazuredeploy.json)

2. En el portal, en la página **Create a Firewall and FirewallPolicy with Rules and Ipgroups** (Crear una instancia de Azure Firewall y una directiva de firewall con reglas y grupos de IP), escriba o seleccione los valores siguientes:
   - Suscripción: seleccione una de las suscripciones existentes.
   - Grupo de recursos:  seleccione uno de los grupos de recursos existentes o elija **Crear nuevo** y, luego, **Aceptar**.
   - Región: Seleccione una región.
   - Nombre del firewall: escriba un nombre para el firewall.

3. Seleccione **Revisar y crear** y, luego, **Crear**. La implementación puede tardar 10 minutos o más en completarse.

## <a name="review-deployed-resources"></a>Revisión de los recursos implementados

Una vez finalizada la implementación, verá los siguientes recursos similares.

:::image type="content" source="media/quick-firewall-policy/qs-deployed-resources.png" alt-text="Recursos implementados":::

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no necesite los recursos que ha creado con el firewall, elimine el grupo de recursos. Esta acción quita el firewall y todos los recursos relacionados.

Para eliminar el grupo de recursos, llame al cmdlet `Remove-AzResourceGroup`:

```azurepowershell-interactive
Remove-AzResourceGroup -Name "<your resource group name>"
```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Información general sobre la directiva de Azure Firewall Manager](policy-overview.md)
