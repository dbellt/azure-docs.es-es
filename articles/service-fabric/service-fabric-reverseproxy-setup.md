---
title: Configuración del proxy inverso de Azure Service Fabric
description: Aprenda a iniciar y configurar el servicio de proxy inverso para una aplicación de Azure Service Fabric.
ms.topic: conceptual
ms.date: 11/13/2018
ms.author: pepogors
ms.openlocfilehash: 1d8f290bf1cad13597376d1d7c27cfaa695f9511
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111956639"
---
# <a name="set-up-and-configure-reverse-proxy-in-azure-service-fabric"></a>Instalación y configuración del proxy inverso en Azure Service Fabric
El proxy inverso es un servicio opcional de Azure Service Fabric que ayuda a los microservicios que se ejecutan en un clúster de Service Fabric a detectar otros servicios que tienen puntos de conexión HTTP y a comunicarse con ellos. Para más información, consulte [Proxy inverso en Azure Service Fabric](service-fabric-reverseproxy.md). Este artículo muestra cómo instalar y configurar el proxy inverso en el clúster. 

## <a name="enable-reverse-proxy-using-azure-portal"></a>Habilitación del proxy inverso mediante Azure Portal

Azure Portal ofrece una opción para habilitar un proxy inverso al crear un clúster de Service Fabric. No puede actualizar un clúster existente para usar el proxy inverso mediante el portal. 

Para configurar el proxy inverso al [crear un clúster mediante Azure Portal](./service-fabric-cluster-creation-via-portal.md), asegúrese de hacer lo siguiente:

1. En el **Paso 2: En Configuración del clúster**, en **Configuración del tipo de nodo**, seleccione **Habilitar proxy inverso**.

   ![Habilitación del proxy inverso en el portal](./media/service-fabric-reverseproxy-setup/enable-rp-portal.png)
