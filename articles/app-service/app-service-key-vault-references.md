---
title: Uso de referencias de Key Vault
description: Aprenda a configurar Azure App Service y Azure Functions para que usen referencias de Azure Key Vault. Haga que los secretos de Key Vault estén disponibles para el código de aplicación.
author: mattchenderson
ms.topic: article
ms.date: 05/25/2021
ms.author: mahender
ms.custom: seodec18
ms.openlocfilehash: 3300f5fbb5613672d7979f161ca0c92126f26a83
ms.sourcegitcommit: e1d5abd7b8ded7ff649a7e9a2c1a7b70fdc72440
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110578122"
---
# <a name="use-key-vault-references-for-app-service-and-azure-functions"></a>Uso de referencias de Key Vault para App Service y Azure Functions

En este tema se muestra cómo trabajar con secretos de Azure Key Vault en la aplicación de App Service o Azure Functions sin necesidad de realizar cambios en el código. [Azure Key Vault](../key-vault/general/overview.md) es un servicio que proporciona administración centralizada de los secretos, con control total sobre las directivas de acceso y el historial de auditoría.

## <a name="granting-your-app-access-to-key-vault"></a>Concesión a la aplicación de acceso a Key Vault

Para leer secretos desde Key Vault, debe tener creado un almacén y proporcionar a la aplicación permiso para acceder a él.

1. Para crear un almacén de claves, siga la [guía de inicio rápido de Key Vault](../key-vault/secrets/quick-create-cli.md).

1. Cree una [identidad administrada asignada por el sistema](overview-managed-identity.md) para la aplicación.

   > [!NOTE] 
   > Las referencias de Key Vault solo admiten actualmente identidades administradas asignadas por el sistema. No se pueden usar identidades asignadas por el usuario.

