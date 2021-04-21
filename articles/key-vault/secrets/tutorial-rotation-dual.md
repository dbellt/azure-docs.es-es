---
title: Tutorial de rotación para recursos con dos conjuntos de credenciales
description: Use este tutorial para aprender a automatizar la rotación de secretos para los recursos que usan dos conjuntos de credenciales de autenticación.
services: key-vault
author: msmbaldwin
manager: rkarlin
tags: rotation
ms.service: key-vault
ms.subservice: secrets
ms.topic: tutorial
ms.date: 06/22/2020
ms.author: jalichwa
ms.custom: devx-track-azurepowershell, devx-track-azurecli
ms.openlocfilehash: 1f656a41b0f447b90f58ec14173e418a2defb72e
ms.sourcegitcommit: afb79a35e687a91270973990ff111ef90634f142
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107484836"
---
# <a name="automate-the-rotation-of-a-secret-for-resources-that-have-two-sets-of-authentication-credentials"></a>Automatización de la rotación de un secreto para recursos que tienen dos conjuntos de credenciales de autenticación

Aunque la mejor forma de realizar la autenticación en los servicios de Azure es mediante una [identidad administrada](../general/authentication.md), hay algunos escenarios en los que no es posible. En esos casos, se usan claves de acceso o contraseñas. Debe rotar las claves de acceso y las contraseñas con frecuencia.

En este tutorial se muestra cómo automatizar la rotación periódica de secretos de bases de datos y servicios que usan dos conjuntos de credenciales de autenticación. En concreto, en este tutorial se muestra cómo rotar las claves de cuentas de Azure Storage almacenadas en Azure Key Vault como secretos. Usará una función desencadenada por notificaciones de Azure Event Grid. 

> [!NOTE]
> Puede hacer que las claves de la cuenta de almacenamiento se administren automáticamente en Key Vault si proporciona tokens de firma de acceso compartido para el acceso delegado a la cuenta de almacenamiento. Hay servicios que requieren cadenas de conexión de la cuenta de almacenamiento con claves de acceso. Esta es la solución recomendada para ese escenario.

Esta es la solución de rotación que se describe en este tutorial: 

![Diagrama que muestra la solución de rotación.](../media/secrets/rotation-dual/rotation-diagram.png)

En esta solución, Azure Key Vault almacena las claves de acceso individuales de la cuenta de almacenamiento como versiones del mismo secreto; la clave principal y la secundaria se alteran en versiones posteriores. Cuando se almacena una clave de acceso en la versión más reciente del secreto, se regenera la clave alternativa y se agrega a Key Vault como la versión nueva más reciente del secreto. La solución proporciona a la aplicación el ciclo completo de rotación para actualizar a la clave regenerada más reciente. 

1. Treinta días antes de la fecha de expiración de un secreto, Key Vault publica el evento de expiración próxima en Event Grid.
1. Event Grid comprueba las suscripciones del evento y usa HTTP POST para llamar al punto de conexión de la aplicación de funciones que se ha suscrito al evento.
1. La aplicación de funciones identifica la clave alternativa (no la más reciente) y llama a la cuenta de almacenamiento para regenerarla.
1. La aplicación de funciones agrega la nueva clave regenerada a Azure Key Vault como la nueva versión del secreto.

