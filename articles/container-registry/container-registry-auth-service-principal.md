---
title: Autenticación con una entidad de servicio
description: Proporcione acceso a las imágenes de su registro de contenedor privado mediante una entidad de servicio de Azure Active Directory.
ms.topic: article
ms.date: 03/15/2021
ms.openlocfilehash: a32538e5fc5354427bafc5098634becdcedd1239
ms.sourcegitcommit: b8995b7dafe6ee4b8c3c2b0c759b874dff74d96f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/03/2021
ms.locfileid: "106285542"
---
# <a name="azure-container-registry-authentication-with-service-principals"></a>Autenticación de Azure Container Registry con entidades de servicio

Puede usar una entidad de servicio de Azure Active Directory (Azure AD) para proporcionar el acceso de inserción, extracción u otro para su registro de contenedor. Al usar una entidad de servicio, podrá proporcionar acceso a aplicaciones y servicios de "equipos sin periféricos".

## <a name="what-is-a-service-principal"></a>¿Qué es una entidad de servicio?

Las *entidades de servicio* de Azure AD proporcionan acceso a los recursos de Azure de su suscripción. Las entidades de servicio son como identidades de usuario para un servicio, donde "servicio" equivale a cualquier aplicación, servicio o plataforma que necesita acceder a los recursos. Puede configurar una entidad de servicio con derechos de acceso limitados solo a aquellos recursos que especifique. A continuación, configure su aplicación o servicio para usar las credenciales de la entidad de servicio y acceder a dichos recursos.

En el contexto de Azure Container Registry, puede crear una entidad de servicio de Azure AD con permisos de extracción, inserción y extracción u otros permisos para su registro de privado de Azure. Para obtener una lista completa, consulte [Roles y permisos de Azure Container Registry](container-registry-roles.md).

## <a name="why-use-a-service-principal"></a>Razones para usar una entidad de servicio

Al usar una entidad de servicio de Azure AD, podrá proporcionar acceso limitado a su registro de contenedor privado. Cree distintas entidades de servicio para cada uno de sus servicios o aplicaciones, cada una con derechos personalizados de acceso a su registro. Además, puesto que es posible evitar el uso compartido de credenciales entre servicios y aplicaciones, podrá alternar las credenciales o revocar el acceso solo a la entidad principal (y, por lo tanto, a la aplicación) que elija.

Por ejemplo, configure su aplicación web para usar una entidad de servicio que solo proporcione acceso a la imagen `pull`, mientras el sistema de compilación utiliza una entidad de servicio que proporciona acceso a las imágenes `push` y `pull`. Si el desarrollo de su aplicación cambia de manos, puede alternar las credenciales de entidad de servicio sin que ello afecte al sistema de compilación.

## <a name="when-to-use-a-service-principal"></a>Cuándo usar una entidad de servicio

Debe usar una entidad de servicio para proporcionar acceso al registro en **escenarios de equipos sin periféricos**. Es decir, para cualquier aplicación, servicio o script que deba insertar o extraer imágenes de contenedor de forma automática o desatendida. Por ejemplo:

  * *Extracción*: se implementan los contenedores desde un Registro en sistemas de orquestación como son Kubernetes, DC/OS y Docker Swarm. También puede incorporar los cambios desde registros de contenedor a servicios de Azure relacionados, como son [Azure Kubernetes Service (AKS)](../aks/cluster-container-registry-integration.md), [Azure Container Instances](container-registry-auth-aci.md), [App Service](../app-service/index.yml), [Batch](../batch/index.yml), [Service Fabric](../service-fabric/index.yml) y otros.

  * *Inserción*: se crean imágenes de contenedor y se insertan en un Registro con soluciones de integración e implementación continuas, por ejemplo, Azure Pipelines o Jenkins.

