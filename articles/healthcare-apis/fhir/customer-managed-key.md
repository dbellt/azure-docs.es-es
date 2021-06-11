---
title: Configuración de claves administradas por el cliente Azure API for FHIR
description: Incorpore su propia característica clave compatible con Azure API for FHIR mediante Cosmos DB
services: healthcare-apis
author: ginalee-dotcom
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: overview
ms.date: 05/04/2021
ms.author: ginle
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 0c32725d020043c3cd62828e4bdccfafef2f53aa
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111970174"
---
# <a name="configure-customer-managed-keys-at-rest"></a>Configuración de claves administradas por el cliente en reposo

Cuando crea una cuenta de Azure API for FHIR, de forma predeterminada los datos se cifran mediante claves administradas por Microsoft. Ahora puede agregar una segunda capa de cifrado de datos mediante una clave que usted mismo elige y administra.

En Azure, para hacerlo se suele utilizar una clave de cifrado en la instancia de Azure Key Vault del cliente. Azure SQL, Azure Storage y Cosmos DB son algunos ejemplos que proporcionan esta funcionalidad hoy en día. Azure API for FHIR aprovecha esta compatibilidad con Cosmos DB. Al crear una cuenta, tendrá la opción de especificar el identificador URI de una clave de Azure Key Vault. Esta clave se pasará a Cosmos DB al aprovisionar la cuenta de la base de datos. Cuando se realiza una solicitud de FHIR, Cosmos DB captura su clave y la usa para cifrar o descifrar los datos. 

Para empezar, consulte los vínculos siguientes:

