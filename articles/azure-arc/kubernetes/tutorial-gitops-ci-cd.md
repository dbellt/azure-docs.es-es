---
title: 'Tutorial: Implementación de CI/CD con GitOps mediante clústeres de Kubernetes habilitados para Azure Arc'
description: Este tutorial le guía en la configuración de una solución de CI/CD con GitOps mediante clústeres de Kubernetes habilitados para Azure Arc. Para ver una introducción conceptual sobre este flujo de trabajo, consulte el artículo Flujo de trabajo de CI/CD con GitOps mediante Kubernetes habilitado para Azure Arc.
author: tcare
ms.author: tcare
ms.service: azure-arc
ms.topic: tutorial
ms.date: 03/03/2021
ms.custom: template-tutorial, devx-track-azurecli
ms.openlocfilehash: 3d7b88007a27b05119ebe93217c64279c8c541ff
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110373411"
---
# <a name="tutorial-implement-cicd-with-gitops-using-azure-arc-enabled-kubernetes-clusters"></a>Tutorial: Implementación de CI/CD con GitOps mediante clústeres de Kubernetes habilitados para Azure Arc


En este tutorial va a configurar una solución de CI/CD con GitOps mediante clústeres de Kubernetes habilitados para Azure Arc. Con la aplicación de votaciones de ejemplo de Azure llevará a cabo las siguientes tareas:

> [!div class="checklist"]
> * Crear un clúster de Kubernetes habilitado para Azure Arc.
> * Conectar los repositorios de la aplicación y de GitOps a Azure Repos.
> * Importar canalizaciones de CI/CD.
> * Conectar la instancia de Azure Container Registry (ACR) a Azure DevOps y Kubernetes.
> * Crear grupos de variables de entorno.
> * Implementar los entornos `dev` y `stage`.
> * Probar los entornos de la aplicación.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="before-you-begin"></a>Antes de empezar

En este tutorial se da por supuesto que está familiarizado con Azure DevOps, Azure Repos, Azure Pipelines y la CLI de Azure.

