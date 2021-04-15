---
title: Implementación de gráficos de Helm mediante GitOps en clústeres de Kubernetes habilitado para Arc
services: azure-arc
ms.service: azure-arc
ms.date: 03/03/2021
ms.topic: article
author: mlearned
ms.author: mlearned
description: Uso de GitOps con Helm para una configuración de clústeres habilitada para Azure Arc
keywords: GitOps, Kubernetes, K8s, Azure, Helm, Arc, AKS, Azure Kubernetes Service, contenedores
ms.openlocfilehash: df9b40764ec463553659803749f282bbc4587bde
ms.sourcegitcommit: 56b0c7923d67f96da21653b4bb37d943c36a81d6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/06/2021
ms.locfileid: "106449550"
---
# <a name="deploy-helm-charts-using-gitops-on-an-arc-enabled-kubernetes-cluster"></a>Implementación de gráficos de Helm mediante GitOps en clústeres de Kubernetes habilitado para Arc

Helm es una herramienta de empaquetado de código abierto que facilita la instalación y administración del ciclo de vida de las aplicaciones de Kubernetes. Al igual que los administradores de paquetes de Linux, como APT y Yum, Helm se usa para administrar los gráficos de Kubernetes, que son paquetes de recursos de Kubernetes preconfigurados.

En este artículo se muestra cómo configurar y usar Helm con Kubernetes habilitado para Azure Arc.

## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Un clúster conectado de Kubernetes habilitado para Azure Arc.
    - Si no ha conectado aún un clúster, siga las indicaciones de [Inicio rápido: Conexión a un clúster de Kubernetes habilitado para Azure Arc](quickstart-connect-cluster.md).
- Entender las ventajas y la arquitectura de esta característica. Más información en el artículo [Configuraciones y GitOps: Kubernetes habilitado para Azure Arc](conceptual-configurations.md).
- Instale la extensión `k8s-configuration` de la CLI de Azure, versión 1.0.0 o posteriores:
  
  ```azurecli
  az extension add --name k8s-configuration
  ```