- [Registro del proveedor de recursos de Azure Cosmos DB para su suscripción a Azure](../../cosmos-db/how-to-setup-cmk.md#register-resource-provider) 
- [Configuración de la instancia de Azure Key Vault](../../cosmos-db/how-to-setup-cmk.md#configure-your-azure-key-vault-instance)
- [Adición de una directiva de acceso a una instancia de Azure Key Vault](../../cosmos-db/how-to-setup-cmk.md#add-access-policy)
- [Generación de una clave en Azure Key Vault](../../cosmos-db/how-to-setup-cmk.md#generate-a-key-in-azure-key-vault)

## <a name="using-azure-portal"></a>Uso de Azure Portal

Al crear la cuenta Azure API for FHIR en Azure Portal, observará  la opción de  configuración Cifrado de datos en Configuración de base de datos en la **pestaña Configuración** adicional. De forma predeterminada, se seleccionará la opción de clave administrada por el servicio.

> [!Important]
> La opción de cifrado de datos solo está disponible cuando el Azure API for FHIR se crea y no se puede cambiar posteriormente. Sin embargo, puede ver y actualizar la clave de cifrado si está seleccionada la opción **Clave** administrada por el cliente. 


La clave se puede elegir en KeyPicker:

:::image type="content" source="media/bring-your-own-key/bring-your-own-key-keypicker.png" alt-text="KeyPicker":::

También puede especificar la clave Azure Key Vault aquí seleccionando la **opción Clave administrada por el** cliente.
 
También puede escribir el URI de clave aquí:

:::image type="content" source="media/bring-your-own-key/bring-your-own-key-create.png" alt-text="Creación de Azure API for FHIR":::

> [!Important]
> Asegúrese de que todos los permisos Azure Key Vault se establecen correctamente. Para obtener más información, consulte [Adición de una directiva de acceso a la Azure Key Vault instancia de](../../cosmos-db/how-to-setup-cmk.md#add-access-policy). Además, asegúrese de que la eliminación automática está habilitada en las propiedades del Key Vault. Si no se completan estos pasos, se producirá un error de implementación. Para más información, consulte [Comprobación de si la eliminación automática está habilitada en un almacén de claves y habilitación de la eliminación automática.](../../key-vault/general/key-vault-recovery.md?tabs=azure-portal#verify-if-soft-delete-is-enabled-on-a-key-vault-and-enable-soft-delete)

En el caso de las cuentas de FHIR existentes, puede ver la  opción de cifrado de claves **(clave** administrada por el servicio o clave administrada por el **cliente)** en la hoja Base de datos, como se muestra a continuación. La opción de configuración no se puede modificar una vez seleccionada. Sin embargo, se puede modificar y actualizar la clave.

:::image type="content" source="media/bring-your-own-key/bring-your-own-key-database.png" alt-text="Base de datos":::

Además, puede crear una nueva versión de la clave especificada, después de la cual los datos se cifrarán con la nueva versión sin ninguna interrupción del servicio. También puede quitar el acceso a la clave para quitar el acceso a los datos. Cuando la clave esté deshabilitada, las consultas generarán un error. Si se vuelve a habilitar, las consultas se volverán a realizar correctamente.

## <a name="using-azure-powershell"></a>Uso de Azure PowerShell

Con el URI de la clave de Azure Key Vault, puede configurar la clave administrada por el cliente con PowerShell mediante la ejecución del comando de PowerShell siguiente:

```powershell
New-AzHealthcareApisService
    -Name "myService"
    -Kind "fhir-R4"
    -ResourceGroupName "myResourceGroup"
    -Location "westus2"
    -CosmosKeyVaultKeyUri "https://<my-vault>.vault.azure.net/keys/<my-key>"
```

## <a name="using-azure-cli"></a>Uso de la CLI de Azure

Al igual que con el método de PowerShell, puede configurar la clave administrada por el cliente pasando el URI de la clave de Azure Key Vault del parámetro `key-vault-key-uri` y ejecutando el comando de la CLI siguiente: 

```azurecli-interactive
az healthcareapis service create
    --resource-group "myResourceGroup"
    --resource-name "myResourceName"
    --kind "fhir-R4"
    --location "westus2"
    --cosmos-db-configuration key-vault-key-uri="https://<my-vault>.vault.azure.net/keys/<my-key>"

```
## <a name="using-azure-resource-manager-template"></a>Uso de la plantilla de Azure Resource Manager

Con el URI de la clave de Azure Key Vault puede configurar la clave administrada por el cliente pasándolo por la propiedad **keyVaultKeyUri** del objeto **properties**.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "services_myService_name": {
            "defaultValue": "myService",
            "type": "String"
        }
    },
    "variables": {},
    "resources": [
        {
            "type": "Microsoft.HealthcareApis/services",
            "apiVersion": "2020-03-30",
            "name": "[parameters('services_myService_name')]",
            "location": "westus2",
            "kind": "fhir-R4",
            "properties": {
                "accessPolicies": [],
                "cosmosDbConfiguration": {
                    "offerThroughput": 400,
                    "keyVaultKeyUri": "https://<my-vault>.vault.azure.net/keys/<my-key>"
                },
                "authenticationConfiguration": {
                    "authority": "https://login.microsoftonline.com/72f988bf-86f1-41af-91ab-2d7cd011db47",
                    "audience": "[concat('https://', parameters('services_myService_name'), '.azurehealthcareapis.com')]",
                    "smartProxyEnabled": false
                },
                "corsConfiguration": {
                    "origins": [],
                    "headers": [],
                    "methods": [],
                    "maxAge": 0,
                    "allowCredentials": false
                }
            }
        }
    ]
}
```

Y puede implementar la plantilla con el siguiente script de PowerShell:

```powershell
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$accountLocation = "West US 2"
$keyVaultKeyUri = "https://<my-vault>.vault.azure.net/keys/<my-key>"

New-AzResourceGroupDeployment `
    -ResourceGroupName $resourceGroupName `
    -TemplateFile "deploy.json" `
    -accountName $accountName `
    -location $accountLocation `
    -keyVaultKeyUri $keyVaultKeyUri
```

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha aprendido a configurar claves administradas por el cliente en reposo mediante Azure Portal, PowerShell, cli y Resource Manager plantilla. Para obtener más información, consulte la Azure Cosmos DB preguntas más frecuentes. 
 
>[!div class="nextstepaction"]
>[Cosmos DB: procedimientos para configurar CMK](../../cosmos-db/how-to-setup-cmk.md#frequently-asked-questions)