---
title: Creación de un clúster de Service Fabric que ejecuta Windows en Azure
description: En este tutorial, aprenderá a implementar un clúster de Azure Service Fabric con Windows en una instancia de Azure Virtual Network mediante PowerShell.
ms.topic: tutorial
ms.date: 07/22/2019
ms.custom: mvc, devx-track-azurepowershell
ms.openlocfilehash: 2c719569fc97121a935c3dcbd4945e50863ecf80
ms.sourcegitcommit: 3bb9f8cee51e3b9c711679b460ab7b7363a62e6b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112082026"
---
# <a name="tutorial-deploy-a-service-fabric-cluster-running-windows-into-an-azure-virtual-network"></a>Tutorial: Implementación de un clúster de Service Fabric con Windows una red virtual de Azure

Este tutorial es la primera parte de una serie. Con él puede aprender a implementar un clúster de Azure Service Fabric con Windows en una instancia de [Azure Virtual Network](../virtual-network/virtual-networks-overview.md) y en un [grupo de seguridad de red](../virtual-network/virtual-network-vnet-plan-design-arm.md) mediante PowerShell y una plantilla. Cuando haya terminado, tendrá un clúster que se ejecuta en la nube en el que puede implementar aplicaciones. Para crear un clúster de Linux con la CLI de Azure, consulte [Implementación de un clúster de Service Fabric de Linux en una red virtual de Azure](service-fabric-tutorial-create-vnet-and-linux-cluster.md).

Este tutorial describe un escenario de producción. Si desea crear un clúster menor para agilizar la prueba, consulte [Creación de un clúster de prueba](./scripts/service-fabric-powershell-create-secure-cluster-cert.md).

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear una red virtual en Azure mediante PowerShell
> * Creación de un almacén de claves y carga de un certificado
> * Configuración de la autenticación de Azure Active Directory
> * Configuración de la colección de diagnóstico
> * Configuración del servicio EventStore
> * Configuración de los registros de Azure Monitor
> * Crear un clúster de Service Fabric seguro en Azure PowerShell
> * Protección del clúster con un certificado X.509
> * Conexión a un clúster con PowerShell
> * Quitar un clúster

En esta serie de tutoriales, se aprende a:
> [!div class="checklist"]
> * Crear un clúster seguro en Azure
> * [Supervisión de un clúster](service-fabric-tutorial-monitor-cluster.md)
> * [Escalado o reducción horizontal](service-fabric-tutorial-scale-cluster.md)
> * [Actualización del entorno en tiempo de ejecución de un clúster](service-fabric-tutorial-upgrade-cluster.md)
> * [Eliminación de un clúster](service-fabric-tutorial-delete-cluster.md)


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Requisitos previos

Antes de empezar este tutorial:

* Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Instalación de [Service Fabric SDK y del módulo de PowerShell](service-fabric-get-started.md).
* Instale [Azure PowerShell](/powershell/azure/install-az-ps).
* Repase los conceptos clave sobre [clústeres de Azure](service-fabric-azure-clusters-overview.md).
* [Planeación y preparación](service-fabric-cluster-azure-deployment-preparation.md) para la implementación de un clúster de producción.

