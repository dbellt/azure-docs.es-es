---
title: Inserción y extracción de un artefacto de OCI
description: Inserción y extracción de artefactos de Open Container Initiative (OCI) mediante un registro de contenedor privado en Azure
author: SteveLasker
manager: gwallace
ms.topic: article
ms.date: 02/03/2021
ms.author: stevelas
ms.openlocfilehash: dbd64d939f823e8dc7b0cd6d76233b07fce462d9
ms.sourcegitcommit: 9ad20581c9fe2c35339acc34d74d0d9cb38eb9aa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110537408"
---
# <a name="push-and-pull-an-oci-artifact-using-an-azure-container-registry"></a>Inserción y extracción de un artefacto de OCI con un registro de contenedor de Azure

Puede usar un registro de contenedor de Azure para almacenar y administrar [artefactos de Open Container Initiative (OCI)](container-registry-image-formats.md#oci-artifacts), así como imágenes de contenedor de Docker y compatibles con Docker.

Para demostrar esta funcionalidad, en este artículo se muestra cómo usar la herramienta [Registro de OCI como almacenamiento (ORAS)](https://github.com/deislabs/oras) para insertar un artefacto de ejemplo, un archivo de texto, en un registro de contenedor de Azure. A continuación, se extrae el artefacto del registro. Puede administrar una variedad de artefactos de OCI en un registro de contenedor de Azure mediante diferentes herramientas de línea de comandos adecuadas para cada artefacto.

## <a name="prerequisites"></a>Prerequisites

* **Registro de contenedor de Azure**: cree un registro de contenedor en la suscripción de Azure. Por ejemplo, use [Azure Portal](container-registry-get-started-portal.md) o la [CLI de Azure](container-registry-get-started-azure-cli.md).
* **Herramienta ORAS**: descargue e instale una versión actual de ORAS para su sistema operativo desde el [repositorio de GitHub](https://github.com/deislabs/oras/releases). La herramienta se presenta como un tarball comprimido (archivo `.tar.gz`). Extraiga e instale el archivo mediante los procedimientos estándar para su sistema operativo.
* **Entidad de servicio de Azure Active Directory (opcional )** : para autenticarse directamente con ORAS, cree una [entidad de servicio](container-registry-auth-service-principal.md) para acceder al registro. Asegúrese de que se asigna a la entidad de servicio un rol como AcrPush para que tenga permisos de inserción y extracción de artefactos.
* **CLI de Azure (opcional)** : para usar una identidad individual, necesita una instalación local de la CLI de Azure. Se recomienda la versión 2.0.71 o posterior. Ejecute `az --version ` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).
* **Docker (opcional)** : para usar una identidad individual, también debe tener Docker instalado localmente para autenticarse con el registro. Docker proporciona paquetes que permiten configurar Docker fácilmente en cualquier sistema [macOS][docker-mac], [Windows][docker-windows] o [Linux][docker-linux].


## <a name="sign-in-to-a-registry"></a>Inicio de sesión en un registro

En esta sección se muestran dos flujos de trabajo sugeridos para iniciar sesión en el registro, en función de la identidad utilizada. Elija el método adecuado para su entorno.

### <a name="sign-in-with-oras"></a>Inicio de sesión con ORAS

Con una [entidad de servicio](container-registry-auth-service-principal.md) con derechos de inserción, ejecute el comando `oras login` para iniciar sesión en el registro con el identificador de aplicación y la contraseña de la entidad de servicio. Especifique el nombre completo del registro (todo en minúsculas), en este caso *myregistry.azurecr.io*. El identificador de aplicación de la entidad de servicio se pasa en la variable de entorno `$SP_APP_ID` y la contraseña en la variable `$SP_PASSWD`.

```bash
oras login myregistry.azurecr.io --username $SP_APP_ID --password $SP_PASSWD
```

Para leer la contraseña desde Stdin, use `--password-stdin`.

### <a name="sign-in-with-azure-cli"></a>Inicio de sesión con la CLI de Azure

[Inicie sesión](/cli/azure/authenticate-azure-cli) en la CLI de Azure con su identidad para insertar y extraer artefactos del registro de contenedor.

A continuación, use el comando [az acr login](/cli/azure/acr#az_acr_login) de la CLI de Azure para acceder al registro. Por ejemplo, para autenticarse en un registro llamado *myregistry*:

```azurecli
az login
az acr login --name myregistry
```

> [!NOTE]
> `az acr login` usa el cliente de Docker para establecer un token de Azure Active Directory en el archivo `docker.config`. El cliente de Docker debe estar instalado y en ejecución para completar el flujo de autenticación individual.

## <a name="push-an-artifact"></a>Inserción de un artefacto

Cree un archivo de texto en un directorio de trabajo local con algún texto de ejemplo. Por ejemplo, en el shell de Bash:

```bash
echo "Here is an artifact" > artifact.txt
```

Use el comando `oras push` para insertar este archivo de texto en el registro. En el ejemplo siguiente se inserta el archivo de texto de ejemplo en el repositorio `samples/artifact`. El registro se identifica con el nombre de registro completo *myregistry.azurecr.io* (todo en minúsculas). El artefacto se etiqueta `1.0`. El artefacto tiene un tipo no definido de forma predeterminada, identificado por la cadena del *tipo de medio* que sigue al nombre de archivo `artifact.txt`. Consulte [Artefactos de OCI](https://github.com/opencontainers/artifacts) para tipos adicionales. 

**Linux o macOS**

```bash
oras push myregistry.azurecr.io/samples/artifact:1.0 \
    --manifest-config /dev/null:application/vnd.unknown.config.v1+json \
    ./artifact.txt:application/vnd.unknown.layer.v1+txt
```

**Windows**

```cmd
.\oras.exe push myregistry.azurecr.io/samples/artifact:1.0 ^
    --manifest-config NUL:application/vnd.unknown.config.v1+json ^
    .\artifact.txt:application/vnd.unknown.layer.v1+txt
```

La salida de una inserción correcta es similar a la siguiente:

```console
Uploading 33998889555f artifact.txt
Pushed myregistry.azurecr.io/samples/artifact:1.0
Digest: sha256:xxxxxxbc912ef63e69136f05f1078dbf8d00960a79ee73c210eb2a5f65xxxxxx
```

Para administrar artefactos en el registro, si usa la CLI de Azure, ejecute los comandos estándar `az acr` para administrar las imágenes. Por ejemplo, obtenga los atributos del artefacto mediante el comando [az acr repository show][az-acr-repository-show]:

```azurecli
az acr repository show \
    --name myregistry \
    --image samples/artifact:1.0
```

La salida es similar a la siguiente:

```json
{
  "changeableAttributes": {
    "deleteEnabled": true,
    "listEnabled": true,
    "readEnabled": true,
    "writeEnabled": true
  },
  "createdTime": "2019-08-28T20:43:31.0001687Z",
  "digest": "sha256:xxxxxxbc912ef63e69136f05f1078dbf8d00960a79ee73c210eb2a5f65xxxxxx",
  "lastUpdateTime": "2019-08-28T20:43:31.0001687Z",
  "name": "1.0",
  "signed": false
}
```

## <a name="pull-an-artifact"></a>Extracción de un artefacto

Ejecute el comando `oras pull` para extraer el artefacto del registro.

En primer lugar, elimine el archivo de texto del directorio de trabajo local:

```bash
rm artifact.txt
```

Ejecute `oras pull` para extraer el artefacto y especifique el tipo de medio utilizado para insertar el artefacto:

```bash
oras pull myregistry.azurecr.io/samples/artifact:1.0 \
    --media-type application/vnd.unknown.layer.v1+txt
```

Compruebe que la extracción se realizó correctamente:

```bash
$ cat artifact.txt
Here is an artifact
```

## <a name="remove-the-artifact-optional"></a>Eliminación del artefacto (opcional)

Para eliminar el artefacto del registro de contenedor de Azure, use el comando [az acr repository delete][az-acr-repository-delete]. En el ejemplo siguiente se elimina el artefacto que almacenó allí:

```azurecli
az acr repository delete \
    --name myregistry \
    --image samples/artifact:1.0
```

## <a name="example-build-docker-image-from-oci-artifact"></a>Ejemplo: Compilación de una imagen de Docker desde el artefacto de OCI

El código fuente y los archivos binarios para compilar una imagen de contenedor se pueden almacenar como artefactos de OCI en el registro de contenedor de Azure. Puede hacer referencia a un artefacto de origen como el contexto de compilación para una [tarea de ACR](container-registry-tasks-overview.md). En este ejemplo se muestra cómo almacenar un Dockerfile como un artefacto de OCI y luego hacer referencia al artefacto para compilar una imagen de contenedor.

Por ejemplo, cree un Dockerfile de una línea:

```bash
echo "FROM mcr.microsoft.com/hello-world" > hello-world.dockerfile
```

Inicie sesión en el registro de contenedor de destino.

```azurecli
az login
az acr login --name myregistry
```

Cree e inserte un nuevo artefacto de OCI en el registro de destino mediante el comando `oras push`. En este ejemplo se establece el tipo de medio predeterminado para el artefacto.

```bash
oras push myregistry.azurecr.io/dockerfile:1.0 hello-world.dockerfile
```

Ejecute el comando [az acr build](/cli/azure/acr#az_acr_build) para compilar la imagen de Hola mundo con el nuevo artefacto como contexto de compilación:

```azurecli
az acr build --registry myregistry --image builds/hello-world:v1 \
  --file hello-world.dockerfile \
  oci://myregistry.azurecr.io/dockerfile:1.0
```

## <a name="next-steps"></a>Pasos siguientes

* Más información acerca de la [Biblioteca ORAS](https://github.com/deislabs/oras), incluido cómo configurar un manifiesto para un artefacto
* Visite el repositorio [Artefactos de OCI](https://github.com/opencontainers/artifacts) para obtener información de referencia sobre los nuevos tipos de artefactos



<!-- LINKS - external -->
[docker-linux]: https://docs.docker.com/engine/installation/#supported-platforms
[docker-mac]: https://docs.docker.com/docker-for-mac/
[docker-windows]: https://docs.docker.com/docker-for-windows/

<!-- LINKS - internal -->
[az-acr-repository-show]: /cli/azure/acr/repository?#az_acr_repository_show
[az-acr-repository-delete]: /cli/azure/acr/repository#az_acr_repository_delete
