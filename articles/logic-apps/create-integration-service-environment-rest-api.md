---
title: Creación de entornos del servicio de integración (ISE) en la API REST de Logic Apps
description: Creación de un entorno del servicio de integración (ISE) mediante la API REST de Logic Apps para acceder a las redes virtuales de Azure desde Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: rarayudu, logicappspm
ms.topic: conceptual
ms.date: 02/03/2021
ms.openlocfilehash: fa0b37c46fd39a1764c356aa910347cafaa7d448
ms.sourcegitcommit: 32ee8da1440a2d81c49ff25c5922f786e85109b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109787264"
---
# <a name="create-an-integration-service-environment-ise-by-using-the-logic-apps-rest-api"></a>Creación de un entorno del servicio de integración (ISE) mediante la API REST de Logic Apps

En escenarios en los que las aplicaciones lógicas y las cuentas de integración necesitan acceder a una [red virtual de Azure](../virtual-network/virtual-networks-overview.md), puede crear un [*entorno de servicio de integración* (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md) mediante el uso de la API de REST de Logic Apps. Para más información sobre los ISE, consulte [Acceso a recursos de Azure Virtual Network desde Azure Logic Apps](connect-virtual-network-vnet-isolated-environment-overview.md).

En este artículo se muestra cómo crear un ISE mediante la API de REST de Logic Apps en general. Opcionalmente, también puede habilitar una [identidad administrada asignada por el sistema o por el usuario](../active-directory/managed-identities-azure-resources/overview.md#managed-identity-types) en el ISE, pero solo con la API de REST de Logic Apps en este momento. Esta identidad permite al ISE autenticar el acceso a recursos seguros, como máquinas virtuales y otros sistemas o servicios que están dentro de una red virtual de Azure o conectados a ella. De este modo, no tiene que iniciar sesión con sus credenciales.

Para obtener más información sobre otras formas de crear un ISE, consulte estos artículos:

* [Creación de un ISE en Azure Portal](../logic-apps/connect-virtual-network-vnet-isolated-environment.md)
* [Creación de un ISE mediante la plantilla de inicio rápido de Azure Resource Manager de ejemplo](https://github.com/Azure/azure-quickstart-templates/tree/master/201-integration-service-environment)
* [Creación de un ISE que admite el uso de claves administradas por el cliente para cifrar datos en reposo](customer-managed-keys-integration-service-environment.md)

## <a name="prerequisites"></a>Requisitos previos

* Los mismos [requisitos previos](../logic-apps/connect-virtual-network-vnet-isolated-environment.md#prerequisites) y [requisitos de acceso](../logic-apps/connect-virtual-network-vnet-isolated-environment.md#enable-access) que cuando se crea un ISE en Azure Portal.

* Cualquier recurso adicional que desee usar con el ISE para que pueda incluir su información en la definición de ISE, por ejemplo: 

  * Para habilitar la compatibilidad con certificados autofirmados, debe incluir información sobre dicho certificado en la definición de ISE.

  * Para habilitar la identidad administrada asignada por el usuario, debe crear esa identidad de antemano e incluir las propiedades `objectId`, `principalId` y `clientId` y sus valores en la definición de ISE. Para obtener más información, vea [Creación de una identidad administrada asignada por el usuario en Azure Portal](../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-portal.md#create-a-user-assigned-managed-identity).

* Una herramienta que puede usar para crear su ISE mediante una llamada a la API de REST de Logic Apps con una solicitud HTTPS PUT. Por ejemplo, puede usar [Postman](https://www.getpostman.com/downloads/) o puede compilar una aplicación lógica que realice esta tarea.

## <a name="create-the-ise"></a>Creación del ISE

Para crear su ISE mediante una llamada a la API de REST de Logic Apps, realice esta solicitud HTTPS PUT.

`PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Logic/integrationServiceEnvironments/{integrationServiceEnvironmentName}?api-version=2019-05-01`

> [!IMPORTANT]
> La versión 2019-05-01 de la API de REST de Logic Apps requiere que realice su propia solicitud HTTP PUT para conectores ISE.

Normalmente, la implementación tarda un máximo de dos horas en completarse. En ocasiones, la implementación puede tardar hasta cuatro horas. Para comprobar el estado de implementación, en [Azure Portal](https://portal.azure.com), en la barra de herramientas de Azure, seleccione el icono de notificaciones, que abre el panel de notificaciones.

> [!NOTE]
> Si se produce un error en la implementación o se elimina el ISE, Azure podría tardar hasta una hora en liberar las subredes. Debido a este retraso, es posible que deba esperar antes de volver a usar esas subredes en otro ISE.
>
> Si elimina su red virtual, Azure generalmente tarda hasta dos horas antes de liberar las subredes, pero esta operación puede llevar más tiempo. 
> Cuando elimine redes virtuales, asegúrese de que no haya recursos conectados. 
> Consulte [Eliminar red virtual](../virtual-network/manage-virtual-network.md#delete-a-virtual-network).

## <a name="request-header"></a>Encabezado de solicitud

En el encabezado de solicitud, incluya estas propiedades:

* `Content-type`: Establezca el valor de propiedad en `application/json`.

* `Authorization`: Establezca este valor de propiedad en el token de portador para el cliente que tiene acceso al grupo de recursos o la suscripción de Azure que desea usar.

<a name="request-body"></a>

## <a name="request-body"></a>Cuerpo de la solicitud

En el cuerpo de la solicitud, proporcione la definición de recursos que se usará para crear el ISE, incluida la información sobre las funcionalidades adicionales que desea habilitar en el ISE, por ejemplo:

* Para crear un ISE que permita el uso de un certificado autofirmado y un certificado emitido por la entidad de certificación de empresa que está instalado en la ubicación `TrustedRoot`, incluya el objeto `certificates` dentro de la sección `properties` de la definición del ISE, como se describe más adelante en este artículo.

* Para crear un ISE que use una identidad administrada asignada por el sistema o por el usuario, incluya el objeto `identity` con el tipo de identidad administrada y otra información necesaria en la definición del ISE, como se describe más adelante en este artículo.

* Para crear un ISE que use claves administradas por el cliente y Azure Key Vault para cifrar datos en reposo, incluya la [información que habilita la compatibilidad con las claves administradas por el cliente](customer-managed-keys-integration-service-environment.md). Puede configurar claves administradas por el cliente *solo durante la creación*, no después.

### <a name="request-body-syntax"></a>Sintaxis del cuerpo de la solicitud

Esta es la sintaxis del cuerpo de la solicitud, que describe las propiedades que se deben usar al crear el ISE:

```json
{
   "id": "/subscriptions/{Azure-subscription-ID}/resourceGroups/{Azure-resource-group}/providers/Microsoft.Logic/integrationServiceEnvironments/{ISE-name}",
   "name": "{ISE-name}",
   "type": "Microsoft.Logic/integrationServiceEnvironments",
   "location": "{Azure-region}",
   "sku": {
      "name": "Premium",
      "capacity": 1
   },
   // Include the `identity` object to enable the system-assigned identity or user-assigned identity
   "identity": {
      "type": <"SystemAssigned" | "UserAssigned">,
      // When type is "UserAssigned", include the following "userAssignedIdentities" object:
      "userAssignedIdentities": {
         "/subscriptions/{Azure-subscription-ID}/resourceGroups/{Azure-resource-group}/providers/Microsoft.ManagedIdentity/userAssignedIdentities/{user-assigned-managed-identity-object-ID}": {
            "principalId": "{principal-ID}",
            "clientId": "{client-ID}"
         }
      }
   },
   "properties": {
      "networkConfiguration": {
         "accessEndpoint": {
            // Your ISE can use the "External" or "Internal" endpoint. This example uses "External".
            "type": "External"
         },
         "subnets": [
            {
               "id": "/subscriptions/{Azure-subscription-ID}/resourceGroups/{Azure-resource-group}/providers/Microsoft.Network/virtualNetworks/{virtual-network-name}/subnets/{subnet-1}",
            },
            {
               "id": "/subscriptions/{Azure-subscription-ID}/resourceGroups/{Azure-resource-group}/providers/Microsoft.Network/virtualNetworks/{virtual-network-name}/subnets/{subnet-2}",
            },
            {
               "id": "/subscriptions/{Azure-subscription-ID}/resourceGroups/{Azure-resource-group}/providers/Microsoft.Network/virtualNetworks/{virtual-network-name}/subnets/{subnet-3}",
            },
            {
               "id": "/subscriptions/{Azure-subscription-ID}/resourceGroups/{Azure-resource-group}/providers/Microsoft.Network/virtualNetworks/{virtual-network-name}/subnets/{subnet-4}",
            }
         ]
      },
      // Include `certificates` object to enable self-signed certificate and the certificate issued by Enterprise Certificate Authority
      "certificates": {
         "testCertificate": {
            "publicCertificate": "{base64-encoded-certificate}",
            "kind": "TrustedRoot"
         }
      }
   }
}
```

### <a name="request-body-example"></a>Ejemplo de cuerpo de solicitud

En este cuerpo de solicitud de ejemplo se muestran los valores de ejemplo:

```json
{
   "id": "/subscriptions/********************/resourceGroups/Fabrikam-RG/providers/Microsoft.Logic/integrationServiceEnvironments/Fabrikam-ISE",
   "name": "Fabrikam-ISE",
   "type": "Microsoft.Logic/integrationServiceEnvironments",
   "location": "WestUS2",
   "identity": {
      "type": "UserAssigned",
      "userAssignedIdentities": {
         "/subscriptions/********************/resourceGroups/Fabrikam-RG/providers/Microsoft.ManagedIdentity/userAssignedIdentities/*********************************": {
            "principalId": "*********************************",
            "clientId": "*********************************"
         }
      }
   },
   "sku": {
      "name": "Premium",
      "capacity": 1
   },
   "properties": {
      "networkConfiguration": {
         "accessEndpoint": {
            // Your ISE can use the "External" or "Internal" endpoint. This example uses "External".
            "type": "External"
         },
         "subnets": [
            {
               "id": "/subscriptions/********************/resourceGroups/Fabrikam-RG/providers/Microsoft.Network/virtualNetworks/Fabrikam-VNET/subnets/subnet-1",
            },
            {
               "id": "/subscriptions/********************/resourceGroups/Fabrikam-RG/providers/Microsoft.Network/virtualNetworks/Fabrikam-VNET/subnets/subnet-2",
            },
            {
               "id": "/subscriptions/********************/resourceGroups/Fabrikam-RG/providers/Microsoft.Network/virtualNetworks/Fabrikam-VNET/subnets/subnet-3",
            },
            {
               "id": "/subscriptions/********************/resourceGroups/Fabrikam-RG/providers/Microsoft.Network/virtualNetworks/Fabrikam-VNET/subnets/subnet-4",
            }
         ]
      },
      "certificates": {
         "testCertificate": {
            "publicCertificate": "LS0tLS1CRUdJTiBDRV...",
            "kind": "TrustedRoot"
         }
      }
   }
}
```

## <a name="add-custom-root-certificates"></a>Adición de certificados raíz personalizados

A menudo se usa un ISE para conectarse a servicios personalizados en la red virtual o en el entorno local. Estos servicios personalizados suelen estar protegidos por un certificado emitido por una entidad de certificación raíz personalizada, como una entidad de certificación de empresa o un certificado autofirmado. Para más información sobre el uso de certificados autofirmados, consulte [Protección del acceso y los datos: acceso para las llamadas salientes a otros servicios y sistemas](../logic-apps/logic-apps-securing-a-logic-app.md#secure-outbound-requests). Para que el ISE se conecte correctamente a estos servicios a través de la seguridad de la capa de transporte (TLS), se necesita acceso a estos certificados raíz.

#### <a name="considerations-for-adding-custom-root-certificates"></a>Consideraciones para agregar certificados raíz personalizados

Antes de actualizar el ISE con un certificado raíz de confianza personalizado, vea estas consideraciones:

* Asegúrese de cargar el certificado raíz *y* todos los certificados intermedios. El número máximo de certificados es 20.

* El nombre del asunto del certificado debe coincidir con el nombre del host del punto de conexión de destino al que desea llamar desde Azure Logic Apps. 

* La carga de certificados raíz es una operación de reemplazo en la que la carga más reciente sobrescribe las cargas anteriores. Por ejemplo, si envía una solicitud que carga un certificado y, luego, envía otra solicitud para cargar otro certificado, el ISE usa solo el segundo certificado. Si necesita usar ambos certificados, agréguelos juntos en la misma solicitud.  

* La carga de certificados raíz es una operación asincrónica que puede tardar algún tiempo. Para comprobar el estado o el resultado, puede enviar una solicitud `GET` con el mismo URI. El mensaje de respuesta tiene un campo `provisioningState` que devuelve el valor `InProgress` cuando la operación de carga sigue funcionando. Cuando el valor `provisioningState` es `Succeeded`, la operación de carga está completa.

#### <a name="request-syntax"></a>Sintaxis de la solicitud

Para actualizar el ISE con un certificado raíz de confianza personalizado, envíe la siguiente solicitud HTTPS PATCH a la [dirección URL de Azure Resource Manager, que difiere en función del entorno de Azure](../azure-resource-manager/management/control-plane-and-data-plane.md#control-plane), por ejemplo:

| Entorno | Dirección URL de Azure Resource Manager |
|-------------|----------------------------|
| Azure global (multiinquilino) | `PATCH https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Logic/integrationServiceEnvironments/{integrationServiceEnvironmentName}?api-version=2019-05-01` |
| Azure Government | `PATCH https://management.usgovcloudapi.net/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Logic/integrationServiceEnvironments/{integrationServiceEnvironmentName}?api-version=2019-05-01` |
| Microsoft Azure China 21Vianet | `PATCH https://management.chinacloudapi.cn/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Logic/integrationServiceEnvironments/{integrationServiceEnvironmentName}?api-version=2019-05-01` |
|||

#### <a name="request-body-syntax-for-adding-custom-root-certificates"></a>Sintaxis del cuerpo de la solicitud para agregar certificados raíz personalizados

Esta es la sintaxis del cuerpo de la solicitud, que describe las propiedades que se deben usar al agregar certificados raíz:

```json
{
   "id": "/subscriptions/{Azure-subscription-ID}/resourceGroups/{Azure-resource-group}/providers/Microsoft.Logic/integrationServiceEnvironments/{ISE-name}",
   "name": "{ISE-name}",
   "type": "Microsoft.Logic/integrationServiceEnvironments",
   "location": "{Azure-region}",
   "properties": {
      "certificates": {
         "testCertificate1": {
            "publicCertificate": "{base64-encoded-certificate}",
            "kind": "TrustedRoot"
         },
         "testCertificate2": {
            "publicCertificate": "{base64-encoded-certificate}",
            "kind": "TrustedRoot"
         }
      }
   }
}
```

## <a name="next-steps"></a>Pasos siguientes

* [Incorporación de recursos a entornos del servicio de integración](../logic-apps/add-artifacts-integration-service-environment-ise.md)
* [Administración de entornos del servicio de integración](../logic-apps/ise-manage-integration-service-environment.md#check-network-health)
