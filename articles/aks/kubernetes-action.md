---
title: Compilación, prueba e implementación de contenedores en Azure Kubernetes Service mediante Acciones de GitHub
description: Más información sobre cómo usar Acciones de GitHub para implementar el contenedor en Kubernetes
services: container-service
author: azooinmyluggage
ms.topic: article
ms.date: 11/06/2020
ms.author: atulmal
ms.custom: github-actions-azure
ms.openlocfilehash: 2539591b7c425750263653d137f904758982ca62
ms.sourcegitcommit: 1b19b8d303b3abe4d4d08bfde0fee441159771e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2021
ms.locfileid: "109751894"
---
# <a name="github-actions-for-deploying-to-kubernetes-service"></a>Acciones de GitHub para la implementación en el servicio Kubernetes

[Acciones de GitHub](https://docs.github.com/en/actions) le ofrece la flexibilidad de compilar un flujo de trabajo del ciclo de vida de desarrollo de software automatizado. Puede usar varias acciones de Kubernetes para realizar implementaciones en contenedores desde el Azure Container Registry al servicio de Azure Kubernetes con Acciones de GitHub. 

## <a name="prerequisites"></a>Prerrequisitos 

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Una cuenta de GitHub. Si no tiene ninguna, regístrese [gratis](https://github.com/join).  
- Un clúster de Kubernetes en funcionamiento
    - [Tutorial: Preparación de una aplicación para Azure Kubernetes Service](tutorial-kubernetes-prepare-app.md)

## <a name="workflow-file-overview"></a>Información general sobre el archivo de flujo de trabajo

Un archivo YAML (.yml) define un flujo de trabajo en la ruta de acceso `/.github/workflows/` de su repositorio. En esta definición se incluyen los diversos pasos y parámetros que componen el flujo de trabajo.

Para un flujo de trabajo que tiene como destino AKS, el archivo tiene tres secciones:

|Sección  |Tareas  |
|---------|---------|
|**Autenticación** | Inicio de sesión en un registro de contenedor privado (ACR) |
|**Compilar** | Compilación e inserción de la imagen de contenedor en el registro  |
|**Implementación** | 1. Establezca el clúster de AKS de destino |
| |2. Cree un secreto genérico o del registro de Docker en el clúster de Kubernetes  |
||3. Implemente el clúster de Kubernetes|

## <a name="create-a-service-principal"></a>Creación de una entidad de servicio

Puede crear una [entidad de servicio](../active-directory/develop/app-objects-and-service-principals.md#service-principal-object) mediante el comando [az ad sp create-for-rbac](/cli/azure/ad/sp#az_ad_sp_create_for_rbac) de la [CLI de Azure](/cli/azure/). Puede ejecutar este comando mediante [Azure Cloud Shell](https://shell.azure.com/) en Azure Portal o seleccionando el botón **Pruébelo**.

```azurecli-interactive
az ad sp create-for-rbac --name "myApp" --role contributor --scopes /subscriptions/<SUBSCRIPTION_ID>/resourceGroups/<RESOURCE_GROUP> --sdk-auth
```

En el comando anterior, reemplace los marcadores de posición por el identificador de la suscripción y el grupo de recursos. La salida son las credenciales de asignación de roles que proporcionan acceso a su aplicación. El comando debe generar un objeto JSON similar a este.

```json
  {
    "clientId": "<GUID>",
    "clientSecret": "<GUID>",
    "subscriptionId": "<GUID>",
    "tenantId": "<GUID>",
    (...)
  }
```
Copie este objeto JSON, el cual puede usar para autenticarse desde GitHub.

## <a name="configure-the-github-secrets"></a>Configuración de los secretos de GitHub

Siga los pasos para configurar los secretos:

1. En [GitHub](https://github.com/), examine el repositorio y seleccione **Configuración > Secretos > Agregar un nuevo secreto**.

    ![Captura de pantalla que muestra el vínculo Agregar un nuevo secreto para un repositorio.](media/kubernetes-action/secrets.png)

2. Pegue el contenido del comando `az cli` anterior como valor de la variable secreta. Por ejemplo, `AZURE_CREDENTIALS`.

3. Del mismo modo, defina los siguientes secretos adicionales para las credenciales del registro de contenedor y establézcalos en la acción de inicio de sesión de Docker. 

    - REGISTRY_USERNAME
    - REGISTRY_PASSWORD

4. Verá los secretos como se muestran a continuación una vez definidos.

    ![Captura de pantalla que muestra los secretos existentes de un repositorio.](media/kubernetes-action/kubernetes-secrets.png)

##  <a name="build-a-container-image-and-deploy-to-azure-kubernetes-service-cluster"></a>Creación de una imagen de contenedor e implementación en el clúster de Azure Kubernetes Service

La compilación y la extracción de las imágenes de contenedor se realizan mediante la acción `Azure/docker-login@v1`. 


```yml
env:
  REGISTRY_NAME: {registry-name}
  CLUSTER_NAME: {cluster-name}
  CLUSTER_RESOURCE_GROUP: {resource-group-name}
  NAMESPACE: {namespace-name}
  APP_NAME: {app-name}
  
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@main
    
    # Connect to Azure Container Registry (ACR)
    - uses: azure/docker-login@v1
      with:
        login-server: ${{ env.REGISTRY_NAME }}.azurecr.io
        username: ${{ secrets.REGISTRY_USERNAME }} 
        password: ${{ secrets.REGISTRY_PASSWORD }}
    
    # Container build and push to a Azure Container Registry (ACR)
    - run: |
        docker build . -t ${{ env.REGISTRY_NAME }}.azurecr.io/${{ env.APP_NAME }}:${{ github.sha }}
        docker push ${{ env.REGISTRY_NAME }}.azurecr.io/${{ env.APP_NAME }}:${{ github.sha }}

```

### <a name="deploy-to-azure-kubernetes-service-cluster"></a>Implementación en un clúster de Azure Kubernetes Service

Para implementar una imagen de contenedor en AKS, debe usar la acción `Azure/k8s-deploy@v1`. Esta acción tiene cinco parámetros:

| **Parámetro**  | **Explicación**  |
|---------|---------|
| **namespace** | (Opcional) Elija el espacio de nombres de Kubernetes de destino. Si no se proporciona el espacio de nombres, los comandos se ejecutarán en el espacio de nombres predeterminado. | 
| **manifests** |  (Obligatorio) Ruta de acceso a los archivos de manifiesto que se usará para la implementación. |
| **images** | (Obligatorio) Dirección URL completa de recursos de las imágenes que se van a usar para sustituciones sobre los archivos de manifiesto. |
| **imagepullsecrets** | (Opcional) Nombre de un secreto del registro de Docker que ya se ha configurado dentro del clúster. Cada uno de estos nombres de secreto se agrega en el campo imagePullSecrets para las cargas de trabajo que se encuentran en los archivos de manifiesto de entrada. |
| **kubectl-version** | (Opcional) Instala una versión específica del archivo binario kubectl. |

> [!NOTE]
> El usuario debe crear manualmente los archivos de manifiesto. Actualmente no hay herramientas que generen estos archivos de forma automatizada. Para obtener más información, consulte [este repositorio de ejemplo con archivos de manifiesto de ejemplo](https://github.com/MicrosoftDocs/mslearn-aks-deploy-container-app/tree/master/kubernetes).

Antes de que pueda realizar la implementación en AKS, debe establecer el espacio de nombres de Kubernetes de destino y crear un secreto de extracción de imagen. Consulte [Extracción de imágenes de Azure Container Registry a un clúster de Kubernetes](../container-registry/container-registry-auth-kubernetes.md) para obtener más información sobre cómo funciona la extracción de imágenes. 

```yaml
  # Create namespace if doesn't exist
  - run: |
      kubectl create namespace ${{ env.NAMESPACE }} --dry-run -o json | kubectl apply -f -
  
  # Create image pull secret for ACR
  - uses: azure/k8s-create-secret@v1
    with:
      container-registry-url: ${{ env.REGISTRY_NAME }}.azurecr.io
      container-registry-username: ${{ secrets.REGISTRY_USERNAME }}
      container-registry-password: ${{ secrets.REGISTRY_PASSWORD }}
      secret-name: ${{ env.SECRET }}
      namespace: ${{ env.NAMESPACE }}
      arguments: --force true
```


Complete la implementación con la acción `k8s-deploy`. Reemplace las variables de entorno por los valores para su aplicación. 

```yaml

on: [push]

# Environment variables available to all jobs and steps in this workflow
env:
  REGISTRY_NAME: {registry-name}
  CLUSTER_NAME: {cluster-name}
  CLUSTER_RESOURCE_GROUP: {resource-group-name}
  NAMESPACE: {namespace-name}
  SECRET: {secret-name}
  APP_NAME: {app-name}
  
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@main
    
    # Connect to Azure Container Registry (ACR)
    - uses: azure/docker-login@v1
      with:
        login-server: ${{ env.REGISTRY_NAME }}.azurecr.io
        username: ${{ secrets.REGISTRY_USERNAME }} 
        password: ${{ secrets.REGISTRY_PASSWORD }}
    
    # Container build and push to a Azure Container Registry (ACR)
    - run: |
        docker build . -t ${{ env.REGISTRY_NAME }}.azurecr.io/${{ env.APP_NAME }}:${{ github.sha }}
        docker push ${{ env.REGISTRY_NAME }}.azurecr.io/${{ env.APP_NAME }}:${{ github.sha }}
    
    # Set the target Azure Kubernetes Service (AKS) cluster. 
    - uses: azure/aks-set-context@v1
      with:
        creds: '${{ secrets.AZURE_CREDENTIALS }}'
        cluster-name: ${{ env.CLUSTER_NAME }}
        resource-group: ${{ env.CLUSTER_RESOURCE_GROUP }}
    
    # Create namespace if doesn't exist
    - run: |
        kubectl create namespace ${{ env.NAMESPACE }} --dry-run -o json | kubectl apply -f -
    
    # Create image pull secret for ACR
    - uses: azure/k8s-create-secret@v1
      with:
        container-registry-url: ${{ env.REGISTRY_NAME }}.azurecr.io
        container-registry-username: ${{ secrets.REGISTRY_USERNAME }}
        container-registry-password: ${{ secrets.REGISTRY_PASSWORD }}
        secret-name: ${{ env.SECRET }}
        namespace: ${{ env.NAMESPACE }}
        force: true
    
    # Deploy app to AKS
    - uses: azure/k8s-deploy@v1
      with:
        manifests: |
          manifests/deployment.yml
          manifests/service.yml
        images: |
          ${{ env.REGISTRY_NAME }}.azurecr.io/${{ env.APP_NAME }}:${{ github.sha }}
        imagepullsecrets: |
          ${{ env.SECRET }}
        namespace: ${{ env.NAMESPACE }}
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando el clúster de Kubernetes, el registro de contenedor y el repositorio ya no sean necesarios, limpie los recursos que implementó eliminando el grupo de recursos y el repositorio de GitHub. 

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Obtenga más información acerca de Azure Kubernetes Service](/azure/architecture/reference-architectures/containers/aks-start-here)

> [!div class="nextstepaction"]
> [Aprenda a crear varias canalizaciones en Acciones de GitHub con AKS](https://docs.microsoft.com/learn/modules/aks-deployment-pipeline-github-actions)

### <a name="more-kubernetes-github-actions"></a>Más Acciones de GitHub para Kubernetes

* [Instalador de la herramienta Kubectl](https://github.com/Azure/setup-kubectl) (`azure/setup-kubectl`): instale una versión específica de kubectl en el ejecutor.
* [Establecimiento del contexto en Kubernetes](https://github.com/Azure/k8s-set-context) (`azure/k8s-set-context`): establezca el contexto del clúster de Kubernetes de destino que usarán otras acciones o ejecute cualquier comando de kubectl.
* [Establecimiento del contexto en AKS](https://github.com/Azure/aks-set-context) (`azure/aks-set-context`): establezca el contexto del clúster de Azure Kubernetes Service de destino.
* [Creación de secretos en Kubernetes](https://github.com/Azure/k8s-create-secret) (`azure/k8s-create-secret`): cree un secreto genérico o un secreto del registro de Docker en el clúster de Kubernetes.
* [Implementación en Kubernetes](https://github.com/Azure/k8s-deploy) (`azure/k8s-deploy`): incorpore e implemente manifiestos en clústeres de Kubernetes.
* [Configuración de Helm](https://github.com/Azure/setup-helm) (`azure/setup-helm`): instale una versión específica del archivo binario de Helm en el ejecutor.
* [Simulación mediante "bake" en Kubernetes](https://github.com/Azure/k8s-bake) (`azure/k8s-bake`): archivo de manifiesto mediante "bake" que se va a usar para las implementaciones con helm2, kustomize o kompose.
* [Acción "lint" de Kubernetes](https://github.com/azure/k8s-lint) (`azure/k8s-lint`): valide/busque errores en los archivos de manifiesto.