1. Cree una [directiva de acceso en Key Vault](../key-vault/general/security-features.md#privileged-access) para la identidad de aplicación que creó anteriormente. Habilite el permiso secreto "Get" en esta directiva. No configure la "aplicación autorizada" o la configuración `applicationId`, ya que no es compatible con una identidad administrada.

### <a name="access-network-restricted-vaults"></a>Acceso a almacenes restringidos de red

> [!NOTE]
> Las aplicaciones basadas en Linux no pueden resolver actualmente los secretos de un almacén de claves con restricciones de red a menos que la aplicación esté hospedada en una instancia de [App Service Environment](./environment/intro.md).

Si el almacén está configurado con [restricciones de red](../key-vault/general/overview-vnet-service-endpoints.md), también deberá asegurarse de que la aplicación tenga acceso a la red.

1. Asegúrese de que la aplicación tenga configuradas funcionalidades de red de salida, tal como se describe en [Características de red de App Service](./networking-features.md) y [Opciones de red de Azure Functions](../azure-functions/functions-networking-options.md).

2. Asegúrese de que la configuración del almacén tenga en cuenta la red o la subred mediante la que cual la aplicación tendrá acceso.

> [!IMPORTANT]
> El acceso a un almacén mediante la integración de red virtual no es compatible actualmente con las [actualizaciones automáticas de secretos sin una versión especificada](#rotation).

## <a name="reference-syntax"></a>Sintaxis de referencia

Una referencia de Key Vault tiene el formato `@Microsoft.KeyVault({referenceString})`, donde `{referenceString}` se sustituye por una de las opciones siguientes:

> [!div class="mx-tdBreakAll"]
> | Cadena de referencia                                                            | Descripción                                                                                                                                                                                 |
> |-----------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
> | SecretUri=_secretUri_                                                       | **SecretUri** debe ser el URI completo del plano de datos de un secreto en Key Vault y, opcionalmente, puede incluir una versión; por ejemplo, `https://myvault.vault.azure.net/secrets/mysecret/` o `https://myvault.vault.azure.net/secrets/mysecret/ec96f02080254f109c51a1f14cdb1931`.  |
> | VaultName=_vaultName_;SecretName=_secretName_;SecretVersion=_secretVersion_ | **VaultName** es un parámetro obligatorio y debe ser el nombre del recurso de Key Vault. **SecretName** es un parámetro obligatorio y debe ser el nombre del secreto de destino. **SecretVersion** es opcional, pero si está presente indica la versión del secreto que se usará. |

Por ejemplo, una referencia completa se parecería a la siguiente:

```
@Microsoft.KeyVault(SecretUri=https://myvault.vault.azure.net/secrets/mysecret/)
```

O bien:

```
@Microsoft.KeyVault(VaultName=myvault;SecretName=mysecret)
```

## <a name="rotation"></a>Rotación

> [!IMPORTANT]
> El [acceso a un almacén mediante la integración de red virtual](#access-network-restricted-vaults) no es compatible actualmente con las actualizaciones automáticas de secretos sin una versión especificada.

Si no se especifica una versión en la referencia, la aplicación usará la versión más reciente que exista en Key Vault. Cuando haya disponibles versiones más recientes, como con un evento de rotación, la aplicación se actualizará automáticamente y comenzará a usar la versión más reciente en el plazo de un día. Los cambios de configuración realizados en la aplicación actualizarán inmediatamente a las versiones más recientes todos los secretos a los que se hace referencia.

## <a name="source-application-settings-from-key-vault"></a>Configuración de la aplicación de origen desde Key Vault

Las referencias de Key Vault pueden usarse como valores en [Configuración de la aplicación](configure-common.md#configure-app-settings), lo que le permite conservar los secretos en Key Vault en lugar de en la configuración del sitio. La configuración de la aplicación se cifra de forma segura en reposo, pero si necesita funcionalidades de administración de secretos, debe incluirse en Key Vault.

Para usar una referencia de Key Vault para la [configuración de la aplicación](configure-common.md#add-or-edit), establezca la referencia como valor de la configuración. La aplicación puede hacer referencia al secreto mediante su clave de la forma habitual. No se requiere ningún cambio de código.

> [!TIP]
> La mayoría de los valores de configuración de la aplicación que usan referencias de Key Vault se deben marcar como configuración de ranuras, así que debe tener distintos almacenes para cada entorno.

### <a name="considerations-for-azure-files-mounting"></a>Consideraciones para el montaje de Azure Files

Las aplicaciones pueden usar la configuración de la aplicación `WEBSITE_CONTENTAZUREFILECONNECTIONSTRING` para montar Azure Files como el sistema de archivos. Esta configuración tiene comprobaciones de validación adicionales para garantizar que la aplicación se puede iniciar correctamente. La plataforma se basa en tener un recurso compartido de contenido dentro de Azure Files y supone un nombre predeterminado a menos que se especifique uno a través de la configuración `WEBSITE_CONTENTSHARE`. Para las solicitudes que modifican esta configuración, la plataforma intentará validar si existe este recurso compartido de contenido, e intentará crearlo si no existe. Si no se puede buscar o crear el recurso compartido de contenido, la solicitud se bloqueará.

Cuando se usen referencias de Key Vault para esta configuración, esta comprobación de validación producirá un error de manera predeterminada, ya que el secreto no se puede resolver al procesar la solicitud entrante. Para evitar este problema, puede omitir la validación al establecer el valor de `WEBSITE_SKIP_CONTENTSHARE_VALIDATION` en "1". Esto omitirá todas las comprobaciones y el recurso compartido de contenido no se creará automáticamente. Debe asegurarse de crearlo de antemano. 

> [!CAUTION]
> Si omite la validación y, además, la cadena de conexión o el recurso compartido de contenido no son válidos, la aplicación no podrá iniciarse correctamente y solo generará errores HTTP 500.

Como parte de la creación del sitio, también es posible que se pueda generar un error al intentar montar el recurso compartido de contenido debido a que no se hayan propagado los permisos de identidad administrada, o a que no se haya configurado la integración de red virtual. Puede aplazar la configuración de Azure Files hasta más adelante en la plantilla de implementación para realizar esto. Para más información, consulte [Implementación de Azure Resource Manager](#azure-resource-manager-deployment). App Service usará sistema de archivos predeterminado hasta que Azure Files esté configurado y los archivos no se hayan copiado, por lo que deberá asegurarse de que no se produzca ningún intento de implementación durante el período transitorio antes de montar Azure Files.

### <a name="azure-resource-manager-deployment"></a>Implementación de Azure Resource Manager

Al automatizar las implementaciones de recursos mediante plantillas de Azure Resource Manager, puede que deba secuenciar sus dependencias en un orden determinado para que esta característica funcione. Cabe señalar que deberá definir la configuración de la aplicación como su propio recurso en lugar de usar una propiedad `siteConfig` en la definición del sitio. El motivo es que primero es necesario definir el sitio para que la identidad asignada por el sistema se cree con él y se pueda usar en la directiva de acceso.

Una seudoplantilla de ejemplo para una aplicación de función podría tener este aspecto:

```json
{
    //...
    "resources": [
        {
            "type": "Microsoft.Storage/storageAccounts",
            "name": "[variables('storageAccountName')]",
            //...
        },
        {
            "type": "Microsoft.Insights/components",
            "name": "[variables('appInsightsName')]",
            //...
        },
        {
            "type": "Microsoft.Web/sites",
            "name": "[variables('functionAppName')]",
            "identity": {
                "type": "SystemAssigned"
            },
            //...
            "resources": [
                {
                    "type": "config",
                    "name": "appsettings",
                    //...
                    "dependsOn": [
                        "[resourceId('Microsoft.Web/sites', variables('functionAppName'))]",
                        "[resourceId('Microsoft.KeyVault/vaults/', variables('keyVaultName'))]",
                        "[resourceId('Microsoft.KeyVault/vaults/secrets', variables('keyVaultName'), variables('storageConnectionStringName'))]",
                        "[resourceId('Microsoft.KeyVault/vaults/secrets', variables('keyVaultName'), variables('appInsightsKeyName'))]"
                    ],
                    "properties": {
                        "AzureWebJobsStorage": "[concat('@Microsoft.KeyVault(SecretUri=', reference(variables('storageConnectionStringResourceId')).secretUriWithVersion, ')')]",
                        "WEBSITE_CONTENTAZUREFILECONNECTIONSTRING": "[concat('@Microsoft.KeyVault(SecretUri=', reference(variables('storageConnectionStringResourceId')).secretUriWithVersion, ')')]",
                        "APPINSIGHTS_INSTRUMENTATIONKEY": "[concat('@Microsoft.KeyVault(SecretUri=', reference(variables('appInsightsKeyResourceId')).secretUriWithVersion, ')')]",
                        "WEBSITE_ENABLE_SYNC_UPDATE_SITE": "true"
                        //...
                    }
                },
                {
                    "type": "sourcecontrols",
                    "name": "web",
                    //...
                    "dependsOn": [
                        "[resourceId('Microsoft.Web/sites', variables('functionAppName'))]",
                        "[resourceId('Microsoft.Web/sites/config', variables('functionAppName'), 'appsettings')]"
                    ],
                }
            ]
        },
        {
            "type": "Microsoft.KeyVault/vaults",
            "name": "[variables('keyVaultName')]",
            //...
            "dependsOn": [
                "[resourceId('Microsoft.Web/sites', variables('functionAppName'))]"
            ],
            "properties": {
                //...
                "accessPolicies": [
                    {
                        "tenantId": "[reference(resourceId('Microsoft.Web/sites/', variables('functionAppName')), '2020-12-01', 'Full').identity.tenantId]",
                        "objectId": "[reference(resourceId('Microsoft.Web/sites/', variables('functionAppName')), '2020-12-01', 'Full').identity.principalId]",
                        "permissions": {
                            "secrets": [ "get" ]
                        }
                    }
                ]
            },
            "resources": [
                {
                    "type": "secrets",
                    "name": "[variables('storageConnectionStringName')]",
                    //...
                    "dependsOn": [
                        "[resourceId('Microsoft.KeyVault/vaults/', variables('keyVaultName'))]",
                        "[resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName'))]"
                    ],
                    "properties": {
                        "value": "[concat('DefaultEndpointsProtocol=https;AccountName=', variables('storageAccountName'), ';AccountKey=', listKeys(variables('storageAccountResourceId'),'2019-09-01').key1)]"
                    }
                },
                {
                    "type": "secrets",
                    "name": "[variables('appInsightsKeyName')]",
                    //...
                    "dependsOn": [
                        "[resourceId('Microsoft.KeyVault/vaults/', variables('keyVaultName'))]",
                        "[resourceId('Microsoft.Insights/components', variables('appInsightsName'))]"
                    ],
                    "properties": {
                        "value": "[reference(resourceId('microsoft.insights/components/', variables('appInsightsName')), '2019-09-01').InstrumentationKey]"
                    }
                }
            ]
        }
    ]
}
```

> [!NOTE] 
> En este ejemplo, la implementación del control de código fuente depende de la configuración de la aplicación. Este comportamiento es normalmente poco seguro, ya que la actualización de la configuración de la aplicación se comporta de forma asincrónica. Sin embargo, como hemos incluido la configuración de la aplicación `WEBSITE_ENABLE_SYNC_UPDATE_SITE`, la actualización es sincrónica. Esto significa que la implementación del control de código fuente solo comenzará una vez que la configuración de la aplicación se haya actualizado completamente.

## <a name="troubleshooting-key-vault-references"></a>Solución de problemas de las referencias de Key Vault

Si alguna referencia no se resuelve correctamente, se usará el valor de la referencia, lo que significa que en la configuración de la aplicación se creará una variable de entorno cuyo valor tiene la sintaxis `@Microsoft.KeyVault(...)`. Esto puede provocar que la aplicación genere errores, ya que esperaba un secreto con una estructura determinada.

Habitualmente, esto se debe a una mala configuración de la [directiva de acceso de Key Vault](#granting-your-app-access-to-key-vault). Sin embargo, también puede deberse a que el secreto ya no existe o a un error de sintaxis de la propia referencia.

Si la sintaxis es correcta, compruebe el estado de resolución actual en el portal para ver otras posibles causas del error. Vaya a Configuración de la aplicación y seleccione "Editar" para la referencia en cuestión. En la configuración de opciones, debe ver la información sobre el estado, incluido cualquier error. La ausencia de estos implica que la sintaxis de referencia no es válida.

También puede usar uno de los detectores integrados para obtener información adicional.

### <a name="using-the-detector-for-app-service"></a>Uso del detector en App Service

1. En el portal, vaya a la aplicación.
2. Seleccione **Diagnóstico y solución de problemas**.
3. Elija **Availability and Performance** (Disponibilidad y rendimiento) y seleccione **Web app down** (La aplicación web no funciona).
4. Busque **Key Vault Application Settings Diagnostics** (Diagnóstico de configuración de la aplicación Key Vault) y haga clic en **Más información**.


### <a name="using-the-detector-for-azure-functions"></a>Uso del detector en Azure Functions

1. En el portal, vaya a la aplicación.
2. Seleccione **Características de la plataforma**.
3. Seleccione **Diagnóstico y solución de problemas**.
4. Elija **Disponibilidad y rendimiento** y seleccione **Function app down or reporting errors** (La aplicación de funciones no funciona o presenta errores).
5. Haga clic en **Key Vault Application Settings Diagnostics** (Diagnóstico de configuración de la aplicación Key Vault).
