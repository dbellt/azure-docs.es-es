---
title: 'Implementación de una instancia de Azure Cloud Services (soporte extendido): plantillas'
description: Implementación de una instancia de Azure Cloud Services (soporte extendido) mediante plantillas de ARM
ms.topic: tutorial
ms.service: cloud-services-extended-support
author: gachandw
ms.author: gachandw
ms.reviewer: mimckitt
ms.date: 10/13/2020
ms.custom: ''
ms.openlocfilehash: 7c013cbaf8a64d12b132e3dd4c599920bd3f6a2a
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108736117"
---
# <a name="deploy-a-cloud-service-extended-support-using-arm-templates"></a>Implementación de una instancia de Cloud Service (soporte extendido) mediante plantillas de ARM

En este tutorial se explica cómo crear la implementación de una instancia de Cloud Service (soporte extendido) mediante [plantillas de ARM](../azure-resource-manager/templates/overview.md). 

## <a name="before-you-begin"></a>Antes de comenzar

1. Consulte los [requisitos previos de implementación](deploy-prerequisite.md) de Cloud Services (soporte extendido) y cree los recursos asociados.

2. Cree un nuevo grupo de recursos con [Azure Portal](../azure-resource-manager/management/manage-resource-groups-portal.md) o [PowerShell](../azure-resource-manager/management/manage-resource-groups-powershell.md). Este paso es opcional si se usa un grupo de recursos existente.
 
3. Cree una cuenta de almacenamiento mediante [Azure Portal](../storage/common/storage-account-create.md?tabs=azure-portal) o [PowerShell](../storage/common/storage-account-create.md?tabs=azure-powershell). Este paso es opcional si se usa una cuenta de almacenamiento existente.

