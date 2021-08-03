---
title: Identidad administrada de Azure IoT Hub | Microsoft Docs
description: Uso de identidades administradas para permitir la conectividad de salida desde IoT Hub a otros recursos de Azure.
author: miag
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 05/11/2021
ms.author: miag
ms.openlocfilehash: 7122cfc12e47734b84aab752901fa9750b7e5164
ms.sourcegitcommit: 34feb2a5bdba1351d9fc375c46e62aa40bbd5a1f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111892002"
---
# <a name="iot-hub-support-for-managed-identities"></a>Compatibilidad de IoT Hub con identidades administradas 

Las identidades administradas proporcionan a los servicios de Azure una identidad administrada automáticamente en Azure AD de manera segura. Esto elimina la necesidad de que los desarrolladores tengan que administrar las credenciales proporcionando una identidad. Hay dos tipos de identidades administradas: asignadas por el sistema y asignadas por el usuario. IoT Hub admite ambas. 

En IoT Hub, las identidades administradas se pueden usar para la conectividad de salida de IoT Hub a otros servicios de Azure para las características como el [enrutamiento de mensajes](iot-hub-devguide-messages-d2c.md), la [carga de archivos](iot-hub-devguide-file-upload.md) y la [importación o exportación masivas de dispositivos](iot-hub-bulk-identity-mgmt.md). En este artículo, aprenderá a usar identidades administradas asignadas por el sistema y asignadas por el usuario en IoT Hub para distintas funcionalidades. 


## <a name="prerequisites"></a>Requisitos previos
- Lea la documentación de [identidades administradas para recursos de Azure](./../active-directory/managed-identities-azure-resources/overview.md) para comprender las diferencias entre las identidades administradas asignadas por el sistema y por el usuario.

- Si no tiene un centro de IoT, [cree uno](iot-hub-create-through-portal.md) antes de continuar.


## <a name="system-assigned-managed-identity"></a>Identidad administrada asignada por el sistema 

### <a name="add-and-remove-a-system-assigned-managed-identity-in-azure-portal"></a>Adición y eliminación de una identidad administrada asignada por el sistema en Azure Portal
1.  Inicie sesión en Azure Portal y vaya al centro de IoT que quiera.
2.  Vaya a **Identidad** en el portal de IoT Hub.
3.  En la pestaña **System-assigned** (Asignada por el sistema), seleccione **Activar** y haga clic en **Guardar**.
4.  Para quitar la identidad administrada asignada por el sistema de un centro de IoT, seleccione **Desactivar** y haga clic en **Guardar**.

    :::image type="content" source="./media/iot-hub-managed-identity/system-assigned.png" alt-text="Captura de pantalla que muestra dónde activar la identidad administrada asignada por el sistema para un centro de IoT":::        

### <a name="enable-system-assigned-managed-identity-at-hub-creation-time-using-arm-template"></a>Habilitación de la identidad administrada asignada por el sistema en el momento de la creación del centro con la plantilla de ARM

