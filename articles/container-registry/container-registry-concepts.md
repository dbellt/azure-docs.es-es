---
title: Acerca de los registros, repositorios, artefactos e imágenes
description: Introducción a los conceptos clave de los registros de contenedor, repositorios e imágenes de contenedor y otros artefactos de Azure.
ms.topic: article
ms.date: 01/29/2021
ms.openlocfilehash: 64ab3812b3f23a7b3a480d3530c82bd39f2d29a5
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107784090"
---
# <a name="about-registries-repositories-and-artifacts"></a>Acerca de los registros, repositorios y artefactos

En este artículo se presentan los conceptos básicos de los registros de contenedor, los repositorios y las imágenes de contenedor, así como los artefactos relacionados. 

:::image type="content" source="media/container-registry-concepts/registry-elements.png" alt-text="Registros, repositorios y artefactos":::

## <a name="registry"></a>Registro

Un *registro* de contenedor es un servicio que almacena y distribuye no solo imágenes de contenedor, sino también los artefactos relacionados. Docker Hub es un ejemplo de un registro de contenedor público que sirve como catálogo general de las imágenes de contenedor de Docker. Azure Container Registry proporciona a los usuarios control directo del contenido de contenedor, con autenticación integrada, [replicación geográfica](container-registry-geo-replication.md) que admite la distribución global y confiabilidad para implementaciones cercanas a la red, la [configuración virtual de red y firewall](container-registry-private-link.md), el [bloqueo con etiquetas](container-registry-image-lock.md) y muchas otras características mejoradas. 

Además de las imágenes de contenedor compatibles con Docker, Azure Container Registry admite un conjunto de [artefactos de contenido](container-registry-image-formats.md) relacionados, entre los que se incluyen los formatos de imagen de gráficos de Helm y de Open Container Initiative (OCI).

## <a name="repository"></a>Repositorio

Un *repositorio* es una colección de imágenes de contenedor u otros artefactos de un registro que tienen el mismo nombre, pero etiquetas diferentes. Por ejemplo, las tres imágenes siguientes están en el repositorio `acr-helloworld`:

- *acr-helloworld:latest*
- *acr-helloworld:v1*
- *acr-helloworld:v2*

