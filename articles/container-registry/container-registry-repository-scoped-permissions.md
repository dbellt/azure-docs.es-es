---
title: Permisos a repositorios en Azure Container Registry
description: Cree un token con permisos orientados a repositorios específicos de un registro Premium para extraer o insertar imágenes, o realizar otras acciones.
ms.topic: article
ms.date: 02/04/2021
ms.openlocfilehash: 8cdcd3e09603f24c37ad7323a273ca97f76fcd0c
ms.sourcegitcommit: bd1a4e4df613ff24e954eb3876aebff533b317ae
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/23/2021
ms.locfileid: "107930504"
---
# <a name="create-a-token-with-repository-scoped-permissions"></a>Creación de un token con permisos orientados al repositorio

En este artículo se describe cómo crear tokens y asignaciones de ámbito para administrar el acceso a repositorios concretos del registro de contenedor. Mediante la creación de tokens, los propietarios del registro pueden proporcionar a los usuarios o servicios acceso de tiempo limitado y orientado a los repositorios con el fin de extraer o insertar imágenes o de realizar otras acciones. Un token proporciona permisos más específicos que otras [opciones de autenticación](container-registry-authentication.md) del registro, que limitan el ámbito de los permisos a un registro completo. 

Los escenarios para crear un token son:

* Permitir que los dispositivos IoT con tokens individuales extraigan una imagen de un repositorio.
* Proporcionar a una organización externa permisos a un repositorio específico. 
* Limitar el acceso del repositorio a diferentes grupos de usuarios de la organización. Por ejemplo, proporcione acceso de lectura y escritura a los desarrolladores que compilan imágenes destinadas a repositorios específicos, y acceso de lectura a los equipos que realizan implementaciones desde estos repositorios.

Esta característica está disponible en el nivel de servicio de un registro de contenedor **Premium**. Para obtener información sobre los límites y los niveles de servicio de los registros, vea [Niveles de servicio de Azure Container Registry](container-registry-skus.md).