Para habilitar la identidad administrada asignada por el sistema en el centro de IoT en el momento del aprovisionamiento de recursos, use la plantilla de Azure Resource Manager (ARM) siguiente. 

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": 
    {
      "iotHubName": {
        "type": "string",
        "metadata": {
          "description": "Name of iothub resource"
        }
      },
      "skuName": {
        "type": "string",
        "defaultValue": "S1",
        "metadata": {
          "description": "SKU name of iothub resource, by default is Standard S1"
        }
      },
      "skuTier": {
        "type": "string",
        "defaultValue": "Standard",
        "metadata": {
          "description": "SKU tier of iothub resource, by default is Standard"
        }
      },
      "location": {
        "type": "string",
        "defaultValue": "[resourceGroup().location]",
        "metadata": {
          "description": "Location of iothub resource. Please provide any of supported-regions of iothub"
        }
      }
    },
  "resources": [
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2020-10-01",
      "name": "createIotHub",
      "properties": {
        "mode": "Incremental",
        "template": {
          "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
          "contentVersion": "1.0.0.0",
          "resources": [
            {
              "type": "Microsoft.Devices/IotHubs",
              "apiVersion": "2021-03-31",
              "name": "[parameters('iotHubName')]",
              "location": "[parameters('location')]",
              "identity": {
                "type": "SystemAssigned"
              },
              "sku": {
              "name": "[parameters('skuName')]",
              "tier": "[parameters('skuTier')]",
              "capacity": 1
              }
            }
          ] 
        }
      }
    }
  ]
}
```

Después de sustituir los valores del recurso `name`, `location`, `SKU.name` y `SKU.tier`, puede usar la CLI de Azure para implementar el recurso en un grupo de recursos existente mediante:

```azurecli-interactive
az deployment group create --name <deployment-name> --resource-group <resource-group-name> --template-file <template-file.json> --parameters iotHubName=<valid-iothub-name> skuName=<sku-name> skuTier=<sku-tier> location=<any-of-supported-regions>
```

Una vez creado el recurso, puede recuperar la identidad asignada por el sistema al centro mediante la CLI de Azure:

```azurecli-interactive
az resource show --resource-type Microsoft.Devices/IotHubs --name <iot-hub-resource-name> --resource-group <resource-group-name>
```
## <a name="user-assigned-managed-identity"></a>Identidad administrada asignada por el usuario 
En esta sección, aprenderá a agregar y quitar una identidad administrada asignada por el usuario desde un centro de IoT mediante Azure Portal.
1.  En primer lugar, debe crear una identidad administrada asignada por el usuario como un recurso independiente. Puede seguir las instrucciones que encontrará [aquí](./../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-portal.md#create-a-user-assigned-managed-identity) para crear una identidad administrada asignada por el usuario.
2.  Vaya al centro de IoT y navegue hasta **Identidad** en el portal de IoT Hub.
3.  En la pestaña **User-Assigned** (Asignada por el usuario), haga clic en **Add user-assigned managed identity** (Agregar identidad administrada por el usuario). Elija la identidad administrada asignada por el usuario que desea agregar al centro y, a continuación, haga clic en **Seleccionar**. 
4.  Puede quitar una identidad asignada por el usuario de un centro de IoT. Elija la identidad asignada por el usuario que quiera quitar y haga clic en el botón **Quitar**. Tenga en cuenta que solo la quita del centro de IoT y no se elimina la identidad asignada por el usuario como recurso. Para eliminar la identidad asignada por el usuario como recurso, siga las instrucciones que se indican [aquí](./../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-portal.md#delete-a-user-assigned-managed-identity).

    :::image type="content" source="./media/iot-hub-managed-identity/user-assigned.png" alt-text="Captura de pantalla que muestra cómo agregar una identidad administrada asignada por el usuario para un centro de IoT":::        


### <a name="enable-user-assigned-managed-identity-at-hub-creation-time-using-arm-template"></a>Habilitación de la identidad administrada asignada por el usuario en el momento de la creación del centro con la plantilla de ARM
A continuación, se muestra la plantilla de ejemplo que se puede usar para crear un centro con una identidad administrada asignada por el usuario. Esta plantilla crea una identidad asignada por el usuario con el nombre *[iothub-name-provided]-identity* y la asigna al centro de IoT creado. Puede cambiar la plantilla para agregar varias identidades asignadas por el usuario según sea necesario.
 
```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "iotHubName": {
      "type": "string",
      "metadata": {
        "description": "Name of iothub resource"
      }
    },
  "skuName": {
    "type": "string",
    "defaultValue": "S1",
    "metadata": {
      "description": "SKU name of iothub resource, by default is Standard S1"
    }
  },
  "skuTier": {
    "type": "string",
    "defaultValue": "Standard",
    "metadata": {
      "description": "SKU tier of iothub resource, by default is Standard"
    }
  },
  "location": {
    "type": "string",
    "defaultValue": "[resourceGroup().location]",
    "metadata": {
      "description": "Location of iothub resource. Please provide any of supported-regions of iothub"
    }
  }
},
  "variables": {
    "identityName": "[concat(parameters('iotHubName'), '-identity')]"
  },
  "resources": [
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2020-10-01",
      "name": "createIotHub",
      "properties": {
        "mode": "Incremental",
        "template": {
          "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
          "contentVersion": "1.0.0.0",
          "resources": [
            {
              "type": "Microsoft.ManagedIdentity/userAssignedIdentities",
              "name": "[variables('identityName')]",
              "apiVersion": "2018-11-30",
              "location": "[resourceGroup().location]"
            },
            {
              "type": "Microsoft.Devices/IotHubs",
              "apiVersion": "2021-03-31",
              "name": "[parameters('iotHubName')]",
              "location": "[parameters('location')]",
              "identity": {
                "type": "UserAssigned",
                "userAssignedIdentities": {
                  "[resourceID('Microsoft.ManagedIdentity/userAssignedIdentities/',variables('identityName'))]": {}
                }
              },
              "sku": {
                "name": "[parameters('skuName')]",
                "tier": "[parameters('skuTier')]",
                "capacity": 1
              },
              "dependsOn": [
                "[resourceID('Microsoft.ManagedIdentity/userAssignedIdentities/',variables('identityName'))]"
              ]
            }
          ]
        }
      }
    }
  ]
}
```
```azurecli-interactive
az deployment group create --name <deployment-name> --resource-group <resource-group-name> --template-file <template-file.json> --parameters iotHubName=<valid-iothub-name> skuName=<sku-name> skuTier=<sku-tier> location=<any-of-supported-regions>
```

Una vez creado el recurso, puede recuperar la identidad administrada asignada por el usuario al centro mediante la CLI de Azure:

```azurecli-interactive
az resource show --resource-type Microsoft.Devices/IotHubs --name <iot-hub-resource-name> --resource-group <resource-group-name>
```
## <a name="egress-connectivity-from-iot-hub-to-other-azure-resources"></a>Conectividad de salida de IoT Hub a otros recursos de Azure
En IoT Hub, las identidades administradas se pueden usar para la conectividad de salida de IoT Hub a otros servicios de Azure para el [enrutamiento de mensajes](iot-hub-devguide-messages-d2c.md), la [carga de archivos](iot-hub-devguide-file-upload.md) y la [importación o exportación masivas de dispositivos](iot-hub-bulk-identity-mgmt.md). Puede elegir qué identidad administrada usar para cada conexión de salida de IoT Hub a los puntos de conexión propiedad del cliente, incluidas las cuentas de almacenamiento, los centros de eventos y los puntos de conexión de Service Bus. 

### <a name="message-routing"></a>Enrutamiento de mensajes
En esta sección, usaremos el [enrutamiento de mensajes](iot-hub-devguide-messages-d2c.md) al punto de conexión personalizado del centro de eventos como ejemplo. Lo mismo se aplica a otros puntos de conexión personalizados de enrutamiento. 

1.  En primer lugar, tenemos que ir al centro de eventos en Azure Portal, para asignar el acceso correcto a la identidad administrada. En el centro de eventos, vaya a la pestaña **Control de acceso (IAM)** y haga clic en **Agregar** y, a continuación, en **Agregar una asignación de roles**.
3.  Seleccione **Event Hubs Data Sender as role** (Remitente de datos de Event Hubs como rol).

    > [!NOTE] 
    > Para la cuenta de almacenamiento, seleccione **Colaborador de datos de Storage Blob** ([*no* Colaborador ni Colaborador de la cuenta de almacenamiento](../storage/common/storage-auth-aad-rbac-portal.md#azure-roles-for-blobs-and-queues)) como **rol**. En Service Bus, seleccione **Remitente de datos de Service Bus** como **rol**.

4.  Para la asignada por el usuario, elija **Identidad administrada asignada por el usuario** en **Asignar acceso a**. Seleccione la suscripción y la identidad administrada asignada por el usuario en la lista desplegable. Haga clic en el botón **Save** (Guardar).

    :::image type="content" source="./media/iot-hub-managed-identity/eventhub-iam-user-assigned.png" alt-text="Enrutamiento de mensajes de IoT Hub con identidad asignada por el usuario":::

5.  En el caso de la asignada por el sistema, en **Asignar acceso a**, elija **User, group, or service principal** (Usuario, grupo o entidad de servicio) y seleccione el nombre del recurso del centro de IoT en la lista desplegable. Haga clic en **Save**(Guardar).

    :::image type="content" source="./media/iot-hub-managed-identity/eventhub-iam-system-assigned.png" alt-text="Enrutamiento de mensajes de IoT Hub con identidad asignada por el sistema":::

    Si necesita restringir la conectividad al punto de conexión personalizado a través de una red virtual, debe activar la excepción de primera entidad de confianza de Microsoft para proporcionar a su centro de IoT acceso al punto de conexión específico. Por ejemplo, si agrega un punto de conexión personalizado del centro de eventos, vaya a la pestaña **Firewalls y redes virtuales** del centro de eventos y habilite la opción **Allow access from selected networks** (Permitir el acceso desde redes seleccionadas). En la lista **Excepciones**, active la casilla situada junto a **Allow trusted Microsoft services to access event hubs** (Permitir que los servicios de Microsoft de confianza accedan a esta instancia de Event Hubs). Haga clic en el botón **Save** (Guardar). Esto también se aplica a la cuenta de almacenamiento y a Service Bus. Obtenga información sobre la [compatibilidad de Azure IoT Hub con redes virtuales](./virtual-network-support.md).

    > [!NOTE]
    > Debe completar los pasos anteriores para asignar a la identidad administrada el acceso correcto antes de agregar el centro de eventos como punto de conexión personalizado en IoT Hub. Espere unos minutos a que se propague la asignación de roles. 

6. A continuación, vaya a IoT Hub. En el centro, vaya a **Enrutamiento de mensajes** y, a continuación, haga clic en **Puntos de conexión personalizados.** Haga clic en **Agregar** y elija el tipo de punto de conexión que quiere usar. En esta sección, usaremos el centro de eventos como ejemplo.
7.  En la parte inferior de la página, elija el **Tipo de autenticación** que prefiera. En esta sección, usaremos **User-Assigned** (Asignada por el usuario) como en el ejemplo. En la lista desplegable, seleccione la identidad administrada asignada por el usuario preferida y, a continuación, haga clic en **Crear**.

    :::image type="content" source="./media/iot-hub-managed-identity/eventhub-routing-endpoint.png" alt-text="Centro de eventos de IoT Hub con identidad asignada por el usuario":::

8. Punto de conexión personalizado creado correctamente. 
9. Después de la creación, todavía puede cambiar el tipo de autenticación. Seleccione el punto de conexión personalizado para el que quiera cambiar el tipo de autenticación y, a continuación, haga clic en **Change authentication type** (Cambiar tipo de autenticación).

    :::image type="content" source="./media/iot-hub-managed-identity/change-authentication-type.png" alt-text="Tipo de autenticación de IoT Hub":::

10. Elija el nuevo tipo de autenticación que se debe actualizar para este punto de conexión y haga clic en **Guardar**.

### <a name="file-upload"></a>Carga de archivos
La característica de [carga de archivos](iot-hub-devguide-file-upload.md) de IoT Hub permite a los dispositivos cargar archivos en una cuenta de almacenamiento propiedad del cliente. Para permitir que la carga de archivos funcione, IoT Hub debe tener conectividad con la cuenta de almacenamiento. De forma similar al enrutamiento de mensajes, puede elegir el tipo de autenticación preferido y la identidad administrada para la conectividad de salida de IoT Hub a la cuenta de Azure Storage. 

1. En Azure Portal, vaya a la pestaña **Control de acceso (IAM)** de la cuenta de almacenamiento y haga clic en **Agregar** en la sección **Agregar una asignación de roles**.
2. Seleccione **Colaborador de datos de Storage Blob** (no Colaborador ni Colaborador de la cuenta de almacenamiento) como rol.
3. Para la asignada por el usuario, elija **Identidad administrada asignada por el usuario** en Asignar acceso a. Seleccione la suscripción y la identidad administrada asignada por el usuario en la lista desplegable. Haga clic en el botón **Save** (Guardar).
4. En el caso de la asignada por el sistema, en **Asignar acceso a**, elija **User, group, or service principal** (Usuario, grupo o entidad de servicio) y seleccione el nombre del recurso del centro de IoT en la lista desplegable. Haga clic en **Save**(Guardar).

    Si necesita restringir la conectividad a la cuenta de almacenamiento a través de una red virtual, debe activar la excepción de primera entidad de confianza de Microsoft para proporcionar a su centro de IoT acceso a la cuenta de almacenamiento. En la página de recursos de la cuenta de almacenamiento, vaya a la pestaña **Firewalls y redes virtuales** y habilite la opción **Allow access from selected networks** (Permitir acceso desde redes seleccionadas). En la lista **Excepciones**, active la casilla situada junto a **Permitir que los servicios de Microsoft de confianza accedan a esta cuenta de almacenamiento**. Haga clic en el botón **Save** (Guardar). Obtenga información sobre la [compatibilidad de Azure IoT Hub con redes virtuales](./virtual-network-support.md). 


    > [!NOTE]
    > Debe completar los pasos anteriores para asignar a la identidad administrada el acceso correcto antes de guardar la cuenta de almacenamiento en IoT Hub para la carga de archivos mediante la identidad administrada. Espere unos minutos a que se propague la asignación de roles. 
 
5. En la página de recursos del centro de IoT, vaya a la pestaña **Carga de archivo**.
6. En la página que se muestra, seleccione el contenedor que quiere usar en Blob Storage, y configure las opciones **Configuración de notificación de archivos, TTL de SAS, TTL predeterminado y Número máximo de entregas**. Elija el tipo de autenticación preferido y haga clic en **Guardar**.

    :::image type="content" source="./media/iot-hub-managed-identity/file-upload.png" alt-text="Carga de archivos de IoT Hub con msi":::

    > [!NOTE]
    > En el escenario de carga de archivos, tanto el centro como el dispositivo deben conectarse con la cuenta de almacenamiento. Los pasos anteriores sirven para conectar el centro de IoT a la cuenta de almacenamiento con el tipo de autenticación deseado. Todavía debe conectar el dispositivo al almacenamiento mediante el URI de SAS. Actualmente, el URI de SAS se genera mediante la cadena de conexión. Pronto se agregará compatibilidad para generar el URI de SAS con la identidad administrada. Siga los pasos descritos en [Carga de archivos](iot-hub-devguide-file-upload.md).

### <a name="bulk-device-importexport"></a>Importación/exportación masiva de dispositivos

IoT Hub admite la funcionalidad para [importar/exportar información de dispositivos](iot-hub-bulk-identity-mgmt.md) de forma masiva desde y hacia una instancia de Storage Blob proporcionada por el cliente. Esta funcionalidad requiere conectividad de IoT Hub a la cuenta de almacenamiento. 

1. En Azure Portal, vaya a la pestaña **Control de acceso (IAM)** de la cuenta de almacenamiento y haga clic en **Agregar** en la sección **Agregar una asignación de roles**.
2. Seleccione **Colaborador de datos de Storage Blob** (no Colaborador ni Colaborador de la cuenta de almacenamiento) como rol.
3. Para la asignada por el usuario, elija **Identidad administrada asignada por el usuario** en Asignar acceso a. Seleccione la suscripción y la identidad administrada asignada por el usuario en la lista desplegable. Haga clic en el botón **Save** (Guardar).
4. En el caso de la asignada por el sistema, en **Asignar acceso a**, elija **User, group, or service principal** (Usuario, grupo o entidad de servicio) y seleccione el nombre del recurso del centro de IoT en la lista desplegable. Haga clic en **Save**(Guardar).


### <a name="using-rest-api-or-sdk-for-import-and-export-jobs"></a>Uso de la API de REST o el SDK para trabajos de importación y exportación

Ahora puede usar la API de REST de Azure IoT para crear trabajos de importación y exportación. Deberá proporcionar las siguientes propiedades en el cuerpo de la solicitud:

- **storageAuthenticationType**: establezca el valor en **identityBased.** 
- **inputBlobContainerUri**: establezca esta propiedad solo en el trabajo de importación.
- **outputBlobContainerUri**: establezca esta propiedad para los trabajos de importación y exportación.
- **identity**: establezca el valor en la identidad administrada que se usará.


Los SDK de Azure IoT Hub también admiten esta funcionalidad en el administrador del Registro del cliente del servicio. El fragmento de código siguiente muestra cómo iniciar un trabajo de importación o exportación al usar el SDK de C#.

**Fragmento de código C#**

```csharp
    // Create an export job
 
    using RegistryManager srcRegistryManager = RegistryManager.CreateFromConnectionString(hubConnectionString);
 
    JobProperties jobProperties = JobProperties.CreateForExportJob(
        outputBlobContainerUri: blobContainerUri,
        excludeKeysInExport: false,
        storageAuthenticationType: StorageAuthenticationType.IdentityBased,
        identity: new ManagedIdentity
        {
            userAssignedIdentity = userDefinedManagedIdentityResourceId
        });
