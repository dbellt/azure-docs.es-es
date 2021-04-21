---
title: CI/CD en contenedores personalizados
description: Aprenda a configurar la implementación continua en un contenedor Windows o Linux personalizado en Azure App Service.
keywords: azure app service, linux, docker, acr,oss
author: msangapu-msft
ms.assetid: a47fb43a-bbbd-4751-bdc1-cd382eae49f8
ms.topic: article
ms.date: 03/12/2021
ms.author: msangapu
ms.custom: seodec18, devx-track-azurecli
zone_pivot_groups: app-service-containers-windows-linux
ms.openlocfilehash: 6519f3fe7335ed41f4d5ef67771aaa738a33e4a8
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107782610"
---
# <a name="continuous-deployment-with-custom-containers-in-azure-app-service"></a>Implementación continua con contenedores personalizados en Azure App Service

En este tutorial, configurará una implementación continua para una imagen de contenedor personalizada desde los repositorios administrados de [Azure Container Registry](https://azure.microsoft.com/services/container-registry/) o [Docker Hub](https://hub.docker.com).

## <a name="1-go-to-deployment-center"></a>1. Ir al centro de implementación

En [Azure Portal](https://portal.azure.com), vaya a la página de administración de la aplicación App Service.

En el menú de la izquierda, haga clic en **Centro de implementación** > **Configuración**. 

::: zone pivot="container-linux"
## <a name="2-choose-deployment-source"></a>2. Elegir origen de la implementación

**Elija** el origen de implementación en función de su escenario:
- **Container Registry** configura CI/CD entre el registro de contenedor y App Service.
- La opción **Acciones de GitHub** se utiliza si se mantiene el código fuente de la imagen de contenedor en GitHub. Desencadenada por nuevas confirmaciones en el repositorio de GitHub, la acción de implementación puede ejecutar `docker build` y `docker push` directamente en el registro de contenedor y, después, actualizar la aplicación App Service para ejecutar la nueva imagen. Para más información, consulte el [funcionamiento de CI/CD con Acciones de GitHub](#how-cicd-works-with-github-actions).
- Para configurar CI/CD con **Azure Pipelines**, consulte [implementación de un contenedor de aplicación web de Azure en Azure Pipelines](/azure/devops/pipelines/targets/webapp-on-container-linux).

> [!NOTE]
> Para una aplicación de Docker Compose, seleccione **Container Registry**.

Si elige Acciones de GitHub, **haga clic en** **Autorizar** y siga las indicaciones de autorización. Si ya ha autorizado antes con GitHub, puede realizar la implementación desde un repositorio de otro usuario haciendo clic en **Cambiar cuenta**.

Cuando autorice su cuenta de Azure con GitHub, **seleccione** la **organización**, el **repositorio** y la **rama** desde donde se va a realizar la implementación.
::: zone-end  

::: zone pivot="container-windows"
## <a name="2-configure-registry-settings"></a>2. Configurar el registro
::: zone-end  
::: zone pivot="container-linux"
## <a name="3-configure-registry-settings"></a>3. Configurar el registro

Para implementar una aplicación de varios contenedores (Docker Compose), **seleccione** **Docker Compose** en **Tipo de contenedor**.

Si no ve la lista desplegable **Tipo de contenedor**, desplácese hacia atrás hasta el **origen** y **seleccione** **Container Registry**.
::: zone-end

En el **origen del registro**, **seleccione** el lugar en el que se encuentra el registro de contenedor. Si no es Azure Container Registry ni Docker Hub, **seleccione** **Registro privado**.

::: zone pivot="container-linux"
> [!NOTE]
> Si la aplicación de varios contenedores (Docker Compose) usa más de una imagen privada, asegúrese de que las imágenes privadas se encuentren en el mismo registro privado y estén accesibles con las mismas credenciales de usuario. Si la aplicación de varios contenedores solo usa imágenes públicas, **seleccione** **Docker Hub**, aunque algunas imágenes no estén en Docker Hub.
::: zone-end  

Siga los pasos siguientes. Para ello, seleccione la pestaña que coincida con su elección.

# <a name="azure-container-registry"></a>[Azure Container Registry](#tab/acr)

La lista desplegable **Registro** muestra los registros de la misma suscripción que la aplicación. **Seleccione** el registro que desee.

> [!NOTE]
> Para implementar desde un registro en una suscripción diferente, **seleccione en cambio** **Registro privado** en el **origen del registro**.

::: zone pivot="container-windows"
**Seleccione** la **imagen** y la **etiqueta** que quiere implementar. Si lo desea, **escriba** el comando de inicio en el **archivo de inicio**. 
::: zone-end
::: zone pivot="container-linux"
Siga este paso según el **tipo de contenedor**:
- En **Docker Compose**, **seleccione** el registro para las imágenes privadas. **Haga clic en** **Elegir archivo** para cargar el archivo de [Docker Compose](https://docs.docker.com/compose/compose-file/)o simplemente **pegue** el contenido del archivo Docker Compose en la **configuración**.
- Para un **solo contenedor**, **seleccione** la **imagen** y la **etiqueta** que quiere implementar. Si lo desea, **escriba** el comando de inicio en el **archivo de inicio**. 
::: zone-end

App Service anexa la cadena del **archivo de inicio** al [final del comando `docker run` (como el segmento `[COMMAND] [ARG...]`)](https://docs.docker.com/engine/reference/run/) al iniciar el contenedor.

# <a name="docker-hub"></a>[Docker Hub](#tab/dockerhub)

::: zone pivot="container-windows"
En **Acceso al repositorio**, **seleccione** si la imagen que se va a implementar es pública o privada.
::: zone-end
::: zone pivot="container-linux"
En **Acceso al repositorio**, **seleccione** si la imagen que se va a implementar es pública o privada. Para una aplicación Docker Compose con una o más imágenes privadas, **seleccione** **Privada**.
::: zone-end

Si selecciona una imagen privada, **especifique** el **inicio de sesión** (nombre de usuario) y la **contraseña** de la cuenta de Docker.

::: zone pivot="container-windows"
**Proporcione** el **nombre de la imagen y la etiqueta completos**, separados por `:` (por ejemplo, `nginx:latest`). Si lo desea, **escriba** el comando de inicio en el **archivo de inicio**. 
::: zone-end
::: zone pivot="container-linux"
Siga este paso según el **tipo de contenedor**:
- En **Docker Compose**, **seleccione** el registro para las imágenes privadas. **Haga clic en** **Elegir archivo** para cargar el archivo de [Docker Compose](https://docs.docker.com/compose/compose-file/)o simplemente **pegue** el contenido del archivo Docker Compose en la **configuración**.
- En el caso de un **solo contenedor**, **proporcione** el **nombre de la imagen y la etiqueta completos**, separados por `:` (por ejemplo, `nginx:latest`). Si lo desea, **escriba** el comando de inicio en el **archivo de inicio**. 
::: zone-end

App Service anexa la cadena del **archivo de inicio** al [final del comando `docker run` (como el segmento `[COMMAND] [ARG...]`)](https://docs.docker.com/engine/reference/run/) al iniciar el contenedor.

# <a name="private-registry"></a>[Registro privado](#tab/private)

En **URL del servidor**, **escriba** la dirección URL del servidor, comenzando por **https://** .

En **Inicio de sesión** y **Contraseña**, **escriba** sus credenciales de inicio de sesión para el registro privado.

::: zone pivot="container-windows"
**Proporcione** el **nombre de la imagen y la etiqueta completos**, separados por `:` (por ejemplo, `nginx:latest`). Si lo desea, **escriba** el comando de inicio en el **archivo de inicio**. 
::: zone-end
::: zone pivot="container-linux"
Siga este paso según el **tipo de contenedor**:
- En **Docker Compose**, **seleccione** el registro para las imágenes privadas. **Haga clic en** **Elegir archivo** para cargar el archivo de [Docker Compose](https://docs.docker.com/compose/compose-file/)o simplemente **pegue** el contenido del archivo Docker Compose en la **configuración**.
- En el caso de un **solo contenedor**, **proporcione** el **nombre de la imagen y la etiqueta completos**, separados por `:` (por ejemplo, `nginx:latest`). Si lo desea, **escriba** el comando de inicio en el **archivo de inicio**. 
::: zone-end

App Service anexa la cadena del **archivo de inicio** al [final del comando `docker run` (como el segmento `[COMMAND] [ARG...]`)](https://docs.docker.com/engine/reference/run/) al iniciar el contenedor.

-----

::: zone pivot="container-windows"
## <a name="3-enable-cicd"></a>3. Habilitar CI/CD
::: zone-end
::: zone pivot="container-linux"
## <a name="4-enable-cicd"></a>4. Habilitar CI/CD
::: zone-end

App Service admite la integración de CI/CD con Azure Container Registry y Docker Hub. Para habilitarla, **seleccione** **Activado** en **Implementación continua**.

::: zone pivot="container-linux"
> [!NOTE]
> Si selecciona **Acciones de GitHub** en **Origen**, no se obtendrá esta opción porque Acciones de GitHub controlan CI/CD directamente. En su lugar, verá una sección de **configuración de flujo de trabajo**, donde puede **hacer clic en** **Vista previa del archivo** para inspeccionar el archivo de flujo de trabajo. Azure confirma este archivo en el repositorio de origen de GitHub seleccionado para controlar las tareas de compilación e implementación. Para más información, consulte el [funcionamiento de CI/CD con Acciones de GitHub](#how-cicd-works-with-github-actions).
::: zone-end

Cuando se habilita esta opción, App Service agrega un webhook al repositorio en Azure Container Registry o Docker Hub. El repositorio se envía a este webhook cada vez que se actualiza la imagen seleccionada con `docker push`. El webhook hace que la aplicación App Service se reinicie y ejecute `docker pull` para obtener la imagen actualizada.

**Para otros registros privados**, puede publicar en el webhook manualmente o como un paso en una canalización de CI/CD. En **URL de webhook**, **haga clic** en el botón **Copiar** para obtener la dirección URL del webhook.

::: zone pivot="container-linux"
> [!NOTE]
> La compatibilidad con aplicaciones de varios contenedores (Docker Compose) es limitada: 
> - Para Azure Container Registry, App Service crea un webhook en el registro seleccionado con el registro como ámbito. Un `docker push` a cualquier repositorio del registro (incluidos los a los que no hace referencia el archivo Docker Compose) desencadena un reinicio de la aplicación. Puede que quiera [modificar el webhook](../container-registry/container-registry-webhook.md) a un ámbito más restringido.
> - Docker Hub no admite webhooks en el nivel del registro. Debe **agregar** los webhooks manualmente a las imágenes especificadas en el archivo de Docker Compose.
::: zone-end

::: zone pivot="container-windows"
## <a name="4-save-your-settings"></a>4. Guardar la configuración
::: zone-end
::: zone pivot="container-linux"
## <a name="5-save-your-settings"></a>5. Guardar la configuración
::: zone-end

**Haga clic en** **Guardar**.

::: zone pivot="container-linux"

## <a name="how-cicd-works-with-github-actions"></a>Funcionamiento de CI/CD con Acciones de GitHub

Si elige **Acciones de GitHub** en **Origen** (consulte [Elección del origen de implementación](#2-choose-deployment-source)), App Service configura CI/CD de las siguientes maneras:

- Deposita un archivo de flujo de trabajo de Acciones de GitHub en el repositorio de GitHub para controlar las tareas de compilación e implementación en App Service.
- Agrega las credenciales del registro privado como secretos de GitHub. El archivo de flujo de trabajo generado ejecuta la acción de [inicio de sesión de Azure/Docker](https://github.com/Azure/docker-login) para iniciar sesión en el registro privado y, a continuación, ejecuta `docker push` para implementarlo.
- Agrega el perfil de publicación de la aplicación como secreto de GitHub. El archivo de flujo de trabajo generado usa este secreto para autenticarse con App Service y, a continuación, ejecuta la acción [Azure/aplicaciones web-deploy](https://github.com/Azure/webapps-deploy) para configurar la imagen actualizada, lo que desencadena un reinicio de la aplicación para extraer la imagen actualizada.
- Captura la información de los [registros de ejecución del flujo de trabajo](https://docs.github.com/actions/managing-workflow-runs/using-workflow-run-logs) y la muestra en la pestaña **Registros** del **Centro de implementación** de la aplicación.

Puede personalizar el proveedor de compilación de Acciones de GitHub de las siguientes maneras:

- Personalice el archivo de flujo de trabajo después de que se genere en el repositorio de GitHub. Para más información, consulte [Sintaxis de flujo de trabajo para Acciones de GitHub](https://docs.github.com/actions/reference/workflow-syntax-for-github-actions). Solo tiene que asegurarse de que el flujo de trabajo finaliza con la acción [Azure/webapps-deploy](https://github.com/Azure/webapps-deploy) para desencadenar un reinicio de la aplicación.
- Si la rama seleccionada está protegida, todavía puede obtener una vista previa del archivo de flujo de trabajo sin guardar la configuración, después, agregarla junto con los secretos de GitHub requeridos manualmente al repositorio. Este método no proporciona la integración de registros con Azure Portal.
- En lugar de un perfil de publicación, implemente con una [entidad de servicio](../active-directory/develop/app-objects-and-service-principals.md#service-principal-object) en Azure Active Directory.

#### <a name="authenticate-with-a-service-principal"></a>Autenticación con una entidad de servicio

Esta configuración opcional reemplaza la autenticación predeterminada por los perfiles de publicación en el archivo de flujo de trabajo generado.

**Genere** una entidad de servicio mediante el comando [az ad sp create-for-rbac](/cli/azure/ad/sp#az_ad_sp_create_for_rbac) de la [CLI de Azure](/cli/azure/). En el ejemplo siguiente, reemplace *\<subscription-id>* , *\<group-name>* y *\<app-name>* por sus propios valores: **Guarde** la salida JSON completa para el paso siguiente, incluido `{}` de nivel superior.

```azurecli-interactive
az ad sp create-for-rbac --name "myAppDeployAuth" --role contributor \
                            --scopes /subscriptions/<subscription-id>/resourceGroups/<group-name>/providers/Microsoft.Web/sites/<app-name> \
                            --sdk-auth
```

> [!IMPORTANT]
> Por seguridad, conceda el acceso mínimo necesario a la entidad de servicio. El ámbito del ejemplo anterior se limita a la aplicación específica de App Service y no a todo el grupo de recursos.

En [GitHub](https://github.com/), **examine** el repositorio y **seleccione** **Configuración > Secretos > Agregar un nuevo secreto**. **Pegue** la salida JSON completa del comando de la CLI de Azure en el campo de valor del secreto. **Asigne** un nombre al secreto, como `AZURE_CREDENTIALS`.

En el archivo de flujo de trabajo generado por el **centro de implementación**, **revise** el paso `azure/webapps-deploy` con código como el ejemplo siguiente:

```yaml
- name: Sign in to Azure 
# Use the GitHub secret you added
- uses: azure/login@v1
    with:
    creds: ${{ secrets.AZURE_CREDENTIALS }}
- name: Deploy to Azure Web App
# Remove publish-profile
- uses: azure/webapps-deploy@v2
    with:
    app-name: '<app-name>'
    slot-name: 'production'
    images: '<registry-server>/${{ secrets.AzureAppService_ContainerUsername_... }}/<image>:${{ github.sha }}'
    - name: Sign out of Azure
    run: |
    az logout
```

::: zone-end

## <a name="automate-with-cli"></a>Automatización con la interfaz de la línea de comandos

Para configurar el registro de contenedor y la imagen de Docker, **ejecute** [az webapp config container set](/cli/azure/webapp/config/container#az_webapp_config_container_set).

# <a name="azure-container-registry"></a>[Azure Container Registry](#tab/acr)

```azurecli-interactive
az webapp config container set --name <app-name> --resource-group <group-name> --docker-custom-image-name '<image>:<tag>' --docker-registry-server-url 'https://<registry-name>.azurecr.io' --docker-registry-server-user '<username>' --docker-registry-server-password '<password>'
```

# <a name="docker-hub"></a>[Docker Hub](#tab/dockerhub)

```azurecli-interactive
# Public image
az webapp config container set --name <app-name> --resource-group <group-name> --docker-custom-image-name <image-name>

# Private image
az webapp config container set --name <app-name> --resource-group <group-name> --docker-custom-image-name <image-name> --docker-registry-server-user <username> --docker-registry-server-password <password>
```

# <a name="private-registry"></a>[Registro privado](#tab/private)

```azurecli-interactive
az webapp config container set --name <app-name> --resource-group <group-name> --docker-custom-image-name '<image>:<tag>' --docker-registry-server-url <private-repo-url> --docker-registry-server-user <username> --docker-registry-server-password <password>
```

-----

::: zone pivot="container-linux"
Para configurar una aplicación de varios contenedores (Docker Compose), **prepare** un archivo de Docker Compose localmente y, después, **ejecute** [az webapp config container set](/cli/azure/webapp/config/container#az_webapp_config_container_set) con el parámetro `--multicontainer-config-file`. Si el archivo de Docker Compose contiene imágenes privadas, **agregue** los parámetros `--docker-registry-server-*` como se muestra en el ejemplo anterior.

```azurecli-interactive
az webapp config container set --resource-group <group-name> --name <app-name> --multicontainer-config-file <docker-compose-file>
```
::: zone-end

Para configurar CI/CD desde el registro de contenedor para la aplicación, **ejecute** [az webapp deployment container config](/cli/azure/webapp/deployment/container#az_webapp_deployment-container-config) con el parámetro `--enable-cd`. El comando genera la dirección URL del webhook, pero debe crear el webhook en el registro manualmente en un paso independiente. En el ejemplo siguiente, se habilita CI/CD en la aplicación y, a continuación, se usa la dirección URL del webhook en la salida para crear el webhook en Azure Container Registry.

```azurecli-interactive
ci_cd_url=$(az webapp deployment container config --name <app-name> --resource-group <group-name> --enable-cd true --query CI_CD_URL --output tsv)

az acr webhook create --name <webhook-name> --registry <registry-name> --resource-group <group-name> --actions push --uri $ci_cd_url --scope '<image>:<tag>'
```

## <a name="more-resources"></a>Más recursos

* [Azure Container Registry](https://azure.microsoft.com/services/container-registry/)
* [Creación de una aplicación web de .NET Core en App Service en Linux](quickstart-dotnetcore.md)
* [Inicio rápido: Ejecución de un contenedor personalizado en App Service](quickstart-custom-container.md)
* [Peguntas más frecuentes sobre App Service en Linux](faq-app-service-linux.md)
* [Configurar contenedores personalizados](configure-custom-container.md)
* [Flujos de trabajo de acciones para implementar en Azure](https://github.com/Azure/actions-workflow-samples)