Para obtener acceso individual a un registro, como, por ejemplo, al extraer manualmente una imagen de contenedor en la estación de trabajo de desarrollo, se recomienda usar su propia [identidad de Azure AD](container-registry-authentication.md#individual-login-with-azure-ad) para acceder al registro (por ejemplo, con [az acr login][az-acr-login]).

[!INCLUDE [container-registry-service-principal](../../includes/container-registry-service-principal.md)]

### <a name="sample-scripts"></a>Muestras de scripts

Puede encontrar los scripts del ejemplo anterior para la CLI de Azure en GitHub, así como versiones para Azure PowerShell:

* [CLI de Azure][acr-scripts-cli]
* [Azure PowerShell][acr-scripts-psh]

## <a name="authenticate-with-the-service-principal"></a>Autenticación con la entidad de servicio

Cuando disponga de una entidad de servicio con acceso a su registro de contenedor, podrá configurar sus credenciales para acceder a aplicaciones y servicios sin periféricos, o especificarlos mediante el comando `docker login`. Use los valores siguientes:

* **Nombre de usuario**: identificador de aplicación de la entidad de servicio (también denominado *identificador de cliente*)
* **Contraseña**: contraseña de la entidad de servicio (también denominada *secreto de cliente*)

Cada valor tiene el formato `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`. 

> [!TIP]
> Puede volver a generar la contraseña de una entidad de servicio mediante el comando [az ad sp reset-credentials](/cli/azure/ad/sp/credential#az-ad-sp-credential-reset).
>

### <a name="use-credentials-with-azure-services"></a>Uso de credenciales con los servicios de Azure

Puede usar las credenciales de la entidad de servicio desde cualquier servicio de Azure que pueda autenticarse en un registro de contenedor de Azure.  Use las credenciales de la entidad de servicio en lugar de las credenciales de administrador del registro para una variedad de escenarios.

Por ejemplo, use las credenciales para extraer una imagen de un registro de contenedor de Azure para [Azure Container Instances](container-registry-auth-aci.md).

### <a name="use-with-docker-login"></a>Uso con el inicio de sesión de Docker

Puede ejecutar `docker login` mediante una entidad de servicio. En el ejemplo siguiente, el identificador de aplicación de la entidad de servicio se pasa en la variable de entorno `$SP_APP_ID` y la contraseña en la variable `$SP_PASSWD`. Si quiere ver los procedimientos recomendados para administrar credenciales de Docker, consulte la referencia del comando [docker login](https://docs.docker.com/engine/reference/commandline/login/).

```bash
# Log in to Docker with service principal credentials
docker login myregistry.azurecr.io --username $SP_APP_ID --password $SP_PASSWD
```

Una vez iniciada la sesión, Docker almacena las credenciales en caché.

### <a name="use-with-certificate"></a>Uso con certificado

Si ha agregado un certificado a la entidad de servicio, puede iniciar sesión en la CLI de Azure con la autenticación basada en certificados y, a continuación, puede usar el comando [az acr login][az-acr-login] para acceder a un registro. El uso de un certificado como secreto en lugar de una contraseña proporciona seguridad adicional cuando se usa la CLI. 

Puede crear un certificado autofirmado durante la [creación de una entidad de servicio](/cli/azure/create-an-azure-service-principal-azure-cli). O bien, puede agregar uno o varios certificados a una entidad de servicio existente. Por ejemplo, si usa uno de los scripts de este artículo para crear o actualizar una entidad de servicio con derechos para extraer o insertar imágenes de un registro, agregue un certificado mediante el comando [az ad sp credential reset][az-ad-sp-credential-reset].

Para usar la entidad de servicio con el certificado para [iniciar sesión en la CLI de Azure](/cli/azure/authenticate-azure-cli#sign-in-with-a-service-principal), el certificado debe tener formato PEM e incluir la clave privada. Si el certificado no tiene el formato requerido, use una herramienta como `openssl` para convertirlo. Al ejecutar [az login][az-login] para iniciar sesión en la CLI con la entidad de servicio, proporcione también el identificador de aplicación de la entidad de servicio y el identificador de inquilino de Active Directory. En el ejemplo siguiente se muestran estos valores como variables de entorno:

```azurecli
az login --service-principal --username $SP_APP_ID --tenant $SP_TENANT_ID  --password /path/to/cert/pem/file
```

A continuación, ejecute [az acr login][az-acr-login] para autenticarse con el registro:

```azurecli
az acr login --name myregistry
```

La CLI usa el token creado cuando se ejecutó `az login` para autenticar su sesión con el registro.

## <a name="next-steps"></a>Pasos siguientes

* Consulte [Información general sobre la autenticación](container-registry-authentication.md) para ver otros escenarios de autenticación con Azure Container Registry.

* Si desea ver un ejemplo del uso de Azure Key Vault para almacenar y recuperar las credenciales de la entidad de servicio de un registro de contenedor, consulte [Tutorial: Compilación e implementación de imágenes de contenedor con ACR Tasks](container-registry-tutorial-quick-task.md).

<!-- LINKS - External -->
[acr-scripts-cli]: https://github.com/Azure/azure-docs-cli-python-samples/tree/master/container-registry
[acr-scripts-psh]: https://github.com/Azure/azure-docs-powershell-samples/tree/master/container-registry

<!-- LINKS - Internal -->
[az-acr-login]: /cli/azure/acr#az-acr-login
[az-login]: /cli/azure/reference-index#az-login
[az-ad-sp-credential-reset]: /cli/azure/ad/sp/credential#az-ad-sp-credential-reset
