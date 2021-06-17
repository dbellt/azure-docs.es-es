---
title: 'Uso compartido fuera de la organización (plantilla de ARM): Inicio rápido de Azure Data Share'
description: En este inicio rápido, aprenderá a compartir datos con clientes y asociados mediante Azure Data Share y una plantilla de Azure Resource Manager en este inicio rápido.
author: mumian
ms.author: jgao
ms.service: data-share
ms.topic: quickstart
ms.custom: subject-armqs, devx-track-azurepowershell
ms.date: 08/19/2020
ms.openlocfilehash: f8a211c6cb3a405b1d3f022dfa02673d6a6db942
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110705180"
---
# <a name="quickstart-share-data-using-azure-data-share-and-arm-template"></a>Inicio rápido: Uso compartido de datos mediante Azure Data Share y una plantilla de Resource Manager

Aprenda a configurar una instancia nueva de Azure Data Share desde una cuenta de Azure Storage mediante una plantilla de Azure Resource Manager. Y comience a compartir datos con clientes y asociados de fuera de la organización de Azure. Para ver una lista de los almacenes de datos compatibles, consulte los [Almacenes de datos admitidos en Azure Data Share](./supported-data-stores.md).

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Si su entorno cumple los requisitos previos y está familiarizado con el uso de plantillas de Resource Manager, seleccione el botón **Implementar en Azure**. La plantilla se abrirá en Azure Portal.

[![Implementación en Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-data-share-share-storage-account%2Fazuredeploy.json)

## <a name="prerequisites"></a>Prerrequisitos

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="review-the-template"></a>Revisión de la plantilla

La plantilla usada en este inicio rápido forma parte de las [plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/101-data-share-share-storage-account/).

:::code language="json" source="~/quickstart-templates/101-data-share-share-storage-account/azuredeploy.json":::

Los recursos siguientes se definen en la plantilla:

* [Microsoft.Storage/storageAccounts](/azure/templates/microsoft.storage/storageaccounts):
* [Microsoft.Storage/storageAccounts/blobServices/containers](/azure/templates/microsoft.storage/storageaccounts/blobservices/containers)
* [Microsoft.DataShare/accounts](/azure/templates/microsoft.datashare/accounts)
* [Microsoft.DataShare/accounts/shares](/azure/templates/microsoft.datashare/accounts/shares)
* [Microsoft.Storage/storageAccounts/providers/roleAssignments](/azure/templates/microsoft.authorization/roleassignments)
* [Microsoft.DataShare/accounts/shares/dataSets](/azure/templates/microsoft.datashare/accounts/shares/datasets)
* [Microsoft.DataShare/accounts/shares/invitations](/azure/templates/microsoft.datashare/accounts/shares/invitations)
* [Microsoft.DataShare/accounts/shares/synchronizationSettings](/azure/templates/microsoft.datashare/accounts/shares/synchronizationsettings)

La plantilla realiza las siguientes tareas:

* Crear una cuenta de almacenamiento y un contenedor, que se usan como origen de datos de Azure Data Share.
* Crear una cuenta de Azure Data Share y un recurso compartido de datos.
* Crear una asignación de roles para conceder el rol Lector de datos de Storage Blob al recurso de recurso compartido de datos de origen. Consulte [Roles y requisitos de Azure Data Share](./concepts-roles-permissions.md).
* Agregue un conjunto de datos a Data Share.
* Agregue destinatarios a Data Share.
* Habilite una programación de instantáneas para Data Share.

Esta plantilla se crea con fines formativos. En la práctica, lo habitual es que haya datos en una cuenta de almacenamiento existente. Sería preciso que creara la asignación de roles antes de ejecutar una plantilla o un script para crear el conjunto de datos. Algunas veces puede aparecer el siguiente mensaje de error cuando se implementa la plantilla:

```plaintext
"Missing permissions for DataShareAcccount on resource 'subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP NAME>/providers/Microsoft.Storage/storageAccounts/<STORAGE ACCOUNT NAME>' (Code: 5006)"
```

Esto se debe a que la implementación intenta crear el conjunto de datos antes de que haya finalizado la asignación de roles de Azure. A pesar del mensaje de error, la implementación puede haberse realizado correctamente. Puede examinar la sección [Revisión de los recursos implementados](#review-deployed-resources).

## <a name="deploy-the-template"></a>Implementación de la plantilla

1. Seleccione la imagen siguiente para iniciar sesión en Azure y abrir la plantilla.

    [![Implementación en Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-data-share-share-storage-account%2Fazuredeploy.json)
1. Seleccione o escriba los siguientes valores:

    * **Suscripción**: seleccione la suscripción de Azure que se usa para crear el recurso compartido de datos y los restantes recursos.
    * **Grupo de recursos**: seleccione **Crear nuevo** para crear un grupo de recursos, o bien seleccione un grupo de recursos existente.
    * **Ubicación**: seleccione una ubicación para el grupo de recursos.
    * **Nombre del proyecto**: escriba un nombre de proyecto.  El nombre de proyecto se utiliza para generar nombres de recursos.  Vea las definiciones de las variables de la plantilla anterior.
    * **Ubicación**: seleccione la ubicación para los recursos.  Puede usar la misma ubicación para el grupo de recursos.
    * **Invitation Email** (Correo electrónico de invitación): escriba la dirección de correo electrónico de inicio de sesión de Azure del destinatario del recurso compartido de datos.  El alias de correo electrónico no funciona.

    Use el valor predeterminado en el resto de la configuración.
1. Seleccione **Acepto los términos y condiciones indicados anteriormente** y, después, **Comprar**.

## <a name="review-deployed-resources"></a>Revisión de los recursos implementados

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Abra la cuenta de Data Share que creó.
1. En el menú izquierdo, seleccione **Send Shares** (Enviar recursos compartidos).  Verá que la cuenta de almacenamiento en la lista.
1. Seleccione la cuenta de almacenamiento.  En **Detalles**, verá el valor de sincronización que ha configurado en la plantilla.

    ![Configuración de sincronización de la cuenta de almacenamiento de Azure Data Share](./media/share-your-data-arm/azure-data-share-storage-account-synchronization-settings.png)
1. Seleccione **Invitaciones** en la parte superior. Verá la dirección de correo electrónico que especificó al implementar la plantilla. El valor de **Estado** debe estar **Pendiente**.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no lo necesite, elimine el grupo de recursos, que elimina los recursos que contiene.

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter the resource group name"
Remove-AzResourceGroup -Name $resourceGroupName
Write-Host "Press [ENTER] to continue..."
```

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha aprendido a crear un recurso compartido de datos de Azure y a invitar a destinatarios. Para más información acerca de la forma en que un consumidor de datos puede aceptar y recibir un recurso compartido de datos, vaya al tutorial para [aceptar y recibir datos](subscribe-to-data-share.md).