## <a name="overview-of-using-gitops-and-helm-with-azure-arc-enabled-kubernetes"></a>Introducción al uso de GitOps y Helm con Kubernetes habilitado para Azure Arc

 El operador Helm proporciona una extensión para Flux que automatiza las versiones Chart de Helm. Una versión del gráfico Helm se describe a través de un recurso personalizado de Kubernetes denominado HelmRelease. Flux sincroniza estos recursos de Git con el clúster mientras el operador de Helm se asegura de que los gráficos Helm se publiquen como se especifica en los recursos.

 El [repositorio de ejemplo](https://github.com/Azure/arc-helm-demo) utilizado en este artículo está estructurado de la manera siguiente:

```console
├── charts
│   └── azure-arc-sample
│       ├── Chart.yaml
│       ├── templates
│       │   ├── NOTES.txt
│       │   ├── deployment.yaml
│       │   └── service.yaml
│       └── values.yaml
└── releases
    └── app.yaml
```

En el repositorio de Git hay dos directorios, uno que contiene un gráfico Helm y otro con la configuración de versiones. En el directorio `releases`, `app.yaml` contiene la configuración de HelmRelease siguiente:

```yaml
apiVersion: helm.fluxcd.io/v1
kind: HelmRelease
metadata:
  name: azure-arc-sample
  namespace: arc-k8s-demo
spec:
  releaseName: arc-k8s-demo
  chart:
    git: https://github.com/Azure/arc-helm-demo
    ref: master
    path: charts/azure-arc-sample
  values:
    serviceName: arc-k8s-demo
```

La configuración de versión de Helm contiene los campos siguientes:

| Campo | Descripción |
| ------------- | ------------- | 
| `metadata.name` | Campo obligatorio. Debe seguir las convenciones de nomenclatura de Kubernetes. |
| `metadata.namespace` | Campo opcional. Determina dónde se crea la versión. |
| `spec.releaseName` | Campo opcional. Si no se proporciona, el nombre de la versión será `$namespace-$name`. |
| `spec.chart.path` | El directorio que contiene el gráfico (en relación con la raíz del repositorio). |
| `spec.values` | Personalizaciones de usuario de los valores de parámetro predeterminados del propio gráfico. |

Las opciones especificadas en `spec.values` de HelmRelease reemplazarán las opciones especificadas en `values.yaml` del origen del gráfico.

Puede obtener más información sobre HelmRelease en la [documentación oficial del operador de Helm](https://docs.fluxcd.io/projects/helm-operator/en/stable/).

## <a name="create-a-configuration"></a>Creación de una configuración

Con la extensión de la CLI de Azure para `k8s-configuration`, vincule el clúster conectado al repositorio de Git de ejemplo. Asigne a esta configuración el nombre `azure-arc-sample` e implemente el operador de Flux en el espacio de nombres `arc-k8s-demo`.

```console
az k8s-configuration create --name azure-arc-sample --cluster-name AzureArcTest1 --resource-group AzureArcTest --operator-instance-name flux --operator-namespace arc-k8s-demo --operator-params='--git-readonly --git-path=releases' --enable-helm-operator --helm-operator-chart-version='1.2.0' --helm-operator-params='--set helm.versions=v3' --repository-url https://github.com/Azure/arc-helm-demo.git --scope namespace --cluster-type connectedClusters
```

### <a name="configuration-parameters"></a>Parámetros de configuración

Para personalizar la creación de la configuración, [obtenga información sobre los parámetros adicionales](./tutorial-use-gitops-connected-cluster.md#additional-parameters).

## <a name="validate-the-configuration"></a>Validación de la configuración

Utilice la CLI de Azure para comprobar que la configuración se hay creado correctamente.

```console
az k8s-configuration show --name azure-arc-sample --cluster-name AzureArcTest1 --resource-group AzureArcTest --cluster-type connectedClusters
```

El recurso de configuración se actualiza con el estado de cumplimiento, mensajes y la información de depuración.

```output
{
  "complianceStatus": {
    "complianceState": "Installed",
    "lastConfigApplied": "2019-12-05T05:34:41.481000",
    "message": "{\"OperatorMessage\":null,\"ClusterState\":null}",
    "messageLevel": "3"
  },
  "enableHelmOperator": "True",
  "helmOperatorProperties": {
    "chartValues": "--set helm.versions=v3",
    "chartVersion": "1.2.0"
  },
  "id": "/subscriptions/57ac26cf-a9f0-4908-b300-9a4e9a0fb205/resourceGroups/AzureArcTest/providers/Microsoft.Kubernetes/connectedClusters/AzureArcTest1/providers/Microsoft.KubernetesConfiguration/sourceControlConfigurations/azure-arc-sample",
  "name": "azure-arc-sample",
  "operatorInstanceName": "flux",
  "operatorNamespace": "arc-k8s-demo",
  "operatorParams": "--git-readonly --git-path=releases",
  "operatorScope": "namespace",
  "operatorType": "Flux",
  "provisioningState": "Succeeded",
  "repositoryPublicKey": "",
  "repositoryUrl": "https://github.com/Azure/arc-helm-demo.git",
  "resourceGroup": "AzureArcTest",
  "type": "Microsoft.KubernetesConfiguration/sourceControlConfigurations"
}
```

## <a name="validate-application"></a>Validación de la aplicación

Ejecute el siguiente comando y vaya a `localhost:8080` en el explorador para comprobar que la aplicación se está ejecutando.

```console
kubectl port-forward -n arc-k8s-demo svc/arc-k8s-demo 8080:8080
```

## <a name="next-steps"></a>Pasos siguientes

Aplique configuraciones de clúster a escala mediante [Azure Policy](./use-azure-policy.md).
