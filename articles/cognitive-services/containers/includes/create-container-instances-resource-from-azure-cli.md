---
title: Compatibilidad con los contenedores
titleSuffix: Azure Cognitive Services
description: Aprenda cómo crear un recurso de instancia de contenedor de Azure con la CLI de Azure.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 04/01/2020
ms.author: aahi
ms.openlocfilehash: 32b00e031e3cf865093c267084117a8704b6e272
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107800173"
---
## <a name="create-an-azure-container-instance-resource-from-the-azure-cli"></a>Creación de un recurso de instancia de contenedor de Azure con la CLI de Azure

El siguiente código YAML define el recurso de instancia de contenedor de Azure. Copie y pegue el contenido en un nuevo archivo llamado `my-aci.yaml` y reemplace los valores comentados por los suyos propios. Consulte el [formato de plantilla][template-format] para ver el código YAML válido. Consulte las [imágenes y los repositorios de contenedor][repositories-and-images] para ver los nombres de imagen disponibles y su repositorio correspondiente. Para más información sobre la referencia de YAML para las instancias de contenedor, consulte [Referencia de YAML: Azure Container Instances][aci-yaml-ref].

```YAML
apiVersion: 2018-10-01
location: # < Valid location >
name: # < Container Group name >
properties:
  imageRegistryCredentials: # This is only required if you are pulling a non-public image that requires authentication to access. For example Text Analytics for health.
  - server: containerpreview.azurecr.io
    username: # < The username for the preview container registry >
    password: # < The password for the preview container registry >
  containers:
  - name: # < Container name >
    properties:
      image: # < Repository/Image name >
      environmentVariables: # These env vars are required
        - name: eula
          value: accept
        - name: billing
          value: # < Service specific Endpoint URL >
        - name: apikey
          value: # < Service specific API key >
      resources:
        requests:
          cpu: 4 # Always refer to recommended minimal resources
          memoryInGb: 8 # Always refer to recommended minimal resources
      ports:
        - port: 5000
  osType: Linux
  volumes: # This node, is only required for container instances that pull their model in at runtime, such as LUIS.
  - name: aci-file-share
    azureFile:
      shareName: # < File share name >
      storageAccountName: # < Storage account name>
      storageAccountKey: # < Storage account key >
  restartPolicy: OnFailure
  ipAddress:
    type: Public
    ports:
    - protocol: tcp
      port: 5000
tags: null
type: Microsoft.ContainerInstance/containerGroups
```

> [!NOTE]
> No todas las ubicaciones tienen la misma disponibilidad de CPU y memoria. Consulte la tabla [ubicación y recursos][location-to-resource] para obtener una lista de los recursos disponibles para los contenedores por ubicación y sistema operativo.

Nos basaremos en el archivo YAML que hemos creado para el comando [`az container create`][azure-container-create]. En la CLI de Azure, ejecute el comando `az container create` reemplazando `<resource-group>` por el suyo propio. Además, para proteger los valores dentro de una implementación de YAML, consulte cómo [proteger los valores][secure-values].

```azurecli
az container create -g <resource-group> -f my-aci.yaml
```

La salida del comando es `Running...` si es válido. Después de unos minutos, la salida cambia a una cadena JSON que representa el recurso ACI recién creado. Es muy probable que la imagen de contenedor no esté disponible durante un tiempo, pero el recurso ya está implementado.

> [!TIP]
> Preste mucha atención a las ubicaciones de las ofertas de la versión preliminar pública de Azure Cognitive Services, ya que el código YAML tendrá que ajustarse según la ubicación.

[azure-container-create]: /cli/azure/container#az_container_create
[template-format]: /azure/templates/Microsoft.ContainerInstance/2018-10-01/containerGroups#template-format
[aci-yaml-ref]: ../../../container-instances/container-instances-reference-yaml.md
[repositories-and-images]: ../container-image-tags.md
[location-to-resource]: ../../../container-instances/container-instances-region-availability.md
[secure-values]: ../../../container-instances/container-instances-environment-variables.md#secure-values