* Inicie sesión en [Azure DevOps Services](https://dev.azure.com/).
* Complete el [tutorial anterior](./tutorial-use-gitops-connected-cluster.md) para obtener información sobre cómo implementar GitOps para su entorno de CI/CD.
* Conozca las [ventajas y arquitectura](./conceptual-configurations.md) de esta característica.
* Compruebe que tiene:
  * Un [clúster de Kubernetes conectado habilitado para Azure Arc](./quickstart-connect-cluster.md#3-connect-an-existing-kubernetes-cluster) con el nombre **arc-cicd-cluster**.
  * Una instancia conectada de Azure Container Registry (ACR) con la [integración de AKS](../../aks/cluster-container-registry-integration.md) o la [autenticación de un clúster que no es de AKS](../../container-registry/container-registry-auth-kubernetes.md).
  * Permisos "Administrador de compilación" y "Administrador de proyectos" para [Azure Repos](/azure/devops/repos/get-started/what-is-repos) y [Azure Pipelines](/azure/devops/pipelines/get-started/pipelines-get-started).
* Instale las siguientes extensiones de versiones de la CLI de Kubernetes habilitado para Azure Arc >= 1.0.0:

  ```azurecli
  az extension add --name connectedk8s
  az extension add --name k8sconfiguration
  ```
  * Para actualizar las extensiones a la versión más reciente, ejecute los siguientes comandos:

    ```azurecli
    az extension update --name connectedk8s
    az extension update --name k8sconfiguration
    ```

## <a name="import-application-and-gitops-repos-into-azure-repos"></a>Importación de los repositorios de la aplicación y de GitOps en Azure Repos

Importe un [repositorio de aplicación](./conceptual-gitops-ci-cd.md#application-repo) y un [repositorio de GitOps](./conceptual-gitops-ci-cd.md#gitops-repo) en Azure Repos. Para este tutorial, use los siguientes repositorios de ejemplo:

* Repositorio de la aplicación **arc-cicd-demo-src**
   * Dirección URL: https://github.com/Azure/arc-cicd-demo-src
   * Contiene la aplicación de votaciones de Azure de ejemplo que se va a implementar con GitOps.
* Repositorio de GitOps **arc-cicd-demo-gitops**
   * Dirección URL: https://github.com/Azure/arc-cicd-demo-gitops
   * Funciona como base para los recursos del clúster que hospedan la aplicación de votaciones de Azure.

Más información en [Importación de un repositorio de Git](/azure/devops/repos/git/import-git-repository).

>[!NOTE]
> La importación y el uso de dos repositorios diferentes para la aplicación y los repositorios de GitOps puede mejorar la seguridad y la simplicidad. Los permisos y la visibilidad de los repositorios de la aplicación y de GitOps se pueden optimizar individualmente.
> Por ejemplo, es posible que el administrador del clúster no encuentre los cambios en el código de la aplicación correspondientes al estado deseado del clúster. Por el contrario, un desarrollador de aplicaciones no necesita conocer los parámetros específicos de cada entorno: un conjunto de valores de prueba que proporcione cobertura para los parámetros puede ser suficiente.

## <a name="connect-the-gitops-repo"></a>Conexión del repositorio de GitOps

Para la implementación continua de la aplicación, conecte el repositorio de la aplicación al clúster mediante GitOps. El repositorio de GitOps **arc-cicd-demo-gitops** contiene los recursos básicos para poner en funcionamiento la aplicación en el clúster **arc-cicd-cluster**.

El repositorio inicial de GitOps solo contiene un [manifiesto](https://github.com/Azure/arc-cicd-demo-gitops/blob/master/arc-cicd-cluster/manifests/namespaces.yml) que crea los espacios de nombres **dev** y **stage** correspondientes a los entornos de implementación.

La conexión de GitOps que va a crear realizará automáticamente las siguientes acciones:
* Sincronizar los manifiestos en el directorio de manifiestos.
* Actualizar el estado del clúster.

El flujo de trabajo de CI/CD rellenará el directorio de manifiestos con manifiestos adicionales para implementar la aplicación.


1. [Cree una nueva conexión de GitOps](./tutorial-use-gitops-connected-cluster.md) al repositorio **arc-cicd-demo-gitops** recién importado en Azure Repos.

   ```azurecli
   az k8sconfiguration create \
      --name cluster-config \
      --cluster-name arc-cicd-cluster \
      --resource-group myResourceGroup \
      --operator-instance-name cluster-config \
      --operator-namespace cluster-config \
      --repository-url https://dev.azure.com/<Your organization>/arc-cicd-demo-gitops \
      --https-user <Azure Repos username> \
      --https-key <Azure Repos PAT token> \
      --scope cluster \
      --cluster-type connectedClusters \
      --operator-params='--git-readonly --git-path=arc-cicd-cluster/manifests'
   ```

1. Asegúrese de que Flux *solo* usa el directorio `arc-cicd-cluster/manifests` como ruta de acceso base. Defina la ruta de acceso mediante el siguiente parámetro de operador:

   `--git-path=arc-cicd-cluster/manifests`

   > [!NOTE]
   > Si usa una cadena de conexión HTTPS y tiene problemas de conexión, asegúrese de omitir el prefijo de nombre de usuario en la dirección URL. Por ejemplo, se debe quitar `alice@` de `https://alice@dev.azure.com/contoso/arc-cicd-demo-gitops`. En su lugar, el parámetro `--https-user` especifica el usuario, por ejemplo `--https-user alice`.

1. Compruebe el estado de la implementación en Azure Portal.
   * Si se realiza correctamente, verá que se han creado en el clúster los espacios de nombres `dev` y `stage`.

## <a name="import-the-cicd-pipelines"></a>Importación de las canalizaciones de CI/CD

Ahora que ha sincronizado una conexión de GitOps, deberá importar las canalizaciones de CI/CD que crean los manifiestos.

El repositorio de la aplicación contiene la carpeta `.pipeline` con las canalizaciones que va a utilizar para las solicitudes de incorporación de cambios (PR), CI y CD. Importe y cambie el nombre de las tres canalizaciones proporcionadas en el repositorio de ejemplo:

| Nombre del archivo de la canalización. | Descripción |
| ------------- | ------------- |
| [`.pipelines/az-vote-pr-pipeline.yaml`](https://github.com/Azure/arc-cicd-demo-src/blob/master/.pipelines/az-vote-pr-pipeline.yaml)  | Canalización de PR de la aplicación, llamada **arc-cicd-demo-src PR** |
| [`.pipelines/az-vote-ci-pipeline.yaml`](https://github.com/Azure/arc-cicd-demo-src/blob/master/.pipelines/az-vote-ci-pipeline.yaml) | Canalización de CI de la aplicación, llamada **arc-cicd-demo-src CI** |
| [`.pipelines/az-vote-cd-pipeline.yaml`](https://github.com/Azure/arc-cicd-demo-src/blob/master/.pipelines/az-vote-cd-pipeline.yaml) | Canalización de CD de la aplicación, llamada **arc-cicd-demo-src CD** |



## <a name="connect-your-acr"></a>Conexión de la instancia de ACR
Tanto las canalizaciones como el clúster utilizarán ACR para almacenar y recuperar imágenes de Docker.

### <a name="connect-acr-to-azure-devops"></a>Conexión de ACR a Azure DevOps
Durante el proceso de CI, implementará los contenedores de la aplicación en un registro. Empiece por crear una conexión de servicio de Azure:

1. En Azure DevOps, abra la página **Conexiones de servicio** desde la página de configuración del proyecto. En TFS, abra la página **Servicios** desde el icono de **Configuración** en la barra de menús superior.
2. Elija **+ Nueva conexión de servicio** y seleccione el tipo de conexión de servicio que necesite.
3. Rellene los parámetros para la conexión de servicio. En este tutorial:
   * Asigne el nombre **arc-demo-acr** a la conexión de servicio. 
   * Seleccione **myResourceGroup** como grupo de recursos.
4. Seleccione **Conceder permiso de acceso a todas las canalizaciones**. 
   * Esta opción autoriza los archivos de canalización de YAML para las conexiones de servicio. 
5. Elija **Aceptar** para crear la conexión.

### <a name="connect-acr-to-kubernetes"></a>Conexión de ACR a Kubernetes
Habilite el clúster de Kubernetes para extraer imágenes de la instancia de ACR. Si es privada, se requerirá autenticación.

#### <a name="connect-acr-to-existing-aks-clusters"></a>Conexión de ACR a los clústeres de AKS existentes

Integre una instancia de ACR existente con los clústeres de AKS existentes mediante el siguiente comando:

```azurecli
az aks update -n arc-cicd-cluster -g myResourceGroup --attach-acr arc-demo-acr
```

#### <a name="create-an-image-pull-secret"></a>Creación de un secreto de extracción de imágenes

Para conectar los clústeres locales y los que no son de AKS a la instancia de ACR, cree un secreto de extracción de imágenes. Kubernetes usa secretos de extracción de imágenes para almacenar la información necesaria para autenticarse en el registro.

Cree un secreto de extracción de imágenes con el siguiente comando `kubectl`: Repita el procedimiento con los espacios de nombres `dev` y `stage`.
```console
kubectl create secret docker-registry <secret-name> \
    --namespace <namespace> \
    --docker-server=<container-registry-name>.azurecr.io \
    --docker-username=<service-principal-ID> \
    --docker-password=<service-principal-password>
```

Para evitar tener que establecer un secreto de extracción de imágenes para cada pod, considere la posibilidad de agregar el secreto de extracción de imágenes a la cuenta de servicio en los espacios de nombres `dev` y `stage`. Consulte el [tutorial de Kubernetes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/#add-imagepullsecrets-to-a-service-account) para más información.

## <a name="create-environment-variable-groups"></a>Creación de grupos de variables de entorno

### <a name="app-repo-variable-group"></a>Grupo de variables del repositorio de la aplicación
[Cree un grupo de variables](/azure/devops/pipelines/library/variable-groups) llamado **az-vote-app-dev**. Establezca los valores siguientes:

| Variable | Value |
| -------- | ----- |
| AZ_ACR_NAME | (instancia de ACR, por ejemplo, azurearctest.azurecr.io) |
| AZURE_SUBSCRIPTION | (conexión de servicio de Azure, que debe ser **arc-demo-acr**, creada anteriormente en el tutorial) |
| AZURE_VOTE_IMAGE_REPO | Ruta de acceso completa al repositorio de la aplicación de votaciones de Azure, por ejemplo, azurearctest.azurecr.io/azvote |
| ENVIRONMENT_NAME | Desarrollo |
| MANIFESTS_BRANCH | `master` |
| MANIFESTS_REPO | Cadena de conexión de Git para el repositorio de GitOps |
| ORGANIZATION_NAME | Nombre de la organización de Azure DevOps |
| PROJECT_NAME | Nombre del proyecto de GitOps en Azure DevOps |
| REPO_URL | Dirección URL completa del repositorio de GitOps |
| SRC_FOLDER | `azure-vote` | 
| TARGET_CLUSTER | `arc-cicd-cluster` |
| TARGET_NAMESPACE | `dev` |

### <a name="stage-environment-variable-group"></a>Grupo de variables del entorno de fase

1. Clone el grupo de variables **az-vote-app-dev**.
1. Cambie el nombre a **az-vote-app-stage**.
1. Asegúrese de establecer los siguientes valores para las variables correspondientes:

| Variable | Value |
| -------- | ----- |
| ENVIRONMENT_NAME | Fase |
| TARGET_NAMESPACE | `stage` |

Ahora está listo para realizar la implementación en los entornos `dev` y `stage`.

## <a name="give-more-permissions-to-the-build-service"></a>Concesión de más permisos al servicio de compilación
La canalización de CD usa el token de seguridad de la compilación en ejecución para autenticarse en el repositorio de GitOps. Se necesitan más permisos para que la canalización cree una nueva rama, inserte cambios y cree solicitudes de extracción.

1. Vaya a `Project settings` desde la página principal del proyecto de Azure DevOps.
1. Seleccione `Repositories`.
1. Seleccione `<GitOps Repo Name>`.
1. Seleccione `Security`. 
1. Para `<Project Name> Build Service (<Organization Name>)`, permita `Contribute`, `Contribute to pull requests` y `Create branch`.

Para más información, consulte:
- [Concesión de permisos de VC al servicio de compilación](https://docs.microsoft.com/azure/devops/pipelines/scripts/git-commands?view=azure-devops&tabs=yaml&preserve-view=true#version-control )
- [Administración de permisos de cuenta de servicio de compilación](https://docs.microsoft.com/azure/devops/pipelines/process/access-tokens?view=azure-devops&tabs=yaml&preserve-view=true#manage-build-service-account-permissions)


## <a name="deploy-the-dev-environment-for-the-first-time"></a>Implementación en el entorno de desarrollo por primera vez
Con las canalizaciones de CI y CD creadas, ejecute la canalización de CI para implementar la aplicación por primera vez.

### <a name="ci-pipeline"></a>Canalización de CI

Durante la ejecución inicial de la canalización de CI, puede obtener un error de autorización de recursos al leer el nombre de la conexión de servicio.
1. Compruebe que la variable a la que se accede es AZURE_SUBSCRIPTION.
1. Autorice el uso.
1. vuelva a ejecutar la canalización.

La canalización de CI:
* Garantiza que el cambio de la aplicación supera todas las comprobaciones de calidad automatizadas para la implementación.
* Realiza cualquier validación adicional que no se pudo completar en la canalización de PR.
    * Específicamente para GitOps, la canalización también publica los artefactos para la confirmación que se implementará mediante la canalización de CD.
* Comprueba que la imagen de Docker ha cambiado y que se ha insertado la nueva imagen.

### <a name="cd-pipeline"></a>Canalización de CD
Durante la ejecución inicial de la canalización de CD, se le pedirá que dé acceso a la canalización al repositorio de GitOps. Seleccione Ver cuando se le indique que la canalización necesita permiso para acceder a un recurso. A continuación, seleccione Permitir para conceder permiso para usar el repositorio de GitOps para las ejecuciones actuales y futuras de la canalización.

La ejecución correcta de la canalización de CI desencadena la canalización de CD para completar el proceso de implementación. Realizará la implementación en cada entorno de forma incremental.

> [!TIP]
> Si la canalización de CD no se desencadena automáticamente:
> 1. Compruebe que el nombre coincide con el desencadenador de rama en [`.pipelines/az-vote-cd-pipeline.yaml`](https://github.com/Azure/arc-cicd-demo-src/blob/master/.pipelines/az-vote-cd-pipeline.yaml).
>    * Debería ser `arc-cicd-demo-src CI`.
> 1. Vuelva a ejecutar la canalización de CI.

Una vez que se hayan generado los cambios de plantilla y de manifiesto en el repositorio de GitOps, la canalización de CD creará una confirmación, la insertará y creará una solicitud de incorporación de cambios para su aprobación.
1. Abra el vínculo de PR proporcionado en la salida de la tarea `Create PR`.
1. Compruebe los cambios en el repositorio de GitOps. Debería ver lo siguiente:
   * La plantilla de Helm de alto nivel ha cambiado.
   * Los manifiestos de Kubernetes de bajo nivel que muestran los cambios subyacentes en el estado deseado. Flux implementa estos manifiestos.
1. Si todo parece correcto, apruebe y complete la solicitud de incorporación de cambios.

1. Después de unos minutos, Flux recoge el cambio e inicia la implementación.
1. Reenvíe el puerto de forma local mediante `kubectl` y asegúrese de que la aplicación funciona correctamente mediante:

   `kubectl port-forward -n dev svc/azure-vote-front 8080:80`

1. Vaya a la aplicación de votaciones de Azure en el explorador en `http://localhost:8080/`.

1. Vote a sus favoritos y prepárese para realizar algunos cambios en la aplicación.

## <a name="set-up-environment-approvals"></a>Configuración de las aprobaciones de entorno
Tras la implementación de la aplicación, no solo puede realizar cambios en el código o las plantillas, sino que también puede poner involuntariamente el clúster en un estado incorrecto.

Si el entorno de desarrollo revela una interrupción después de la implementación, evite que siga en entornos posteriores con las aprobaciones de entorno.

1. En el proyecto de Azure DevOps, vaya al entorno que se va a proteger.
1. Vaya a **Aprobaciones y comprobaciones** en el recurso.
1. Seleccione **Crear**.
1. Proporcione los aprobadores y un mensaje opcional.
1. Seleccione de nuevo **Crear** para completar la adición de la comprobación de aprobación manual.

Para más información, consulte el tutorial [Definición de aprobaciones y comprobaciones](/azure/devops/pipelines/process/approvals).

La próxima vez que se ejecute la canalización de CD, la canalización se pondrá en pausa después de la creación de la PR de GitOps. Compruebe que el cambio se haya sincronizado correctamente y de que supera la funcionalidad básica. Apruebe la comprobación desde la canalización para permitir que el cambio fluya al siguiente entorno.

## <a name="make-an-application-change"></a>Realización de un cambio en la aplicación

Con este conjunto de plantillas y manifiestos de línea base que representan el estado del clúster, va a realizar un pequeño cambio en la aplicación.

1. En el repositorio **arc-cicd-demo-src**, edite el archivo [`azure-vote/src/azure-vote-front/config_file.cfg`](https://github.com/Azure/arc-cicd-demo-src/blob/master/azure-vote/src/azure-vote-front/config_file.cfg).

2. Como "gatos frente a perros" no está obteniendo suficientes votos, cámbielo a "tabuladores frente a espacios" para impulsar la participación.

3. Confirme el cambio en una nueva rama, insértelo y cree una solicitud de incorporación de cambios.
   * Este es el flujo de desarrollador típico que iniciará el ciclo de vida de CI/CD.

## <a name="pr-validation-pipeline"></a>Canalización de validación de PR

La canalización de PR es la primera línea de defensa contra un cambio erróneo. Las comprobaciones de la calidad del código de la aplicación habituales incluyen el linting y el análisis estático. Desde la perspectiva de GitOps, también debe asegurarse de la misma calidad para la infraestructura resultante que se va a implementar.

El archivo Dockerfile y los gráficos de Helm de la aplicación pueden usar el linting de una manera similar a la aplicación.

Los errores encontrados durante el linting pueden ir desde:
* Archivos YAML con formato incorrecto, hasta
* Sugerencias de procedimientos recomendados, como la configuración de límites de CPU y memoria para la aplicación.

> [!NOTE]
> Para obtener la mejor cobertura del linting de Helm en una aplicación real, deberá sustituir los valores que sean razonablemente similares a los que se usan en un entorno real.

Los errores encontrados durante la ejecución de la canalización aparecen en la sección de resultados de las pruebas de la ejecución. Desde aquí, puede:
* Realizar un seguimiento de las estadísticas de utilidad sobre tipos de error.
* Buscar la primera confirmación en la que se detectaron.
* Realizar un seguimiento de la pila de los vínculos de estilo a las secciones de código que produjeron el error.

Una vez finalizada la ejecución de la canalización, habrá asegurado la calidad del código de la aplicación y de la plantilla que lo implementará. Ahora puede aprobar y completar la solicitud de incorporación de cambios. La canalización de CI se ejecutará de nuevo, con lo que se volverán a generar las plantillas y los manifiestos antes de desencadenar la canalización de CD.

> [!TIP]
> En un entorno real, no olvide establecer directivas de rama para asegurarse de que la solicitud de incorporación de cambios supera las comprobaciones de calidad. Para más información, consulte el artículo [Mejora de la calidad del código con directivas de rama](/azure/devops/repos/git/branch-policies).

## <a name="cd-process-approvals"></a>Aprobaciones del proceso de CD

La ejecución correcta de la canalización de CI desencadena la canalización de CD para completar el proceso de implementación. De forma similar a la primera vez que ejecutó la canalización de CD, va a realizar la implementación en cada entorno de forma incremental. Esta vez, la canalización requiere que apruebe cada entorno de implementación.

1. Apruebe la implementación en el entorno `dev`.
1. Una vez que se hayan generado los cambios de plantilla y de manifiesto en el repositorio de GitOps, la canalización de CD creará una confirmación, la insertará y creará una solicitud de incorporación de cambios para su aprobación.
1. Abra el vínculo de PR proporcionado en la tarea.
1. Compruebe los cambios en el repositorio de GitOps. Debería ver lo siguiente:
   * La plantilla de Helm de alto nivel ha cambiado.
   * Los manifiestos de Kubernetes de bajo nivel que muestran los cambios subyacentes en el estado deseado.
1. Si todo parece correcto, apruebe y complete la solicitud de incorporación de cambios.
1. Espere a que la implementación se complete.
1. Como prueba de aceptación de la compilación básica, vaya a la página de la aplicación y compruebe que la aplicación de votación ahora muestra la encuesta Tabuladores frente a espacios.
   * Reenvíe el puerto de forma local mediante `kubectl` y asegúrese de que la aplicación funciona correctamente mediante: `kubectl port-forward -n dev svc/azure-vote-front 8080:80`
   * Vaya a la aplicación de votaciones de Azure en el explorador en http://localhost:8080/ y compruebe que las opciones de votación han cambiado a la encuesta Tabuladores frente a espacios. 
1. Repita los pasos del 1 al 7 para el entorno `stage`.

La implementación ha finalizado. Esto finaliza el flujo de trabajo de CI/CD.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si no va a seguir usando esta aplicación, elimine los recursos mediante los siguientes pasos:

1. Elimine la conexión de configuración de GitOps de Azure Arc:
   ```azurecli
   az k8sconfiguration delete \
   --name cluster-config \
   --cluster-name arc-cicd-cluster \
   --resource-group myResourceGroup \
   --cluster-type connectedClusters
   ```

2. Quite el espacio de nombres `dev`:
   * `kubectl delete namespace dev`

3. Quite el espacio de nombres `stage`:
   * `kubectl delete namespace stage`

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha configurado un flujo de trabajo completo de CI/CD que implementa DevOps desde el desarrollo de la aplicación hasta la implementación. Los cambios en la aplicación desencadenan automáticamente la validación y la implementación, que se vigilan mediante aprobaciones manuales.

Avance a nuestro artículo conceptual para más información sobre GitOps y configuraciones con Kubernetes habilitado para Azure Arc.

> [!div class="nextstepaction"]
> [Flujo de trabajo de CI/CD con GitOps: Kubernetes habilitado para Azure Arc](./conceptual-gitops-ci-cd.md)