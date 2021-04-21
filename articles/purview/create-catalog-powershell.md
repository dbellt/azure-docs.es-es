---
title: 'Inicio rápido: Creación de una cuenta de Azure Purview mediante Azure PowerShell o la CLI de Azure (versión preliminar)'
description: En este inicio rápido se describe cómo crear una cuenta de Azure Purview mediante Azure PowerShell o la CLI de Azure.
author: hophanms
ms.author: hophan
ms.date: 11/23/2020
ms.topic: quickstart
ms.service: purview
ms.subservice: purview-data-catalog
ms.custom:
- mode-api
ms.openlocfilehash: 6266aedaec8f171a1a6ff3e0d15abdad0263767a
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107530877"
---
# <a name="quickstart-create-an-azure-purview-account-using-azure-powershellazure-cli"></a>Inicio rápido: Creación de una cuenta de Azure Purview mediante Azure PowerShell o la CLI de Azure

> [!IMPORTANT]
> Azure Purview se encuentra actualmente en VERSIÓN PRELIMINAR. Los [Términos de uso complementarios para las versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) incluyen términos legales adicionales que se aplican a las características de Azure que se encuentran en la versión beta, en versión preliminar o que todavía no se han publicado para que estén disponibles con carácter general.

En este inicio rápido se crea una cuenta de Azure Purview mediante Azure PowerShell o la CLI de Azure.

## <a name="prerequisites"></a>Requisitos previos

* Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

* La cuenta de usuario que use para iniciar sesión en Azure debe ser miembro de los roles Colaborador o Propietario, o bien ser administrador de la suscripción de Azure.

* Su propio [inquilino de Azure Active Directory](../active-directory/fundamentals/active-directory-access-create-new-tenant.md).

* Instale Azure PowerShell o la CLI de Azure en la máquina cliente para implementar la plantilla: [Implementación desde la línea de comandos](../azure-resource-manager/templates/template-tutorial-create-first-template.md?tabs=azure-cli#command-line-deployment)

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en [Azure Portal](https://portal.azure.com) con su cuenta de Azure.

## <a name="configure-your-subscription"></a>Configuración de su suscripción

Si fuera necesario, siga estos pasos para configurar una suscripción para que Azure Purview se pueda ejecutar en ella:

   1. En Azure Portal, seleccione **Suscripciones**.

   1. En la lista de suscripciones, seleccione la que desee usar. Se requiere el permiso de acceso administrativo para la suscripción.

      :::image type="content" source="./media/create-catalog-portal/select-subscription.png" alt-text="Captura de pantalla que muestra cómo seleccionar una suscripción en Azure Portal.":::

   1. Para la suscripción, seleccione **Proveedores de recursos**. En el panel **Proveedores de recursos**, busque los tres proveedores de recursos siguientes y regístrelos: 
       1. **Microsoft.Purview**
       1. **Microsoft.Storage**
       1. **Microsoft.EventHub** 
      
      Si no están registrados, regístrelos ahora, para lo que debe seleccionar **Registrar**.

      :::image type="content" source="./media/create-catalog-portal/register-purview-resource-provider.png" alt-text="Captura de pantalla que muestra cómo registrar el proveedor de recursos Microsoft.Purview en Azure Portal.":::

## <a name="create-an-azure-purview-account-instance"></a>Creación de una instancia de una cuenta de Azure Purview

1. Inicie sesión con su credencial de Azure

    # <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
    
    ```azurepowershell
    Connect-AzAccount
    ```
    
    # <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)
    
    ```azurecli
    az login
    ```
    
    ---

1. Si tiene varias suscripciones de Azure, seleccione la suscripción que desee usar:

    # <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
    
    ```azurepowershell
    Set-AzContext [SubscriptionID/SubscriptionName]
    ```
    
    # <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)
    
    ```azurecli
    az account set --subscription [SubscriptionID/SubscriptionName]
    ```
    
    ---

1. Cree un grupo de recursos para su cuenta de Purview. Si ya tiene una, puede omitir este paso:

    # <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
    
    ```azurepowershell
    New-AzResourceGroup `
      -Name myResourceGroup `
      -Location "East US"
    ```
    
    # <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)
    
    ```azurecli
    az group create \
      --name myResourceGroup \
      --location "East US"
    ```
    
    ---

1. Cree un archivo de plantilla de Purview como `purviewtemplate.json`. Puede actualizar `name`, `location` y `capacity` (`4` o `16`):

    ```json
    {
      "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "resources": [
        {
          "name": "<yourPurviewAccountName>",
          "type": "Microsoft.Purview/accounts",
          "apiVersion": "2020-12-01-preview",
          "location": "EastUs",
          "identity": {
            "type": "SystemAssigned"
          },
          "properties": {
            "networkAcls": {
              "defaultAction": "Allow"
            }
          },
          "dependsOn": [],
          "sku": {
            "name": "Standard",
            "capacity": "4"
          },
          "tags": {}
        }
      ],
      "outputs": {}
    }
    ```

1. Implementación de una plantilla de Purview

    # <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
    
    ```azurepowershell
    New-AzResourceGroupDeployment -ResourceGroupName "<myResourceGroup>" -TemplateFile "<PATH TO purviewtemplate.json>"
    ```
    
    # <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)
    
    Para ejecutar este comando de implementación, debe tener la [última versión](/cli/azure/install-azure-cli) de la CLI de Azure.
    
    ```azurecli
    az deployment group create --resource-group "<myResourceGroup>" --template-file "<PATH TO purviewtemplate.json>"
    ```
    
    ---

1. El comando de implementación devuelve los resultados. Busque `ProvisioningState` para ver si la implementación se realizó correctamente.
    
## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha aprendido a crear una cuenta de Azure Purview.

Vaya al siguiente artículo, donde aprenderá a permitir que los usuarios accedan a su cuenta de Azure Purview. 

> [!div class="nextstepaction"]
> [Adición de usuarios a una cuenta de Azure Purview](catalog-permissions.md)
