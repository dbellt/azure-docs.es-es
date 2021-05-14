---
title: Transferencia de artefactos
description: Transferir colecciones de imágenes u otros artefactos de un registro de contenedor a otro registro mediante la creación de una canalización de transferencia con cuentas de Azure Storage
ms.topic: article
ms.date: 10/07/2020
ms.custom: ''
ms.openlocfilehash: c966600b0ca9d65cf533c3c2f0aca211c84917bd
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107780782"
---
# <a name="transfer-artifacts-to-another-registry"></a>Transferir artefactos a otro registro

En este artículo se muestra cómo transferir colecciones de imágenes u otros artefactos del registro de un contenedor de Azure a otro registro. Los registros de origen y destino pueden estar en la misma suscripción o en distintas suscripciones, inquilinos de Active Directory, nubes de Azure o nubes físicamente desconectadas. 

Para transferir artefactos, cree una *canalización de transferencia* que replique artefactos entre dos registros mediante [Blob Storage](../storage/blobs/storage-blobs-introduction.md):

* Los artefactos de un registro de origen se exportan a un blob en una cuenta de almacenamiento de origen 
* El blob se copia de la cuenta de almacenamiento de origen a una cuenta de almacenamiento de destino
* El blob de la cuenta de almacenamiento de destino se importa como artefactos en el registro de destino. Puede configurar la canalización de importación para que se desencadene siempre que el blob del artefacto se actualice en el almacenamiento de destino.

La transferencia es ideal para copiar contenido entre dos registros de contenedor de Azure en nubes físicamente desconectadas, administrada por las cuentas de almacenamiento en cada nube. Si, en su lugar, desea copiar imágenes de los registros de contenedor en nubes conectadas, incluido Docker Hub y otros proveedores en la nube, se recomienda la [importación de imágenes](container-registry-import-images.md).

En este artículo, usará implementaciones de plantilla de Azure Resource Manager para crear y ejecutar la canalización de transferencia. La CLI de Azure se usa para aprovisionar los recursos asociados, como los secretos de almacenamiento. Se recomienda la CLI de Azure versión 2.2.0 o posterior. Si necesita instalarla o actualizarla, consulte [Instalación de la CLI de Azure][azure-cli].

Esta característica está disponible en el nivel de servicio de un registro de contenedor **Premium**. Para obtener información sobre los límites y niveles de servicio de registro, consulte [Niveles de Azure Container Registry](container-registry-skus.md).

> [!IMPORTANT]
> Esta funcionalidad actualmente está en su versión preliminar. Las versiones preliminares están a su disposición con la condición de que acepte los [términos de uso adicionales][terms-of-use]. Es posible que algunos de los aspectos de esta característica cambien antes de ofrecer disponibilidad general.

## <a name="prerequisites"></a>Requisitos previos

* **Registros de contenedor**: necesita un registro de origen existente con artefactos para transferir y un registro de destino. La transferencia de ACR está pensada para el movimiento entre nubes físicamente desconectadas. Para las pruebas, los registros de origen y de destino pueden estar en la misma o en otra suscripción de Azure, inquilino de Active Directory o nube. 

   Si necesita crear un registro, consulte [Inicio rápido: Creación de un registro de contenedor privado con la CLI de Azure](container-registry-get-started-azure-cli.md). 
* **Cuentas de almacenamiento**: cree cuentas de almacenamiento de origen y de destino en una suscripción y ubicación de su elección. Con fines de prueba, puede usar la misma suscripción o suscripciones que los registros de origen y de destino. En el caso de escenarios entre nubes, normalmente se crea una cuenta de almacenamiento independiente en cada nube. 

  Si es necesario, cree las cuentas de almacenamiento con la [CLI de Azure](../storage/common/storage-account-create.md?tabs=azure-cli) u otras herramientas. 

  Cree un contenedor de blobs para la transferencia de artefactos en cada cuenta. Por ejemplo, cree un contenedor denominado *transfer*. Dos o más canalizaciones de transferencia pueden compartir la misma cuenta de almacenamiento, pero deben usar diferentes ámbitos de contenedor de almacenamiento.