## <a name="prerequisites"></a>Requisitos previos
* Suscripción a Azure. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Azure [Cloud Shell](https://shell.azure.com/). En este tutorial se usa Cloud Shell del portal con ENV de PowerShell.
* Azure Key Vault.
* Dos cuentas de Azure Storage.

Puede usar este vínculo de implementación si no tiene un almacén de claves ni cuentas de almacenamiento:

[![Vínculo con la etiqueta de implementación en Azure.](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/1-CONTRIBUTION-GUIDE/images/deploytoazure.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure-Samples%2FKeyVault-Rotation-StorageAccountKey-PowerShell%2Fmaster%2FARM-Templates%2FInitial-Setup%2Fazuredeploy.json)

1. En **Grupo de recursos**, seleccione **Crear nuevo**. Asigne al grupo el nombre **vaultrotation** y, luego, seleccione **Aceptar**.
1. Seleccione **Revisar + crear**.
1. Seleccione **Crear**.

    ![Captura de pantalla que muestra cómo se crea un grupo de recursos.](../media/secrets/rotation-dual/dual-rotation-1.png)

Ahora tendrá un almacén de claves y dos cuentas de almacenamiento. Puede comprobar esta configuración en la CLI de Azure o en Azure PowerShell mediante la ejecución de este comando:
# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)
```azurecli
az resource list -o table -g vaultrotation
```
# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azurepowershell)

```azurepowershell
Get-AzResource -Name 'vaultrotation*' | Format-Table
```
---

El resultado deberá tener un aspecto similar a esta salida:

```console
Name                     ResourceGroup         Location    Type                               Status
-----------------------  --------------------  ----------  ---------------------------------  --------
vaultrotation-kv         vaultrotation      westus      Microsoft.KeyVault/vaults
vaultrotationstorage     vaultrotation      westus      Microsoft.Storage/storageAccounts
vaultrotationstorage2    vaultrotation      westus      Microsoft.Storage/storageAccounts
```

## <a name="create-and-deploy-the-key-rotation-function"></a>Creación e implementación de la función de rotación de claves

A continuación, va a crear una aplicación de funciones con una identidad administrada por el sistema, además de otros componentes necesarios. También va a implementar la función de rotación de las claves de la cuenta de almacenamiento.

La función de rotación de la aplicación de funciones requiere los siguientes componentes y configuración:
- Un plan de Azure App Service
- Una cuenta de almacenamiento para administrar los desencadenadores de la aplicación de funciones
- Una directiva de acceso para acceder a los secretos de Key Vault
- El rol de servicio del operador de claves de cuentas de almacenamiento asignado a la aplicación de funciones para que pueda acceder a las claves de acceso de la cuenta de almacenamiento
- Una función de rotación de claves con un desencadenador de eventos y un desencadenador HTTP (rotación a petición)
- Una suscripción a eventos de Event Grid para el evento **SecretNearExpiry**

1. Seleccione el vínculo de implementación de la plantilla de Azure: 

   [![Vínculo de implementación de la plantilla de Azure.](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/1-CONTRIBUTION-GUIDE/images/deploytoazure.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure-Samples%2FKeyVault-Rotation-StorageAccountKey-PowerShell%2Fmaster%2FARM-Templates%2FFunction%2Fazuredeploy.json)

1. En la lista **Grupo de recursos**, seleccione **vaultrotation**.
1. En el cuadro **Storage Account RG** (Grupo de recursos de la cuenta de almacenamiento), escriba el nombre del grupo de recursos en el que se encuentra la cuenta de almacenamiento. Conserve el valor predeterminado **[resourceGroup().name]** si la cuenta de almacenamiento ya se encuentra en el mismo grupo de recursos donde va a implementar la función de rotación de claves.
1. En el cuadro **Nombre de la cuenta de almacenamiento**, escriba el nombre de la cuenta de almacenamiento que contiene las claves de acceso que se van a rotar. Mantenga el valor predeterminado **[concat(resourceGroup().name, "storage")]** si usa la cuenta de almacenamiento creada en [Requisitos previos](#prerequisites).
1. En el cuadro **Key Vault RG** (Grupo de recursos del almacén de claves), escriba el nombre del grupo de recursos donde se encuentra el almacén de claves. Conserve el valor predeterminado **[resourceGroup().name]** si el almacén de claves ya existe en el mismo grupo de recursos donde va a implementar la función de rotación de claves.
1. En el cuadro **Nombre del almacén de claves**, escriba el nombre del almacén de claves. Mantenga el valor predeterminado **[concat(resourceGroup().name, "-kv")]** si usa el almacén de claves creado en [Requisitos previos](#prerequisites).
1. En el cuadro **App Service Plan Type** (Tipo de plan de App Service), seleccione el plan de hospedaje. El **plan Premium** solo es necesario cuando el almacén de claves está detrás del firewall.
1. En el cuadro **Nombre de la aplicación de funciones**, escriba el nombre de la aplicación de funciones.
1. En el cuadro **Nombre del secreto**, escriba el nombre del secreto en el que almacenará las claves de acceso.
1. En el cuadro **Repo URL** (Dirección URL del repositorio), escriba la ubicación de GitHub del código de la función. En este tutorial, puede usar **https://github.com/Azure-Samples/KeyVault-Rotation-StorageAccountKey-PowerShell.git** .
1. Seleccione **Revisar + crear**.
1. Seleccione **Crear**.

   ![Captura de pantalla que muestra cómo crear e implementar la función.](../media/secrets/rotation-dual/dual-rotation-2.png)

Tras completar los pasos anteriores tendrá una cuenta de almacenamiento, una granja de servidores, una aplicación de funciones y Application Insights. Una vez completada la implementación, verá esta página:

   ![Captura de pantalla que muestra la página Se completó la implementación.](../media/secrets/rotation-dual/dual-rotation-3.png)
> [!NOTE]
> Si se produce un error, puede seleccionar **Volver a implementar** para finalizar la implementación de los componentes.


Puede encontrar plantillas de implementación y código para la función de rotación en [Azure Samples](https://github.com/Azure-Samples/KeyVault-Rotation-StorageAccountKey-PowerShell).

## <a name="add-the-storage-account-access-keys-to-key-vault"></a>Incorporación de las claves de acceso de la cuenta de almacenamiento a Key Vault

En primer lugar, establezca la directiva de acceso para conceder a la entidad de seguridad de usuario los permisos para **administrar secretos**:
# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)
```azurecli
az keyvault set-policy --upn <email-address-of-user> --name vaultrotation-kv --secret-permissions set delete get list
```
# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azurepowershell)

```azurepowershell
Set-AzKeyVaultAccessPolicy -UserPrincipalName <email-address-of-user> --name vaultrotation-kv -PermissionsToSecrets set,delete,get,list
```
---

Ahora puede crear un nuevo secreto con una clave de acceso de la cuenta de almacenamiento como valor. También necesitará el identificador de recurso de la cuenta de almacenamiento, el período de validez del secreto y el identificador de clave para agregarlos al secreto, para que la función de rotación pueda volver a generar la clave en la cuenta de almacenamiento.

Determine el identificador de recurso de la cuenta de almacenamiento. Puede encontrar este valor en la propiedad `id`.

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)
```azurecli
az storage account show -n vaultrotationstorage
```
# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azurepowershell)

```azurepowershell
Get-AzStorageAccount -Name vaultrotationstorage -ResourceGroupName vaultrotation | Select-Object -Property *
```
---

Enumere las claves de acceso de la cuenta de almacenamiento para que pueda obtener los valores de clave:
# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)
```azurecli
az storage account keys list -n vaultrotationstorage
```
# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azurepowershell)

```azurepowershell
Get-AzStorageAccountKey -Name vaultrotationstorage -ResourceGroupName vaultrotation
```
---

Agregue el secreto al almacén de claves con la fecha de expiración establecida en mañana, el período de validez de 60 días y el identificador de recurso de la cuenta de almacenamiento. Ejecute este comando con los valores recuperados para `key1Value` y `storageAccountResourceId`:

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)
```azurecli
$tomorrowDate = (get-date).AddDays(+1).ToString("yyy-MM-ddTHH:mm:ssZ")
az keyvault secret set --name storageKey --vault-name vaultrotation-kv --value <key1Value> --tags "CredentialId=key1" "ProviderAddress=<storageAccountResourceId>" "ValidityPeriodDays=60" --expires $tomorrowDate
```
# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azurepowershell)

```azurepowershell
$tomorrowDate = (Get-Date).AddDays(+1).ToString('yyy-MM-ddTHH:mm:ssZ')
$secretVaule = ConvertTo-SecureString -String '<key1Value>' -AsPlainText -Force
$tags = @{
    CredentialId='key1'
    ProviderAddress='<storageAccountResourceId>'
    ValidityPeriodDays='60'
}
Set-AzKeyVaultSecret -Name storageKey -VaultName vaultrotation-kv -SecretValue $secretVaule -Tag $tags -Expires $tomorrowDate
```
---

El secreto anterior desencadenará el evento `SecretNearExpiry` en unos minutos. Este evento desencadenará a su vez la función para rotar el secreto con la expiración establecida en 60 días. En esa configuración, el evento "SecretNearExpiry" se desencadenaría cada 30 días (30 días antes de la expiración) y la función de rotación alternaría la rotación entre key1 y key2.

Para comprobar que las claves de acceso se han regenerado, recupere la clave de la cuenta de almacenamiento y el secreto de Key Vault y compárelos.

Use este comando para obtener la información del secreto:
# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)
```azurecli
az keyvault secret show --vault-name vaultrotation-kv --name storageKey
```
# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azurepowershell)

```azurepowershell
Get-AzKeyVaultSecret -VaultName vaultrotation-kv -Name storageKey -AsPlainText
```
---

Observe que el valor de `CredentialId` se actualiza al valor de `keyName` alternativo y que se regenera `value`:

![Captura de pantalla que muestra la salida del comando a z keyvault secret show para la primera cuenta de almacenamiento.](../media/secrets/rotation-dual/dual-rotation-4.png)

Recupere las claves de acceso para comparar los valores:
# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)
```azurecli
az storage account keys list -n vaultrotationstorage 
```
# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azurepowershell)

```azurepowershell
Get-AzStorageAccountKey -Name vaultrotationstorage -ResourceGroupName vaultrotation
```
---

Tenga en cuenta que el elemento `value` de la clave es igual que el secreto en el almacén de claves:

![Captura de pantalla que muestra la salida del comando a z storage account keys list para la primera cuenta de almacenamiento.](../media/secrets/rotation-dual/dual-rotation-5.png)

## <a name="add-storage-accounts-for-rotation"></a>Incorporación de cuentas de almacenamiento para la rotación

Puede reutilizar la misma aplicación de funciones para rotar las claves de varias cuentas de almacenamiento. 

Para agregar claves de cuenta de almacenamiento a una función existente para la rotación, necesita:
- El rol de servicio del operador de claves de cuentas de almacenamiento asignado a la aplicación de funciones para que pueda acceder a las claves de acceso de la cuenta de almacenamiento.
- Una suscripción a eventos de Event Grid para el evento **SecretNearExpiry**.

1. Seleccione el vínculo de implementación de la plantilla de Azure: 

   [![Vínculo de implementación de la plantilla de Azure.](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/1-CONTRIBUTION-GUIDE/images/deploytoazure.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure-Samples%2FKeyVault-Rotation-StorageAccountKey-PowerShell%2Fmaster%2FARM-Templates%2FAdd-Event-Subscriptions%2Fazuredeploy.json)

1. En la lista **Grupo de recursos**, seleccione **vaultrotation**.
1. En el cuadro **Storage Account RG** (Grupo de recursos de la cuenta de almacenamiento), escriba el nombre del grupo de recursos en el que se encuentra la cuenta de almacenamiento. Conserve el valor predeterminado **[resourceGroup().name]** si la cuenta de almacenamiento ya se encuentra en el mismo grupo de recursos donde va a implementar la función de rotación de claves.
1. En el cuadro **Nombre de la cuenta de almacenamiento**, escriba el nombre de la cuenta de almacenamiento que contiene las claves de acceso que se van a rotar.
1. En el cuadro **Key Vault RG** (Grupo de recursos del almacén de claves), escriba el nombre del grupo de recursos donde se encuentra el almacén de claves. Conserve el valor predeterminado **[resourceGroup().name]** si el almacén de claves ya existe en el mismo grupo de recursos donde va a implementar la función de rotación de claves.
1. En el cuadro **Nombre del almacén de claves**, escriba el nombre del almacén de claves.
1. En el cuadro **Nombre de la aplicación de funciones**, escriba el nombre de la aplicación de funciones.
1. En el cuadro **Nombre del secreto**, escriba el nombre del secreto en el que almacenará las claves de acceso.
1. Seleccione **Revisar + crear**.
1. Seleccione **Crear**.

   ![Captura de pantalla que muestra cómo se crea una cuenta de almacenamiento adicional.](../media/secrets/rotation-dual/dual-rotation-7.png)

### <a name="add-another-storage-account-access-key-to-key-vault"></a>Incorporación de otra clave de acceso de cuenta de almacenamiento a Key Vault

Determine el identificador de recurso de la cuenta de almacenamiento. Puede encontrar este valor en la propiedad `id`.
# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)
```azurecli
az storage account show -n vaultrotationstorage2
```
# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azurepowershell)

```azurepowershell
Get-AzStorageAccount -Name vaultrotationstorage -ResourceGroupName vaultrotation | Select-Object -Property *
```
---

Enumere las claves de acceso de la cuenta de almacenamiento para que pueda obtener el valor de key2:
# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)
```azurecli
az storage account keys list -n vaultrotationstorage2
```
# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azurepowershell)

```azurepowershell
Get-AzStorageAccountKey -Name vaultrotationstorage2 -ResourceGroupName vaultrotation
```
---

Agregue el secreto al almacén de claves con la fecha de expiración establecida en mañana, el período de validez de 60 días y el identificador de recurso de la cuenta de almacenamiento. Ejecute este comando con los valores recuperados para `key2Value` y `storageAccountResourceId`:

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)
```azurecli
$tomorrowDate = (Get-Date).AddDays(+1).ToString('yyy-MM-ddTHH:mm:ssZ')
az keyvault secret set --name storageKey2 --vault-name vaultrotation-kv --value <key2Value> --tags "CredentialId=key2" "ProviderAddress=<storageAccountResourceId>" "ValidityPeriodDays=60" --expires $tomorrowDate
```
# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azurepowershell)

```azurepowershell
$tomorrowDate = (get-date).AddDays(+1).ToString("yyy-MM-ddTHH:mm:ssZ")
$secretVaule = ConvertTo-SecureString -String '<key1Value>' -AsPlainText -Force
$tags = @{
    CredentialId='key2';
    ProviderAddress='<storageAccountResourceId>';
    ValidityPeriodDays='60'
}
Set-AzKeyVaultSecret -Name storageKey2 -VaultName vaultrotation-kv -SecretValue $secretVaule -Tag $tags -Expires $tomorrowDate
```
---

Use este comando para obtener la información del secreto:
# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)
```azurecli
az keyvault secret show --vault-name vaultrotation-kv --name storageKey2
```
# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azurepowershell)

```azurepowershell
Get-AzKeyVaultSecret -VaultName vaultrotation-kv -Name storageKey2 -AsPlainText
```
---

Observe que el valor de `CredentialId` se actualiza al valor de `keyName` alternativo y que se regenera `value`:

![Captura de pantalla que muestra la salida del comando a z keyvault secret show para la segunda cuenta de almacenamiento.](../media/secrets/rotation-dual/dual-rotation-8.png)

Recupere las claves de acceso para comparar los valores:
# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)
```azurecli
az storage account keys list -n vaultrotationstorage 
```
# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azurepowershell)

```azurepowershell
Get-AzStorageAccountKey -Name vaultrotationstorage -ResourceGroupName vaultrotation
```
---

Tenga en cuenta que el elemento `value` de la clave es igual que el secreto en el almacén de claves:

![Captura de pantalla que muestra la salida del comando a z storage account keys list para la segunda cuenta de almacenamiento.](../media/secrets/rotation-dual/dual-rotation-9.png)

## <a name="key-vault-rotation-functions-for-two-sets-of-credentials"></a>Funciones de rotación de Key Vault para dos conjuntos de credenciales

Plantilla de funciones de rotación para dos conjuntos de credenciales y varias funciones listas para usar:

- [Plantilla de proyecto](https://serverlesslibrary.net/sample/bc72c6c3-bd8f-4b08-89fb-c5720c1f997f)
- [Redis Cache](https://serverlesslibrary.net/sample/0d42ac45-3db2-4383-86d7-3b92d09bc978)
- [Storage Account](https://serverlesslibrary.net/sample/0e4e6618-a96e-4026-9e3a-74b8412213a4)
- [Cosmos DB](https://serverlesslibrary.net/sample/bcfaee79-4ced-4a5c-969b-0cc3997f47cc)

> [!NOTE]
> Las funciones de rotación anteriores han sido creadas por un miembro de la comunidad y no por Microsoft. Las funciones de Azure Functions de la comunidad no reciben ningún tipo de soporte en ningún servicio ni programa de soporte de Microsoft. Dichas plantillas se proporcionan tal cual, sin garantía de ningún tipo.

## <a name="next-steps"></a>Pasos siguientes

- Tutorial: [Rotación de secretos para un conjunto de credenciales](./tutorial-rotation.md)
- Introducción: [Supervisión de Key Vault con Azure Event Grid](../general/event-grid-overview.md)
- Procedimientos: [Creación de su primera función en Azure Portal](../../azure-functions/functions-get-started.md)
- Procedimiento: [Recibir un correo electrónico al cambiar un secreto de Key Vault](../general/event-grid-logicapps.md)
- Referencia: [Esquema de eventos de Azure Event Grid para Azure Key Vault](../../event-grid/event-schema-key-vault.md)
