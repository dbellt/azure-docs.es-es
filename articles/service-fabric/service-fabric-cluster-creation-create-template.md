---
title: Creación de una plantilla de clúster de Azure Service Fabric
description: Aprenda a crear una plantilla de Resource Manager para un clúster de Service Fabric. Configure la seguridad, Azure Key Vault y Azure Active Directory (Azure AD) para la autenticación de clientes.
ms.topic: conceptual
ms.date: 08/16/2018
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 5a216f0863569bba0c743b086e3747f8ae1902a8
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110671073"
---
# <a name="create-a-service-fabric-cluster-resource-manager-template"></a>Creación de una plantilla de Resource Manager para el clúster de Service Fabric

Un [clúster de Azure Service Fabric](service-fabric-deploy-anywhere.md) es un conjunto de máquinas virtuales conectadas a la red, en las que se implementan y administran los microservicios. Un clúster de Service Fabric que se ejecuta en Azure es un recurso de Azure y se implementa, administra y supervisa mediante Resource Manager.  En este artículo se describe cómo crear una plantilla de Resource Manager para un clúster de Service Fabric que se ejecuta en Azure.  Una vez completada la plantilla, puede [implementar el clúster en Azure](service-fabric-cluster-creation-via-arm.md).

La seguridad del clúster se configura cuando el clúster se configura por primera vez y no se puede cambiar más adelante. Antes de configurar un clúster, lea [Escenarios de seguridad de los clústeres de Service Fabric][service-fabric-cluster-security]. En Azure, Service Fabric usa un certificado x509 para proteger el clúster y sus puntos de conexión, autenticar a los clientes y cifrar los datos. También se recomienda Azure Active Directory para proteger el acceso a los puntos de conexión de administración. Los inquilinos y usuarios de Azure AD deben crearse antes de crear el clúster.  Para obtener más información, lea [Set up Azure AD to authenticate clients](service-fabric-cluster-creation-setup-aad.md) (Configuración de Azure AD para autenticar clientes).