4. Cargue los archivos de configuración de paquete (.csfg) y servicio (.cscfg) en la cuenta de almacenamiento mediante [Azure Portal](../storage/blobs/storage-quickstart-blobs-portal.md#upload-a-block-blob) o [PowerShell](../storage/blobs/storage-quickstart-blobs-powershell.md#upload-blobs-to-the-container). Obtenga los URI de SAS de ambos archivos que se van a agregar a la plantilla de ARM más adelante en este tutorial.

5. (Opcional) Cree un almacén de claves y cargue los certificados.

    -  Los certificados pueden adjuntarse a los servicios en la nube para permitir la comunicación segura hacia el servicio, y desde él. Para poder usar certificados, se deben especificar sus huellas digitales en el archivo de configuración del servicio (.cscfg) y cargarse en un almacén de claves. Un almacén de claves se puede crear mediante [Azure Portal](../key-vault/general/quick-create-portal.md) o [PowerShell](../key-vault/general/quick-create-powershell.md).
    - El almacén de claves asociado se debe crear en la misma región y suscripción que el servicio en la nube.
    - El almacén de claves asociado debe tener los permisos adecuados para que el recurso de Cloud Services (soporte extendido) pueda recuperar certificados de la instancia de Key Vault. Para más información, consulte [Certificados y Key Vault](certificates-and-key-vault.md).
    - Se debe hacer referencia al almacén de claves en la sección OsProfile de la plantilla de ARM que se muestra en los pasos siguientes.

## <a name="deploy-a-cloud-service-extended-support"></a>Implementación de una instancia de Cloud Services (soporte extendido)

> [!NOTE]
> Una manera más fácil y rápida de generar la plantilla de ARM y el archivo de parámetros es a través de [Azure Portal](https://portal.azure.com). Puede [descargar la plantilla de ARM generada](generate-template-portal.md) del portal para crear su servicio en la nube mediante PowerShell.
 
1. Cree una red virtual. El nombre de la red virtual debe coincidir con las referencias del archivo de configuración del servicio (.cscfg). Si usa una red virtual existente, omita esta sección de la plantilla de ARM.

    ```json
    "resources": [ 
        { 
          "apiVersion": "2019-08-01", 
          "type": "Microsoft.Network/virtualNetworks", 
          "name": "[parameters('vnetName')]", 
          "location": "[parameters('location')]", 
          "properties": { 
            "addressSpace": { 
              "addressPrefixes": [ 
                "10.0.0.0/16" 
              ] 
            }, 
            "subnets": [ 
              { 
                "name": "WebTier", 
                "properties": { 
                  "addressPrefix": "10.0.0.0/24" 
                } 
              } 
            ] 
          } 
        } 
    ] 
    ```
    
     Si crea una nueva red virtual, agregue lo siguiente a la sección `dependsOn` para asegurarse de que la plataforma crea la red virtual antes de crear el servicio en la nube.

    ```json
    "dependsOn": [ 
            "[concat('Microsoft.Network/virtualNetworks/', parameters('vnetName'))]" 
     ] 
    ```
 
2. Cree una dirección IP pública y, si lo desea, establezca la propiedad de la etiqueta DNS de la dirección IP pública. Si usa una dirección IP estática, debe hacer referencia a ella como IP reservada del archivo de configuración de servicio (.cscfg). Si usa una dirección IP existente, omita este paso y agregue la información de la dirección IP directamente en la configuración del equilibrador de carga de la plantilla de ARM.
 
    ```json
    "resources": [ 
        { 
          "apiVersion": "2019-08-01", 
          "type": "Microsoft.Network/publicIPAddresses", 
          "name": "[parameters('publicIPName')]", 
          "location": "[parameters('location')]", 
          "properties": { 
            "publicIPAllocationMethod": "Dynamic", 
            "idleTimeoutInMinutes": 10, 
            "publicIPAddressVersion": "IPv4", 
            "dnsSettings": { 
              "domainNameLabel": "[variables('dnsName')]" 
            } 
          }, 
          "sku": { 
            "name": "Basic" 
          } 
        } 
    ] 
    ```
     
     Si crea una nueva dirección IP, agregue lo siguiente a la sección `dependsOn` para asegurarse de que la plataforma crea la dirección IP antes de crear el servicio en la nube.
    
    ```json
    "dependsOn": [ 
            "[concat('Microsoft.Network/publicIPAddresses/', parameters('publicIPName'))]" 
          ] 
    ```
 
3. Cree un objeto de perfil de red y asocie la dirección IP pública al front-end del equilibrador de carga. La plataforma crea automáticamente un equilibrador de carga.

    ```json
    "networkProfile": { 
        "loadBalancerConfigurations": [ 
          { 
            "id": "[concat(variables('resourcePrefix'), 'Microsoft.Network/loadBalancers/', variables('lbName'))]", 
            "name": "[variables('lbName')]", 
            "properties": { 
              "frontendIPConfigurations": [ 
                { 
                  "name": "[variables('lbFEName')]", 
                  "properties": { 
                    "publicIPAddress": { 
                      "id": "[concat(variables('resourcePrefix'), 'Microsoft.Network/publicIPAddresses/', parameters('publicIPName'))]" 
                    } 
                  } 
                } 
              ] 
            } 
          } 
        ] 
      } 
    ```
 

4. Agregue la referencia al almacén de claves en la sección `OsProfile` de la plantilla de ARM. Key Vault se usa para almacenar certificados asociados a Cloud Services (soporte extendido). Agregue los certificados a la instancia de Key Vault y haga referencia a las huellas digitales del certificado en el archivo de configuración de servicio (.cscfg). También debe habilitar "Directivas de acceso" en "Azure Virtual Machines para la implementación" en Key Vault, de modo que el recurso de Cloud Services (soporte extendido) pueda recuperar el certificado almacenado como secretos de Key Vault. El almacén de claves debe estar ubicado en la misma región y suscripción que el servicio en la nube, y debe tener un nombre único. Para más información, consulte [Uso de certificados con Cloud Services (soporte extendido)](certificates-and-key-vault.md).
     
    ```json
    "osProfile": { 
          "secrets": [ 
            { 
              "sourceVault": { 
                "id": "/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/Microsoft.KeyVault/vaults/{keyvault-name}" 
              }, 
              "vaultCertificates": [ 
                { 
                  "certificateUrl": "https://{keyvault-name}.vault.azure.net:443/secrets/ContosoCertificate/{secret-id}" 
                } 
              ] 
            } 
          ] 
        } 
    ```
  
    > [!NOTE]
    > SourceVault es el identificador de recurso de ARM para el almacén de claves. Para encontrar esta información, busque el identificador de recurso en la sección de propiedades del almacén de claves.
    > - Para encontrar el objeto certificateUrl, vaya al certificado en el almacén de claves etiquetado como **Identificador secreto**.  
   >  - El objeto certificateUrl debe tener el formato https://{keyvault-endpoin}/secrets/{secretname}/{secret-id}.

5. Cree un perfil de rol. Asegúrese de que el número de roles, nombres de roles, número de instancias de cada rol y tamaños sea el mismo en la sección de configuración del servicio (.cscfg), definición de servicio (.csdef) y de perfil de rol de la plantilla de ARM.
    
    ```json
    "roleProfile": {
      "roles": {
        "value": [
          {
            "name": "WebRole1",
            "sku": {
              "name": "Standard_D1_v2",
              "capacity": "1"
            }
          },
          {
            "name": "WorkerRole1",
            "sku": {
              "name": "Standard_D1_v2",
              "capacity": "1"
            } 
          } 
        ]
      }
    }   
    ```

6. Si lo desea, cree un objeto de perfil de extensión que desee agregar al servicio en la nube. En este ejemplo, vamos a agregar el escritorio remoto y la extensión de diagnósticos de Windows Azure.
   > [!Note] 
   > La contraseña del escritorio remoto debe tener una longitud de entre 8 y 123 caracteres, y debe cumplir al menos 3 de los siguientes requisitos de complejidad de contraseña: 1) Contener un carácter en mayúsculas. 2) Contener un carácter en minúsculas. 3) Contener un dígito numérico. 4) Contener un carácter especial. 5) No se permiten los caracteres de control.

    ```json
        "extensionProfile": {
          "extensions": [
            {
              "name": "RDPExtension",
              "properties": {
                "autoUpgradeMinorVersion": true,
                "publisher": "Microsoft.Windows.Azure.Extensions",
                "type": "RDP",
                "typeHandlerVersion": "1.2.1",
                "settings": "<PublicConfig>\r\n <UserName>[Insert Username]</UserName>\r\n <Expiration>1/21/2022 12:00:00 AM</Expiration>\r\n</PublicConfig>",
                "protectedSettings": "<PrivateConfig>\r\n <Password>[Insert Password]</Password>\r\n</PrivateConfig>"
              }
            },
            {
              "name": "Microsoft.Insights.VMDiagnosticsSettings_WebRole1",
              "properties": {
                "autoUpgradeMinorVersion": true,
                "publisher": "Microsoft.Azure.Diagnostics",
                "type": "PaaSDiagnostics",
                "typeHandlerVersion": "1.5",
                "settings": "[parameters('wadPublicConfig_WebRole1')]",
                "protectedSettings": "[parameters('wadPrivateConfig_WebRole1')]",
                "rolesAppliedTo": [
                  "WebRole1"
                ]
              }
            }
          ]
        }
    ```

