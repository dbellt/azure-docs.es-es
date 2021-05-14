---
title: Registro con redundancia de zona para lograr alta disponibilidad
description: Aprenda a habilitar la redundancia de zona en Azure Container Registry. Cree un registro de contenedor o una replicación en una zona de disponibilidad de Azure. La redundancia de zona es una característica del nivel de servicio Premium.
ms.topic: article
ms.date: 02/23/2021
ms.custom: references_regions, devx-track-azurecli
ms.openlocfilehash: 731962437c4890c665513241e756dbbc2acfc5de
ms.sourcegitcommit: 5ce88326f2b02fda54dad05df94cf0b440da284b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107891597"
---
# <a name="enable-zone-redundancy-in-azure-container-registry-for-resiliency-and-high-availability"></a>Habilitación de la redundancia de zona en Azure Container Registry para lograr resistencia y alta disponibilidad

Además de la [replicación geográfica](container-registry-geo-replication.md), que replica los datos del registro en una o varias regiones de Azure para proporcionar disponibilidad y reducir la latencia en las operaciones regionales, Azure Container Registry admite la *redundancia de zona* opcional. La [redundancia de zona](../availability-zones/az-overview.md#availability-zones) proporciona resistencia y alta disponibilidad a un recurso de registro o de replicación (réplica) en una región específica.

En este artículo se muestra cómo configurar un registro de contenedor o una réplica con redundancia de zona mediante la CLI de Azure, Azure Portal o una plantilla de Azure Resource Manager. 

La redundancia de zona es una característica en **versión preliminar** del nivel de servicio Premium del registro de contenedor. Para obtener información sobre los límites y los niveles de servicio de los registros, consulte [SKU de Azure Container Registry](container-registry-skus.md).

## <a name="preview-limitations"></a>Limitaciones de vista previa

* Actualmente se admite en las siguientes regiones: Este de EE. UU., Este de EE. UU. 2, Oeste de EE. UU. 2, Norte de Europa, Oeste de Europa, Japón Oriental.
* Actualmente no se admite la conversión de regiones en zonas de disponibilidad. Para habilitar la compatibilidad con las zonas de disponibilidad en una región, el registro debe crearse en la región deseada, con la compatibilidad con zonas de disponibilidad habilitada, o bien debe agregarse una región replicada que tenga habilitada la compatibilidad con zonas de disponibilidad.
* No se puede deshabilitar la redundancia de zona en una región.
* [ACR Tasks](container-registry-tasks-overview.md) todavía no admite las zonas de disponibilidad.

## <a name="about-zone-redundancy"></a>Acerca de la redundancia de zona

Use las [zonas de disponibilidad](../availability-zones/az-overview.md) de Azure para crear un registro de contenedor de Azure resistente y de alta disponibilidad en una región de Azure. Por ejemplo, las organizaciones pueden configurar un registro de contenedor de Azure con redundancia de zona con otros [recursos de Azure compatibles](../availability-zones/az-region.md) para satisfacer los requisitos de residencia de datos u otros requisitos de cumplimiento, a la vez que se proporciona alta disponibilidad dentro de una región.

Azure Container Registry también admite la [replicación geográfica](container-registry-geo-replication.md), que replica el servicio en varias regiones, lo que permite aplicar la redundancia y la localidad para procesar recursos de otras ubicaciones. La combinación de zonas de disponibilidad para la redundancia dentro de una región y la replicación geográfica en varias regiones mejora la confiabilidad y el rendimiento de un registro.

Las zonas de disponibilidad son ubicaciones físicas exclusivas dentro de una región de Azure. Para garantizar la resistencia, hay un mínimo de tres zonas independientes en todas las regiones habilitadas. Cada zona tiene uno o varios centros de datos equipados con alimentación, refrigeración y redes independientes. Cuando se configura para la redundancia de zona, un registro (o una réplica del registro en otra región) se replica en todas las zonas de disponibilidad de la región, lo que permite que siga disponible si hay errores en el centro de datos.

## <a name="create-a-zone-redundant-registry---cli"></a>Creación de un registro con redundancia de zona: CLI

Para usar la CLI de Azure para habilitar la redundancia de zona, necesita la versión 2.17.0 o posterior de esta CLI, o Azure Cloud Shell. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

### <a name="create-a-resource-group"></a>Crear un grupo de recursos

Si es necesario, ejecute el comando [az group create](/cli/azure/group#az_group_create) para crear un grupo de recursos para el registro.

```azurecli
az group create --name <resource-group-name> --location <location>
```

### <a name="create-zone-enabled-registry"></a>Creación de un registro habilitado para zona

Ejecute el comando [az acr create](/cli/azure/acr#az_acr_create) para crear un registro con redundancia de zona en el nivel de servicio Premium. Elija una región que [admita zonas de disponibilidad](../availability-zones/az-region.md) para Azure Container Registry. En el ejemplo siguiente, la redundancia de zona está habilitada en la región *eastus*. Consulte la ayuda del comando `az acr create` para ver más opciones de registro.

```azurecli
az acr create \
  --resource-group <resource-group-name> \
  --name <container-registry-name> \
  --location eastus \
  --zone-redundancy enabled \
  --sku Premium
```

En la salida del comando, observe la propiedad `zoneRedundancy` del registro. Cuando está habilitada, el registro tiene redundancia de zona:

```JSON
{
 [...]
"zoneRedundancy": "Enabled",
}
```

### <a name="create-zone-redundant-replication"></a>Creación de una replicación con redundancia de zona

Ejecute el comando [az acr replication create](/cli/azure/acr/replication#az_acr_replication_create) para crear una réplica de registro con redundancia de zona en una región que [admita zonas de disponibilidad](../availability-zones/az-region.md) en Azure Container Registry, como *westus2*. 

```azurecli
az acr replication create \
  --location westus2 \
  --resource-group <resource-group-name> \
  --registry <container-registry-name> \
  --zone-redundancy enabled
```
 
En la salida del comando, observe la propiedad `zoneRedundancy` de la réplica. Cuando está habilitada, la réplica tiene redundancia de zona:

```JSON
{
 [...]
"zoneRedundancy": "Enabled",
}
```

## <a name="create-a-zone-redundant-registry---portal"></a>Creación de un registro con redundancia de zona: Azure Portal

1. Inicie sesión en Azure Portal en [https://portal.azure.com](https://portal.azure.com).
1. Seleccione **Crear un recurso** > **Contenedores** > **Registro de contenedor**.
1. En la pestaña **Datos básicos**, seleccione o cree un grupo de recursos y escriba un nombre de registro único. 
1. En **Ubicación**, seleccione una región que admita la redundancia de zona para Azure Container Registry, como *Este de EE. UU.*
1. En **SKU**, seleccione **Premium**.
1. En **Zonas de disponibilidad**, seleccione **Habilitado**.
1. Si lo desea, configure otras opciones del registro y, a continuación, seleccione **Revisar y crear**.
1. Seleccione **Crear** para implementar la instancia del registro.

    :::image type="content" source="media/zone-redundancy/enable-availability-zones-portal.png" alt-text="Habilitación de la redundancia de zona en Azure Portal":::

Para crear una replicación con redundancia de zona:

1. Vaya al registro de contenedor de nivel Premium y seleccione **Replicaciones**.
1. En el mapa que aparece, seleccione un hexágono verde en una región que admita la redundancia de zona para Azure Container Registry, como **Oeste de EE. UU. 2**. O bien, seleccione **+ Agregar**.
1. En la ventana **Crear replicación**, confirme los datos de **Ubicación**. En **Zonas de disponibilidad**, seleccione **Habilitado** y, luego, elija **Crear**.

    :::image type="content" source="media/zone-redundancy/enable-availability-zones-replication-portal.png" alt-text="Habilitación de la replicación con redundancia de zona en Azure Portal":::

## <a name="create-a-zone-redundant-registry---template"></a>Creación de un registro con redundancia de zona: plantilla

### <a name="create-a-resource-group"></a>Crear un grupo de recursos

Si es necesario, ejecute el comando [az group create](/cli/azure/group#az_group_create) para crear un grupo de recursos para el registro en una región que [admita zonas de disponibilidad](../availability-zones/az-region.md) para Azure Container Registry, como *eastus*. Esta región se usa en la plantilla para establecer la ubicación del registro.

```azurecli
az group create --name <resource-group-name> --location eastus
```

### <a name="deploy-the-template"></a>Implementación de la plantilla 

Puede usar la siguiente plantilla de Resource Manager para crear un registro con replicación geográfica y redundancia de zona. De forma predeterminada, la plantilla permite la redundancia de zona en el registro y en una réplica regional. 

Copie el siguiente contenido en un archivo nuevo y guárdelo con un nombre de archivo como `registryZone.json`.

```JSON
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
      "acrName": {
        "type": "string",
        "defaultValue": "[concat('acr', uniqueString(resourceGroup().id))]",
        "minLength": 5,
        "maxLength": 50,
        "metadata": {
          "description": "Globally unique name of your Azure Container Registry"
        }
      },
      "location": {
        "type": "string",
        "defaultValue": "[resourceGroup().location]",
        "metadata": {
          "description": "Location for registry home replica."
        }
      },
      "acrSku": {
        "type": "string",
        "defaultValue": "Premium",
        "allowedValues": [
          "Premium"
        ],
        "metadata": {
          "description": "Tier of your Azure Container Registry. Geo-replication and zone redundancy require Premium SKU."
        }
      },
      "acrZoneRedundancy": {
        "type": "string",
        "defaultValue": "Enabled",
        "metadata": {
          "description": "Enable zone redundancy of registry's home replica. Requires registry location to support availability zones."
        }
      },
      "acrReplicaLocation": {
        "type": "string",
        "metadata": {
          "description": "Short name for registry replica location."
        }
      },
      "acrReplicaZoneRedundancy": {
        "type": "string",
        "defaultValue": "Enabled",
        "metadata": {
          "description": "Enable zone redundancy of registry replica. Requires replica location to support availability zones."
        }
      }
    },
    "resources": [
      {
        "comments": "Container registry for storing docker images",
        "type": "Microsoft.ContainerRegistry/registries",
        "apiVersion": "2020-11-01-preview",
        "name": "[parameters('acrName')]",
        "location": "[parameters('location')]",
        "sku": {
          "name": "[parameters('acrSku')]",
          "tier": "[parameters('acrSku')]"
        },
        "tags": {
          "displayName": "Container Registry",
          "container.registry": "[parameters('acrName')]"
        },
        "properties": {
          "adminUserEnabled": "[parameters('acrAdminUserEnabled')]",
          "zoneRedundancy": "[parameters('acrZoneRedundancy')]"
        }
      },
      {
        "type": "Microsoft.ContainerRegistry/registries/replications",
        "apiVersion": "2020-11-01-preview",
        "name": "[concat(parameters('acrName'), '/', parameters('acrReplicaLocation'))]",
        "location": "[parameters('acrReplicaLocation')]",
          "dependsOn": [
          "[resourceId('Microsoft.ContainerRegistry/registries/', parameters('acrName'))]"
        ],
        "properties": {
          "zoneRedundancy": "[parameters('acrReplicaZoneRedundancy')]"
        }
      }
    ],
    "outputs": {
      "acrLoginServer": {
        "value": "[reference(resourceId('Microsoft.ContainerRegistry/registries',parameters('acrName')),'2019-12-01-preview').loginServer]",
        "type": "string"
      }
    }
  }
```

Ejecute el siguiente comando [az deployment group create](/cli/azure/group/deployment#az_group_deployment_create) para crear el registro con el archivo de plantilla anterior. Donde se indique, proporcione la siguiente información:

* un nombre de registro único; también puede implementar la plantilla sin parámetros y se creará un nombre único automáticamente
* una ubicación para la réplica que admita zonas de disponibilidad, como *westus2*

```azurecli
az deployment group create \
  --resource-group <resource-group-name> \
  --template-file registryZone.json \
  --parameters acrName=<registry-name> acrReplicaLocation=<replica-location>
```

En la salida del comando, observe la propiedad `zoneRedundancy` del registro y la réplica. Cuando está habilitada, cada recurso tiene redundancia de zona:

```JSON
{
 [...]
"zoneRedundancy": "Enabled",
}
```

## <a name="next-steps"></a>Pasos siguientes

* Obtenga más información sobre las [regiones que admiten zonas de disponibilidad](../availability-zones/az-region.md).
* Obtenga más información sobre el desarrollo de la [confiabilidad](/azure/architecture/framework/resiliency/overview) en Azure.