Antes de implementar un clúster de producción para ejecutar cargas de trabajo de producción, asegúrese de leer la [Lista de comprobación sobre la preparación de producción](service-fabric-production-readiness-checklist.md).


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="create-the-resource-manager-template"></a>Creación de la plantilla de Resource Manager
Las plantillas de ejemplo de Resource Manager están disponibles en los [ejemplos de Azure de GitHub](https://github.com/Azure-Samples/service-fabric-cluster-templates). Estas plantillas se pueden usar como punto de partida para crear la plantilla de clúster.

En este artículo se usa la plantilla de ejemplo [five-node secure cluster][service-fabric-secure-cluster-5-node-1-nodetype] y sus parámetros de plantilla. Descargue *azuredeploy.json* y *azuredeploy.parameters.json* en el equipo y abra ambos archivos en un editor de texto.

> [!NOTE]
> Para las nubes nacionales (Azure Government, Azure China, Azure Alemania), también debe agregar el siguiente `fabricSettings` a la plantilla: `AADLoginEndpoint`, `AADTokenEndpointFormat` y `AADCertEndpointFormat`.

## <a name="add-certificates"></a>Adición de certificados
Para agregar certificados a una plantilla de Resource Manager para el clúster, haga referencia al almacén de claves que contiene las claves de certificado. Agregue esos parámetros y valores de almacén de claves en un archivo de parámetros de plantilla de Resource Manager (*azuredeploy.parameters.json*).

### <a name="add-all-certificates-to-the-virtual-machine-scale-set-osprofile"></a>Incorporación de todos los certificados a osProfile del conjunto de escalado de máquinas virtuales
Todos los certificados instalados en el clúster deben estar configurados en la sección **osProfile** del recurso del conjunto de escalado (Microsoft.Compute/virtualMachineScaleSets). Con ello se indica al proveedor de recursos que instale el certificado en las máquinas virtuales. Esta instalación incluye tanto el certificado del clúster como los certificados de seguridad de la aplicación que planee usar para las aplicaciones:

```json
{
  "apiVersion": "[variables('vmssApiVersion')]",
  "type": "Microsoft.Compute/virtualMachineScaleSets",
  ...
  "properties": {
    ...
    "osProfile": {
      ...
      "secrets": [
        {
          "sourceVault": {
            "id": "[parameters('sourceVaultValue')]"
          },
          "vaultCertificates": [
            {
              "certificateStore": "[parameters('clusterCertificateStorevalue')]",
              "certificateUrl": "[parameters('clusterCertificateUrlValue')]"
            },
            {
              "certificateStore": "[parameters('applicationCertificateStorevalue')",
              "certificateUrl": "[parameters('applicationCertificateUrlValue')]"
            },
            ...
          ]
        }
      ]
    }
  }
}
```

### <a name="configure-the-service-fabric-cluster-certificate"></a>Configuración del certificado de clúster de Service Fabric

El certificado de autenticación del clúster debe estar configurado tanto en el recurso del clúster de Service Fabric (Microsoft.ServiceFabric/clusters) como en la extensión de Service Fabric para conjuntos de escalado de máquinas virtuales en el recurso de conjuntos de escalado de máquinas virtuales. Esta disposición permite al proveedor de recursos de Service Fabric configurarlo para su uso en la autenticación del clúster y la autenticación del servidor en los puntos de conexión de administración.

#### <a name="add-the-certificate-information-the-virtual-machine-scale-set-resource"></a>Agregue la información de certificado al recurso del conjunto de escalado de máquinas virtuales

```json
{
  "apiVersion": "[variables('vmssApiVersion')]",
  "type": "Microsoft.Compute/virtualMachineScaleSets",
  ...
  "properties": {
    ...
    "virtualMachineProfile": {
      "extensionProfile": {
        "extensions": [
          {
            "name": "[concat('ServiceFabricNodeVmExt_',variables('vmNodeType0Name'))]",
            "properties": {
              ...
              "settings": {
                ...
                "certificate": {
                  "commonNames": ["[parameters('certificateCommonName')]"],
                  "x509StoreName": "[parameters('clusterCertificateStoreValue')]"
                },
                ...
              }
            }
          }
        ]
      }
    }
  }
}
```

#### <a name="add-the-certificate-information-to-the-service-fabric-cluster-resource"></a>Agregue la información de certificado al recurso de clúster de Service Fabric

```json
{
  "apiVersion": "2018-02-01",
  "type": "Microsoft.ServiceFabric/clusters",
  "name": "[parameters('clusterName')]",
  "location": "[parameters('clusterLocation')]",
  "dependsOn": [
    "[concat('Microsoft.Storage/storageAccounts/', variables('supportLogStorageAccountName'))]"
  ],
  "properties": {
    "certificateCommonNames": {
        "commonNames": [
        {
            "certificateCommonName": "[parameters('certificateCommonName')]",
            "certificateIssuerThumbprint": ""
        }
        ],
        "x509StoreName": "[parameters('certificateStoreValue')]"
    },
    ...
  }
}
```

## <a name="add-azure-ad-configuration-to-use-azure-ad-for-client-access"></a>Adición de la configuración de Azure AD para usar Azure AD para el acceso de cliente

Para agregar la configuración de Azure AD a una plantilla de Resource Manager del clúster, haga referencia al almacén de claves que contiene las claves de certificado. Agregue esos parámetros y valores de Azure AD en un archivo de parámetros de plantilla de Resource Manager (*azuredeploy.parameters.json*). 

> [!NOTE]
> En Linux, los inquilinos y usuarios de Azure AD deben crearse antes de crear el clúster.  Para obtener más información, lea [Set up Azure AD to authenticate clients](service-fabric-cluster-creation-setup-aad.md) (Configuración de Azure AD para autenticar clientes).

```json
{
  "apiVersion": "2018-02-01",
  "type": "Microsoft.ServiceFabric/clusters",
  "name": "[parameters('clusterName')]",
  ...
  "properties": {
    "certificateCommonNames": {
        "commonNames": [
        {
            "certificateCommonName": "[parameters('certificateCommonName')]",
            "certificateIssuerThumbprint": ""
        }
        ],
        "x509StoreName": "[parameters('certificateStoreValue')]"
    },
    ...
    "azureActiveDirectory": {
      "tenantId": "[parameters('aadTenantId')]",
      "clusterApplication": "[parameters('aadClusterApplicationId')]",
      "clientApplication": "[parameters('aadClientApplicationId')]"
    },
    ...
  }
}
```

## <a name="populate-the-parameter-file-with-the-values"></a>Relleno del archivo de parámetros con los valores

Por último, use los valores de salida del almacén de claves y los comandos de PowerShell de Azure AD para rellenar el archivo de parámetros.

Si tiene previsto usar los módulos de PowerShell de RM de Azure Service Fabric, no es necesario rellenar la información de certificado del clúster. Si desea que el sistema genere el certificado autofirmado para la seguridad del clúster, basta con mantenerlos como null. 

> [!NOTE]
> Para que los módulos de RM seleccionen y rellenen estos valores de parámetro vacíos, los nombres de parámetros deben coincidir en gran medida con los nombres siguientes.

```json
"clusterCertificateThumbprint": {
    "value": ""
},
"certificateCommonName": {
    "value": ""
},
"clusterCertificateUrlValue": {
    "value": ""
},
"sourceVaultvalue": {
    "value": ""
},
```

Si va a usar certificados de aplicación o usa un clúster existente que ha cargado en el almacén de claves, debe obtener esta información y rellenarla.

Los módulos de RM no tienen la posibilidad de generar la configuración de Azure AD automáticamente. Por ello, si tiene previsto usar Azure AD para el acceso de cliente, debe rellenarla.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        ...
        "clusterCertificateStoreValue": {
            "value": "My"
        },
        "clusterCertificateThumbprint": {
            "value": "<thumbprint>"
        },
        "clusterCertificateUrlValue": {
            "value": "https://myvault.vault.azure.net:443/secrets/myclustercert/4d087088df974e869f1c0978cb100e47"
        },
        "applicationCertificateStorevalue": {
            "value": "My"
        },
        "applicationCertificateUrlValue": {
            "value": "https://myvault.vault.azure.net:443/secrets/myapplicationcert/2e035058ae274f869c4d0348ca100f08"
        },
        "sourceVaultvalue": {
            "value": "/subscriptions/<guid>/resourceGroups/mycluster-keyvault/providers/Microsoft.KeyVault/vaults/myvault"
        },
        "aadTenantId": {
            "value": "<guid>"
        },
        "aadClusterApplicationId": {
            "value": "<guid>"
        },
        "aadClientApplicationId": {
            "value": "<guid>"
        },
        ...
    }
}
```

## <a name="test-your-template"></a>Prueba de la plantilla
Use el siguiente comando de PowerShell para probar la plantilla de Resource Manager con un archivo de parámetros:

```powershell
Test-AzResourceGroupDeployment -ResourceGroupName "myresourcegroup" -TemplateFile .\azuredeploy.json -TemplateParameterFile .\azuredeploy.parameters.json
```

Si se encuentra con problemas y recibe mensajes cifrados, use "-Depurar" como opción.

```powershell
Test-AzResourceGroupDeployment -ResourceGroupName "myresourcegroup" -TemplateFile .\azuredeploy.json -TemplateParameterFile .\azuredeploy.parameters.json -Debug
```

En el diagrama siguiente se ilustra el lugar que ocupa el almacén de claves y la configuración de Azure AD en la plantilla de Resource Manager.

![Asignación de dependencias de Resource Manager][cluster-security-arm-dependency-map]

## <a name="next-steps"></a>Pasos siguientes
Ahora que tiene una plantilla para el clúster, obtenga información sobre cómo [implementar el clúster en Azure](service-fabric-cluster-creation-via-arm.md).  Si no lo ha hecho ya, lea el [Lista de comprobación sobre la preparación de producción](service-fabric-production-readiness-checklist.md) antes de implementar un clúster de producción.

Para información sobre la sintaxis JSON y las propiedades de los recursos implementados en este artículo, consulte:

* [Microsoft.ServiceFabric/clusters](/azure/templates/microsoft.servicefabric/clusters)
* [Microsoft.Storage/storageAccounts](/azure/templates/microsoft.storage/storageaccounts)
* [Microsoft.Network/virtualNetworks](/azure/templates/microsoft.network/virtualnetworks)
* [Microsoft.Network/publicIPAddresses](/azure/templates/microsoft.network/publicipaddresses)
* [Microsoft.Network/loadBalancers](/azure/templates/microsoft.network/loadbalancers)
* [Microsoft.Compute/virtualMachineScaleSets](/azure/templates/microsoft.compute/virtualmachinescalesets)

<!-- Links -->
[service-fabric-cluster-security]: service-fabric-cluster-security.md
[service-fabric-secure-cluster-5-node-1-nodetype]: https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/5-VM-Windows-1-NodeTypes-Secure
[resource-group-template-deploy]: ../azure-resource-manager/templates/deploy-powershell.md

<!-- Images -->
[cluster-security-arm-dependency-map]: ./media/service-fabric-cluster-creation-create-template/cluster-security-arm-dependency-map.png
