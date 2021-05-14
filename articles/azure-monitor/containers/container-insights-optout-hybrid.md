---
title: Cómo detener la supervisión del clúster híbrido de Kubernetes | Microsoft Docs
description: En este artículo se describe cómo puede detener la supervisión del clúster híbrido de Kubernetes con Container Insights.
ms.topic: conceptual
ms.date: 06/16/2020
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: bd13c3e3c3a1aca3253bc43377a15324db2cc4fb
ms.sourcegitcommit: 52491b361b1cd51c4785c91e6f4acb2f3c76f0d5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108319942"
---
# <a name="how-to-stop-monitoring-your-hybrid-cluster"></a>Cómo detener la supervisión del clúster híbrido

Después de habilitar la supervisión del clúster de Kubernetes, puede detenerla con Container Insights si decide que ya no quiere continuar supervisándolo. En este artículo se muestra cómo hacerlo para los siguientes entornos:

- Motor de AKS en Azure y Azure Stack
- OpenShift versión 4 y versiones posteriores
- Kubernetes habilitado para Azure Arc (versión preliminar)

## <a name="how-to-stop-monitoring-using-helm"></a>Cómo detener la supervisión con Helm

Los pasos siguientes se aplican a los siguientes entornos:

- Motor de AKS en Azure y Azure Stack
- OpenShift versión 4 y versiones posteriores

1. Para identificar primero la versión del gráfico de Helm de Container Insights instalada en el clúster, ejecute el siguiente comando de Helm.

    ```
    helm list
    ```

    La salida será similar a la siguiente:

    ```
    NAME                            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                           APP VERSION
    azmon-containers-release-1      default         3               2020-04-21 15:27:24.1201959 -0700 PDT   deployed        azuremonitor-containers-2.7.0   7.0.0-1
    ```

    *azmon-containers-release-1* representa la versión del gráfico de Helm para Container Insights.

2. Para eliminar la versión del gráfico, ejecute el siguiente comando de Helm.

    `helm delete <releaseName>`

    Ejemplo:

    `helm delete azmon-containers-release-1`

    Esto eliminará la versión del clúster. Puede comprobarlo mediante la ejecución del comando `helm list`:

    ```
    NAME                            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                           APP VERSION
    ```

El cambio de configuración puede tardar unos minutos en completarse. Dado que Helm realiza un seguimiento de las versiones incluso después de eliminarlas, es posible auditar el historial de un clúster e incluso recuperar una versión con `helm rollback`.

## <a name="how-to-stop-monitoring-on-arc-enabled-kubernetes"></a>Cómo detener la supervisión en Kubernetes habilitado para Arc

### <a name="using-powershell"></a>Usar PowerShell

1. Descargue y guarde el script que configura el clúster en una carpeta local con el complemento de supervisión mediante el uso de los siguientes comandos:

    ```powershell
    wget https://aka.ms/disable-monitoring-powershell-script -OutFile disable-monitoring.ps1
    ```

2. Configure la variable `$azureArcClusterResourceId`; para ello, establezca los valores correspondientes para `subscriptionId`, `resourceGroupName` y `clusterName`, que representan el identificador de recurso del recurso de clúster de Kubernetes habilitado para Azure Arc.

    ```powershell
    $azureArcClusterResourceId = "/subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.Kubernetes/connectedClusters/<clusterName>"
    ```

3. Configure la variable `$kubeContext` con el valor de **kube-context** del clúster mediante la ejecución del comando `kubectl config get-contexts`. Si desea utilizar el contexto actual, establezca el valor en `""`.

    ```powershell
    $kubeContext = "<kubeContext name of your k8s cluster>"
    ```

4. Ejecute el siguiente comando para detener la supervisión del clúster.

    ```powershell
    .\disable-monitoring.ps1 -clusterResourceId $azureArcClusterResourceId -kubeContext $kubeContext
    ```

