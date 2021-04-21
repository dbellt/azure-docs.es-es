---
title: Creación de un panel de Azure Portal con la CLI de Azure
description: 'Inicio rápido: Aprenda a crear un panel en Azure Portal mediante la CLI de Azure. Un panel es una vista concentrada y organizada de los recursos en la nube.'
ms.topic: quickstart
ms.custom: devx-track-azurepowershell, devx-track-azurecli
ms.date: 12/4/2020
ms.openlocfilehash: d951c692c7d3c282ae68c5f9b53e9cda5407df10
ms.sourcegitcommit: afb79a35e687a91270973990ff111ef90634f142
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107481028"
---
# <a name="quickstart-create-an-azure-portal-dashboard-with-azure-cli"></a>Inicio rápido: Creación de un panel de Azure Portal con la CLI de Azure

En Azure Portal, un panel es una vista organizada de los recursos en la nube. Este artículo se centra en el proceso de utilizar la CLI de Azure para crear un panel.
El panel muestra el rendimiento de una máquina virtual y algunos vínculos e información estática.


[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- Si tiene varias suscripciones a Azure, elija la adecuada en la que se facturarán los recursos.
Seleccione una suscripción mediante el comando [az account set](/cli/azure/account#az_account_set):

  ```azurecli
  az account set --subscription 00000000-0000-0000-0000-000000000000
  ```

- Cree un [grupo de recursos de Azure](../azure-resource-manager/management/overview.md) mediante el comando [az group create](/cli/azure/group#az_group_create) o use un grupo de recursos existente:

  ```azurecli
  az group create --name myResourceGroup --location centralus
  ```

   Un grupo de recursos es un contenedor lógico en el que se implementan y se administran recursos de Azure como un grupo.

## <a name="create-a-virtual-machine"></a>Crear una máquina virtual

Cree una máquina virtual con el comando [az vm create](/cli/azure/vm#az_vm_create):

```azurecli
az vm create --resource-group myResourceGroup --name SimpleWinVM --image win2016datacenter \
   --admin-username azureuser --admin-password 1StrongPassword$
```

> [!Note]
> La contraseña debe ser compleja.
> Debe ser un nombre de usuario y una contraseña nuevos,
> no la cuenta que se usa para iniciar sesión en Azure, por ejemplo.
> Para más información, consulte los [requisitos de nombre de usuario](../virtual-machines/windows/faq.md#what-are-the-username-requirements-when-creating-a-vm) y los [requisitos de contraseña](../virtual-machines/windows/faq.md#what-are-the-password-requirements-when-creating-a-vm).

La implementación se inicia; esta operación normalmente tarda unos minutos en completarse.
Una vez finalizada la implementación, pase a la sección siguiente.

## <a name="download-the-dashboard-template"></a>Descarga de la plantilla del panel

Como los paneles de Azure son recursos, se pueden representar como JSON.
Para más información, consulte [La estructura de los paneles de Azure](./azure-portal-dashboards-structure.md).

Descargue el siguiente archivo: [portal-dashboard-template-testvm.json](https://raw.githubusercontent.com/Azure/azure-docs-powershell-samples/master/azure-portal/portal-dashboard-template-testvm.json).

Personalice la plantilla descargada cambiando los valores siguientes por los suyos:

* `<subscriptionID>`: Su suscripción
* `<rgName>`: grupo de recursos, por ejemplo `myResourceGroup`.
* `<vmName>`: nombre de la máquina virtual, por ejemplo `SimpleWinVM`.
* `<dashboardTitle>`: título del panel, por ejemplo `Simple VM Dashboard`.
* `<location>`: la región de Azure, por ejemplo, `centralus`.

Para más información, consulte el artículo sobre la [referencia de las plantillas de paneles del portal de Microsoft](/azure/templates/microsoft.portal/dashboards).

## <a name="deploy-the-dashboard-template"></a>Implementación de la plantilla del panel

Ahora puede implementar la plantilla desde la CLI de Azure.

1. Ejecute el comando [az portal dashboard create](/cli/azure/ext/portal/portal/dashboard#ext_portal_az_portal_dashboard_create) para implementar la plantilla:

   ```azurecli
   az portal dashboard create --resource-group myResourceGroup --name 'Simple VM Dashboard' \
      --input-path portal-dashboard-template-testvm.json --location centralus
   ```

1. Compruebe que el panel se ha creado correctamente ejecutando el comando [az portal dashboard show](/cli/azure/ext/portal/portal/dashboard#ext_portal_az_portal_dashboard_show):

   ```azurecli
   az portal dashboard show --resource-group myResourceGroup --name 'Simple VM Dashboard'
   ```

Para ver todos los paneles de la suscripción actual, use [az portal dashboard list](/cli/azure/ext/portal/portal/dashboard#ext_portal_az_portal_dashboard_list):

```azurecli
az portal dashboard list
```

También puede ver todos los paneles de un grupo de recursos:

```azurecli
az portal dashboard list --resource-group myResourceGroup
```

Puede actualizar un panel mediante el comando [az portal dashboard update](/cli/azure/ext/portal/portal/dashboard#ext_portal_az_portal_dashboard_update):

```azurecli
az portal dashboard update --resource-group myResourceGroup --name 'Simple VM Dashboard' \
   --input-path portal-dashboard-template-testvm.json --location centralus
```

[!INCLUDE [azure-portal-review-deployed-resources](../../includes/azure-portal-review-deployed-resources.md)]

## <a name="clean-up-resources"></a>Limpieza de recursos

Para quitar la máquina virtual y el panel asociado, elimine el grupo de recursos que los contiene.

> [!CAUTION]
> En el ejemplo siguiente se elimina el grupo de recursos especificado y todos los recursos que contiene.
> Si los recursos que están fuera del ámbito de este artículo existen en el grupo de recursos especificado, también se eliminarán.

```azurecli
az group delete --name myResourceGroup
```

Para quitar solo el panel, use el comando [az portal dashboard delete](/cli/azure/ext/portal/portal/dashboard#ext_portal_az_portal_dashboard_delete):

```azurecli
az portal dashboard delete --resource-group myResourceGroup --name "Simple VM Dashboard"
```

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la compatibilidad de la CLI de Azure con los paneles, consulte [az portal dashboard](/cli/azure/ext/portal/portal/dashboard).