> [!IMPORTANT]
> Esta funcionalidad actualmente está en su versión preliminar y se [aplican algunas limitaciones](#preview-limitations). Las versiones preliminares están a su disposición con la condición de que acepte los [términos de uso adicionales][terms-of-use]. Es posible que algunos de los aspectos de esta característica cambien antes de ofrecer disponibilidad general.

## <a name="preview-limitations"></a>Limitaciones de vista previa

* Actualmente no se pueden asignar permisos orientados al repositorio a una identidad de Azure Active Directory, como una entidad de servicio o una identidad administrada.
* No se puede crear una asignación de ámbito en un registro habilitado para el [acceso de extracción anónimo](container-registry-faq.md#how-do-i-enable-anonymous-pull-access).

## <a name="concepts"></a>Conceptos

Para configurar los permisos orientados al repositorio, debe crear un *token* con una *asignación de ámbito* asociada. 

* Un **token** junto con una contraseña generada permiten al usuario autenticarse con el registro. Puede establecer una fecha de expiración para la contraseña del token o deshabilitar un token en cualquier momento.  

  Después de autenticarse con un token, el usuario o el servicio pueden realizar una o varias *acciones* que tengan como ámbito uno o varios repositorios.

  |Acción  |Descripción  | Ejemplo |
  |---------|---------|--------|
  |`content/delete`    | Quitar datos del repositorio  | Eliminar un repositorio o un manifiesto |
  |`content/read`     |  Leer datos del repositorio |  Extracción de un artefacto |
  |`content/write`     |  Escribir datos en el repositorio     | Úsela con `content/read` para insertar un artefacto. |
  |`metadata/read`    | Leer metadatos del repositorio   | Enumerar etiquetas o manifiestos |
  |`metadata/write`     |  Escribir metadatos en el repositorio  | Habilitar o deshabilitar operaciones de lectura, escritura o eliminación |

* Una **asignación de ámbito** agrupa los permisos de repositorio que se aplican a un token. Estos permisos se pueden volver a aplicar a otros tokens. Cada token está asociado con una única asignación de ámbito. 

   Con una asignación de ámbito puede:

    * Configurar varios tokens con permisos idénticos para un conjunto de repositorios
    * Actualizar permisos de token al agregar o quitar acciones de repositorio en la asignación de ámbito o aplicar una asignación de ámbito diferente 

  Azure Container Registry también proporciona varias asignaciones de ámbito definidas por el sistema que se pueden aplicar al crear tokens. Los permisos de las asignaciones de ámbito definidas por el sistema se aplican a todos los repositorios del registro.

En la imagen siguiente se muestra la relación entre los tokens y las asignaciones de ámbito. 

![Tokens del registro y asignaciones de ámbito](media/container-registry-repository-scoped-permissions/token-scope-map-concepts.png)

## <a name="prerequisites"></a>Requisitos previos

* **CLI de Azure**: Los ejemplos de comandos de la CLI de Azure de este artículo requieren la versión 2.17.0, o cualquier versión posterior, de la CLI de Azure. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).
* **Docker**: para autenticarse con el registro para insertar o extraer imágenes, necesita una instalación local de Docker. Docker ofrece instrucciones de instalación para sistemas [macOS](https://docs.docker.com/docker-for-mac/), [Windows](https://docs.docker.com/docker-for-windows/) y [Linux](https://docs.docker.com/engine/installation/#supported-platforms).
* **Registro de contenedor**: si no tiene uno, cree un registro de contenedor premium en la suscripción de Azure o actualice un registro existente. Por ejemplo, use [Azure Portal](container-registry-get-started-portal.md) o la [CLI de Azure](container-registry-get-started-azure-cli.md). 

## <a name="create-token---cli"></a>Creación de un token: CLI

### <a name="create-token-and-specify-repositories"></a>Creación de un token y especificación de los repositorios

Cree un token con el comando [az acr token create][az-acr-token-create]. Al crear un token, puede especificar uno o varios repositorios y acciones asociadas en cada uno. No es necesario que los repositorios estén todavía en el registro. Para crear un token mediante la especificación de una asignación de ámbito existente, consulte la [sección siguiente](#create-token-and-specify-scope-map).

En el ejemplo siguiente se crea un token en el registro *myregistry* con los permisos siguientes en el repositorio `samples/hello-world`: `content/write` y `content/read`. De forma predeterminada, el comando establece el estado predeterminado del token en `enabled`, pero puede actualizarlo a `disabled` en cualquier momento.

```azurecli
az acr token create --name MyToken --registry myregistry \
  --repository samples/hello-world \
  content/write content/read \
  --output json
```

La salida muestra detalles sobre el token. De forma predeterminada, se generan dos contraseñas que no expiran, pero también se puede establecer una fecha de expiración. Se recomienda guardar las contraseñas en un lugar seguro para usarlas más adelante para la autenticación. Las contraseñas no se pueden recuperar de nuevo, pero se pueden generar otras.

```console
{
  "creationDate": "2020-01-18T00:15:34.066221+00:00",
  "credentials": {
    "certificates": [],
    "passwords": [
      {
        "creationTime": "2020-01-18T00:15:52.837651+00:00",
        "expiry": null,
        "name": "password1",
        "value": "uH54BxxxxK7KOxxxxRbr26dAs8JXxxxx"
      },
      {
        "creationTime": "2020-01-18T00:15:52.837651+00:00",
        "expiry": null,
        "name": "password2",
        "value": "kPX6Or/xxxxLXpqowxxxxkA0idwLtmxxxx"
      }
    ],
    "username": "MyToken"
  },
  "id": "/subscriptions/xxxxxxxx-adbd-4cb4-c864-xxxxxxxxxxxx/resourceGroups/myresourcegroup/providers/Microsoft.ContainerRegistry/registries/myregistry/tokens/MyToken",
  "name": "MyToken",
  "objectId": null,
  "provisioningState": "Succeeded",
  "resourceGroup": "myresourcegroup",
  "scopeMapId": "/subscriptions/xxxxxxxx-adbd-4cb4-c864-xxxxxxxxxxxx/resourceGroups/myresourcegroup/providers/Microsoft.ContainerRegistry/registries/myregistry/scopeMaps/MyToken-scope-map",
  "status": "enabled",
  "type": "Microsoft.ContainerRegistry/registries/tokens"
```

> [!NOTE]
> Para volver a generar contraseñas de token y períodos de expiración, consulte el apartado [Regeneración de contraseñas de token](#regenerate-token-passwords) de este mismo artículo.

La salida incluye detalles sobre la asignación de ámbito que creó el comando. Puede usar la asignación de ámbito, llamada aquí `MyToken-scope-map`, para aplicar las mismas acciones de repositorio a otros tokens. También, puede actualizar la asignación de ámbito más adelante para cambiar los permisos de los tokens asociados.

### <a name="create-token-and-specify-scope-map"></a>Creación de un token y especificación de la asignación de ámbito

Una manera alternativa de crear un token es especificar una asignación de ámbito existente. Si aún no tiene una asignación de ámbito, primero debe crear una especificando los repositorios y las acciones asociadas. Luego, especifique la asignación de ámbito al crear un token. 

Para crear una asignación de ámbito, use el comando [az acr scope-map create][az-acr-scope-map-create]. El siguiente comando crea una asignación de ámbito con los mismos permisos en el repositorio `samples/hello-world` usado anteriormente. 

```azurecli
az acr scope-map create --name MyScopeMap --registry myregistry \
  --repository samples/hello-world \
  content/write content/read \
  --description "Sample scope map"
```

Ejecute [az acr token create][az-acr-token-create] para crear un token y especifique la asignación de ámbito *MyScopeMap*. Como en el ejemplo anterior, el comando establece el estado predeterminado del token en `enabled`.

```azurecli
az acr token create --name MyToken \
  --registry myregistry \
  --scope-map MyScopeMap
```

La salida muestra detalles sobre el token. De manera predeterminada, se generan dos contraseñas. Se recomienda guardar las contraseñas en un lugar seguro para usarlas más adelante para la autenticación. Las contraseñas no se pueden recuperar de nuevo, pero se pueden generar otras.

> [!NOTE]
> Para volver a generar contraseñas de token y períodos de expiración, consulte el apartado [Regeneración de contraseñas de token](#regenerate-token-passwords) de este mismo artículo.

## <a name="create-token---portal"></a>Creación de un token: portal

Puede usar Azure Portal para crear tokens y asignaciones de ámbito. Al igual que con el comando de la CLI `az acr token create`, puede aplicar una asignación de ámbito existente o crear una al crear un token mediante la especificación de uno o varios repositorios y acciones asociadas. No es necesario que los repositorios estén todavía en el registro. 

En el ejemplo siguiente se crea un token y se crea una asignación de ámbito con los siguientes permisos en el repositorio `samples/hello-world`: `content/write` y `content/read`.

1. En el portal, vaya al registro de contenedor.
1. En **Permisos del repositorio**, seleccione **Tokens (versión preliminar) > +Agregar**.

      :::image type="content" source="media/container-registry-repository-scoped-permissions/portal-token-add.png" alt-text="Creación de un token en el portal":::
1. Escriba un nombre de token.
1. En **Asignación de ámbito**, seleccione **Crear nuevo**.
1. Configure la asignación de ámbito:
    1. Escriba un nombre y una descripción para la asignación de ámbito. 
    1. En **Repositorios**, escriba `samples/hello-world` y, en **Permisos**, seleccione `content/read` y `content/write`. Luego, seleccione **+Agregar**.  

        :::image type="content" source="media/container-registry-repository-scoped-permissions/portal-scope-map-add.png" alt-text="Creación de una asignación de ámbito en el portal":::

    1. Después de agregar los repositorios y los permisos, seleccione **Agregar** para agregar la asignación de ámbito.
1. Acepte el valor de **Estado** del token predeterminado de **Habilitado** y seleccione **Crear**.

Después de validar y crear el token, los detalles del token aparecen en la pantalla **Tokens**.

### <a name="add-token-password"></a>Adición de la contraseña del token

Para usar un token creado en el portal, debe generar una contraseña. Puede generar una o dos contraseñas y establecer una fecha de expiración para cada una de ellas. 

1. En el portal, vaya al registro de contenedor.
1. En **Permisos del repositorio**, seleccione **Tokens (versión preliminar)** y seleccione un token.
1. En los detalles del token, seleccione **password1** o **password2**, y elija el icono Generar.
1. En la pantalla de contraseña, establezca opcionalmente una fecha de expiración para la contraseña y seleccione **Generar**. Se recomienda establecer una fecha de expiración.
1. Después de generar una contraseña, cópiela y guárdela en una ubicación segura. No se puede recuperar una contraseña generada después de cerrar la pantalla, pero se puede generar una nueva.

    :::image type="content" source="media/container-registry-repository-scoped-permissions/portal-token-password.png" alt-text="Creación de una contraseña de token en el portal":::

## <a name="authenticate-with-token"></a>Autenticación con un token

Cuando un usuario o un servicio usan un token para autenticarse con registro de destino, se proporciona el nombre del token como un nombre de usuario y una de sus contraseñas generadas. 

El método de autenticación depende de las acciones configuradas asociadas con el token.

|Acción  |Cómo autenticarse  |
  |---------|---------|
  |`content/delete`    | `az acr repository delete` en la CLI de Azure<br/><br/>Ejemplo: `az acr repository delete --name myregistry --repository myrepo --username MyToken --password xxxxxxxxxx`|
  |`content/read`     |  `docker login`<br/><br/>`az acr login` en la CLI de Azure<br/><br/>Ejemplo: `az acr login --name myregistry --username MyToken --password xxxxxxxxxx`  |
  |`content/write`     |  `docker login`<br/><br/>`az acr login` en la CLI de Azure     |
  |`metadata/read`    | `az acr repository show`<br/><br/>`az acr repository show-tags`<br/><br/>`az acr repository show-manifests` en la CLI de Azure   |
  |`metadata/write`     |  `az acr repository untag`<br/><br/>`az acr repository update` en la CLI de Azure |

## <a name="examples-use-token"></a>Ejemplos: Uso del token

En los siguientes ejemplos se usa el token creado anteriormente en este artículo para realizar operaciones comunes en un repositorio: insertar y extraer imágenes, eliminar imágenes y mostrar etiquetas del repositorio. El token se configuró inicialmente con permisos de extracción (acciones `content/write` y `content/read`) en el repositorio `samples/hello-world`.

### <a name="pull-and-tag-test-images"></a>Inserción y etiquetado de imágenes de prueba

En los siguientes ejemplos, extraiga las imágenes `hello-world` y `nginx` públicas de Microsoft Container Registry y etiquételas para su registro y repositorio.

```bash
docker pull mcr.microsoft.com/hello-world
docker pull mcr.microsoft.com/oss/nginx/nginx:1.15.5-alpine
docker tag mcr.microsoft.com/hello-world myregistry.azurecr.io/samples/hello-world:v1
docker tag mcr.microsoft.com/oss/nginx/nginx:1.15.5-alpine myregistry.azurecr.io/samples/nginx:v1
```

### <a name="authenticate-using-token"></a>Autenticación mediante el token

Ejecute `docker login` o `az acr login` para autenticarse con el registro para insertar o extraer imágenes. Proporcione el nombre del token como el nombre de usuario y proporcione una de sus contraseñas. El token debe tener el estado `Enabled`.

El siguiente ejemplo tiene el formato para el shell de Bash y se proporcionan los valores mediante variables de entorno.

```bash
TOKEN_NAME=MyToken
TOKEN_PWD=<token password>

echo $TOKEN_PWD | docker login --username $TOKEN_NAME --password-stdin myregistry.azurecr.io
```

La salida debería mostrar la autenticación correcta:

```console
Login Succeeded
```

### <a name="push-images-to-registry"></a>Inserción de imágenes en el registro

Después iniciar sesión correctamente, intente insertar las imágenes etiquetadas en el registro. Como el token tiene permisos para insertar imágenes en el repositorio `samples/hello-world`, la siguiente operación de inserción se realiza correctamente:

```bash
docker push myregistry.azurecr.io/samples/hello-world:v1
```

El token no tiene permisos para el repositorio `samples/nginx`, así que el siguiente intento de inserción produce un error similar a `requested access to the resource is denied`:

```bash
docker push myregistry.azurecr.io/samples/nginx:v1
```

### <a name="update-token-permissions"></a>Actualización de permisos de token

Para actualizar los permisos de un token, actualice los permisos en la asignación de ámbito asociada. La asignación de ámbito actualizada se aplica inmediatamente a todos los tokens asociados. 

Por ejemplo, actualice `MyToken-scope-map` con acciones `content/write` y `content/read` en el repositorio `samples/ngnx`, y elimine la acción `content/write` en el repositorio `samples/hello-world`.  

Para usar la CLI de Azure, ejecute [az acr scope-map update][az-acr-scope-map-update] para actualizar la asignación de ámbito:

```azurecli
az acr scope-map update \
  --name MyScopeMap \
  --registry myregistry \
  --add-repository samples/nginx content/write content/read \
  --remove-repository samples/hello-world content/write 
```

En Azure Portal:

1. Vaya al registro de contenedor.
1. En **Permisos del repositorio**, seleccione **Asignaciones de ámbito (versión preliminar)** y seleccione la asignación de ámbito que se va a actualizar.
1. En **Repositorios**, escriba `samples/nginx` y, en **Permisos**, seleccione `content/read` y `content/write`. Luego, seleccione **+Agregar**.
1. En **Repositorios**, seleccione `samples/hello-world` y, en **Permisos**, anule la selección de `content/write`. Después, seleccione **Guardar**.

Después de actualizar la asignación de ámbito, la siguiente inserción se realiza correctamente:

```bash
docker push myregistry.azurecr.io/samples/nginx:v1
```

Dado que la asignación de ámbito solo tiene el permiso `content/read` en el repositorio `samples/hello-world`, el intento de inserción en el repositorio `samples/hello-world` ahora genera un error:
 
```bash
docker push myregistry.azurecr.io/samples/hello-world:v1
```

La extracción de imágenes de ambos repositorios se realiza correctamente, porque la asignación de ámbito proporciona permisos `content/read` en ambos repositorios:

```bash
docker pull myregistry.azurecr.io/samples/nginx:v1
docker pull myregistry.azurecr.io/samples/hello-world:v1
```
### <a name="delete-images"></a>Eliminación de imágenes

Para actualizar la asignación de ámbito, agregue la acción `content/delete` al repositorio `nginx`. Esta acción permite la eliminación de imágenes en el repositorio o la eliminación de todo el repositorio.

Por motivos de brevedad, solo se muestra el comando [az acr scope-map update][az-acr-scope-map-update] para actualizar la asignación de ámbito:

```azurecli
az acr scope-map update \
  --name MyScopeMap \
  --registry myregistry \
  --add-repository samples/nginx content/delete
``` 

Para actualizar la asignación de ámbito mediante el portal, consulte la [sección anterior](#update-token-permissions).

Use el siguiente comando [az acr repository delete][az-acr-repository-delete] para eliminar el repositorio `samples/nginx`. Para eliminar las imágenes o los repositorios, pase el nombre y la contraseña del token al comando. En el ejemplo siguiente se usan las variables de entorno creadas anteriormente en el artículo:

```azurecli
az acr repository delete \
  --name myregistry --repository samples/nginx \
  --username $TOKEN_NAME --password $TOKEN_PWD
```

### <a name="show-repo-tags"></a>Presentación de las etiquetas de repositorio 

Para actualizar la asignación de ámbito, agregue la acción `metadata/read` al repositorio `hello-world`. Esta acción permite leer los datos de manifiesto y etiqueta en el repositorio.

Por motivos de brevedad, solo se muestra el comando [az acr scope-map update][az-acr-scope-map-update] para actualizar la asignación de ámbito:

```azurecli
az acr scope-map update \
  --name MyScopeMap \
  --registry myregistry \
  --add-repository samples/hello-world metadata/read 
```  

Para actualizar la asignación de ámbito mediante el portal, consulte la [sección anterior](#update-token-permissions).

Para leer los metadatos del repositorio `samples/hello-world`, ejecute el comando [az acr repository show-manifests][az-acr-repository-show-manifests] o [az acr repository show-tags][az-acr-repository-show-tags]. 

Para leer los metadatos, pase el nombre y la contraseña del token a cualquiera de los comandos. En el ejemplo siguiente se usan las variables de entorno creadas anteriormente en el artículo:

```azurecli
az acr repository show-tags \
  --name myregistry --repository samples/hello-world \
  --username $TOKEN_NAME --password $TOKEN_PWD
```

Salida del ejemplo:

```console
[
  "v1"
]
```

## <a name="manage-tokens-and-scope-maps"></a>Administración de tokens y asignaciones de ámbito

### <a name="list-scope-maps"></a>Presentación de asignaciones de ámbito

Use el comando [az acr scope-map list][az-acr-scope-map-list] o la pantalla **Scope maps (Preview)** (Asignaciones de ámbito [versión preliminar]) en el portal para mostrar todas las asignaciones de ámbito configuradas en un registro. Por ejemplo:

```azurecli
az acr scope-map list \
  --registry myregistry --output table
```

La salida consta de las tres asignaciones de ámbito definidas por el sistema y otras asignaciones generadas por el usuario. Los tokens se pueden configurar con cualquiera de estas asignaciones de ámbito.

```
NAME                 TYPE           CREATION DATE         DESCRIPTION
-------------------  -------------  --------------------  ------------------------------------------------------------
_repositories_admin  SystemDefined  2020-01-20T09:44:24Z  Can perform all read, write and delete operations on the ...
_repositories_pull   SystemDefined  2020-01-20T09:44:24Z  Can pull any repository of the registry
_repositories_push   SystemDefined  2020-01-20T09:44:24Z  Can push to any repository of the registry
MyScopeMap           UserDefined    2019-11-15T21:17:34Z  Sample scope map
```

### <a name="show-token-details"></a>Presentación de los detalles del token

Para ver los detalles de un token, como su estado o las fechas de expiración de la contraseña, ejecute el comando [az acr token show][az-acr-token-show] o seleccione el token en la pantalla **Tokens (Preview)** (Tokens [versión preliminar]) del portal. Por ejemplo:

```azurecli
az acr scope-map show \
  --name MyScopeMap --registry myregistry
```

Use el comando [az acr token list][az-acr-token-list] o la pantalla **Tokens (Preview)** (Tokens [versión preliminar]) en el portal para mostrar todos los tokens configurados en un registro. Por ejemplo:

```azurecli
az acr token list --registry myregistry --output table
```

### <a name="regenerate-token-passwords"></a>Regeneración de contraseñas de token

Si no generó una contraseña de token o quiere generar otras nuevas, ejecute el comando [az acr token credential generate][az-acr-token-credential-generate]. 

En el ejemplo siguiente se genera un nuevo valor para password1 para el token *MyToken*, con un período de expiración de 30 días. La contraseña se almacena en la variable de entorno `TOKEN_PWD`. El fragmento de código tiene el formato para el shell de Bash.

```azurecli
TOKEN_PWD=$(az acr token credential generate \
  --name MyToken --registry myregistry --expiration-in-days 30 \
  --password1 --query 'passwords[0].value' --output tsv)
```

Para usar Azure Portal para generar una contraseña de token, consulte los pasos descritos en [Creación de un token: portal](#create-token---portal) anteriormente en este artículo.

### <a name="update-token-with-new-scope-map"></a>Actualización de un token con una nueva asignación de ámbito

Si quiere actualizar un token con una asignación de ámbito diferente, ejecute [az acr token update][az-acr-token-update] y especifique la nueva asignación de ámbito. Por ejemplo:

```azurecli
az acr token update --name MyToken --registry myregistry \
  --scope-map MyNewScopeMap
```

En el portal, en la pantalla **Tokens (preview)** (Tokens [versión preliminar]), seleccione el token y, en **Asignación de ámbito**, seleccione una asignación de ámbito diferente.

> [!TIP]
> Después de actualizar un token con una nueva asignación de ámbito, podría querer generar nuevas contraseñas de token. Use el comando [az acr token credential generate][az-acr-token-credential-generate] o vuelva a generar una contraseña de token en Azure Portal.

## <a name="disable-or-delete-token"></a>Deshabilitación o eliminación de un token

Puede que tenga que deshabilitar temporalmente el uso de las credenciales de token para un usuario o un servicio. 

Mediante la CLI de Azure, ejecute el comando [az acr token update][az-acr-token-update] para establecer `status` en `disabled`:

```azurecli
az acr token update --name MyToken --registry myregistry \
  --status disabled
```

En el portal, seleccione el token en la pantalla **Tokens (Preview)** (Tokens [versión preliminar]) y seleccione **Deshabilitado** en **Estado**.

Para eliminar un token a fin de invalidar de forma permanente el acceso por parte de cualquiera que use sus credenciales, ejecute el comando [az acr token delete][az-acr-token-delete]. 

```azurecli
az acr token delete --name MyToken --registry myregistry
```

En el portal, seleccione el token en la pantalla **Tokens (Preview)** (Tokens [versión preliminar]) y elija **Descartar**.

## <a name="next-steps"></a>Pasos siguientes

* Para administrar asignaciones de ámbito y tokens, use otros comandos de los grupos de comandos [az acr scope-map][az-acr-scope-map] y [az acr token][az-acr-token].
* Consulte la [información general sobre la autenticación](container-registry-authentication.md) para conocer otras opciones de autenticarse con un registro de contenedor de Azure, como el uso de una identidad de Azure Active Directory, una entidad de servicio o una cuenta de administrador.


<!-- LINKS - External -->
[terms-of-use]: https://azure.microsoft.com/support/legal/preview-supplemental-terms/

<!-- LINKS - Internal -->
[az-acr-login]: /cli/azure/acr#az_acr_login
[az-acr-repository]: /cli/azure/acr/repository/
[az-acr-repository-show-tags]: /cli/azure/acr/repository/#az_acr_repository_show_tags
[az-acr-repository-show-manifests]: /cli/azure/acr/repository/#az_acr_repository_show_manifests
[az-acr-repository-delete]: /cli/azure/acr/repository/#az_acr_repository_delete
[az-acr-scope-map]: /cli/azure/acr/scope-map/
[az-acr-scope-map-create]: /cli/azure/acr/scope-map/#az_acr_scope_map_create
[az-acr-scope-map-list]: /cli/azure/acr/scope-map/#az_acr_scope_map_show
[az-acr-scope-map-show]: /cli/azure/acr/scope-map/#az_acr_scope_map_list
[az-acr-scope-map-update]: /cli/azure/acr/scope-map/#az_acr_scope_map_update
[az-acr-scope-map-list]: /cli/azure/acr/scope-map/#az_acr_scope_map_list
[az-acr-token]: /cli/azure/acr/token/
[az-acr-token-show]: /cli/azure/acr/token/#az_acr_token_show
[az-acr-token-list]: /cli/azure/acr/token/#az_acr_token_list
[az-acr-token-delete]: /cli/azure/acr/token/#az_acr_token_delete
[az-acr-token-create]: /cli/azure/acr/token/#az_acr_token_create
[az-acr-token-update]: /cli/azure/acr/token/#az_acr_token_update
[az-acr-token-credential-generate]: /cli/azure/acr/token/credential/#az_acr_token_credential_generate