```
 
```csharp
    // Create an import job
    
    using RegistryManager destRegistryManager = RegistryManager.CreateFromConnectionString(hubConnectionString);
 
    JobProperties jobProperties = JobProperties.CreateForImportJob(
        inputBlobContainerUri: blobContainerUri,
        outputBlobContainerUri: blobContainerUri,
        storageAuthenticationType: StorageAuthenticationType.IdentityBased,
        identity: new ManagedIdentity
        {
            userAssignedIdentity = userDefinedManagedIdentityResourceId
        });
```  

**Fragmento de código Python**

```python
# see note below
iothub_job_manager = IoTHubJobManager("<IoT Hub connection string>")

# Create an import job
result = iothub_job_manager.create_import_export_job(JobProperties(
    type="import",
    input_blob_container_uri="<input container URI>",
    output_blob_container_uri="<output container URI>",
    storage_authentication_type="identityBased",
    identity=ManagedIdentity(
        user_assigned_identity="<resource ID of user assigned managed identity>"
    )
))

# Create an export job
result = iothub_job_manager.create_import_export_job(JobProperties(
    type="export",
    output_blob_container_uri="<output container URI>",
    storage_authentication_type="identityBased",
    exclude_keys_in_export=True,
    identity=ManagedIdentity(
        user_assigned_identity="<resource ID of user assigned managed identity>"
    ) 
))
```

> [!NOTE]
> - Si **storageAuthenticationType** se establece en **identityBased** y la propiedad **userAssignedIdentity** no es **null**, los trabajos usarán la identidad administrada asignada por el usuario especificada.
> - Si el centro de IoT no está configurado con la identidad administrada asignada por el usuario especificada en **userAssignedIdentity**, se producirá un error en el trabajo.
> - Si **storageAuthenticationType** se establece en **identityBased**, la propiedad **userAssignedIdentity** es NULL, los trabajos usarán la identidad asignada por el sistema.
> - Si el centro de IoT no está configurado con la identidad administrada asignada por el usuario, se producirá un error en el trabajo.
> - Si **storageAuthenticationType** se establece en **identityBased** y no se configuran identidades administradas **asignadas por el usuario** ni **asignadas por el sistema** en el centro, se producirá un error en el trabajo.

## <a name="sdk-samples"></a>Ejemplos del SDK
- [Ejemplo de SDK de .NET](https://aka.ms/iothubmsicsharpsample)
- [Ejemplo de SDK de Java](https://aka.ms/iothubmsijavasample)
- [Ejemplo de SDK de Python](https://aka.ms/iothubmsipythonsample)
- SDK de Node.js: [importación masiva de dispositivos](https://aka.ms/iothubmsinodesampleimport), [exportación masiva de dispositivos](https://aka.ms/iothubmsinodesampleexport)

## <a name="next-steps"></a>Pasos siguientes

Use los vínculos siguientes para obtener más información sobre las características de IoT Hub:

* [Enrutamiento de mensajes](./iot-hub-devguide-messages-d2c.md)
* [Carga de archivos](./iot-hub-devguide-file-upload.md)
* [Importación/exportación masiva de dispositivos](./iot-hub-bulk-identity-mgmt.md)
