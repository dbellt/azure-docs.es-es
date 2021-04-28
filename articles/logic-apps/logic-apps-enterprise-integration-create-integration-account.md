---
title: Creación o administración de cuentas de integración B2B
description: Creación, vinculación y administración de cuentas de integración para la integración empresarial con Azure Logic Apps
services: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.author: divswa
ms.reviewer: estfan, logicappspm
ms.topic: conceptual
ms.date: 11/04/2020
ms.openlocfilehash: ae5ca6ac822dabd32b6463c3a742901f32b34323
ms.sourcegitcommit: 2aeb2c41fd22a02552ff871479124b567fa4463c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107862262"
---
# <a name="create-and-manage-integration-accounts-for-b2b-enterprise-integrations-in-azure-logic-apps"></a>Creación y administración de cuentas de integración para la integración empresarial B2B en Azure Logic Apps

Para poder compilar [soluciones B2B y de integración empresarial](../logic-apps/logic-apps-enterprise-integration-overview.md) mediante [Azure Logic Apps](../logic-apps/logic-apps-overview.md), debe crear una cuenta de integración, que es un recurso de Azure independiente que proporciona un contenedor seguro, escalable y administrable para los artefactos de integración que el usuario define y usa con los flujos de trabajo de la aplicación lógica.

Por ejemplo, puede crear, almacenar y administrar artefactos B2B, tales como asociados comerciales, contratos, mapas, esquemas, certificados y configuraciones de lotes. Además, para que la aplicación lógica pueda trabajar con estos artefactos y usar los conectores de Logic Apps B2B, debe [vincular su cuenta de integración](#link-account) a la aplicación lógica. La cuenta de integración y la aplicación lógica deben estar en la *misma* ubicación o región.

> [!IMPORTANT]
> En función del tipo de cuenta de integración que seleccione, la creación de una cuenta de integración conlleva un costo. Para más información, vea [Modelos de precios y facturación en Azure Logic Apps](logic-apps-pricing.md#integration-accounts) y [Precios de Azure Logic Apps](https://azure.microsoft.com/pricing/details/logic-apps/).

En este tema se muestra cómo realizar estas tareas:

* Crear la cuenta de integración.

  > [!TIP]
  > Para crear una cuenta de integración en un [entorno de servicio de integración](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md), consulte [Creación de cuentas de integración en un ISE](../logic-apps/add-artifacts-integration-service-environment-ise.md#create-integration-account-environment).

* Vincular la cuenta de integración a una aplicación lógica

* Cambiar el plan de tarifa de la cuenta de integración.

* Desvincular la cuenta de integración de una aplicación lógica.

* Mover la cuenta de integración a otra suscripción o grupo de recursos de Azure.

* Eliminar la cuenta de integración.

## <a name="prerequisites"></a>Prerrequisitos

* Una cuenta y una suscripción de Azure. Si no tiene una suscripción de Azure, [regístrese para obtener una cuenta gratuita de Azure](https://azure.microsoft.com/free/).

## <a name="create-integration-account"></a>Creación de una cuenta de integración

### <a name="portal"></a>[Portal](#tab/azure-portal)

Para esta tarea, puede usar Azure Portal si sigue los pasos de esta sección, [Azure PowerShell](/powershell/module/Az.LogicApp/New-AzIntegrationAccount) o la [CLI de Azure](/cli/azure/resource#az_resource_create).

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con sus credenciales de su cuenta de Azure.

1. En el menú principal de Azure, seleccione **Crear un recurso**. En el cuadro de búsqueda, especifique "cuenta de integración" como filtro y seleccione **Cuenta de integración**.

   ![Creación de una cuenta de integración](./media/logic-apps-enterprise-integration-create-integration-account/create-integration-account.png)

1. En **Cuenta de integración**, seleccione **Crear**.

   ![Elija "Agregar" para crear una cuenta de integración](./media/logic-apps-enterprise-integration-create-integration-account/add-integration-account.png)

1. Proporcione esta información sobre la cuenta de integración:

   ![Proporcionar los detalles de la cuenta de integración](./media/logic-apps-enterprise-integration-create-integration-account/integration-account-details.png)

   | Propiedad | Obligatorio | Value | Descripción |
   |----------|----------|-------|-------------|
   | **Nombre** | Sí | <*integration-account-name*> | El nombre de la cuenta de integración, que solo puede contener letras, números, guiones (`-`), caracteres de subrayado (`_`), paréntesis (`(`,`)`) y puntos (`.`). En este ejemplo se usa "Fabrikam-Integration". |
   | **Suscripción** | Sí | <*Azure-subscription-name*> | El nombre de la suscripción a Azure |
   | **Grupos de recursos** | Sí | <*nombre del grupo de recursos de Azure*> | Nombre del [grupo de recursos de Azure](../azure-resource-manager/management/overview.md) que se va a usar para organizar recursos relacionados. Para este ejemplo, cree un grupo de recursos con el nombre "FabrikamIntegration-RG". |
   | **Plan de tarifa** | Sí | <*pricing-level*> | Plan de tarifa de la cuenta de integración, que puede cambiar más adelante. En este ejemplo, seleccione **Gratis**. Para más información, consulte los temas siguientes: <p>- [Modelo de precios de Logic Apps](../logic-apps/logic-apps-pricing.md#integration-accounts) <p>- [Límites y configuración de Logic Apps](../logic-apps/logic-apps-limits-and-config.md#integration-account-limits) <p>- [Precios de Logic Apps](https://azure.microsoft.com/pricing/details/logic-apps/) |
   | **Ubicación** | Sí | <*Azure-region*> | Región en la que quiere almacenar los metadatos de la cuenta de integración. Seleccione la misma ubicación que la aplicación lógica o cree las aplicaciones lógicas en la misma ubicación que la cuenta de integración. En este ejemplo, use "Oeste de EE. UU.". <p>**Nota**: Para crear una cuenta de integración en un [entorno de servicio de integración (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md), seleccione ese ISE como ubicación. Para obtener más información, consulte [Creación de cuentas de integración en un ISE](../logic-apps/add-artifacts-integration-service-environment-ise.md#create-integration-account-environment). |
   | **Log Analytics** | No | Activado, desactivado | Mantenga el valor **Activado** para este ejemplo. |
   |||||

1. Cuando haya finalizado, seleccione **Crear**.

   Una vez completada la implementación, Azure abre la cuenta de integración.

   ![Azure abre la cuenta de integración](./media/logic-apps-enterprise-integration-create-integration-account/integration-account-created.png)

1. Antes de que la aplicación lógica pueda usar su cuenta de integración, siga los pasos que se indican a continuación para vincular la cuenta de integración y la aplicación lógica.

### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Puede crear una cuenta de integración mediante los comandos de la CLI de Azure de esta sección.

[!INCLUDE [azure-cli-prepare-your-environment-h3.md](../../includes/azure-cli-prepare-your-environment-h3.md)]

### <a name="create-an-integration-account"></a>Creación de una cuenta de integración

Use estos comandos para crear una cuenta de integración.

1. Para agregar la extensión [az logic integration-account](/cli/azure/logic/integration-account), use el comando [az extension add](/cli/azure/extension#az_extension_add):

   ```azurecli
   az extension add –-name logic
   ```

1. Para crear un grupo de recursos o usar uno ya existente, ejecute el comando [az group create](/cli/azure/group#az_group_create):

   ```azurecli
   az group create --name myresourcegroup --location westus
   ```

   Para enumerar las cuentas de integración de un grupo de recursos, use el comando [az logic integration-account list](/cli/azure/logic/integration-account#az_logic_integration_account_list):

   ```azurecli
   az logic integration-account list --resource-group myresourcegroup
   ```

1. Para crear una cuenta de integración, ejecute el comando [az logic integration-account create](/cli/azure/logic/integration-account#az_logic_integration_account_create):

   ```azurecli
   az logic integration-account create --resource-group myresourcegroup \
       --name integration_account_01 --location westus --sku name=Standard
   ```

   El nombre de la cuenta de integración solo puede contener letras, números, guiones (-), guiones bajos (_), paréntesis ((,)) y puntos (.).

   > [!TIP]
   > Para crear una cuenta de integración en un [entorno de servicio de integración (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md), seleccione ese ISE como ubicación. Para obtener más información, consulte [Creación de cuentas de integración en un ISE](../logic-apps/add-artifacts-integration-service-environment-ise.md#create-integration-account-environment).

   Para ver una cuenta de integración concreta, use el comando [az logic integration-account show](/cli/azure/logic/integration-account#az_logic_integration_account_show):

   ```azurecli
   az logic integration-account show --name integration_account_01 --resource-group myresourcegroup
   ```

   Puede cambiar la SKU o el plan de tarifa mediante el comando [az logic integration-account update](/cli/azure/logic/integration-account#az_logic_integration_account_update):

   ```azurecli
   az logic integration-account update --sku name=Basic --name integration_account_01 \
       --resource-group myresourcegroup
   ```

   Para más información sobre los precios, consulte estos recursos:

   * [Modelo de precios de Logic Apps](../logic-apps/logic-apps-pricing.md#integration-accounts)
   * [Límites y configuración de Logic Apps](../logic-apps/logic-apps-limits-and-config.md#integration-account-limits)
   * [Precios de Logic Apps](https://azure.microsoft.com/pricing/details/logic-apps/)

Para importar una cuenta de integración mediante un archivo JSON, use el comando [az logic integration-account import](/cli/azure/logic/integration-account#az_logic_integration_account_import):

```azurecli
az logic integration-account import --name integration_account_01 \
    --resource-group myresourcegroup --input-path integration.json
```

Puede eliminar una cuenta de integración mediante el comando [az logic integration-account delete](/cli/azure/logic/integration-account#az_logic_integration_account_delete):

```azurecli
az logic integration-account delete --name integration_account_01 --resource-group myresourcegroup
```

Antes de que la aplicación lógica pueda usar su cuenta de integración, vincule la cuenta de integración con la aplicación lógica. En la siguiente sección se describe este paso.

---
<a name="link-account"></a>

## <a name="link-to-logic-app"></a>Vincular a la aplicación lógica

Para que las aplicaciones lógicas accedan a una cuenta de integración que contiene los artefactos B2B, primero debe vincular la cuenta de integración a la aplicación lógica. La cuenta de integración y la aplicación lógica deben estar en la misma región. Para completar esta tarea, puede utilizar Azure Portal. Si usa Visual Studio y la aplicación lógica está en un [proyecto de grupo de recursos de Azure](../azure-resource-manager/templates/create-visual-studio-deployment-project.md), puede [vincular la aplicación lógica a una cuenta de integración mediante Visual Studio](../logic-apps/manage-logic-apps-with-visual-studio.md#link-integration-account).

1. En Azure Portal, busque y abra la aplicación lógica.

1. En [Azure Portal](https://portal.azure.com), abra una aplicación lógica existente o cree una.

1. En el menú de la aplicación lógica, en **Configuración**, seleccione **Configuración de flujo de trabajo**. En **Cuenta de integración**, abra la lista **Seleccione una cuenta de integración**. Seleccione la cuenta de integración que quiere vincular a la aplicación lógica.

   ![Selección de la cuenta de integración](./media/logic-apps-enterprise-integration-create-integration-account/select-integration-account.png)

1. Para finalizar la vinculación, seleccione **Guardar**.

   ![Captura de pantalla que muestra dónde seleccionar Guardar para elegir la cuenta de integración.](./media/logic-apps-enterprise-integration-create-integration-account/save-link.png)

   Una vez que la cuenta de integración se haya vinculado correctamente, Azure muestra un mensaje de confirmación.

   ![Azure confirma el vínculo correcto](./media/logic-apps-enterprise-integration-create-integration-account/link-confirmation.png)

Ahora, la aplicación lógica puede usar los artefactos de la cuenta de integración, además de conectores B2B, como la validación XML y la codificación o descodificación de archivos sin formato.  

<a name="change-pricing-tier"></a>

## <a name="change-pricing-tier"></a>Cambiar el plan de tarifa

Para aumentar los [límites](../logic-apps/logic-apps-limits-and-config.md#integration-account-limits) de una cuenta de integración, puede [actualizar a un plan de tarifa superior](#upgrade-pricing-tier), si está disponible. Por ejemplo, puede actualizar del nivel Gratis al nivel básico o estándar. También puede [cambiar a un nivel inferior](#downgrade-pricing-tier), si está disponible. Para obtener más información sobre los precios, consulte estos temas:

* [Precios de Logic Apps](https://azure.microsoft.com/pricing/details/logic-apps/)
* [Modelo de precios de Logic Apps](../logic-apps/logic-apps-pricing.md#integration-accounts)

<a name="upgrade-pricing-tier"></a>

### <a name="upgrade-pricing-tier"></a>Actualización del plan de tarifa

Para realizar este cambio, puede usar Azure Portal o la CLI de Azure.

#### <a name="portal"></a>[Portal](#tab/azure-portal)

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con sus credenciales de su cuenta de Azure.

1. En el cuadro de búsqueda principal de Azure, especifique "cuentas de integración" como filtro y seleccione **Cuentas de integración**.

   ![Búsqueda de la cuenta de integración](./media/logic-apps-enterprise-integration-create-integration-account/find-integration-account.png)

   Azure muestra todas las cuentas de integración en las suscripciones de Azure.

1. En **Cuentas de integración**, seleccione la cuenta de integración que quiera mover. En el menú de la cuenta de integración, seleccione **Información general**.

   ![En el menú de la cuenta de integración, seleccione "Información general".](./media/logic-apps-enterprise-integration-create-integration-account/integration-account-overview.png)

1. En el panel Información general, seleccione **Actualizar plan de tarifa**, donde se enumeran todos los niveles superiores disponibles. Al seleccionar un nivel, el cambio surte efecto de inmediato.

<a name="upgrade-tier-azure-cli"></a>

#### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

1. Si aún no lo ha hecho, [instale los requisitos previos de la CLI de Azure](/cli/azure/get-started-with-azure-cli).

1. En Azure Portal, abra el entorno de [Azure Cloud Shell](../cloud-shell/overview.md).

   ![Apertura de Azure Cloud Shell](./media/logic-apps-enterprise-integration-create-integration-account/open-azure-cloud-shell-window.png)

1. En el símbolo del sistema, escriba el comando [**az resource**](/cli/azure/resource#az_resource_update) y establezca `skuName` en el nivel superior que quiera.

   ```azurecli
   az resource update --resource-group {ResourceGroupName} --resource-type Microsoft.Logic/integrationAccounts --name {IntegrationAccountName} --subscription {AzureSubscriptionID} --set sku.name={SkuName}
   ```
  
   Por ejemplo, si tiene el nivel básico, puede establecer `skuName` en `Standard`:

   ```azurecli
   az resource update --resource-group FabrikamIntegration-RG --resource-type Microsoft.Logic/integrationAccounts --name Fabrikam-Integration --subscription XXXXXXXXXXXXXXXXX --set sku.name=Standard
   ```

---

<a name="downgrade-pricing-tier"></a>

### <a name="downgrade-pricing-tier"></a>Cambio del plan de tarifa a un nivel inferior

Para cambiarlo, use la [CLI de Azure](/cli/azure/get-started-with-azure-cli).

1. Si aún no lo ha hecho, [instale los requisitos previos de la CLI de Azure](/cli/azure/get-started-with-azure-cli).

1. En Azure Portal, abra el entorno de [Azure Cloud Shell](../cloud-shell/overview.md).

   ![Apertura de Azure Cloud Shell](./media/logic-apps-enterprise-integration-create-integration-account/open-azure-cloud-shell-window.png)

1. En el símbolo del sistema, escriba el comando [**az resource**](/cli/azure/resource#az_resource_update) y establezca `skuName` en el nivel inferior que quiera.

   ```azurecli
   az resource update --resource-group <resourceGroupName> --resource-type Microsoft.Logic/integrationAccounts --name <integrationAccountName> --subscription <AzureSubscriptionID> --set sku.name=<skuName>
   ```
  
   Por ejemplo, si tiene el nivel estándar, puede establecer `skuName` en `Basic`:

   ```azurecli
   az resource update --resource-group FabrikamIntegration-RG --resource-type Microsoft.Logic/integrationAccounts --name Fabrikam-Integration --subscription XXXXXXXXXXXXXXXXX --set sku.name=Basic
   ```

## <a name="unlink-from-logic-app"></a>Desvinculación de aplicación lógica

Si quiere vincular la aplicación lógica a otra cuenta de integración o dejar de usar una cuenta de integración con la aplicación lógica, elimine el vínculo mediante Azure Resource Explorer.

1. Abra la ventana del explorador y vaya a [Azure Resource Explorer (https://resources.azure.com)](https://resources.azure.com). Inicie sesión con las mismas credenciales de la cuenta de Azure.

   ![Azure Resource Explorer](./media/logic-apps-enterprise-integration-create-integration-account/resource-explorer.png)

1. En el cuadro de búsqueda, escriba el nombre de la aplicación lógica para poder buscar y seleccionar la aplicación lógica.

   ![Buscar y seleccionar la aplicación lógica](./media/logic-apps-enterprise-integration-create-integration-account/resource-explorer-find-logic-app.png)

1. En la barra de título del explorador, seleccione **Lectura/escritura**.

   ![Activar el modo de "lectura/escritura"](./media/logic-apps-enterprise-integration-create-integration-account/resource-explorer-select-read-write.png)

1. En la pestaña **Datos**, seleccione **Editar**.

   ![En la pestaña "Datos", seleccione "Editar".](./media/logic-apps-enterprise-integration-create-integration-account/resource-explorer-select-edit.png)

1. En el editor, busque el objeto `integrationAccount` y elimine esa propiedad, que tiene este formato:

   ```json
   {
      // <other-attributes>
      "integrationAccount": {
         "name": "<integration-account-name>",
         "id": "<integration-account-resource-ID>",
         "type": "Microsoft.Logic/integrationAccounts"  
   },
   ```

   Por ejemplo:

   ![Búsqueda del objeto "integrationAccount"](./media/logic-apps-enterprise-integration-create-integration-account/resource-explorer-delete-integration-account.png)

1. En la pestaña **Datos**, seleccione **Colocar** para guardar los cambios.

   ![Para guardar los cambios, seleccione "Colocar".](./media/logic-apps-enterprise-integration-create-integration-account/resource-explorer-save-changes.png)

1. En Azure Portal, busque y seleccione la aplicación lógica. En la **configuración del flujo de trabajo** de la aplicación, compruebe que la propiedad **Cuenta de integración** esté vacía.

   ![Comprobar que la cuenta de integración no está vinculada](./media/logic-apps-enterprise-integration-create-integration-account/unlinked-account.png)

## <a name="move-integration-account"></a>Mover la cuenta de integración

Puede mover la cuenta de integración a otro grupo de recursos de Azure o suscripción a Azure. Al mover recursos, Azure crea nuevos identificadores de recurso, por lo que debe asegurarse de usar los nuevos identificadores en su lugar y actualizar los scripts o las herramientas asociados a los recursos que ha movido. Si quiere cambiar la suscripción, también debe especificar un grupo de recursos nuevo o existente.

Para esta tarea, puede usar Azure Portal si sigue los pasos de esta sección o la [CLI de Azure](/cli/azure/resource#az_resource_move).

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con sus credenciales de su cuenta de Azure.

1. En el cuadro de búsqueda principal de Azure, especifique "cuentas de integración" como filtro y seleccione **Cuentas de integración**.

   ![Búsqueda de la cuenta de integración](./media/logic-apps-enterprise-integration-create-integration-account/find-integration-account.png)

   Azure muestra todas las cuentas de integración en las suscripciones de Azure.

1. En **Cuentas de integración**, seleccione la cuenta de integración que quiera mover. En el menú de la cuenta de integración, seleccione **Información general**.

   ![En el menú de la cuenta de integración, seleccione "Información general".](./media/logic-apps-enterprise-integration-create-integration-account/integration-account-overview.png)

1. Junto a **Grupo de recursos** o **Nombre de la suscripción**, seleccione **cambiar**.

   ![Cambio del grupo de recursos o la suscripción](./media/logic-apps-enterprise-integration-create-integration-account/change-resource-group-subscription.png)

1. Seleccione los recursos relacionados que también quiera mover.

1. En función de lo que seleccione, siga estos pasos para cambiar el grupo de recursos o la suscripción:

   * Grupo de recursos: En la lista **Grupo de recursos**, seleccione el grupo de recursos de destino. O bien, para crear otro grupo de recursos, seleccione **Crear un nuevo grupo de recursos**.

   * Suscripción: En la lista **Suscripción**, seleccione la suscripción de destino. En la lista **Grupo de recursos**, seleccione el grupo de recursos de destino. O bien, para crear otro grupo de recursos, seleccione **Crear un nuevo grupo de recursos**.

1. Para confirmar que comprende que los scripts o las herramientas asociados a los recursos que se han movido no funcionarán hasta que los actualice con los nuevos identificadores de recurso, seleccione el cuadro de confirmación y después seleccione **Aceptar**.

1. Cuando finalice, asegúrese de actualizar todos los scripts con los nuevos identificadores de los recursos que ha movido.  

## <a name="delete-integration-account"></a>Eliminar una cuenta de integración

Para esta tarea, puede usar Azure Portal si sigue los pasos de esta sección, la [CLI de Azure](/cli/azure/resource#az_resource_delete) o [Azure PowerShell](/powershell/module/az.logicapp/remove-azintegrationaccount).

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con sus credenciales de su cuenta de Azure.

1. En el cuadro de búsqueda principal de Azure, especifique "cuentas de integración" como filtro y seleccione **Cuentas de integración**.

   ![Búsqueda de la cuenta de integración](./media/logic-apps-enterprise-integration-create-integration-account/find-integration-account.png)

   Azure muestra todas las cuentas de integración en las suscripciones de Azure.

1. En **Cuentas de integración**, seleccione la cuenta de integración que quiera eliminar. En el menú de la cuenta de integración, seleccione **Información general**.

   ![En el menú de la cuenta de integración, seleccione "Información general".](./media/logic-apps-enterprise-integration-create-integration-account/integration-account-overview.png)

1. En el panel Información general, seleccione **Eliminar**.

   ![En el panel "Información general", seleccione "Eliminar".](./media/logic-apps-enterprise-integration-create-integration-account/delete-integration-account.png)

1. Para confirmar que quiere eliminar la cuenta de integración, seleccione **Sí**.

   ![Para confirmar la eliminación, seleccione "Sí".](./media/logic-apps-enterprise-integration-create-integration-account/confirm-delete.png)

## <a name="next-steps"></a>Pasos siguientes

* [Creación de asociados comerciales en la cuenta de integración](../logic-apps/logic-apps-enterprise-integration-partners.md)
* [Creación de contratos entre asociados en la cuenta de integración](../logic-apps/logic-apps-enterprise-integration-agreements.md)
