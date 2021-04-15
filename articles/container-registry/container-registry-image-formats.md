---
title: Formatos de contenido compatibles
description: Obtenga información sobre los formatos de contenido que admite Azure Container Registry, como imágenes de contenedor compatibles con Docker, gráficos de Helm, e imágenes y artefactos OCI.
ms.topic: article
ms.date: 03/02/2021
ms.openlocfilehash: 218d98f3f16e8d0ca76a24692afbb2b69606564b
ms.sourcegitcommit: 3f684a803cd0ccd6f0fb1b87744644a45ace750d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/02/2021
ms.locfileid: "106223071"
---
# <a name="content-formats-supported-in-azure-container-registry"></a>Formatos de contenido admitidos en Azure Container Registry

Use un repositorio privado en Azure Container Registry para administrar uno de los siguientes formatos de contenido. 

## <a name="docker-compatible-container-images"></a>Imágenes de contenedor compatibles con Docker

Se admiten los siguientes formatos de imagen de contenedor de Docker:

* [Manifiesto de imagen de Docker V2, esquema 1](https://docs.docker.com/registry/spec/manifest-v2-1/)

* [Manifiesto de imagen de Docker V2, esquema 2](https://docs.docker.com/registry/spec/manifest-v2-2/): incluye listas de manifiesto que permiten que los registros almacenen [imágenes con varias arquitecturas](push-multi-architecture-images.md) en una referencia `image:tag` única.

## <a name="oci-images"></a>Imágenes OCI

Azure Container Registry admite imágenes que cumplen la [especificación de formato de imagen de Open Container Initiative (OCI)](https://github.com/opencontainers/image-spec/blob/master/spec.md) que incluyen la especificación de [índice de imágenes](https://github.com/opencontainers/image-spec/blob/master/image-index.md) opcional. Entre los formatos de empaquetado se incluye [Singularity Image Format (SIF)](https://github.com/sylabs/sif).

## <a name="oci-artifacts"></a>Artefactos de OCI

Azure Container Registry admite la [especificación de distribución OCI](https://github.com/opencontainers/distribution-spec), una especificación independiente de la nube y del proveedor para almacenar, compartir, proteger e implementar imágenes de contenedor y otros tipos de contenido (artefactos). La especificación permite que un registro almacene una amplia variedad de artefactos además de imágenes de contenedor. Use las herramientas adecuadas para el artefacto para insertar y extraer artefactos. Para ver un ejemplo, consulte [Inserción y extracción de un artefacto de OCI con un registro de contenedor de Azure](container-registry-oci-artifacts.md).

Para más información sobre los artefactos de OCI, consulte los repositorios [OCI Registry as Storage (ORAS)](https://github.com/deislabs/oras) y [OCI Artifacts](https://github.com/opencontainers/artifacts) en GitHub.

## <a name="helm-charts"></a>Gráficos de Helm

Azure Container Registry también puede hospedar repositorios para [gráficos de Helm](https://helm.sh/), un formato de empaquetado que se usa para administrar e implementar rápidamente aplicaciones de Kubernetes. Se recomienda el [cliente Helm](https://docs.helm.sh/using_helm/#installing-helm) versión 3. Consulte [Inserción y extracción de gráficos de Helm en Azure Container Registry](container-registry-helm-repos.md).

## <a name="next-steps"></a>Pasos siguientes

* Vea cómo [insertar y extraer](container-registry-get-started-docker-cli.md) imágenes con Azure Container Registry.

* Use [ACR Tasks](container-registry-tasks-overview.md) para compilar y probar imágenes de contenedor. 

* Use [Moby BuildKit](https://github.com/moby/buildkit) para compilar y empaquetar contenedores en formato OCI.

* Configure un [repositorio de Helm](container-registry-helm-repos.md) hospedado en Azure Container Registry. 