2. (Opcional) Para configurar un proxy inverso seguro, tiene que configurar un certificado TLS/SSL. En **Paso 3: Seguridad**, en **Configurar opciones de seguridad del clúster**, en **Tipo de configuración**, seleccione **Personalizado**. A continuación, en **Certificado SSL de proxy inverso**, seleccione **Include a SSL certificate for reverse proxy** (Incluir un certificado SSL para proxy inverso) y escriba los detalles del certificado.

   ![Configuración del proxy inverso seguro en el portal](./media/service-fabric-reverseproxy-setup/configure-rp-certificate-portal.png)

   Si decide no configurar el proxy inverso con un certificado al crear el clúster, puede hacerlo más adelante mediante la plantilla de Resource Manager para el grupo de recursos del clúster. Para más información, consulte [Habilitar proxy inverso mediante plantillas de Azure Resource Manager](#enable-reverse-proxy-via-azure-resource-manager-templates).

## <a name="enable-reverse-proxy-via-azure-resource-manager-templates"></a>Habilitar proxy inverso mediante plantillas de Azure Resource Manager

En los clústeres de Azure, puede usar la plantilla de Azure Resource Manager para habilitar el proxy inverso en Service Fabric. Puede habilitar el proxy inverso al crear el clúster o bien al actualizar el clúster en un momento posterior. 

Si desea un clúster nuevo, puede [crear una plantilla de Resource Manager personalizada](service-fabric-cluster-creation-via-arm.md) o puede usar una plantilla de ejemplo. 

Puede encontrar plantillas de Resource Manager de ejemplo que pueden servirle de ayuda para configurar el proxy inverso seguro para un clúster de Azure en [Secure Reverse Proxy Sample Templates](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/Reverse-Proxy-Sample) (Plantillas de ejemplo de proxy inverso seguro) en GitHub. Consulte [Configure HTTPS Reverse Proxy in a secure cluster](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/Reverse-Proxy-Sample/README.md#configure-https-reverse-proxy-in-a-secure-cluster) (Configuración del proxy inverso HTTPS en un clúster seguro) en el archivo LÉAME para obtener instrucciones y las plantillas para configurar un proxy inverso seguro con un certificado y administrar la sustitución de certificados.

Para un clúster existente, puede exportar la plantilla de Resource Manager para el grupo de recursos del clúster con los [Azure Portal](../azure-resource-manager/templates/export-template-portal.md), [PowerShell](../azure-resource-manager/management/manage-resources-powershell.md) o la [CLI de Azure](../azure-resource-manager/management/manage-resources-cli.md).

Una vez que tenga una plantilla de Resource Manager, puede seguir estos pasos para habilitar el proxy inverso:

1. Defina un puerto para el proxy inverso en la [sección de parámetros](../azure-resource-manager/templates/syntax.md) de la plantilla.

    ```json
    "SFReverseProxyPort": {
        "type": "int",
        "defaultValue": 19081,
        "metadata": {
            "description": "Endpoint for Service Fabric Reverse proxy"
        }
    },
    ```
2. Especifique el puerto para cada uno de los objetos nodetype de la sección [**Microsoft. ServiceFabric/clusters**](/azure/templates/microsoft.servicefabric/clusters) [tipo de recurso](../azure-resource-manager/templates/syntax.md).

    El puerto se identifica con el nombre de parámetro reverseProxyEndpointPort.

    ```json
    {
        "apiVersion": "2016-09-01",
        "type": "Microsoft.ServiceFabric/clusters",
        "name": "[parameters('clusterName')]",
        "location": "[parameters('clusterLocation')]",
        ...
       "nodeTypes": [
          {
           ...
           "reverseProxyEndpointPort": "[parameters('SFReverseProxyPort')]",
           ...
          },
        ...
        ],
        ...
    }
    ```
3. Para configurar certificados TLS o SSL en el puerto del proxy inverso, agregue el certificado a la propiedad **reverseProxyCertificate** de la sección [Tipo de recurso](../azure-resource-manager/templates/syntax.md) de *Microsoft.ServiceFabric/clusters** .

    ```json
    {
        "apiVersion": "2016-09-01",
        "type": "Microsoft.ServiceFabric/clusters",
        "name": "[parameters('clusterName')]",
        "location": "[parameters('clusterLocation')]",
        "dependsOn": [
            "[concat('Microsoft.Storage/storageAccounts/', parameters('supportLogStorageAccountName'))]"
        ],
        "properties": {
            ...
            "reverseProxyCertificate": {
                "thumbprint": "[parameters('sfReverseProxyCertificateThumbprint')]",
                "x509StoreName": "[parameters('sfReverseProxyCertificateStoreName')]"
            },
            ...
            "clusterState": "Default",
        }
    }
    ```

### <a name="supporting-a-reverse-proxy-certificate-thats-different-from-the-cluster-certificate"></a>Compatibilidad de un certificado de proxy inverso distinto del certificado de clúster
 Si el certificado de proxy inverso es distinto del certificado que protege el clúster, el certificado especificado anteriormente debe instalarse en la máquina virtual y agregarse a la lista de control de acceso (ACL) para que Service Fabric pueda acceder a él. Esto puede hacerse en la [**sección del** tipo de recurso](/azure/templates/microsoft.compute/virtualmachinescalesets) [Microsoft. Compute/virtualMachineScaleSets](../azure-resource-manager/templates/syntax.md). Para la instalación, agregue el certificado a osProfile. La sección de extensión de la plantilla puede actualizar el certificado en la ACL.

  ```json
  {
    "apiVersion": "[variables('vmssApiVersion')]",
    "type": "Microsoft.Compute/virtualMachineScaleSets",
    ....
      "osProfile": {
          "adminPassword": "[parameters('adminPassword')]",
          "adminUsername": "[parameters('adminUsername')]",
          "computernamePrefix": "[parameters('vmNodeType0Name')]",
          "secrets": [
            {
              "sourceVault": {
                "id": "[parameters('sfReverseProxySourceVaultValue')]"
              },
              "vaultCertificates": [
                {
                  "certificateStore": "[parameters('sfReverseProxyCertificateStoreValue')]",
                  "certificateUrl": "[parameters('sfReverseProxyCertificateUrlValue')]"
                }
              ]
            }
          ]
        }
   ....
   "extensions": [
          {
              "name": "[concat(parameters('vmNodeType0Name'),'_ServiceFabricNode')]",
              "properties": {
                      "type": "ServiceFabricNode",
                      "autoUpgradeMinorVersion": false,
                      ...
                      "publisher": "Microsoft.Azure.ServiceFabric",
                      "settings": {
                        "clusterEndpoint": "[reference(parameters('clusterName')).clusterEndpoint]",
                        "nodeTypeRef": "[parameters('vmNodeType0Name')]",
                        "dataPath": "D:\\\\SvcFab",
                        "durabilityLevel": "Bronze",
                        "testExtension": true,
                        "reverseProxyCertificate": {
                          "thumbprint": "[parameters('sfReverseProxyCertificateThumbprint')]",
                          "x509StoreName": "[parameters('sfReverseProxyCertificateStoreValue')]"
                        },
                  },
                  "typeHandlerVersion": "1.0"
              }
          },
      ]
    }
  ```
> [!NOTE]
> Cuando use certificados que sean distintos al certificado del clúster para habilitar el proxy inverso en un clúster existente, instale el certificado del proxy inverso y actualice la ACL en el clúster antes de habilitar el proxy inverso. Complete la implementación de la [plantilla de Azure Resource Manager](service-fabric-cluster-creation-via-arm.md) con la configuración mencionada anteriormente antes de iniciar una implementación para habilitar el proxy inverso con los pasos 1 a 3.

## <a name="enable-reverse-proxy-on-standalone-clusters"></a>Habilitación del proxy inverso en clústeres independientes

Para los clústeres independientes, habilite el proxy inverso en el archivo ClusterConfig.json. Puede habilitar el proxy inverso al crear el clúster o al actualizar la configuración para un clúster existente. Para más información acerca de las opciones de configuración disponibles en los archivos ClusterConfig.json, consulte [Configuración de un clúster independiente](./service-fabric-cluster-manifest.md).

Los pasos siguientes muestran la configuración que se usará para habilitar el proxy inverso y, opcionalmente, para proteger el proxy inverso con un certificado X.509. 

1. Para habilitar el proxy inverso, establezca el valor de **reverseProxyEndpointPort** para el tipo de nodo en **Propiedades** en la configuración del clúster. El siguiente archivo JSON muestra cómo establecer el puerto del extremo del proxy inverso en 19081 para los nodos con el tipo "NodeType0":

   ```json
       "properties": {
          ... 
           "nodeTypes": [
               {
                   "name": "NodeType0",
                   ...
                   "reverseProxyEndpointPort": "19081",
                   ...
               }
           ],
          ...
       }
   ```
2. (Opcional) Si desea un proxy inverso seguro, configure un certificado en la sección **Seguridad**, en **Propiedades**. 
   - En un entorno de desarrollo o de prueba, puede usar la opción **ReverseProxyCertificate**:

      ```json
          "properties": {
              ...
              "security": {
                  ...
                  "CertificateInformation": {
                      ...
                      "ReverseProxyCertificate": {
                          "Thumbprint": "[Thumbprint]",
                          "ThumbprintSecondary": "[Thumbprint]",
                          "X509StoreName": "My"
                      },
                      ...
                  }
              },
              ...
          }
      ```
   - En un entorno de producción, se recomienda la opción **ReverseProxyCertificateCommonNames**:

      ```json
          "properties": {
              ...
              "security": {
                  ...
                  "CertificateInformation": {
                      ...
                      "ReverseProxyCertificateCommonNames": {
                        "CommonNames": [
                            {
                              "CertificateCommonName": "[CertificateCommonName]"
                            }
                          ],
                          "X509StoreName": "My"
                      },
                      ...
                  }
              },
              ...
          }
      ```

   Para más información sobre la configuración y administración de certificados para un clúster independiente, así como más detalles acerca de cómo configurar los certificados usados para el proxy inverso seguro, consulte [Seguridad basada en certificados X509](./service-fabric-windows-cluster-x509-security.md).

Cuando haya modificado el archivo ClusterConfig.json para habilitar el proxy inverso, siga las instrucciones de [Actualización de la configuración del clúster](service-fabric-cluster-config-upgrade-windows-server.md) para insertar los cambios en el clúster.


## <a name="expose-reverse-proxy-on-a-public-port-through-azure-load-balancer"></a>Exposición de un proxy inverso en un puerto público mediante Azure Load Balancer

Para comunicarse con el proxy inverso desde fuera de un clúster de Azure, configure las reglas de Azure Load Balancer y sondeos de estado de Azure para el puerto del proxy inverso. Estos pasos pueden realizarse mediante Azure Portal o con la plantilla de Resource Manager en cualquier momento después de haber creado el clúster. 

> [!WARNING]
> Al configurar el puerto del proxy inverso de Load Balancer, se podrá acceder de manera externa a todos los microservicios del clúster que exponen un punto de conexión HTTP. Esto significa que un usuario malintencionado determinado podría detectar microservicios destinados a ser internos. Potencialmente, implica vulnerabilidades graves que podrían aprovecharse, como, por ejemplo:
>
> * Un usuario malintencionado puede iniciar un ataque de denegación de servicio mediante llamadas reiteradas a un servicio interno que no tenga una superficie expuesta a ataques suficientemente protegida.
> * Un usuario malintencionado puede entregar paquetes con formato incorrecto a un servicio interno y provocar un comportamiento no deseado.
> * Un servicio destinado a ser interno podría devolver información privada o confidencial que no se pretende exponer a servicios fuera del clúster, y ponerla así a disposición de un usuario malintencionado. 
>
> Asegúrese de entender completamente y de mitigar las posibles repercusiones para la seguridad del clúster y las aplicaciones que se ejecutan en este, antes de hacer público el puerto de proxy inverso. 
>

Si desea exponer un proxy inverso públicamente para un clúster independiente, la manera en que lo haga dependerá del sistema que hospede el clúster y esto está fuera del ámbito de este artículo. Sin embargo, la advertencia anterior sobre la exposición pública de un proxy inverso se sigue aplicando.

### <a name="expose-the-reverse-proxy-using-azure-portal"></a>Exposición del proxy inverso mediante Azure Portal 

1. En Azure Portal, haga clic en el grupo de recursos para el clúster y, luego, haga clic en el equilibrador de carga para el clúster.
2. Para agregar un sondeo de estado para el puerto de proxy inverso, en el panel izquierdo de la ventana del equilibrador de carga, en **CONFIGURACIÓN**, haga clic en **Sondeos de estado**. A continuación, haga clic en **Agregar** en la parte superior de la ventana Sondeos de estado y especifique los detalles para el puerto de proxy inverso; a continuación, haga clic en **Aceptar**. De forma predeterminada, el puerto de proxy inverso es 19081, a menos que lo cambiara al crear el clúster.

   ![Configuración de sondeos de estado del proxy inverso](./media/service-fabric-reverseproxy-setup/lb-rp-probe.png)
3. Para agregar una regla de Azure Load Balancer para exponer el puerto de proxy inverso, en el panel izquierdo de la ventana del equilibrador de carga, en **CONFIGURACIÓN**, haga clic en **Reglas de equilibrio de carga**. A continuación, haga clic en **Agregar** en la parte superior de la ventana Reglas de equilibrio de carga y especifique los detalles para el puerto de proxy inverso. Asegúrese de establecer el valor de **Puerto** en el puerto en el que desee que el proxy inverso se exponga, el valor de **Puerto back-end** en el puerto que estableció al habilitar el proxy inverso y el valor de **Sondeo de estado** en el sondeo de estado que configuró en el paso anterior. Establezca los demás campos según corresponda y haga clic en **Aceptar**.

   ![Configuración de la regla del equilibrador de carga para el proxy inverso](./media/service-fabric-reverseproxy-setup/lb-rp-rule.png)

### <a name="expose-the-reverse-proxy-via-resource-manager-templates"></a>Exposición del proxy inverso mediante plantillas de Resource Manager

El siguiente archivo JSON hace referencia a la misma plantilla que se usa en [Habilitación del proxy inverso mediante plantillas de Azure Resource Manager](#enable-reverse-proxy-via-azure-resource-manager-templates). Haga referencia a esa sección del documento para obtener información acerca de cómo crear una plantilla de Resource Manager o exportar una plantilla para un clúster existente.  Los cambios se realizan en la [**sección de cambios de recurso**](/azure/templates/microsoft.network/loadbalancers) [Microsoft. Network/loadBalancers](../azure-resource-manager/templates/syntax.md).

```json
{
    "apiVersion": "[variables('lbApiVersion')]",
    "type": "Microsoft.Network/loadBalancers",
    ...
    ...
    "loadBalancingRules": [
        ...
        {
            "name": "LBSFReverseProxyRule",
            "properties": {
                "backendAddressPool": {
                    "id": "[variables('lbPoolID0')]"
                },
                "backendPort": "[parameters('SFReverseProxyPort')]",
                "enableFloatingIP": "false",
                "frontendIPConfiguration": {
                    "id": "[variables('lbIPConfig0')]"
                },
                "frontendPort": "[parameters('SFReverseProxyPort')]",
                "idleTimeoutInMinutes": "5",
                "probe": {
                    "id": "[concat(variables('lbID0'),'/probes/SFReverseProxyProbe')]"
                },
                "protocol": "tcp"
            }
        }
    ],
    "probes": [
        ...
        {
            "name": "SFReverseProxyProbe",
            "properties": {
                "intervalInSeconds": 5,
                "numberOfProbes": 2,
                "port":     "[parameters('SFReverseProxyPort')]",
                "protocol": "tcp"
            }
        }  
    ]
}
```


## <a name="customize-reverse-proxy-behavior-using-fabric-settings"></a>Personalización del comportamiento del proxy inverso mediante la configuración del tejido

Puede personalizar el comportamiento del proxy inverso configurando el tejido en la plantilla de Resource Manager para los clústeres hospedados en Azure o en el archivo ClusterConfig.json para los clústeres independientes. La configuración que controla el comportamiento del proxy inverso se encuentra en la sección **fabricSettings**[ de **ApplicationGateway/Http**](./service-fabric-cluster-fabric-settings.md#applicationgatewayhttp), en la sección **Propiedades** del clúster. 

Por ejemplo, puede establecer el valor de **DefaultHttpRequestTimeout** para establecer el tiempo de espera para las solicitudes en el proxy inverso en 180 segundos como se muestra en el siguiente archivo JSON:

   ```json
   {
   "fabricSettings": [
             ...
             {
               "name": "ApplicationGateway/Http",
               "parameters": [
                 {
                   "name": "DefaultHttpRequestTimeout",
                   "value": "180"
                 }
               ]
             }
           ],
           ...
   }
   ``` 

Para más información acerca de cómo actualizar la configuración del tejido para clústeres de Azure, consulte [Personalización de la configuración de clústeres mediante plantillas de Resource Manager](service-fabric-cluster-config-upgrade-azure.md). Para los clústeres independientes, consulte [Personalización de la configuración de clúster de clústeres independientes](service-fabric-cluster-config-upgrade-windows-server.md). 

Varias configuraciones de tejido se usan para establecer una comunicación segura entre el proxy inverso y los servicios. Para información detallada sobre esta configuración, consulte [Conexión a un servicio seguro con el proxy inverso](service-fabric-reverseproxy-configure-secure-communication.md).

## <a name="next-steps"></a>Pasos siguientes
* [Configuración del reenvío para proteger el servicio HTTP con el proxy inverso](service-fabric-reverseproxy-configure-secure-communication.md)
* Para opciones de configuración del proxy inverso, consulte la [sección de ApplicationGateway/Http en Personalización de la configuración del clúster de Service Fabric](service-fabric-cluster-fabric-settings.md#applicationgatewayhttp).