Los nombres de repositorio también pueden incluir [espacios de nombres](container-registry-best-practices.md#repository-namespaces). Los espacios de nombres permiten identificar la propiedad de los artefactos y los repositorios relacionados en la organización mediante el uso de nombres delimitados por barras diagonales. Sin embargo, el registro administra todos los repositorios de forma independiente, no como una jerarquía. Por ejemplo:

- *marketing/campaign10-18/web:v2*
- *marketing/campaign10-18/api:v3*
- *marketing/campaign10-18/email-sender:v2*
- *product-returns/web-submission:20180604*
- *product-returns/legacy-integrator:20180715*

Los nombres de repositorio solo pueden incluir caracteres alfanuméricos en minúscula, puntos, guiones, guiones bajos y barras diagonales. 

Para obtener las reglas de nomenclatura de repositorios completas, vea la [Especificación sobre la distribución de Open Container Initiative](https://github.com/docker/distribution/blob/master/docs/spec/api.md#overview).

## <a name="artifact"></a>Artefacto

Una imagen de contenedor u otro artefacto dentro de un registro está asociado con una o varias etiquetas, tiene una o más capas y se identifica mediante un manifiesto. Entender cómo se relacionan entre sí estos componentes puede ayudarle a administrar el registro de forma eficaz.

### <a name="tag"></a>Etiqueta

La *etiqueta* de una imagen u otro artefacto especifica su versión. A un solo artefacto dentro de un repositorio se le pueden asignar una o varias etiquetas y también puede ser "sin etiqueta". Es decir, puede eliminar todas las etiquetas de una imagen, mientras que los datos de la imagen (sus capas) permanecen en el registro.

El repositorio (o repositorio y espacio de nombres) junto con una etiqueta definen el nombre de una imagen. Puede insertar y extraer una imagen especificando su nombre en la operación de inserción o extracción. De forma predeterminada, se usa la etiqueta `latest` si no se proporciona ninguna en los comandos de Docker.

La forma de etiquetar las imágenes de contenedor está guiada por los escenarios de desarrollo o implementación. Por ejemplo, se recomiendan etiquetas estables para mantener las imágenes base y etiquetas únicas para implementar las imágenes. Para más información, consulte [Recomendaciones para el etiquetado y control de versiones de las imágenes de contenedor](container-registry-image-tag-version.md).

Para obtener información sobre las reglas de nomenclatura de etiquetas, vea la [documentación de Docker](https://docs.docker.com/engine/reference/commandline/tag/).

### <a name="layer"></a>Nivel

Las imágenes de contenedor y los artefactos constan de una o varias *capas*. Los distintos tipos de artefacto definen las capas de manera diferente. Por ejemplo, en una imagen de contenedor de Docker, cada capa corresponde a una línea del Dockerfile que define la imagen:

:::image type="content" source="media/container-registry-concepts/container-image-layers.png" alt-text="Capas de una imagen de contenedor":::

Los artefactos de un registro comparten capas comunes, lo que aumenta la eficacia de almacenamiento. Por ejemplo, varias imágenes de distintos repositorios pueden tener la misma capa base de ASP.NET Core, pero en el registro solo se almacena una copia de esa capa. El uso compartido de las capas optimiza la distribución de estas en nodos y que varios artefactos compartan capas comunes. Por ejemplo, si una imagen que ya se encuentra en un nodo incluye la capa de ASP.NET Core como base, la posterior extracción de otra imagen que haga referencia a la misma capa no transfiere la capa al nodo. En su lugar, hace referencia a la capa que ya existe en el nodo.

Para proporcionar aislamiento seguro y protección frente a posibles manipulaciones de capa, las capas no se comparten entre los registros.

### <a name="manifest"></a>Manifest

Cada imagen de contenedor o artefacto que se inserta en un registro de contenedor está asociado con un *manifiesto*. El manifiesto, que lo ha generado el registro cuando se inserta el contenido, identifica de forma única los artefactos y especifica las capas. Puede enumerar los manifiestos de un repositorio con el comando de la CLI de Azure [az acr repository show-manifests][az-acr-repository-show-manifests]. 

Un manifiesto básico de una imagen `hello-world` de Linux tiene un aspecto similar al siguiente:

  ```json
  {
    "schemaVersion": 2,
    "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
    "config": {
      "mediaType": "application/vnd.docker.container.image.v1+json",
      "size": 1510,
      "digest": "sha256:fbf289e99eb9bca977dae136fbe2a82b6b7d4c372474c9235adc1741675f587e"
    },
    "layers": [
      {
        "mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
        "size": 977,
        "digest": "sha256:2c930d010525941c1d56ec53b97bd057a67ae1865eebf042686d2a2d18271ced"
      }
    ]
  }
  ```

Puede enumerar los manifiestos de un repositorio con el comando de la CLI de Azure [az acr repository show-manifests][az-acr-repository-show-manifests]:

```azurecli
az acr repository show-manifests --name <acrName> --repository <repositoryName>
```

Por ejemplo, enumere los manifiestos del repositorio "acr-helloworld":

```azurecli
az acr repository show-manifests --name myregistry --repository acr-helloworld
```

```output
[
  {
    "digest": "sha256:0a2e01852872580b2c2fea9380ff8d7b637d3928783c55beb3f21a6e58d5d108",
    "tags": [
      "latest",
      "v3"
    ],
    "timestamp&quot;: &quot;2018-07-12T15:52:00.2075864Z"
  },
  {
    "digest": "sha256:3168a21b98836dda7eb7a846b3d735286e09a32b0aa2401773da518e7eba3b57",
    "tags": [
      "v2"
    ],
    "timestamp&quot;: &quot;2018-07-12T15:50:53.5372468Z"
  },
  {
    "digest": "sha256:7ca0e0ae50c95155dbb0e380f37d7471e98d2232ed9e31eece9f9fb9078f2728",
    "tags": [
      "v1"
    ],
    "timestamp&quot;: &quot;2018-07-11T21:38:35.9170967Z"
  }
]
```

### <a name="manifest-digest"></a>Hash de manifiesto

Los manifiestos se identifican mediante un hash SHA-256 único o *hash de manifiesto*. Cada imagen o artefacto, etiquetados o no, se identifican por el hash. El valor del hash es único, incluso si los datos de la capa del artefacto son idénticos a los de otro artefacto. Este mecanismo es lo que le permite insertar varias veces imágenes etiquetadas de forma idéntica a un registro. Por ejemplo, puede insertar varias veces `myimage:latest` en el registro sin errores porque cada imagen se identifica mediante su hash único.

Puede extraer un artefacto de un registro especificando su hash en la operación de extracción. Algunos sistemas se pueden configurar para extraer por hash porque se garantiza la versión de la imagen que se va a extraer, incluso si una imagen etiquetada de forma idéntica se inserta posteriormente en el registro.

> [!IMPORTANT]
> Si inserta repetidamente artefactos modificados con etiquetas idénticas, puede crear artefactos "huérfanos" que están sin etiquetas, pero consumen espacio en el registro. Las imágenes sin etiquetas no se muestran en la CLI de Azure ni en Azure Portal al enumerar o visualizar las imágenes por etiqueta. Sin embargo, sus capas existen y consumen espacio en el registro. La eliminación de una imagen sin etiqueta libera espacio del registro cuando el manifiesto es el único, o el último, que apunta a una capa determinada. Para más información acerca de cómo liberar el espacio usado mediante imágenes sin etiquetar, consulte [Eliminación de imágenes de contenedor en Azure Container Registry](container-registry-delete.md).

## <a name="addressing-an-artifact"></a>Direccionamiento de un artefacto

Para direccionar un artefacto del registro en las operaciones de inserción y extracción con Docker, u otras herramientas de cliente, combine el nombre completo del registro, el nombre del repositorio (incluida la ruta de acceso del espacio de nombres, si procede) y una etiqueta del artefacto o un resumen del manifiesto. En las secciones anteriores encontrará explicaciones de estos términos.

  **Dirección por etiqueta**: `[loginServerUrl]/[repository][:tag]`
    
  **Dirección por hash**: `[loginServerUrl]/[repository@sha256][:digest]`  

Si se usan Docker u otras herramientas de cliente para extraer o insertar artefactos en un registro de contenedor de Azure, se debe utilizar la dirección URL completa del registro, también denominada nombre del *servidor de inicio de sesión*. En la nube de Azure, la dirección URL completa de un registro de contenedor de Azure está en formato `myregistry.azurecr.io` (todo en minúsculas).

> [!NOTE]
> * No puede especificar un número de puerto en la dirección URL del servidor de inicio de sesión del registro, como `myregistry.azurecr.io:443`. 
> * De forma predeterminada, se usa la etiqueta `latest` si no se incluye ninguna en el comando.  

   
### <a name="push-by-tag"></a>Inserción por etiqueta

Ejemplos: 

   `docker push myregistry.azurecr.io/samples/myimage:20210106`

   `docker push myregistry.azurecr.io/marketing/email-sender`

### <a name="pull-by-tag"></a>Extracción por etiqueta

Ejemplo: 

  `docker pull myregistry.azurecr.io/marketing/campaign10-18/email-sender:v2`

### <a name="pull-by-manifest-digest"></a>Extracción por hash de manifiesto


Ejemplo:

  `docker pull myregistry.azurecr.io/acr-helloworld@sha256:0a2e01852872580b2c2fea9380ff8d7b637d3928783c55beb3f21a6e58d5d108`



## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre el [almacenamiento en el registro](container-registry-storage.md) y los [formatos de contenido admitidos](container-registry-image-formats.md) en Azure Container Registry.

Aprenda a [insertar y extraer imágenes ](container-registry-get-started-docker-cli.md) desde Azure Container Registry.

<!-- LINKS - Internal -->
[az-acr-repository-show-manifests]: /cli/azure/acr/repository#az_acr_repository_show_manifests