* **Almacenes de claves**: los almacenes de claves son necesarios para almacenar los secretos de token de SAS que se usan para tener acceso a las cuentas de almacenamiento de origen y de destino. Cree los almacenes de claves de origen y de destino en la misma suscripción o suscripciones de Azure que los registros de origen y de destino. Para demostraciones, las plantillas y los comandos que se usan en este artículo también suponen que los almacenes de claves de origen y de destino se encuentran en los mismos grupos de recursos que los registros de origen y de destino, respectivamente. Este uso de grupos de recursos comunes no es necesario, pero simplifica las plantillas y los comandos que se usan en este artículo.

   Si es necesario, cree los almacenes de claves con la [CLI de Azure](../key-vault/secrets/quick-create-cli.md) u otras herramientas.

* **Variables de entorno**: por ejemplo, los comandos de este artículo establecen las siguientes variables de entorno para los entornos de origen y de destino. Todos los ejemplos tienen el formato del shell de Bash.
  ```console
  SOURCE_RG="<source-resource-group>"
  TARGET_RG="<target-resource-group>"
  SOURCE_KV="<source-key-vault>"
  TARGET_KV="<target-key-vault>"
  SOURCE_SA="<source-storage-account>"
  TARGET_SA="<target-storage-account>"
  ```

## <a name="scenario-overview"></a>Información general de escenario

Puede crear los siguientes tres recursos de canalización para la transferencia de imágenes entre los registros. Todos se crean mediante las operaciones PUT. Estos recursos operan en los registros y cuentas de almacenamiento de *origen* y *destino*. 

La autenticación de almacenamiento usa tokens de SAS, administrados como secretos en los almacenes de claves. Las canalizaciones usan identidades administradas para leer los secretos de los almacenes.

