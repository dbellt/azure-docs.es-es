---
title: Configuración de su propia clave para cifrar datos en reposo de Azure Event Hubs
description: En este artículo se proporciona información sobre cómo configurar su propia clave para cifrar datos en reposo de Azure Event Hubs.
ms.topic: conceptual
ms.date: 05/04/2021
ms.openlocfilehash: 89d12079195406e4b3c6da77105dc359cc1dacae
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110377249"
---
# <a name="configure-customer-managed-keys-for-encrypting-azure-event-hubs-data-at-rest-by-using-the-azure-portal"></a>Configuración de claves administradas por el cliente para cifrar datos en reposo de Azure Event Hubs mediante Azure Portal
Azure Event Hubs proporciona cifrado de datos en reposo con Azure Storage Service Encryption (Azure SSE). El servicio de Event Hubs usa Azure Storage para almacenar los datos. Todos los datos almacenados con Azure Storage se cifran con claves administradas por Microsoft. Si usa su propia clave (también conocida como Bring Your Own Key [BYOK] o clave administrada por el cliente), los datos se cifran mediante la clave administrada por Microsoft, pero además la clave administrada por Microsoft se cifrará mediante la clave administrada por el cliente. Esta característica permite crear, rotar, deshabilitar y revocar el acceso a las claves administradas por el cliente que se usan para cifrar claves administradas por Microsoft. La habilitación de la característica BYOK es un proceso que solo hay que configurar una vez en el espacio de nombres.

> [!IMPORTANT]
> - La funcionalidad BYOK se admite en los niveles **Premium** y **Dedicado** de Event Hubs.
> - El cifrado solo se puede habilitar para espacios de nombres nuevos o vacíos. Si el espacio de nombres contiene centros de eventos, se producirá un error en la operación de cifrado.

Puede usar Azure Key Vault para administrar las claves y auditar su uso. Puede crear sus propias claves y almacenarlas en un almacén de claves, o puede usar las API de Azure Key Vault para generarlas. Para obtener más información sobre Azure Key Vault, consulte [¿Qué es Azure Key Vault?](../key-vault/general/overview.md)

En este artículo se muestra cómo configurar un almacén de claves con claves administradas del cliente mediante Azure Portal. Para obtener información sobre cómo crear un almacén de claves mediante Azure Portal, consulte [Inicio rápido: Creación de una instancia de Azure Key Vault mediante Azure Portal](../key-vault/general/quick-create-portal.md).

> [!IMPORTANT]
> El uso de claves administradas por el cliente con Azure Event Hubs requiere que el almacén de claves tenga configuradas dos propiedades obligatorias. Son las siguientes:  **Eliminación temporal** y **No purgar**. Estas propiedades están habilitadas de manera predeterminada cuando crea un nuevo almacén de claves en Azure Portal. Sin embargo, si tiene que habilitar estas propiedades en un almacén de claves existente, deberá usar PowerShell o la CLI de Azure.

## <a name="enable-customer-managed-keys"></a>Habilitar claves administradas del cliente
Para habilitar las claves administradas por el cliente en Azure Portal, siga estos pasos. Si usa el nivel Dedicado, vaya primero al clúster dedicado de Event Hubs.

1. Seleccione el espacio de nombres en el que desea habilitar BYOK.
1. En la página **Configuración** del espacio de nombres de Event Hubs, seleccione **Cifrado**. 
1. Seleccione **Cifrado de claves en reposo que administra el cliente** como se muestra en la siguiente imagen. 

    ![Habilitación de clave administrada por el cliente](./media/configure-customer-managed-key/enable-customer-managed-key.png)

## <a name="set-up-a-key-vault-with-keys"></a>Configuración de un almacén de claves con claves
Después de habilitar las claves administradas por el cliente, debe asociar la clave administrada por el cliente con su espacio de nombres de Azure Event Hubs. Event Hubs solo admite Azure Key Vault. Si habilita la opción **Encryption with customer-managed key** (Cifrado con clave administrada por el cliente) en la sección anterior, debe importar la clave en Azure Key Vault. Recuerde que las claves deben tener habilitadas las opciones **Eliminación temporal** y **No purgar**. Estas opciones se pueden configurar mediante [PowerShell](../key-vault/general/key-vault-recovery.md) o la [CLI](../key-vault/general/key-vault-recovery.md).