#### <a name="using-service-principal"></a>Uso de una entidad de servicio
El script *disable-monitoring.ps1* usa el inicio de sesión de dispositivo interactivo. Si prefiere el inicio de sesión no interactivo, puede usar una entidad de servicio existente o crear otra que tenga los permisos necesarios, tal y como se describe en [Requisitos previos](container-insights-enable-arc-enabled-clusters.md#prerequisites). Para usar la entidad de servicio, tendrá que pasar los parámetros $servicePrincipalClientId, $servicePrincipalClientSecret y $tenantId con los valores de la entidad de servicio que ha pensado usar para el script enable-monitoring.ps1.

```powershell
$subscriptionId = "<subscription Id of the Azure Arc connected cluster resource>"
$servicePrincipal = New-AzADServicePrincipal -Role Contributor -Scope "/subscriptions/$subscriptionId"

$servicePrincipalClientId =  $servicePrincipal.ApplicationId.ToString()
$servicePrincipalClientSecret = [System.Net.NetworkCredential]::new("", $servicePrincipal.Secret).Password
$tenantId = (Get-AzSubscription -SubscriptionId $subscriptionId).TenantId
```

Por ejemplo:

```powershell
\disable-monitoring.ps1 -clusterResourceId $azureArcClusterResourceId -kubeContext $kubeContext -servicePrincipalClientId $servicePrincipalClientId -servicePrincipalClientSecret $servicePrincipalClientSecret -tenantId $tenantId
```


### <a name="using-bash"></a>Uso de Bash

1. Descargue y guarde el script que configura el clúster en una carpeta local con el complemento de supervisión mediante el uso de los siguientes comandos:

    ```bash
    curl -o disable-monitoring.sh -L https://aka.ms/disable-monitoring-bash-script
    ```

2. Configure la variable `azureArcClusterResourceId`; para ello, establezca los valores correspondientes para `subscriptionId`, `resourceGroupName` y `clusterName`, que representan el identificador de recurso del recurso de clúster de Kubernetes habilitado para Azure Arc.

    ```bash
    export azureArcClusterResourceId="/subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.Kubernetes/connectedClusters/<clusterName>"
    ```

3. Configure la variable `kubeContext` con el valor de **kube-context** del clúster mediante la ejecución del comando `kubectl config get-contexts`.

    ```bash
    export kubeContext="<kubeContext name of your k8s cluster>"
    ```

4. Para detener la supervisión del clúster, se proporcionan comandos diferentes según el escenario de implementación.

    Ejecute el siguiente comando para detener la supervisión del clúster con el contexto actual.

    ```bash
    bash disable-monitoring.sh --resource-id $azureArcClusterResourceId
    ```

    Ejecute el siguiente comando para detener la supervisión del clúster mediante la especificación de un contexto.

    ```bash
    bash disable-monitoring.sh --resource-id $azureArcClusterResourceId --kube-context $kubeContext
    ```

#### <a name="using-service-principal"></a>Uso de una entidad de servicio
El script *disable-monitoring.sh* de Bash usa el inicio de sesión de dispositivo interactivo. Si prefiere el inicio de sesión no interactivo, puede usar una entidad de servicio existente o crear otra que tenga los permisos necesarios, tal y como se describe en [Requisitos previos](container-insights-enable-arc-enabled-clusters.md#prerequisites). Para usar la entidad de servicio, tendrá que pasar los valores --client-id, --client-secret y --tenant-id de la entidad de servicio que ha pensado usar para el script *enable-monitoring.sh* de Bash.

```bash
subscriptionId="<subscription Id of the Azure Arc connected cluster resource>"
servicePrincipal=$(az ad sp create-for-rbac --role="Contributor" --scopes="/subscriptions/${subscriptionId}")
servicePrincipalClientId=$(echo $servicePrincipal | jq -r '.appId')

servicePrincipalClientSecret=$(echo $servicePrincipal | jq -r '.password')
tenantId=$(echo $servicePrincipal | jq -r '.tenant')
```

Por ejemplo:

```bash
bash disable-monitoring.sh --resource-id $azureArcClusterResourceId --kube-context $kubeContext --client-id $servicePrincipalClientId --client-secret $servicePrincipalClientSecret  --tenant-id $tenantId
```

## <a name="next-steps"></a>Pasos siguientes

Si el área de trabajo de Log Analytics se creó solamente para admitir la supervisión del clúster y ya no es necesaria, tiene que eliminarla manualmente. Si no está familiarizado con la eliminación de un área de trabajo, consulte [Eliminación de un área de trabajo de Azure Log Analytics](../logs/delete-workspace.md).