Los siguientes procedimientos crean un clúster de Service Fabric de siete nodos. Use la [Calculadora de precios de Azure](https://azure.microsoft.com/pricing/calculator/) para calcular el costo producido por la ejecución de un clúster de Service Fabric en Azure.

## <a name="download-and-explore-the-template"></a>Descarga y exploración de la plantilla

Descargue los siguientes archivos de plantilla de Azure Resource Manager:

* [azuredeploy.json][template]
* [azuredeploy.parameters.json][parameters]

Esta plantilla implementa un clúster seguro de siete máquinas virtuales y tres tipos de nodo en una red virtual y un grupo de seguridad de red.  Se pueden encontrar otras plantillas de ejemplo en [GitHub](https://github.com/Azure-Samples/service-fabric-cluster-templates). El archivo [azuredeploy.json][template] permite implementar varios recursos, incluidos los siguientes.

### <a name="service-fabric-cluster"></a>Clúster de Service Fabric

En el recurso **Microsoft.ServiceFabric/clusters**, un clúster de Windows está configurado con las siguientes características:

* Tres tipos de nodo.
* Cinco nodos en el tipo de nodo principal (configurable en los parámetros de la plantilla) y un nodo en cada uno de los otros tipos de nodo.
* Sistema operativo: Windows Server 2016 Datacenter con Containers (configurable en los parámetros de la plantilla).
* Protección con certificado (configurable en los parámetros de la plantilla).
* Se habilita el [proxy inverso](service-fabric-reverseproxy.md).
* Se habilita el [servicio DNS](service-fabric-dnsservice.md).
* [Nivel de durabilidad](service-fabric-cluster-capacity.md#durability-characteristics-of-the-cluster) de Bronze (configurable en los parámetros de la plantilla).
* [Nivel de confiabilidad](service-fabric-cluster-capacity.md#reliability-characteristics-of-the-cluster) de Silver (configurable en los parámetros de la plantilla).
* Punto de conexión de la conexión de cliente: 19000 (configurable en los parámetros de la plantilla).
* punto de conexión de la puerta de enlace HTTP: 19080 (configurable en los parámetros de la plantilla).

### <a name="azure-load-balancer"></a>Azure Load Balancer

En el recurso **Microsoft.Network/loadBalancers** se han configurado un equilibrador de carga. Los sondeos y reglas se configuran para los puertos siguientes:

* Punto de conexión de la conexión de cliente: 19000
* punto de conexión de la puerta de enlace HTTP: 19080
* Puerto de la aplicación: 80
* Puerto de la aplicación: 443
* Proxy inverso de Service Fabric: 19081

Si se necesitan otros puertos de aplicación, debe ajustar el recurso **Microsoft.Network/loadBalancers** y el recurso **Microsoft.Network/networkSecurityGroups** para permitir el tráfico de entrada.

### <a name="virtual-network-subnet-and-network-security-group"></a>Red virtual, subred y grupo de seguridad de red

Los nombres de la red virtual, la subred y el grupo de seguridad de red se declaran en los parámetros de la plantilla. Los espacios de direcciones de la red virtual y de la subred se declaran también en los parámetros de plantilla y se configuran en el recurso **Microsoft.Network/virtualNetworks**:

* Espacio de direcciones de red virtual: 172.16.0.0/20
* espacio de direcciones de subred de Service Fabric: 172.16.2.0/23

Las siguientes reglas de tráfico de entrada están habilitadas en el recurso **Microsoft.Network/networksecuritygroups**. Puede cambiar los valores de puerto cambiando las variables de la plantilla.

* ClientConnectionEndpoint (TCP): 19000
* HttpGatewayEndpoint (HTTP/TCP): 19080
* SMB: 445
* Internodecommunication: 1025, 1026, 1027
* Intervalo de puertos efímeros: 49152 a 65534 (necesita un mínimo de 256 puertos).
* Puertos para el uso de las aplicaciones: 80 y 443
* Intervalo de puertos de destino: 49152 a 65534 (se usan para la comunicación entre servicios. Otros puertos no se abren en el equilibrador de carga).
* Bloqueo de todos los puertos restantes

Si se necesitan otros puertos de aplicación, debe ajustar el recurso **Microsoft.Network/loadBalancers** y el recurso **Microsoft.Network/networkSecurityGroups** para permitir el tráfico de entrada.

### <a name="windows-defender"></a>Windows Defender
De forma predeterminada, el [programa antivirus Windows Defender](/windows/security/threat-protection/windows-defender-antivirus/windows-defender-antivirus-on-windows-server-2016) está instalado y operativo en Windows Server 2016. La interfaz de usuario se instala de forma predeterminada en algunas SKU, pero no es necesaria. En cada tipo de nodo o conjunto de escalado de máquinas virtuales que se declara en la plantilla, se usa la [extensión de antimalware para máquinas virtuales de Azure](../virtual-machines/extensions/iaas-antimalware-windows.md) para excluir los directorios y procesos de Service Fabric:

```json
{
"name": "[concat('VMIaaSAntimalware','_vmNodeType0Name')]",
"properties": {
        "publisher": "Microsoft.Azure.Security",
        "type": "IaaSAntimalware",
        "typeHandlerVersion": "1.5",
        "settings": {
        "AntimalwareEnabled": "true",
        "Exclusions": {
                "Paths": "D:\\SvcFab;D:\\SvcFab\\Log;C:\\Program Files\\Microsoft Service Fabric",
                "Processes": "Fabric.exe;FabricHost.exe;FabricInstallerService.exe;FabricSetup.exe;FabricDeployer.exe;ImageBuilder.exe;FabricGateway.exe;FabricDCA.exe;FabricFAS.exe;FabricUOS.exe;FabricRM.exe;FileStoreService.exe"
        },
        "RealtimeProtectionEnabled": "true",
        "ScheduledScanSettings": {
                "isEnabled": "true",
                "scanType": "Quick",
                "day": "7",
                "time": "120"
        }
        },
        "protectedSettings": null
}
}
```

## <a name="set-template-parameters"></a>Configuración de los parámetros de plantilla

El archivo de parámetros [azuredeploy.parameters.json][parameters] permite declarar muchos valores que se usan para implementar el clúster y los recursos asociados. Los siguientes son los parámetros para modificar la implementación:

**Parámetro** | **Valor de ejemplo** | **Notas** 
|---|---|---|
|adminUserName|vmadmin| Nombre de usuario del administrador de las máquinas virtuales del clúster. [Requisitos de nombre de usuario para la máquina virtual](../virtual-machines/windows/faq.yml#what-are-the-username-requirements-when-creating-a-vm-). |
|adminPassword|Password#1234| Contraseña del administrador de las máquinas virtuales del clúster. [Requisitos de contraseña para la máquina virtual](../virtual-machines/windows/faq.yml#what-are-the-password-requirements-when-creating-a-vm-).|
|clusterName|mysfcluster123| Nombre del clúster. Solo puede contener letras y números. Puede tener entre 3 y 23 caracteres.|
|ubicación|southcentralus| Ubicación del clúster. |
|certificateThumbprint|| <p>El valor debe estar vacío si se va a crear un certificado autofirmado o a proporcionar un archivo de certificados.</p><p>Para usar un certificado existente cargado previamente en un almacén de claves, rellene el valor de huella digital SHA1 del certificado. Por ejemplo, "6190390162C988701DB5676EB81083EA608DCCF3".</p> |
|certificateUrlValue|| <p>El valor debe estar vacío si se va a crear un certificado autofirmado o a proporcionar un archivo de certificados. </p><p>Para usar un certificado existente cargado previamente en un almacén de claves, especifique la dirección URL del certificado. Por ejemplo, "https:\//mykeyvault.vault.azure.net:443/secrets/mycertificate/02bea722c9ef4009a76c5052bcbf8346".</p>|
|sourceVaultValue||<p>El valor debe estar vacío si se va a crear un certificado autofirmado o a proporcionar un archivo de certificados.</p><p>Para usar un certificado existente cargado previamente en un almacén de claves, especifique el valor del almacén de claves de origen. Por ejemplo, "/subscriptions/333cc2c84-12fa-5778-bd71-c71c07bf873f/resourceGroups/MyTestRG/providers/Microsoft.KeyVault/vaults/MYKEYVAULT".</p>|

## <a name="set-up-azure-active-directory-client-authentication"></a>Configuración de la autenticación de cliente de Azure Active Directory
Para los clústeres de Service Fabric implementados en una red pública hospedada en Azure, la recomendación para la autenticación mutua de cliente a nodo es:
* Usar Azure Active Directory para la identidad del cliente.
* Usar un certificado para la identidad del servidor y el cifrado TLS de la comunicación HTTP.

La configuración de Azure Active Directory (Azure AD) para autenticar a los clientes de un clúster de Service Fabric se debe realizar antes de [crear el clúster](#createvaultandcert). Azure AD permite a las organizaciones (conocidas como inquilinos) administrar el acceso de los usuarios a las aplicaciones. 

Un clúster de Service Fabric ofrece diversos puntos de entrada a su funcionalidad de administración, como [Service Fabric Explorer](service-fabric-visualizing-your-cluster.md) y [Visual Studio](service-fabric-manage-application-in-visual-studio.md) basados en web. Como consecuencia, creará dos aplicaciones de Azure AD para controlar el acceso al clúster: una aplicación web y una aplicación nativa.  Una vez creadas las aplicaciones, debe asignar usuarios a los roles de solo lectura y administrador.

> [!NOTE]
> Debe realizar los pasos siguientes antes de crear el clúster. Dado que los scripts esperan los puntos de conexión y los nombres de los clústeres, los valores deben planearse y no se pueden usar los que ya haya creado.

En este artículo, se supone que ya ha creado un inquilino. En caso de que no lo haya hecho, lea [Obtención de un inquilino de Azure Active Directory](../active-directory/develop/quickstart-create-new-tenant.md).

Para simplificar los pasos necesarios para configurar Azure AD con un clúster de Service Fabric, hemos creado un conjunto de scripts de Windows PowerShell. [Descargue los scripts](https://github.com/Azure-Samples/service-fabric-aad-helpers) en el equipo.

### <a name="create-azure-ad-applications-and-assign-users-to-roles"></a>Creación de aplicaciones de Azure AD y asignación de usuarios a roles
Cree dos aplicaciones de Azure AD para controlar el acceso al clúster: una aplicación web y una aplicación nativa. Una vez que haya creado las aplicaciones para representar el clúster, debe asignar los usuarios a los [roles compatibles con Service Fabric](service-fabric-cluster-security-roles.md): solo lectura y administrador.

Ejecute `SetupApplications.ps1` y proporcione el identificador de inquilino, el nombre del clúster y la dirección URL de respuesta de la aplicación web como parámetros. Especifique los nombres de usuario y las contraseñas para los usuarios. Por ejemplo:

```powershell
$Configobj = .\SetupApplications.ps1 -TenantId '<MyTenantID>' -ClusterName 'mysfcluster123' -WebApplicationReplyUrl 'https://mysfcluster123.eastus.cloudapp.azure.com:19080/Explorer/index.html' -AddResourceAccess
.\SetupUser.ps1 -ConfigObj $Configobj -UserName 'TestUser' -Password 'P@ssword!123'
.\SetupUser.ps1 -ConfigObj $Configobj -UserName 'TestAdmin' -Password 'P@ssword!123' -IsAdmin
```

> [!NOTE]
> Para las nubes nacionales (por ejemplo, Azure Government, Azure China, Azure Alemania), especifique el parámetro `-Location`.

Puede encontrar el identificador *TenantId* o el identificador de directorio en [Azure Portal](https://portal.azure.com). Seleccione **Azure Active Directory** > **Propiedades** y copie el valor de **Id. de directorio**.

El valor de *ClusterName* se usa como prefijo en las aplicaciones de Azure AD que crea el script. No es necesario que coincida exactamente con el nombre del clúster real. Solo facilita la asignación de artefactos de Azure AD para el clúster de Service Fabric en uso.

El valor de *WebApplicationReplyUrl* es el punto de conexión predeterminado al que Azure AD devuelve a los usuarios cuando terminan el inicio de sesión. Establézcalo como el punto de conexión de Service Fabric Explorer para el clúster, que de manera predeterminada es el siguiente:

https://&lt;cluster_domain&gt;:19080/Explorer

Se le pedirá que inicie sesión en una cuenta con privilegios administrativos para el inquilino de Azure AD. Una vez iniciada la sesión, el script creará las aplicaciones web y nativa para representar el clúster de Service Fabric. En las aplicaciones del inquilino en [Azure Portal](https://portal.azure.com), debería ver dos entradas nuevas:

   * *ClusterName*\_Clúster
   * *ClusterName*\_Cliente

El script imprimirá el código JSON que necesita la plantilla de Azure Resource Manager al crear el clúster, por lo que es buena idea mantener abierta la ventana de PowerShell.

```json
"azureActiveDirectory": {
  "tenantId":"<guid>",
  "clusterApplication":"<guid>",
  "clientApplication":"<guid>"
},
```

### <a name="add-azure-ad-configuration-to-use-azure-ad-for-client-access"></a>Adición de la configuración de Azure AD para usar Azure AD para el acceso de cliente
En [azuredeploy.json][template], configure Azure AD en la sección **Microsoft.ServiceFabric/clusters**. Agregue parámetros para el identificador de inquilino, el identificador de la aplicación del clúster y el identificador de la aplicación cliente.  

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json",
  "contentVersion": "1.0.0.0",
  "parameters": {
    ...

    "aadTenantId": {
      "type": "string",
      "defaultValue": "0e3d2646-78b3-4711-b8be-74a381d9890c"
    },
    "aadClusterApplicationId": {
      "type": "string",
      "defaultValue": "cb147d34-b0b9-4e77-81d6-420fef0c4180"
    },
    "aadClientApplicationId": {
      "type": "string",
      "defaultValue": "7a8f3b37-cc40-45cc-9b8f-57b8919ea461"
    }
  },

...

{
  "apiVersion": "2018-02-01",
  "type": "Microsoft.ServiceFabric/clusters",
  "name": "[parameters('clusterName')]",
  ...
  "properties": {
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

Agregue los valores de los parámetros en el archivo de parámetros [azuredeploy.parameters.json][parameters]. Por ejemplo:

```json
"aadTenantId": {
"value": "0e3d2646-78b3-4711-b8be-74a381d9890c"
},
"aadClusterApplicationId": {
"value": "cb147d34-b0b9-4e77-81d6-420fef0c4180"
},
"aadClientApplicationId": {
"value": "7a8f3b37-cc40-45cc-9b8f-57b8919ea461"
}
```
<a id="configurediagnostics" name="configurediagnostics_anchor"></a>

## <a name="configure-diagnostics-collection-on-the-cluster"></a>Configuración de la colección de diagnóstico en el clúster
Cuando se ejecuta un clúster de Azure Service Fabric, es conveniente recopilar los registros de todos los nodos en una ubicación central. La presencia de los registros en una ubicación central facilita el análisis y la solución de los problemas del clúster o de las aplicaciones y los servicios que se ejecutan en ese clúster.

Uno de los métodos para cargar y recopilar registros es usar la extensión Azure Diagnostics (WAD), que carga registros en Azure Storage y también tiene la opción de enviar registros a Azure Application Insights o Event Hubs. Además, puede usar un proceso externo para leer los eventos desde el almacenamiento y colocarlos en un producto de plataforma de análisis, como los registros de Azure Monitor u otra solución de análisis de registros.

Si está siguiendo este tutorial, la recopilación de diagnóstico ya está configurada en la [plantilla][template].

Si tiene un clúster existente sin la extensión de Diagnostics implementada, puede agregarla o actualizarla con la plantilla del clúster. Modifique la plantilla de Resource Manager usada para crear el clúster existente o descargue la plantilla desde el portal. Modifique el archivo template.json mediante la realización de las siguientes tareas:

Agregue un nuevo recurso de almacenamiento a la sección de recursos en la plantilla:
```json
"resources": [
...
{
  "apiVersion": "2015-05-01-preview",
  "type": "Microsoft.Storage/storageAccounts",
  "name": "[parameters('applicationDiagnosticsStorageAccountName')]",
  "location": "[parameters('computeLocation')]",
  "sku": {
    "accountType": "[parameters('applicationDiagnosticsStorageAccountType')]"
  },
  "tags": {
    "resourceType": "Service Fabric",
    "clusterName": "[parameters('clusterName')]"
  }
},
...
]
```

A continuación, agregue parámetros para el nombre de la cuenta de almacenamiento y el tipo a la sección de parámetros de la plantilla. Reemplace el texto del marcador de posición aquí va el nombre de la cuenta de almacenamiento por el nombre de la cuenta de almacenamiento que desee.

```json
"parameters": {
...
"applicationDiagnosticsStorageAccountType": {
    "type": "string",
    "allowedValues": [
    "Standard_LRS",
    "Standard_GRS"
    ],
    "defaultValue": "Standard_LRS",
    "metadata": {
    "description": "Replication option for the application diagnostics storage account"
    }
},
"applicationDiagnosticsStorageAccountName": {
    "type": "string",
    "defaultValue": "**STORAGE ACCOUNT NAME GOES HERE**",
    "metadata": {
    "description": "Name for the storage account that contains application diagnostics data from the cluster"
    }
},
...
}
```

A continuación, agregue la extensión **IaaSDiagnostics** a la matriz de extensiones de la propiedad **VirtualMachineProfile** de cada recurso **Microsoft.Compute/virtualMachineScaleSets** del clúster.  Si usa la [plantilla de muestra][template], hay tres conjuntos de escalado de máquinas virtuales (uno para cada tipo de nodo del clúster).

```json
"apiVersion": "2018-10-01",
"type": "Microsoft.Compute/virtualMachineScaleSets",
"name": "[variables('vmNodeType1Name')]",
"properties": {
    ...
    "virtualMachineProfile": {
        "extensionProfile": {
            "extensions": [
                {
                    "name": "[concat(parameters('vmNodeType0Name'),'_Microsoft.Insights.VMDiagnosticsSettings')]",
                    "properties": {
                        "type": "IaaSDiagnostics",
                        "autoUpgradeMinorVersion": true,
                        "protectedSettings": {
                        "storageAccountName": "[parameters('applicationDiagnosticsStorageAccountName')]",
                        "storageAccountKey": "[listKeys(resourceId('Microsoft.Storage/storageAccounts', parameters('applicationDiagnosticsStorageAccountName')),'2015-05-01-preview').key1]",
                        "storageAccountEndPoint": "https://core.windows.net/"
                        },
                        "publisher": "Microsoft.Azure.Diagnostics",
                        "settings": {
                        "WadCfg": {
                            "DiagnosticMonitorConfiguration": {
                            "overallQuotaInMB": "50000",
                            "EtwProviders": {
                                "EtwEventSourceProviderConfiguration": [
                                {
                                    "provider": "Microsoft-ServiceFabric-Actors",
                                    "scheduledTransferKeywordFilter": "1",
                                    "scheduledTransferPeriod": "PT5M",
                                    "DefaultEvents": {
                                    "eventDestination": "ServiceFabricReliableActorEventTable"
                                    }
                                },
                                {
                                    "provider": "Microsoft-ServiceFabric-Services",
                                    "scheduledTransferPeriod": "PT5M",
                                    "DefaultEvents": {
                                    "eventDestination": "ServiceFabricReliableServiceEventTable"
                                    }
                                }
                                ],
                                "EtwManifestProviderConfiguration": [
                                {
                                    "provider": "cbd93bc2-71e5-4566-b3a7-595d8eeca6e8",
                                    "scheduledTransferLogLevelFilter": "Information",
                                    "scheduledTransferKeywordFilter": "4611686018427387904",
                                    "scheduledTransferPeriod": "PT5M",
                                    "DefaultEvents": {
                                    "eventDestination": "ServiceFabricSystemEventTable"
                                    }
                                }
                                ]
                            }
                            }
                        },
                        "StorageAccount": "[parameters('applicationDiagnosticsStorageAccountName')]"
                        },
                        "typeHandlerVersion": "1.5"
                    }
                }
            ...
            ]
        }
    }
}
```
<a id="configureeventstore" name="configureeventstore_anchor"></a>

## <a name="configure-the-eventstore-service"></a>Configuración del servicio EventStore
El servicio EventStore es una opción de supervisión de Service Fabric. EventStore proporciona una manera de comprender el estado del clúster o las cargas de trabajo en un momento dado. EventStore es un servicio de Service Fabric con estado que mantiene los eventos del clúster. El evento se expone a través de las API, de REST y de Service Fabric Explorer. EventStore consulta el clúster directamente para obtener datos de diagnóstico sobre cualquier entidad del clúster y debe usarse como ayuda para lo siguiente:

* Diagnosticar problemas en el desarrollo, prueba o allí donde use una canalización de supervisión
* Confirmar que el clúster procesa correctamente las acciones de administración que realiza
* Obtener una "instantánea" de cómo Service Fabric está interactuando con una entidad determinada



Para habilitar el servicio EventStore en el clúster, agregue lo siguiente a la propiedad **fabricSettings** del recurso **Microsoft.ServiceFabric/clusters**:

```json
"apiVersion": "2018-02-01",
"type": "Microsoft.ServiceFabric/clusters",
"name": "[parameters('clusterName')]",
"properties": {
    ...
    "fabricSettings": [
        ...
        {
            "name": "EventStoreService",
            "parameters": [
                {
                "name": "TargetReplicaSetSize",
                "value": "3"
                },
                {
                "name": "MinReplicaSetSize",
                "value": "1"
                }
            ]
        }
    ]
}
```
<a id="configureloganalytics" name="configureloganalytics_anchor"></a>

## <a name="set-up-azure-monitor-logs-for-the-cluster"></a>Configuración de los registros de Azure Monitor para el clúster

Para supervisar los eventos de nivel de clúster, recomendamos los registros de Azure Monitor. Para configurar los registros de Azure Monitor a fin de supervisar el clúster, debe tener [habilitado Diagnostics para ver los eventos de nivel de clúster](#configure-diagnostics-collection-on-the-cluster).  

El área de trabajo debe conectarse a los datos de diagnóstico que proceden del clúster.  Estos datos de registro se almacenan en la cuenta de almacenamiento *applicationDiagnosticsStorageAccountName* en las tablas WADServiceFabric*EventTable, WADWindowsEventLogsTable y WADETWEventTable.

Agregue el área de trabajo de Azure Log Analytics y la solución al área de trabajo:

```json
"resources": [
    ...
    {
        "apiVersion": "2015-11-01-preview",
        "location": "[parameters('omsRegion')]",
        "name": "[parameters('omsWorkspacename')]",
        "type": "Microsoft.OperationalInsights/workspaces",
        "properties": {
            "sku": {
                "name": "Free"
            }
        },
        "resources": [
            {
                "apiVersion": "2015-11-01-preview",
                "name": "[concat(variables('applicationDiagnosticsStorageAccountName'),parameters('omsWorkspacename'))]",
                "type": "storageinsightconfigs",
                "dependsOn": [
                    "[concat('Microsoft.OperationalInsights/workspaces/', parameters('omsWorkspacename'))]",
                    "[concat('Microsoft.Storage/storageAccounts/', variables('applicationDiagnosticsStorageAccountName'))]"
                ],
                "properties": {
                    "containers": [],
                    "tables": [
                        "WADServiceFabric*EventTable",
                        "WADWindowsEventLogsTable",
                        "WADETWEventTable"
                    ],
                    "storageAccount": {
                        "id": "[resourceId('Microsoft.Storage/storageaccounts/', variables('applicationDiagnosticsStorageAccountName'))]",
                        "key": "[listKeys(resourceId('Microsoft.Storage/storageAccounts', variables('applicationDiagnosticsStorageAccountName')),'2015-06-15').key1]"
                    }
                }
            },
            {
                "apiVersion": "2015-11-01-preview",
                "type": "datasources",
                "name": "sampleWindowsPerfCounter",
                "dependsOn": [
                    "[concat('Microsoft.OperationalInsights/workspaces/', parameters('omsWorkspacename'))]"
                ],
                "kind": "WindowsPerformanceCounter",
                "properties": {
                    "objectName": "Memory",
                    "instanceName": "*",
                    "intervalSeconds": 10,
                    "counterName": "Available MBytes"
                }
            },
            {
                "apiVersion": "2015-11-01-preview",
                "type": "datasources",
                "name": "sampleWindowsPerfCounter2",
                "dependsOn": [
                    "[concat('Microsoft.OperationalInsights/workspaces/', parameters('omsWorkspacename'))]"
                ],
                "kind": "WindowsPerformanceCounter",
                "properties": {
                    "objectName": "Service Fabric Service",
                    "instanceName": "*",
                    "intervalSeconds": 10,
                    "counterName": "Average milliseconds per request"
                }
            }
        ]
    },
    {
        "apiVersion": "2015-11-01-preview",
        "location": "[parameters('omsRegion')]",
        "name": "[variables('solution')]",
        "type": "Microsoft.OperationsManagement/solutions",
        "dependsOn": [
            "[concat('Microsoft.OperationalInsights/workspaces/', parameters('omsWorkspacename'))]"
        ],
        "properties": {
            "workspaceResourceId": "[resourceId('Microsoft.OperationalInsights/workspaces/', parameters('omsWorkspacename'))]"
        },
        "plan": {
            "name": "[variables('solution')]",
            "publisher": "Microsoft",
            "product": "[Concat('OMSGallery/', variables('solutionName'))]",
            "promotionCode": ""
        }
    }
]
```

Después, agregue parámetros
```json
"parameters": {
    ...
    "omsWorkspacename": {
        "type": "string",
        "defaultValue": "mysfomsworkspace",
        "metadata": {
            "description": "Name of your OMS Log Analytics Workspace"
        }
    },
    "omsRegion": {
        "type": "string",
        "defaultValue": "West Europe",
        "allowedValues": [
            "West Europe",
            "East US",
            "Southeast Asia"
        ],
        "metadata": {
            "description": "Specify the Azure Region for your OMS workspace"
        }
    }
}
```

A continuación, agregue las variables:
```json
"variables": {
    ...
    "solution": "[Concat('ServiceFabric', '(', parameters('omsWorkspacename'), ')')]",
    "solutionName": "ServiceFabric"
}
```

Agregue la extensión del agente de Log Analytics a cada conjunto de escalado de máquinas virtuales en el clúster y conecte el agente al área de trabajo de Log Analytics. Esto permite recopilar datos de diagnóstico sobre los contenedores, las aplicaciones y la supervisión de rendimiento. Si lo agrega como una extensión al recurso del conjunto de escalado de máquinas virtuales, Azure Resource Manager garantiza la instalación en todos los nodos, incluso cuando se ajusta la escala del clúster.

```json
"apiVersion": "2018-10-01",
"type": "Microsoft.Compute/virtualMachineScaleSets",
"name": "[variables('vmNodeType1Name')]",
"properties": {
    ...
    "virtualMachineProfile": {
        "extensionProfile": {
            "extensions": [
                {
                    "name": "[concat(variables('vmNodeType0Name'),'OMS')]",
                    "properties": {
                        "publisher": "Microsoft.EnterpriseCloud.Monitoring",
                        "type": "MicrosoftMonitoringAgent",
                        "typeHandlerVersion": "1.0",
                        "autoUpgradeMinorVersion": true,
                        "settings": {
                            "workspaceId": "[reference(resourceId('Microsoft.OperationalInsights/workspaces/', parameters('omsWorkspacename')), '2015-11-01-preview').customerId]"
                        },
                        "protectedSettings": {
                            "workspaceKey": "[listKeys(resourceId('Microsoft.OperationalInsights/workspaces/', parameters('omsWorkspacename')),'2015-11-01-preview').primarySharedKey]"
                        }
                    }
                }
            ...
            ]
        }
    }
}
```

<a id="createvaultandcert" name="createvaultandcert_anchor"></a>

## <a name="deploy-the-virtual-network-and-cluster"></a>Implementación de la red virtual y el clúster

A continuación, configure la topología de red e implemente el clúster de Service Fabric. La plantilla de Resource Manager [azuredeploy.json][template] permite crear una red virtual, una subred y un grupo de seguridad de red para Service Fabric. La plantilla permite también implementar un clúster con seguridad mediante certificados habilitada. Para los clústeres de producción, use un certificado de una entidad de certificación (CA) como certificado del clúster. Un certificado autofirmado se puede usar para proteger los clústeres de prueba.

La plantilla de este artículo implementa un clúster que usa la huella digital para identificar el certificado del clúster. No hay dos certificados que puedan tener la misma huella digital, lo que dificulta la administración de certificados. Si cambia un clúster implementado para que use nombres comunes del certificado en vez de las huellas digitales del mismo, se simplifica la administración de los certificados. Para más información sobre cómo actualizar el clúster para que use nombres comunes del certificado para la administración de certificados, consulte cómo [cambiar el clúster a administración de nombre común del certificado](service-fabric-cluster-change-cert-thumbprint-to-cn.md).

### <a name="create-a-cluster-by-using-an-existing-certificate"></a>Creación de un clúster mediante un certificado existente

El script siguiente usa el cmdlet [New-AzServiceFabricCluster](/powershell/module/az.servicefabric/New-azServiceFabricCluster) y una plantilla para implementar un clúster nuevo en Azure. El cmdlet crea un nuevo almacén de claves de Azure y carga el certificado.

```powershell
# Variables.
$groupname = "sfclustertutorialgroup"
$clusterloc="southcentralus"  # Must match the location parameter in the template
$templatepath="C:\temp\cluster"

$certpwd="q6D7nN%6ck@6" | ConvertTo-SecureString -AsPlainText -Force
$clustername = "mysfcluster123"  # Must match the clustername parameter in the template
$vaultname = "clusterkeyvault123"
$vaultgroupname="clusterkeyvaultgroup123"
$subname="$clustername.$clusterloc.cloudapp.azure.com"

# Sign in to your Azure account and select your subscription
Connect-AzAccount
Get-AzSubscription
Set-AzContext -SubscriptionId <guid>

# Create a new resource group for your deployment, and give it a name and a location.
New-AzResourceGroup -Name $groupname -Location $clusterloc

# Create the Service Fabric cluster.
New-AzServiceFabricCluster  -ResourceGroupName $groupname -TemplateFile "$templatepath\azuredeploy.json" `
-ParameterFile "$templatepath\azuredeploy.parameters.json" -CertificatePassword $certpwd `
-KeyVaultName $vaultname -KeyVaultResourceGroupName $vaultgroupname -CertificateFile $certpath
```

### <a name="create-a-cluster-by-using-a-new-self-signed-certificate"></a>Creación de un clúster mediante un nuevo certificado autofirmado

El script siguiente usa el cmdlet [New-AzServiceFabricCluster](/powershell/module/az.servicefabric/New-azServiceFabricCluster) y una plantilla para implementar un clúster nuevo en Azure. El cmdlet crea un almacén de claves en Azure, agrega un certificado autofirmado nuevo en el almacén de claves y carga el certificado en el archivo de certificados de forma local.

```powershell
# Variables.
$groupname = "sfclustertutorialgroup"
$clusterloc="southcentralus"  # Must match the location parameter in the template
$templatepath="C:\temp\cluster"

$certpwd="q6D7nN%6ck@6" | ConvertTo-SecureString -AsPlainText -Force
$certfolder="c:\mycertificates\"
$clustername = "mysfcluster123"
$vaultname = "clusterkeyvault123"
$vaultgroupname="clusterkeyvaultgroup123"
$subname="$clustername.$clusterloc.cloudapp.azure.com"

# Sign in to your Azure account and select your subscription
Connect-AzAccount
Get-AzSubscription
Set-AzContext -SubscriptionId <guid>

# Create a new resource group for your deployment, and give it a name and a location.
New-AzResourceGroup -Name $groupname -Location $clusterloc

# Create the Service Fabric cluster.
New-AzServiceFabricCluster  -ResourceGroupName $groupname -TemplateFile "$templatepath\azuredeploy.json" `
-ParameterFile "$templatepath\azuredeploy.parameters.json" -CertificatePassword $certpwd `
-CertificateOutputFolder $certfolder -KeyVaultName $vaultname -KeyVaultResourceGroupName $vaultgroupname -CertificateSubjectName $subname

```

## <a name="connect-to-the-secure-cluster"></a>Conexión al clúster seguro

Conéctese al clúster mediante el módulo PowerShell de Service Fabric instalado con el SDK de Service Fabric.  En primer lugar, instale el certificado en el almacén personal (Mi) del usuario actual en el equipo. Ejecute el siguiente comando de PowerShell:

```powershell
$certpwd="q6D7nN%6ck@6" | ConvertTo-SecureString -AsPlainText -Force
Import-PfxCertificate -Exportable -CertStoreLocation Cert:\CurrentUser\My `
        -FilePath C:\mycertificates\mysfcluster20170531104310.pfx `
        -Password $certpwd
```

Ahora está preparado para conectarse al clúster seguro.

El módulo PowerShell de **Service Fabric** proporciona muchos cmdlets para administrar clústeres, aplicaciones y servicios de Service Fabric. Use el cmdlet [Connect-ServiceFabricCluster](/powershell/module/servicefabric/connect-servicefabriccluster) para conectarse al clúster seguro. Los detalles de la huella digital SHA1 del certificado y del punto de conexión se encuentran en la salida del paso anterior.

Si anteriormente configuró la autenticación de cliente de Azure AD, ejecute el comando siguiente: 
```powershell
Connect-ServiceFabricCluster -ConnectionEndpoint mysfcluster123.southcentralus.cloudapp.azure.com:19000 `
        -KeepAliveIntervalInSec 10 `
        -AzureActiveDirectory `
        -ServerCertThumbprint C4C1E541AD512B8065280292A8BA6079C3F26F10
```

Si configuró la autenticación de cliente de Azure AD, ejecute el comando siguiente:
```powershell
Connect-ServiceFabricCluster -ConnectionEndpoint mysfcluster123.southcentralus.cloudapp.azure.com:19000 `
          -KeepAliveIntervalInSec 10 `
          -X509Credential -ServerCertThumbprint C4C1E541AD512B8065280292A8BA6079C3F26F10 `
          -FindType FindByThumbprint -FindValue C4C1E541AD512B8065280292A8BA6079C3F26F10 `
          -StoreLocation CurrentUser -StoreName My
```

Compruebe que está conectado y que el estado del clúster es correcto mediante el cmdlet [Get-ServiceFabricClusterHealth](/powershell/module/servicefabric/get-servicefabricclusterhealth).

```powershell
Get-ServiceFabricClusterHealth
```

## <a name="clean-up-resources"></a>Limpieza de recursos

En el resto de artículos de esta serie de tutoriales se usa el clúster que creó anteriormente. Si no va a pasar inmediatamente al siguiente artículo, puede [eliminar el clúster](./service-fabric-tutorial-delete-cluster.md) para evitar incurrir en cargos.

## <a name="next-steps"></a>Pasos siguientes

Avance hasta el tutorial siguiente para obtener información sobre cómo escalar el clúster.

> [!div class="checklist"]
> * Crear una red virtual en Azure mediante PowerShell
> * Creación de un almacén de claves y carga de un certificado
> * Configuración de la autenticación de Azure Active Directory
> * Configuración de la colección de diagnóstico
> * Configuración del servicio EventStore
> * Configuración de los registros de Azure Monitor
> * Crear un clúster de Service Fabric seguro en Azure PowerShell
> * Protección del clúster con un certificado X.509
> * Conexión a un clúster con PowerShell
> * Quitar un clúster

Luego, avance hasta el tutorial siguiente para obtener información sobre cómo supervisar el clúster.
> [!div class="nextstepaction"]
> [Supervisión de un clúster](service-fabric-tutorial-monitor-cluster.md)

[template]:https://github.com/Azure-Samples/service-fabric-cluster-templates/blob/master/7-VM-Windows-3-NodeTypes-Secure-NSG/AzureDeploy.json
[parameters]:https://github.com/Azure-Samples/service-fabric-cluster-templates/blob/master/7-VM-Windows-3-NodeTypes-Secure-NSG/AzureDeploy.Parameters.json