7. Revise la plantilla completa.

    ```json
    {
      "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {
        "cloudServiceName": {
          "type": "string",
          "metadata": {
            "description": "Name of the cloud service"
          }
        },
        "location": {
          "type": "string",
          "metadata": {
            "description": "Location of the cloud service"
          }
        },
        "deploymentLabel": {
          "type": "string",
          "metadata": {
            "description": "Label of the deployment"
          }
        },
        "packageSasUri": {
          "type": "securestring",
          "metadata": {
            "description": "SAS Uri of the CSPKG file to deploy"
          }
        },
        "configurationSasUri": {
          "type": "securestring",
          "metadata": {
            "description": "SAS Uri of the service configuration (.cscfg)"
          }
        },
        "roles": {
          "type": "array",
          "metadata": {
            "description": "Roles created in the cloud service application"
          }
        },
        "wadPublicConfig_WebRole1": {
          "type": "string",
          "metadata": {
             "description": "Public configuration of Windows Azure Diagnostics extension"
          }
        },
        "wadPrivateConfig_WebRole1": {
          "type": "securestring",
          "metadata": {
            "description": "Private configuration of Windows Azure Diagnostics extension"
          }
        },
        "vnetName": {
          "type": "string",
          "defaultValue": "[concat(parameters('cloudServiceName'), 'VNet')]",
          "metadata": {
            "description": "Name of vitual network"
          }
        },
        "publicIPName": {
          "type": "string",
          "defaultValue": "contosocsIP",
          "metadata": {
            "description": "Name of public IP address"
          }
        },
        "upgradeMode": {
          "type": "string",
          "defaultValue": "Auto",
          "metadata": {
            "UpgradeMode": "UpgradeMode of the CloudService"
          }
        }
      },
      "variables": {
        "cloudServiceName": "[parameters('cloudServiceName')]",
        "subscriptionID": "[subscription().subscriptionId]",
        "dnsName": "[variables('cloudServiceName')]",
        "lbName": "[concat(variables('cloudServiceName'), 'LB')]",
        "lbFEName": "[concat(variables('cloudServiceName'), 'LBFE')]",
        "resourcePrefix": "[concat('/subscriptions/', variables('subscriptionID'), '/resourceGroups/', resourceGroup().name, '/providers/')]"
      },
      "resources": [
        {
          "apiVersion": "2019-08-01",
          "type": "Microsoft.Network/virtualNetworks",
          "name": "[parameters('vnetName')]",
          "location": "[parameters('location')]",
          "properties": {
            "addressSpace": {
              "addressPrefixes": [
                "10.0.0.0/16"
              ]
            },
            "subnets": [
              {
                "name": "WebTier",
                "properties": {
                  "addressPrefix": "10.0.0.0/24"
                }
              }
            ]
          }
        },
        {
          "apiVersion": "2019-08-01",
          "type": "Microsoft.Network/publicIPAddresses",
          "name": "[parameters('publicIPName')]",
          "location": "[parameters('location')]",
          "properties": {
            "publicIPAllocationMethod": "Dynamic",
            "idleTimeoutInMinutes": 10,
            "publicIPAddressVersion": "IPv4",
            "dnsSettings": {
              "domainNameLabel": "[variables('dnsName')]"
            }
          },
          "sku": {
            "name": "Basic"
          }
        },
        {
          "apiVersion": "2021-03-01",
          "type": "Microsoft.Compute/cloudServices",
          "name": "[variables('cloudServiceName')]",
          "location": "[parameters('location')]",
          "tags": {
            "DeploymentLabel": "[parameters('deploymentLabel')]",
            "DeployFromVisualStudio": "true"
          },
          "dependsOn": [
            "[concat('Microsoft.Network/virtualNetworks/', parameters('vnetName'))]",
            "[concat('Microsoft.Network/publicIPAddresses/', parameters('publicIPName'))]"
          ],
          "properties": {
            "packageUrl": "[parameters('packageSasUri')]",
            "configurationUrl": "[parameters('configurationSasUri')]",
            "upgradeMode": "[parameters('upgradeMode')]",
            "roleProfile": {
              "roles": [
                {
                  "name": "WebRole1",
                  "sku": {
                    "name": "Standard_D1_v2",
                    "capacity": "1"
                  }
                },
                {
                  "name": "WorkerRole1",
                  "sku": {
                    "name": "Standard_D1_v2",
                    "capacity": "1"
                  }
                }
              ]
            },
            "networkProfile": {
              "loadBalancerConfigurations": [
                {
                  "id": "[concat(variables('resourcePrefix'), 'Microsoft.Network/loadBalancers/', variables('lbName'))]",
                  "name": "[variables('lbName')]",
                  "properties": {
                    "frontendIPConfigurations": [
                      {
                        "name": "[variables('lbFEName')]",
                        "properties": {
                          "publicIPAddress": {
                            "id": "[concat(variables('resourcePrefix'), 'Microsoft.Network/publicIPAddresses/', parameters('publicIPName'))]"
                          }
                        }
                      }
                    ]
                  }
                }
              ]
            },
            "osProfile": {
              "secrets": [
                {
                  "sourceVault": {
                    "id": "/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/Microsoft.KeyVault/vaults/{keyvault-name}"
                  },
                  "vaultCertificates": [
                    {
                      "certificateUrl": "https://{keyvault-name}.vault.azure.net:443/secrets/ContosoCertificate/{secret-id}"
                    }
                  ]
                }
              ]
            },
            "extensionProfile": {
              "extensions": [
                {
                  "name": "RDPExtension",
                  "properties": {
                    "autoUpgradeMinorVersion": true,
                    "publisher": "Microsoft.Windows.Azure.Extensions",
                    "type": "RDP",
                    "typeHandlerVersion": "1.2.1",
                    "settings": "<PublicConfig>\r\n <UserName>[Insert Username]</UserName>\r\n <Expiration>1/21/2022 12:00:00 AM</Expiration>\r\n</PublicConfig>",
                    "protectedSettings": "<PrivateConfig>\r\n <Password>[Insert Password]</Password>\r\n</PrivateConfig>"
                  }
                },
                {
                  "name": "Microsoft.Insights.VMDiagnosticsSettings_WebRole1",
                  "properties": {
                    "autoUpgradeMinorVersion": true,
                    "publisher": "Microsoft.Azure.Diagnostics",
                    "type": "PaaSDiagnostics",
                    "typeHandlerVersion": "1.5",
                    "settings": "[parameters('wadPublicConfig_WebRole1')]",
                    "protectedSettings": "[parameters('wadPrivateConfig_WebRole1')]",
                    "rolesAppliedTo": [
                      "WebRole1"
                  ]
                }
              }
            ]
          }
        }
       }
      ]
    }
    ```

8. Implemente la plantilla y el archivo de parámetros (definiendo parámetros en el archivo de plantilla) para crear la implementación de Cloud Services (soporte extendido). Consulte estas [plantillas de ejemplo](https://github.com/Azure-Samples/cloud-services-extended-support) según sea necesario.

    ```powershell
    New-AzResourceGroupDeployment -ResourceGroupName "ContosOrg" -TemplateFile "file path to your template file" -TemplateParameterFile "file path to your parameter file"
    ```

## <a name="next-steps"></a>Pasos siguientes 

- Vea las [preguntas más frecuentes](faq.md) sobre Cloud Services (soporte extendido).
- Implemente una instancia de Cloud Services (soporte extendido) mediante [Azure Portal](deploy-portal.md), [PowerShell](deploy-powershell.md), una [plantilla](deploy-template.md) o [Visual Studio](deploy-visual-studio.md).
- Visite el [repositorio de ejemplos de Cloud Services (soporte extendido)](https://github.com/Azure-Samples/cloud-services-extended-support)