1. Para crear un nuevo almacén de claves, siga el [inicio rápido](../key-vault/general/overview.md) de Azure Key Vault. Para más información sobre cómo importar claves existentes, consulte [Información acerca de claves, secretos y certificados](../key-vault/general/about-keys-secrets-certificates.md).
1. Para activar la eliminación temporal y la protección de purgas al crear un almacén, use el comando [az keyvault create](/cli/azure/keyvault#az_keyvault_create).

    ```azurecli-interactive
    az keyvault create --name ContosoVault --resource-group ContosoRG --location westus --enable-soft-delete true --enable-purge-protection true
    ```    
1. Para agregar la protección de purgas a un almacén existente (que ya tenga habilitada la eliminación temporal), use el comando [az keyvault update](/cli/azure/keyvault#az_keyvault_update).

    ```azurecli-interactive
    az keyvault update --name ContosoVault --resource-group ContosoRG --enable-purge-protection true
    ```
1. Siga estos pasos para crear claves:
    1. Para crear una nueva clave, seleccione **Generar/Importar** en el menú **Claves** en **Configuración**.
        
        ![Seleccione el botón Generate/Import (Generar/importar).](./media/configure-customer-managed-key/select-generate-import.png)
    1. Establezca **Opciones** en **Generar** y asigne un nombre a la clave.

        ![Crear una clave](./media/configure-customer-managed-key/create-key.png) 
    1. Ahora puede seleccionar esta clave para asociarla con el espacio de nombres de Event Hubs para el cifrado en la lista desplegable. 

        ![Selección de clave del almacén de claves](./media/configure-customer-managed-key/select-key-from-key-vault.png)
    1. Rellene los detalles de la clave y haga clic en **Seleccionar**. Esto habilitará el cifrado de la clave administrada por Microsoft con su clave (clave administrada por el cliente). 


## <a name="rotate-your-encryption-keys"></a>Rotación de las claves de cifrado
Puede rotar la clave en el almacén de claves mediante el mecanismo de rotación de los Azure Key Vault. También es posible establecer fechas de activación y expiración para automatizar la rotación de claves. El servicio Event Hubs detectará nuevas versiones de clave y comenzará a usarlas automáticamente.

## <a name="revoke-access-to-keys"></a>Revocación del acceso a las claves
Al revocar el acceso a las claves de cifrado, no se purgan los datos de Event Hubs. Sin embargo, no se podrá acceder a los datos desde el espacio de nombres de Event Hubs. Puede revocar la clave de cifrado mediante la directiva de acceso o eliminando la clave. Obtenga más información sobre las directivas de acceso y la protección del almacén de claves en [Protección del acceso a un almacén de claves](../key-vault/general/security-features.md).

Una vez revocada la clave de cifrado, el servicio Event Hubs en el espacio de nombres cifrado dejará de ser operativo. Si el acceso a la clave está habilitado o si se ha restaurado la clave eliminada, el servicio Event Hubs seleccionará la clave para que pueda acceder a los datos desde el espacio de nombres de Event Hubs cifrado.

## <a name="set-up-diagnostic-logs"></a>Configuración de registros de diagnósticos 
La configuración de los registros de diagnóstico para los espacios de nombres habilitados para BYOK proporciona la información necesaria sobre las operaciones. Estos registros pueden habilitarse y, posteriormente, transmitirse a un centro de eventos, analizarse mediante análisis de registros o transmitirse al almacenamiento para realizar análisis personalizados. Para más información acerca de los registros de diagnóstico, consulte [Información general sobre los registros de diagnóstico de Azure](../azure-monitor/essentials/platform-logs-overview.md).

## <a name="enable-user-logs"></a>Habilitación de registros de usuario
Siga estos pasos para habilitar registros para las claves administradas por el cliente.

1. En Azure Portal, desplácese hasta el espacio de nombres que tiene habilitado BYOK.
1. Seleccione **Configuración de diagnóstico** en **Supervisión**.

    ![Selección de Configuración de diagnóstico](./media/configure-customer-managed-key/select-diagnostic-settings.png)
1. Seleccione **+Agregar configuración de diagnóstico**. 

    ![Selección de Agregar configuración de diagnóstico](./media/configure-customer-managed-key/select-add-diagnostic-setting.png)
1. Proporcione un **nombre** y seleccione dónde desea transmitir los registros.
1. Seleccione **CustomerManagedKeyUserLogs** y **Guardar**. Esta acción habilita los registros de BYOK en el espacio de nombres.

    ![Selección de la opción de registros de usuario de clave administrada por el cliente](./media/configure-customer-managed-key/select-customer-managed-key-user-logs.png)

## <a name="log-schema"></a>Esquema de registro 
Todos los registros se almacenan en el formato de notación de objetos JavaScript (JSON). Cada entrada tiene campos de cadena que usan el formato descrito en la siguiente tabla. 

| Nombre | Descripción |
| ---- | ----------- | 
| TaskName | La descripción de la tarea que generó el error. |
| ActivityId | El identificador interno, usado con fines de seguimiento. |
| category | Define la clasificación de la tarea. Por ejemplo, si la clave del almacén de claves se va a deshabilitar, sería una categoría de información, o si no se puede desencapsular una clave, podría tratarse de un error. |
| resourceId | El identificador de recursos de Azure Resource Manager |
| keyVault | Nombre completo del almacén de claves. |
| key | Nombre de clave que se usa para cifrar el espacio de nombres de Event Hubs. |
| version | Versión de la clave que se va a usar. |
| operation | La operación que se realiza en la clave en el almacén de claves. Por ejemplo, deshabilitar o habilitar la clave, encapsular o desencapsular. |
| código | Código asociado a la operación. Ejemplo: Código de error; 404 indica que no se ha encontrado la clave. |
| message | Cualquier mensaje de error asociado con la operación. |

El siguiente es un ejemplo del registro de una clave administrada por el cliente:

```json
{
   "TaskName": "CustomerManagedKeyUserLog",
   "ActivityId": "11111111-1111-1111-1111-111111111111",
   "category": "error"
   "resourceId": "/SUBSCRIPTIONS/11111111-1111-1111-1111-11111111111/RESOURCEGROUPS/DEFAULT-EVENTHUB-CENTRALUS/PROVIDERS/MICROSOFT.EVENTHUB/NAMESPACES/FBETTATI-OPERA-EVENTHUB",
   "keyVault": "https://mykeyvault.vault-int.azure-int.net",
   "key": "mykey",
   "version": "1111111111111111111111111111111",
   "operation": "wrapKey",
   "code": "404",
   "message": "Key not found: ehbyok0/111111111111111111111111111111",
}



{
   "TaskName": "CustomerManagedKeyUserLog",
   "ActivityId": "11111111111111-1111-1111-1111111111111",
   "category": "info"
   "resourceId": "/SUBSCRIPTIONS/111111111-1111-1111-1111-11111111111/RESOURCEGROUPS/DEFAULT-EVENTHUB-CENTRALUS/PROVIDERS/MICROSOFT.EVENTHUB/NAMESPACES/FBETTATI-OPERA-EVENTHUB",
   "keyVault": "https://mykeyvault.vault-int.azure-int.net",
   "key": "mykey",
   "version": "111111111111111111111111111111",
   "operation": "disable" | "restore",
   "code": "",
   "message": "",
}
```

## <a name="use-resource-manager-template-to-enable-encryption"></a>Uso de la plantilla de Resource Manager para habilitar el cifrado
En esta sección se muestra cómo realizar las siguientes tareas con **plantillas de Azure Resource Manager**. 

1. Cree un **espacio de nombres de Event Hubs** con una identidad de servicio administrada.
2. Cree un **almacén de claves** y conceda a la identidad de servicio acceso a él. 
3. Actualice el espacio de nombres de Event Hubs con la información del almacén de claves (clave/valor). 


### <a name="create-an-event-hubs-cluster-and-namespace-with-managed-service-identity"></a>Creación de un clúster y un espacio de nombres de Event Hubs con la identidad de servicio administrada
En esta sección se muestra cómo crear un espacio de nombres de Azure Event Hubs con la identidad de servicio administrada mediante una plantilla de Azure Resource Manager y PowerShell. 

1. Cree una plantilla de Azure Resource Manager para crear un espacio de nombres de Event Hubs con una identidad de servicio administrada. Asigne al archivo el nombre: **CreateEventHubClusterAndNamespace.json**: 

    ```json
    {
       "$schema":"https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
       "contentVersion":"1.0.0.0",
       "parameters":{
          "clusterName":{
             "type":"string",
             "metadata":{
                "description":"Name for the Event Hub cluster."
             }
          },
          "namespaceName":{
             "type":"string",
             "metadata":{
                "description":"Name for the Namespace to be created in cluster."
             }
          },
          "location":{
             "type":"string",
             "defaultValue":"[resourceGroup().location]",
             "metadata":{
                "description":"Specifies the Azure location for all resources."
             }
          }
       },
       "resources":[
          {
             "type":"Microsoft.EventHub/clusters",
             "apiVersion":"2018-01-01-preview",
             "name":"[parameters('clusterName')]",
             "location":"[parameters('location')]",
             "sku":{
                "name":"Dedicated",
                "capacity":1
             }
          },
          {
             "type":"Microsoft.EventHub/namespaces",
             "apiVersion":"2018-01-01-preview",
             "name":"[parameters('namespaceName')]",
             "location":"[parameters('location')]",
             "identity":{
                "type":"SystemAssigned"
             },
             "sku":{
                "name":"Standard",
                "tier":"Standard",
                "capacity":1
             },
             "properties":{
                "isAutoInflateEnabled":false,
                "maximumThroughputUnits":0,
                "clusterArmId":"[resourceId('Microsoft.EventHub/clusters', parameters('clusterName'))]"
             },
             "dependsOn":[
                "[resourceId('Microsoft.EventHub/clusters', parameters('clusterName'))]"
             ]
          }
       ],
       "outputs":{
          "EventHubNamespaceId":{
             "type":"string",
             "value":"[resourceId('Microsoft.EventHub/namespaces',parameters('namespaceName'))]"
          }
       }
    }
    ```
2. Cree un archivo de parámetros de plantilla llamado: **CreateEventHubClusterAndNamespaceParams.json**. 

    > [!NOTE]
    > Reemplace los siguientes valores: 
    > - `<EventHubsClusterName>`: nombre del clúster de Event Hubs.    
    > - `<EventHubsNamespaceName>`: nombre del espacio de nombres de Event Hubs.
    > - `<Location>`: ubicación del espacio de nombres de Event Hubs.

    ```json
    {
       "$schema":"https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
       "contentVersion":"1.0.0.0",
       "parameters":{
          "clusterName":{
             "value":"<EventHubsClusterName>"
          },
          "namespaceName":{
             "value":"<EventHubsNamespaceName>"
          },
          "location":{
             "value":"<Location>"
          }
       }
    }
    
    ```
3. Ejecute el siguiente comando de PowerShell para implementar la plantilla y crear un espacio de nombres de Event Hubs. A continuación, recupere el identificador del espacio de nombres de Event Hubs para usarlo más adelante. Antes de ejecutar el comando, reemplace `{MyRG}` por el nombre del grupo de recursos.  

    ```powershell
    $outputs = New-AzResourceGroupDeployment -Name CreateEventHubClusterAndNamespace -ResourceGroupName {MyRG} -TemplateFile ./CreateEventHubClusterAndNamespace.json -TemplateParameterFile ./CreateEventHubClusterAndNamespaceParams.json

    $EventHubNamespaceId = $outputs.Outputs["eventHubNamespaceId"].value
    ```
 
### <a name="grant-event-hubs-namespace-identity-access-to-key-vault"></a>Concesión de acceso al almacén de claves a la identidad del espacio de nombres de Event Hubs

1. Ejecute el siguiente comando para crear un almacén de claves con las características **protección de purga** y **eliminación temporal** habilitadas. 

    ```powershell
    New-AzureRmKeyVault -Name {keyVaultName} -ResourceGroupName {RGName}  -Location {location} -EnableSoftDelete -EnablePurgeProtection    
    ```     
    
    O BIEN    
    
    Ejecute el siguiente comando para actualizar un **almacén de claves existente**. Antes de ejecutar el comando, especifique valores para los nombres de grupo de recursos y almacén de claves. 
    
    ```powershell
    ($updatedKeyVault = Get-AzureRmResource -ResourceId (Get-AzureRmKeyVault -ResourceGroupName {RGName} -VaultName {keyVaultName}).ResourceId).Properties| Add-Member -MemberType "NoteProperty" -Name "enableSoftDelete" -Value "true"-Force | Add-Member -MemberType "NoteProperty" -Name "enablePurgeProtection" -Value "true" -Force
    ``` 
2. Establezca la directiva de acceso del almacén de claves para que la identidad administrada del espacio de nombres de Event Hubs pueda acceder al valor de la clave del almacén de claves. Use el identificador del espacio de nombres de Event Hubs de la sección anterior. 

    ```powershell
    $identity = (Get-AzureRmResource -ResourceId $EventHubNamespaceId -ExpandProperties).Identity
    
    Set-AzureRmKeyVaultAccessPolicy -VaultName {keyVaultName} -ResourceGroupName {RGName} -ObjectId $identity.PrincipalId -PermissionsToKeys get,wrapKey,unwrapKey,list
    ```

### <a name="encrypt-data-in-event-hubs-namespace-with-customer-managed-key-from-key-vault"></a>Cifrado de datos en el espacio de nombres de Event Hubs con la clave administrada por el cliente desde el almacén de claves
Hasta el momento ha realizado los pasos siguientes: 

1. Ha creado un espacio de nombres premium con una identidad administrada.
2. Cree un almacén de claves y conceda a la identidad administrada acceso al almacén de claves. 

En este paso, actualizará el espacio de nombres de Event Hubs con la información del almacén de claves. 

1. Cree un archivo JSON llamado **CreateEventHubClusterAndNamespace.json** con el contenido siguiente: 

    ```json
    {
       "$schema":"https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
       "contentVersion":"1.0.0.0",
       "parameters":{
          "clusterName":{
             "type":"string",
             "metadata":{
                "description":"Name for the Event Hub cluster."
             }
          },
          "namespaceName":{
             "type":"string",
             "metadata":{
                "description":"Name for the Namespace to be created in cluster."
             }
          },
          "location":{
             "type":"string",
             "defaultValue":"[resourceGroup().location]",
             "metadata":{
                "description":"Specifies the Azure location for all resources."
             }
          },
          "keyVaultUri":{
             "type":"string",
             "metadata":{
                "description":"URI of the KeyVault."
             }
          },
          "keyName":{
             "type":"string",
             "metadata":{
                "description":"KeyName."
             }
          }
       },
       "resources":[
          {
             "type":"Microsoft.EventHub/namespaces",
             "apiVersion":"2018-01-01-preview",
             "name":"[parameters('namespaceName')]",
             "location":"[parameters('location')]",
             "identity":{
                "type":"SystemAssigned"
             },
             "sku":{
                "name":"Standard",
                "tier":"Standard",
                "capacity":1
             },
             "properties":{
                "isAutoInflateEnabled":false,
                "maximumThroughputUnits":0,
                "clusterArmId":"[resourceId('Microsoft.EventHub/clusters', parameters('clusterName'))]",
                "encryption":{
                   "keySource":"Microsoft.KeyVault",
                   "keyVaultProperties":[
                      {
                         "keyName":"[parameters('keyName')]",
                         "keyVaultUri":"[parameters('keyVaultUri')]"
                      }
                   ]
                }
             }
          }
       ]
    }
    ``` 

2. Cree un archivo de parámetros de plantilla: **UpdateEventHubClusterAndNamespaceParams.json**. 

    > [!NOTE]
    > Reemplace los siguientes valores: 
    > - `<EventHubsClusterName>`: nombre del clúster de Event Hubs.        
    > - `<EventHubsNamespaceName>`: nombre del espacio de nombres de Event Hubs.
    > - `<Location>`: ubicación del espacio de nombres de Event Hubs.
    > - `<KeyVaultName>`: nombre de su almacén de claves.
    > - `<KeyName>`: nombre de la clave del almacén de claves.

    ```json
    {
       "$schema":"https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
       "contentVersion":"1.0.0.0",
       "parameters":{
          "clusterName":{
             "value":"<EventHubsClusterName>"
          },
          "namespaceName":{
             "value":"<EventHubsNamespaceName>"
          },
          "location":{
             "value":"<Location>"
          },
          "keyName":{
             "value":"<KeyName>"
          },
          "keyVaultUri":{
             "value":"https://<KeyVaultName>.vault.azure.net"
          }
       }
    }
    ```             
3. Ejecute el siguiente comando de PowerShell para implementar la plantilla de Resource Manager. Antes de ejecutar el comando, reemplace `{MyRG}` por el nombre del grupo de recursos. 

    ```powershell
    New-AzResourceGroupDeployment -Name UpdateEventHubNamespaceWithEncryption -ResourceGroupName {MyRG} -TemplateFile ./UpdateEventHubClusterAndNamespace.json -TemplateParameterFile ./UpdateEventHubClusterAndNamespaceParams.json 
    ```

## <a name="troubleshoot"></a>Solución de problemas
Como procedimiento recomendado, habilite siempre los registros como se muestra en la sección anterior. Esto ayuda a realizar el seguimiento de las actividades cuando está habilitado el cifrado de BYOK. También ayuda a limitar los problemas.

A continuación se muestran los códigos de error comunes que buscar cuando está habilitado el cifrado de BYOK.

| Acción | Código de error | Estado resultante de los datos |
| ------ | ---------- | ----------------------- | 
| Quitar el permiso de encapsular/desencapsular de un almacén de claves | 403 |    Inaccessible |
| Quitar la pertenencia al rol de AAD de una entidad de seguridad de AAD que concedió el permiso de encapsular/desencapsular | 403 |  Inaccessible |
| Eliminar una clave de cifrado del almacén de claves | 404 | Inaccessible |
| Eliminar el almacén de claves | 404 | Inaccesible (se da por supuesto que la eliminación temporal está habilitada, al ser una opción obligatoria) |
| Cambiar el período de expiración de la clave de cifrado para que ya haya expirado | 403 |   Inaccessible  |
| Cambiar el valor NBF (no antes), de modo que la clave de cifrado de clave no esté activa | 403 | Inaccessible  |
| Seleccionar la opción **Allow MSFT Services** (Permitir servicios MSFT) para el firewall del almacén de claves o bloquear el acceso de red al almacén de claves que tiene la clave de cifrado | 403 | Inaccessible |
| Mover el almacén de claves a un inquilino diferente | 404 | Inaccessible |  
| Problema de red intermitente o interrupción de DNS/AAD/MSI |  | Accesible mediante clave de cifrado de datos en caché |

> [!IMPORTANT]
> Para habilitar la recuperación ante desastres con localización geográfica en un espacio de nombres que use el cifrado de BYOK, el espacio de nombres secundario para el emparejamiento debe estar en un clúster dedicado y debe tener habilitada en este una identidad administrada asignada por el sistema. Para más información, consulte [Identidades administradas para recursos de Azure](../active-directory/managed-identities-azure-resources/overview.md).

## <a name="next-steps"></a>Pasos siguientes
Vea los artículos siguientes:
- [Información general de Event Hubs](event-hubs-about.md)
- [Introducción a Azure Key Vault](../key-vault/general/overview.md)