* **[ExportPipeline](#create-exportpipeline-with-resource-manager)** : recurso duradero que contiene información de alto nivel sobre el registro y la cuenta de almacenamiento de *origen*. Esta información incluye el URI del contenedor de blobs de almacenamiento de origen y el almacén de claves que administra el token de SAS de origen. 
* **[ImportPipeline](#create-importpipeline-with-resource-manager)** : recurso duradero que contiene información de alto nivel sobre el registro y la cuenta de almacenamiento de *destino*. Esta información incluye el URI del contenedor de blobs de almacenamiento de destino y el almacén de claves que administra el token de SAS de destino. Un desencadenador de importación está habilitado de forma predeterminada, por lo que la canalización se ejecuta automáticamente cuando un blob de artefacto llega al contenedor de almacenamiento de destino. 
* **[PipelineRun](#create-pipelinerun-for-export-with-resource-manager)** : recurso que se usa para invocar un recurso ExportPipeline o ImportPipeline.  
  * Para ejecutar el ExportPipeline manualmente, cree un recurso PipelineRun y especifique los artefactos que se van a exportar.  
  * Si se habilita un desencadenador de importación, el ImportPipeline se ejecuta automáticamente. También se puede ejecutar manualmente mediante un PipelineRun. 
  * Actualmente, un máximo de **50 artefactos** se pueden transferir con cada PipelineRun.

### <a name="things-to-know"></a>Cosas que debe saber
* ExportPipeline e ImportPipeline normalmente estarán en distintos inquilinos de Active Directory asociados a las nubes de origen y de destino. Este escenario requiere identidades administradas independientes y almacenes de claves para los recursos de exportación e importación. Con fines de prueba, estos recursos se pueden colocar en la misma nube, compartiendo identidades.
* De forma predeterminada, las plantillas ExportPipeline e ImportPipeline permiten a una identidad administrada asignada por el sistema acceder a los secretos del almacén de claves. Las plantillas ExportPipeline e ImportPipeline también admiten la identidad asignada por el usuario que proporcione. 

## <a name="create-and-store-sas-keys"></a>Creación y almacenamiento de claves SAS

La transferencia usa tokens de firma de acceso compartido (SAS) para tener acceso a las cuentas de almacenamiento en los entornos de origen y de destino. Genere y almacene los tokens tal como se describe en las secciones siguientes.  

### <a name="generate-sas-token-for-export"></a>Generar el token de SAS para la exportación

Ejecute el comando [az storage container generate-sas][az-storage-container-generate-sas] para generar un token de SAS para el contenedor en la cuenta de almacenamiento de origen, que se usa para la exportación de artefactos.

*Permisos de token recomendados*: Lectura, escritura, lista, adición. 

En el ejemplo siguiente, la salida del comando se asigna a la variable de entorno EXPORT_SAS, precedida del carácter "? ". Actualice el valor `--expiry` del entorno:

```azurecli
EXPORT_SAS=?$(az storage container generate-sas \
  --name transfer \
  --account-name $SOURCE_SA \
  --expiry 2021-01-01 \
  --permissions alrw \
  --https-only \
  --output tsv)
```

### <a name="store-sas-token-for-export"></a>Almacenar el token de SAS para la exportación

Almacene el token de SAS en el almacén de claves de Azure de origen con [az keyvault secret set][az-keyvault-secret-set]:

```azurecli
az keyvault secret set \
  --name acrexportsas \
  --value $EXPORT_SAS \
  --vault-name $SOURCE_KV 
```

### <a name="generate-sas-token-for-import"></a>Generar el token de SAS para la importación

Ejecute el comando [az storage container generate-sas][az-storage-container-generate-sas] para generar un token de SAS para el contenedor en la cuenta de almacenamiento de destino, que se usa para la importación de artefactos.

*Permisos de token recomendados*: Lectura, eliminación, lista

En el ejemplo siguiente, la salida del comando se asigna a la variable de entorno IMPORT_SAS, precedida del carácter "? ". Actualice el valor `--expiry` del entorno:

```azurecli
IMPORT_SAS=?$(az storage container generate-sas \
  --name transfer \
  --account-name $TARGET_SA \
  --expiry 2021-01-01 \
  --permissions dlr \
  --https-only \
  --output tsv)
```

### <a name="store-sas-token-for-import"></a>Almacenar el token de SAS para la importación

Almacene el token de SAS en el almacén de claves de Azure de destino con [az keyvault secret set][az-keyvault-secret-set]:

```azurecli
az keyvault secret set \
  --name acrimportsas \
  --value $IMPORT_SAS \
  --vault-name $TARGET_KV 
```

## <a name="create-exportpipeline-with-resource-manager"></a>Crear ExportPipeline con Resource Manager

Cree un recurso ExportPipeline para el registro de contenedor de origen mediante la implementación de plantillas de Azure Resource Manager.

Copie los [archivos de plantilla](https://github.com/Azure/acr/tree/master/docs/image-transfer/ExportPipelines) de Resource Manager del recurso ExportPipeline en una carpeta local.

Escriba los siguientes valores de parámetro en el archivo `azuredeploy.parameters.json`:

|Parámetro  |Value  |
|---------|---------|
|registryName     | Nombre del registro de contenedor de origen      |
|exportPipelineName     |  Nombre que elija para la canalización de exportación       |
|targetUri     |  URI del contenedor de almacenamiento en el entorno de origen (el destino de la canalización de exportación).<br/>Ejemplo: `https://sourcestorage.blob.core.windows.net/transfer`       |
|keyVaultName     |  Nombre del almacén de claves de origen  |
|sasTokenSecretName  | Nombre del secreto del token de SAS en el almacén de claves de origen <br/>Ejemplo: acrexportsas

### <a name="export-options"></a>Opciones de exportación

La propiedad `options` para las canalizaciones de exportación admite valores booleanos opcionales. Se recomiendan los siguientes valores:

|Parámetro  |Value  |
|---------|---------|
|opciones | OverwriteBlobs: sobrescribe los blobs de destino existentes<br/>ContinueOnErrors: continúa la exportación de artefactos restantes en el registro de origen si se produce un error en la exportación de un artefacto.

### <a name="create-the-resource"></a>Crear el recurso

Ejecute [az deployment group create][az-deployment-group-create] para crear un recurso denominado *exportPipeline*, tal como se muestra en estos ejemplos. De forma predeterminada, con la primera opción, la plantilla de ejemplo habilita una identidad asignada por el sistema en el recurso ExportPipeline. 

Con la segunda opción, puede proporcionar el recurso con una identidad asignada por el usuario. (No se muestra la creación de la identidad asignada por el usuario).

Con cualquiera de las dos opciones, la plantilla configura la identidad para tener acceso al token de SAS en el almacén de claves de exportación. 

#### <a name="option-1-create-resource-and-enable-system-assigned-identity"></a>Opción 1: Crear un recurso y habilitar una identidad asignada por el sistema

```azurecli
az deployment group create \
  --resource-group $SOURCE_RG \
  --template-file azuredeploy.json \
  --name exportPipeline \
  --parameters azuredeploy.parameters.json
```

#### <a name="option-2-create-resource-and-provide-user-assigned-identity"></a>Opción 2: Crear un recurso y proporcionar una identidad asignada por el usuario

En este comando, proporcione el identificador de recurso de la identidad asignada por el usuario como parámetro adicional.

```azurecli
az deployment group create \
  --resource-group $SOURCE_RG \
  --template-file azuredeploy.json \
  --name exportPipeline \
  --parameters azuredeploy.parameters.json \
  --parameters userAssignedIdentity="/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/myUserAssignedIdentity"
```

En la salida del comando, anote el identificador de recurso (`id`) de la canalización. Puede almacenar este valor en una variable de entorno para su uso posterior si ejecuta [az deployment group show][az-deployment-group-show]. Por ejemplo:

```azurecli
EXPORT_RES_ID=$(az deployment group show \
  --resource-group $SOURCE_RG \
  --name exportPipeline \
  --query 'properties.outputResources[1].id' \
  --output tsv)
```

## <a name="create-importpipeline-with-resource-manager"></a>Crear ImportPipeline con Resource Manager 

Cree un recurso ImportPipeline en el registro de contenedor de destino mediante la implementación de plantillas de Azure Resource Manager. De forma predeterminada, la canalización está habilitada para importar automáticamente cuando la cuenta de almacenamiento en el entorno de destino tiene un blob de artefacto.

Copie los [archivos de plantilla](https://github.com/Azure/acr/tree/master/docs/image-transfer/ImportPipelines) de Resource Manager del recurso ImportPipeline en una carpeta local.

Escriba los siguientes valores de parámetro en el archivo `azuredeploy.parameters.json`:

Parámetro  |Value  |
|---------|---------|
|registryName     | Nombre del registro de contenedor de destino      |
|importPipelineName     |  Nombre que elija para la canalización de importación       |
|SourceUri     |  URI del contenedor de almacenamiento en el entorno de destino (el origen de la canalización de importación).<br/>Ejemplo: `https://targetstorage.blob.core.windows.net/transfer/`|
|keyVaultName     |  Nombre del almacén de claves de destino |
|sasTokenSecretName     |  Nombre del secreto del token de SAS en el almacén de claves de destino<br/>Ejemplo: acr importsas |

### <a name="import-options"></a>Opciones de importación

La propiedad `options` para las canalizaciones de importación admite valores booleanos opcionales. Se recomiendan los siguientes valores:

|Parámetro  |Value  |
|---------|---------|
|opciones | OverwriteTags: sobrescribe las etiquetas de destino existentes<br/>DeleteSourceBlobOnSuccess: elimina el blob de almacenamiento de origen después de la importación correcta en el registro de destino<br/>ContinueOnErrors: continúa la importación de artefactos restantes en el registro de destino si se produce un error en la importación de un artefacto.

### <a name="create-the-resource"></a>Crear el recurso

Ejecute [az deployment group create][az-deployment-group-create] para crear un recurso denominado *importPipeline*, tal como se muestra en estos ejemplos. De forma predeterminada, con la primera opción, la plantilla de ejemplo habilita una identidad asignada por el sistema en el recurso ImportPipeline. 

Con la segunda opción, puede proporcionar el recurso con una identidad asignada por el usuario. (No se muestra la creación de la identidad asignada por el usuario).

Con cualquiera de las dos opciones, la plantilla configura la identidad para tener acceso al token de SAS en el almacén de claves de importación. 

#### <a name="option-1-create-resource-and-enable-system-assigned-identity"></a>Opción 1: Crear un recurso y habilitar una identidad asignada por el sistema

```azurecli
az deployment group create \
  --resource-group $TARGET_RG \
  --template-file azuredeploy.json \
  --name importPipeline \
  --parameters azuredeploy.parameters.json 
```

#### <a name="option-2-create-resource-and-provide-user-assigned-identity"></a>Opción 2: Crear un recurso y proporcionar una identidad asignada por el usuario

En este comando, proporcione el identificador de recurso de la identidad asignada por el usuario como parámetro adicional.

```azurecli
az deployment group create \
  --resource-group $TARGET_RG \
  --template-file azuredeploy.json \
  --name importPipeline \
  --parameters azuredeploy.parameters.json \
  --parameters userAssignedIdentity="/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/myUserAssignedIdentity"
```

Si tiene previsto ejecutar la importación manualmente, anote el identificador de recurso (`id`) de la canalización. Puede almacenar este valor en una variable de entorno para su uso posterior si ejecuta el comando [az deployment group show][az-deployment-group-show]. Por ejemplo:

```azurecli
IMPORT_RES_ID=$(az deployment group show \
  --resource-group $TARGET_RG \
  --name importPipeline \
  --query 'properties.outputResources[1].id' \
  --output tsv)
```

## <a name="create-pipelinerun-for-export-with-resource-manager"></a>Crear PipelineRun para la exportación con Resource Manager 

Cree un recurso PipelineRun para el registro de contenedor de origen mediante la implementación de plantillas de Azure Resource Manager. Este recurso ejecuta el recurso ExportPipeline que creó anteriormente y exporta los artefactos especificados desde el registro de contenedor como un blob a la cuenta de almacenamiento de origen.

Copie los [archivos de plantilla](https://github.com/Azure/acr/tree/master/docs/image-transfer/PipelineRun/PipelineRun-Export) de Resource Manager del recurso PipelineRun en una carpeta local.

Escriba los siguientes valores de parámetro en el archivo `azuredeploy.parameters.json`:

|Parámetro  |Value  |
|---------|---------|
|registryName     | Nombre del registro de contenedor de origen      |
|pipelineRunName     |  Nombre que elija para la ejecución       |
|pipelineResourceId     |  Identificador de recurso de la canalización de exportación.<br/>Ejemplo: `/subscriptions/<subscriptionID>/resourceGroups/<resourceGroupName>/providers/Microsoft.ContainerRegistry/registries/<sourceRegistryName>/exportPipelines/myExportPipeline`|
|targetName     |  Nombre que elija para el blob de artefactos exportado a la cuenta de almacenamiento de origen, como *myblob*
|artifacts | Matriz de artefactos de origen que se van a transferir, como etiquetas o resúmenes de manifiestos<br/>Ejemplo: `[samples/hello-world:v1", "samples/nginx:v1" , "myrepository@sha256:0a2e01852872..."]` |

Si vuelve a implementar un recurso PipelineRun con propiedades idénticas, también debe usar la propiedad [forceUpdateTag](#redeploy-pipelinerun-resource).

Ejecute [az deployment group create][az-deployment-group-create] para crear el recurso PipelineRun. En el ejemplo siguiente se nombra a la implementación como *exportPipelineRun*.

```azurecli
az deployment group create \
  --resource-group $SOURCE_RG \
  --template-file azuredeploy.json \
  --name exportPipelineRun \
  --parameters azuredeploy.parameters.json
```

Para su uso posterior, guarde el identificador de recurso de la ejecución de canalización en una variable de entorno:

```azurecli
EXPORT_RUN_RES_ID=$(az deployment group show \
  --resource-group $SOURCE_RG \
  --name exportPipelineRun \
  --query 'properties.outputResources[0].id' \
  --output tsv)
```

Los artefactos pueden tardar varios minutos en exportarse. Cuando la implementación se complete correctamente, compruebe la exportación de artefactos; para ello, registre el blob exportado en el contenedor *transfer* de la cuenta de almacenamiento de origen. Por ejemplo, ejecute el comando [az storage blob list][az-storage-blob-list]:

```azurecli
az storage blob list \
  --account-name $SOURCE_SA \
  --container transfer \
  --output table
```

## <a name="transfer-blob-optional"></a>Transferir blob (opcional) 

Use la herramienta AzCopy u otros métodos para [transferir datos de blob](../storage/common/storage-use-azcopy-v10.md#transfer-data) de la cuenta de almacenamiento de origen a la cuenta de almacenamiento de destino.

Por ejemplo, el siguiente [`azcopy copy`](../storage/common/storage-ref-azcopy-copy.md) comando copia myblob del contenedor *transfer* de la cuenta de origen en el contenedor *transfer* de la cuenta de destino. Si el blob existe en la cuenta de destino, se sobrescribe. La autenticación usa tokens de SAS con los permisos adecuados para los contenedores de origen y de destino. (No se muestran los pasos para crear tokens).

```console
azcopy copy \
  'https://<source-storage-account-name>.blob.core.windows.net/transfer/myblob'$SOURCE_SAS \
  'https://<destination-storage-account-name>.blob.core.windows.net/transfer/myblob'$TARGET_SAS \
  --overwrite true
```

## <a name="trigger-importpipeline-resource"></a>Desencadenar recurso ImportPipeline

Si ha habilitado el parámetro `sourceTriggerStatus` de ImportPipeline (el valor predeterminado), la canalización se desencadena después de que el blob se copia en la cuenta de almacenamiento de destino. Los artefactos pueden tardar varios minutos en importarse. Cuando la importación finalice correctamente, compruebe la importación de artefactos; para ello, registre los repositorios en el registro de contenedor de destino. Por ejemplo, ejecute [az acr repository list][az-acr-repository-list]:

```azurecli
az acr repository list --name <target-registry-name>
```

Si no ha habilitado el parámetro `sourceTriggerStatus` de la canalización de importación, ejecute el recurso ImportPipeline manualmente, tal como se muestra en la sección siguiente. 

## <a name="create-pipelinerun-for-import-with-resource-manager-optional"></a>Crear PipelineRun para la importación con Resource Manager (opcional) 
 
También puede usar un recurso PipelineRun para desencadenar un ImportPipeline para la importación de artefactos en el registro de contenedor de destino.

Copie los [archivos de plantilla](https://github.com/Azure/acr/tree/master/docs/image-transfer/PipelineRun/PipelineRun-Import) de Resource Manager del recurso PipelineRun en una carpeta local.

Escriba los siguientes valores de parámetro en el archivo `azuredeploy.parameters.json`:

|Parámetro  |Value  |
|---------|---------|
|registryName     | Nombre del registro de contenedor de destino      |
|pipelineRunName     |  Nombre que elija para la ejecución       |
|pipelineResourceId     |  Identificador de recurso de la canalización de importación.<br/>Ejemplo: `/subscriptions/<subscriptionID>/resourceGroups/<resourceGroupName>/providers/Microsoft.ContainerRegistry/registries/<sourceRegistryName>/importPipelines/myImportPipeline`       |
|sourceName     |  Nombre del blob existente para los artefactos exportados en la cuenta de almacenamiento, como *myblob*

Si vuelve a implementar un recurso PipelineRun con propiedades idénticas, también debe usar la propiedad [forceUpdateTag](#redeploy-pipelinerun-resource).

Ejecute [az deployment group create][az-deployment-group-create] para ejecutar el recurso.

```azurecli
az deployment group create \
  --resource-group $TARGET_RG \
  --name importPipelineRun \
  --template-file azuredeploy.json \
  --parameters azuredeploy.parameters.json
```

Para su uso posterior, guarde el identificador de recurso de la ejecución de canalización en una variable de entorno:

```azurecli
IMPORT_RUN_RES_ID=$(az deployment group show \
  --resource-group $TARGET_RG \
  --name importPipelineRun \
  --query 'properties.outputResources[0].id' \
  --output tsv)
```

Cuando la implementación finalice correctamente, compruebe la importación de artefactos; para ello, registre los repositorios en el registro de contenedor de destino. Por ejemplo, ejecute [az acr repository list][az-acr-repository-list]:

```azurecli
az acr repository list --name <target-registry-name>
```

## <a name="redeploy-pipelinerun-resource"></a>Nueva implementación del recurso PipelineRun

Si vuelve a implementar un recurso PipelineRun con *propiedades idénticas*, debe aprovechar la propiedad **forceUpdateTag**. Esta propiedad indica que el recurso PipelineRun debe volver a crearse incluso si la configuración no ha cambiado. Asegúrese de que forceUpdateTag sea diferente cada vez que vuelva a implementar el recurso PipelineRun. En el ejemplo siguiente se vuelve a crear un recurso PipelineRun para exportarlo. La fecha y hora actual se utiliza para establecer forceUpdateTag, lo que garantiza que esta propiedad sea siempre única.

```console
CURRENT_DATETIME=`date +"%Y-%m-%d:%T"`
```

```azurecli
az deployment group create \
  --resource-group $SOURCE_RG \
  --template-file azuredeploy.json \
  --name exportPipelineRun \
  --parameters azuredeploy.parameters.json \
  --parameters forceUpdateTag=$CURRENT_DATETIME
```

## <a name="delete-pipeline-resources"></a>Eliminar recursos de canalización

En el siguiente ejemplo, los comandos usan [az resource delete][az-resource-delete] para suprimir los recursos de canalización creados en este artículo. Los identificadores de recursos se almacenaron previamente en variables de entorno.

```
# Delete export resources
az resource delete \
--resource-group $SOURCE_RG \
--ids $EXPORT_RES_ID $EXPORT_RUN_RES_ID \
--api-version 2019-12-01-preview

# Delete import resources
az resource delete \
--resource-group $TARGET_RG \
--ids $IMPORT_RES_ID $IMPORT_RUN_RES_ID \
--api-version 2019-12-01-preview
```

## <a name="troubleshooting"></a>Solución de problemas

* **Errores o fallos de la implementación de plantillas**
  * Si se produce un error en una ejecución de canalización, examine la propiedad `pipelineRunErrorMessage` del recurso de ejecución.
  * Para ver los errores comunes de la implementación de plantillas, consulte [Solución de problemas de las implementaciones de plantillas de Resource Manager](../azure-resource-manager/templates/template-tutorial-troubleshoot.md)
* **Problemas al acceder al almacenamiento**<a name="problems-accessing-storage"></a>
  * Si ve un error `403 Forbidden` del almacenamiento, es probable que tenga un problema con el token de SAS.
  * Es posible que el token de SAS no sea válido. Es posible que el token de SAS haya expirado o que las claves de la cuenta de almacenamiento hayan cambiado desde que se creó el token de SAS. Compruebe que el token de SAS es válido, para lo que debe intentar usarlo para autenticarse para acceder al contenedor de la cuenta de almacenamiento. Por ejemplo, coloque un punto de conexión de blob existente, seguido del token de SAS, en la barra de direcciones de una nueva ventana InPrivate de Microsoft Edge, o bien cargue un blob en el contenedor con el token de SAS mediante `az storage blob upload`.
  * Es posible que el token de SAS no tenga suficientes tipos de recursos permitidos. Compruebe que se han concedido permisos al token de SAS para Servicio, Contenedor y Objeto en Tipos de recursos permitidos (`srt=sco` en el token de SAS).
  * Es posible que el token de SAS no tenga suficientes permisos. En el caso de las canalizaciones de exportación, los permisos de token de SAS necesarios son Lectura, Escritura, Lista y Agregar. En el caso de las canalizaciones de importación, los permisos de token de SAS necesarios son Lectura, Eliminación y Lista (el permiso de eliminación solo es necesario si la canalización de importación tiene la opción `DeleteSourceBlobOnSuccess` habilitada).
  * Es posible que el token de SAS no esté configurado para funcionar solo con HTTPS. Compruebe que el token de SAS está configurado para funcionar solo con HTTPS (`spr=https` en el token de SAS).
* **Problemas con la exportación o importación de blobs de almacenamiento**
  * Puede que el token de SAS no sea válido o que no tenga permisos suficientes para la ejecución de exportación o importación especificada. Consulte [Problemas al acceder al almacenamiento](#problems-accessing-storage).
  * El blob de almacenamiento existente en la cuenta de almacenamiento de origen no se puede sobrescribir durante varias ejecuciones de exportación. Confirme que la opción OverwriteBlob está establecida en la ejecución de exportación y que el token de SAS tiene permisos suficientes.
  * No se puede eliminar el blob de almacenamiento de la cuenta de almacenamiento de destino después de la ejecución correcta de la importación. Confirme que la opción DeleteBlobOnSuccess está establecida en la ejecución de importación y que el token de SAS tiene permisos suficientes.
  * Blob de almacenamiento no creado o eliminado. Confirme que el contenedor especificado en la ejecución de exportación o importación existe o que el blob de almacenamiento especificado para la ejecución de importación manual existe. 
* **Problemas de AzCopy**
  * Consulte [Solución de problemas de AzCopy](../storage/common/storage-use-azcopy-configure.md).  
* **Problemas de transferencia de artefactos**
  * No se transfieren todos los artefactos o ninguno. Confirme la ortografía de los artefactos en la ejecución de la exportación y el nombre del blob en las ejecuciones de exportación e importación. Confirme que está transfiriendo un máximo de 50 artefactos.
  * Es posible que no se haya completado la ejecución de canalización. Una ejecución de exportación o importación puede tardar algún tiempo. 
  * Para otros problemas de canalización, proporcione el [id. de correlación](../azure-resource-manager/templates/deployment-history.md) de implementación de la ejecución de la exportación o la ejecución de la importación al equipo de Azure Container Registry.
* **Problemas al extraer la imagen en un entorno físicamente aislado**
  * Si ve errores relacionados con las capas externas o intenta resolver mcr.microsoft.com al intentar extraer una imagen en un entorno físicamente aislado, es probable que el manifiesto de imagen tenga capas no redistribuibles. Debido a la naturaleza de un entorno físicamente aislado, estas imágenes no se pueden extraer a menudo. Puede confirmar que este es el caso comprobando el manifiesto de la imagen para las referencias a registros externos. En este caso, tendrá que enviar las capas no redistribuibles al ACR de la nube pública antes de implementar una canalización de exportación: ejecute la imagen. Para obtener instrucciones sobre cómo hacerlo, consulte [¿Cómo enviar capas no redistribuibles a un registro?](./container-registry-faq.md#how-do-i-push-non-distributable-layers-to-a-registry)

## <a name="next-steps"></a>Pasos siguientes

Para importar imágenes de contenedor único a un registro de contenedor de Azure desde un registro público u otro registro privado, consulte la referencia del comando [az acr import][az-acr-import].

<!-- LINKS - External -->
[terms-of-use]: https://azure.microsoft.com/support/legal/preview-supplemental-terms/



<!-- LINKS - Internal -->
[azure-cli]: /cli/azure/install-azure-cli
[az-login]: /cli/azure/reference-index#az_login
[az-keyvault-secret-set]: /cli/azure/keyvault/secret#az_keyvault_secret_set
[az-keyvault-secret-show]: /cli/azure/keyvault/secret#az_keyvault_secret_show
[az-keyvault-set-policy]: /cli/azure/keyvault#az_keyvault_set_policy
[az-storage-container-generate-sas]: /cli/azure/storage/container#az_storage_container_generate_sas
[az-storage-blob-list]: /cli/azure/storage/blob#az_storage-blob-list
[az-deployment-group-create]: /cli/azure/deployment/group#az_deployment_group_create
[az-deployment-group-delete]: /cli/azure/deployment/group#az_deployment_group_delete
[az-deployment-group-show]: /cli/azure/deployment/group#az_deployment_group_show
[az-acr-repository-list]: /cli/azure/acr/repository#az_acr_repository_list
[az-acr-import]: /cli/azure/acr#az_acr_import
[az-resource-delete]: /cli/azure/resource#az_resource_delete
