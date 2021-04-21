---
title: Visualización de los registros de controlador de Azure Kubernetes Service (AKS)
description: Aprenda a habilitar y ver los registros del plano de control de Kubernetes en Azure Kubernetes Service (AKS).
services: container-service
ms.topic: article
ms.date: 01/27/2020
ms.openlocfilehash: 395422ca97b43488a7d7ad1c7434b20ccc59f2b0
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107767298"
---
# <a name="enable-and-review-kubernetes-control-plane-logs-in-azure-kubernetes-service-aks"></a>Habilitación y revisión de los registros del plano de control de Kubernetes en Azure Kubernetes Service (AKS)

Con Azure Kubernetes Service (AKS), los componentes del plano de control, como *kube-apiserver* y *kube-controller-manager* se proporcionan como un servicio administrado. Cree y administre los nodos que ejecutan *kubelet* y el entorno de tiempo de ejecución del contenedor, e implemente las aplicaciones mediante el servidor de API de Kubernetes administrado. Para ayudar a solucionar los problemas de sus aplicaciones y servicios, quizás deba ver los registros generados por estos componentes del plano de control. En este artículo se muestra cómo usar registros de Azure Monitor para habilitar y consultar los registros de los componentes del plano de control de Kubernetes.

## <a name="before-you-begin"></a>Antes de empezar

En este artículo se requiere un clúster de AKS existente que se ejecute en su cuenta de Azure. Si todavía no tiene un clúster de AKS, cree uno con la [CLI de Azure][cli-quickstart] o [Azure Portal][portal-quickstart]. Los registros de Azure Monitor funcionan tanto con clústeres de AKS habilitados para RBAC de Kubernetes, RBAC de Azure y no habilitados para RBAC.

## <a name="enable-resource-logs"></a>Habilitación de registros de recursos

Para ayudar a recopilar y revisar los datos de varios orígenes, registros de Azure Monitor proporciona un motor de lenguaje y análisis de consultas que ofrece información detallada sobre su entorno. Un área de trabajo se usa para intercalar y analizar los datos, y se puede integrar con otros servicios de Azure como Application Insights y Security Center. Si desea utilizar una plataforma diferente para analizar los registros, puede decidir en su lugar enviar los registros de recursos a una cuenta de almacenamiento o un centro de eventos de Azure. Para más información, consulte el artículo sobre los [registros de Azure Monitor][log-analytics-overview].

Los registros de Azure Monitor se habilitan y administran en Azure Portal. Para habilitar la recopilación de registros para los componentes del plano de control de Kubernetes en el clúster de AKS, abra Azure Portal en un explorador web y realice los pasos siguientes:

1. Seleccione el grupo de recursos del clúster de AKS, como *myResourceGroup*. No seleccione el grupo de recursos que contiene los recursos del clúster de AKS individuales, como *MC_myResourceGroup_myAKSCluster_eastus*.

2. En la parte izquierda, elija **Configuración de diagnóstico**.

3. Seleccione el clúster de AKS, como *myAKSCluster*, y elija **Agregar configuración de diagnóstico**.
  :::image type="content" source="media\view-control-plane-logs\select-add-diagnostic-setting.PNG" alt-text="Captura de pantalla de Azure Portal en una ventana del explorador en la que se muestra la configuración de diagnóstico y se debe seleccionar &quot;Agregar configuración de diagnóstico&quot;":::

4. Escriba un nombre, como *myAKSClusterLogs* y seleccione la opción **Send to Log Analytics workspace** (Enviar al área de trabajo de Log Analytics).

5. Seleccione un área de trabajo existente o cree uno. Al crear un área de trabajo, proporciónele un nombre, un grupo de recursos y una ubicación.

6. En la lista de registros disponibles, seleccione los que desea habilitar. En este ejemplo, habilite los registros *kube-audit* y *kube-audit-admin*. Los registros típicos incluyen *kube-apiserver*, *kube-controller-manager* y *kube-scheduler*. Puede volver y cambiar los registros recopilados una vez que las áreas de trabajo de Log Analytics está habilitadas.

7. Cuando esté listo, seleccione **Guardar** para habilitar la recopilación de los registros seleccionados.
  :::image type="content" source="media\view-control-plane-logs\settings-selected.PNG" alt-text="Captura de pantalla de la pantalla &quot;Agregar configuración de diagnóstico&quot; de Azure Portal. Está seleccionado el destino &quot;Enviar al área de trabajo de Log Analytics&quot; y los registros &quot;kube-audit&quot; y &quot;kube-audit-admin&quot;":::

## <a name="log-categories"></a>Categorías de registro

Además de las entradas escritas por Kubernetes, los registros de auditoría del proyecto también tienen entradas de AKS.

Los registros de auditoría se registran en tres categorías: *kube-audit*, *kube-audit-admin* y *guard*.

- La categoría *kube-audit* contiene todos los datos de registro de auditoría para cada evento de auditoría, los que incluyen *get*, *list*, *create*, *update*, *delete*, *patch* y *post*.
- La categoría *kube-audit-admin* es un subconjunto de la categoría de registro *kube-audit*. *kube-audit-admin* reduce considerablemente el número de registros al excluir los eventos de auditoría *get* y *list* del registro.
- La categoría *guard* está administrada por Azure AD y las auditorías de Azure RBAC. En el caso de estar administrada por Azure AD: el token debe estar dentro y la información de usuario fuera. Para Azure RBAC: revisiones de acceso dentro y fuera.

## <a name="schedule-a-test-pod-on-the-aks-cluster"></a>Programación de un pod de prueba en el clúster de AKS

Para generar algunos registros, cree un nuevo pod en el clúster de AKS. El siguiente ejemplo de manifiesto de YAML puede usarse para crear una instancia básica de NGINX. Cree un archivo denominado `nginx.yaml` en el editor que prefiera y pegue el contenido siguiente:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  nodeSelector:
    "beta.kubernetes.io/os": linux
  containers:
  - name: mypod
    image: mcr.microsoft.com/oss/nginx/nginx:1.15.5-alpine
    resources:
      requests:
        cpu: 100m
        memory: 128Mi
      limits:
        cpu: 250m
        memory: 256Mi
    ports:
    - containerPort: 80
```

Cree el pod con el comando [kubectl create][kubectl-create] y especifique el archivo YAML, como se muestra en el ejemplo siguiente:

```
$ kubectl create -f nginx.yaml

pod/nginx created
```

## <a name="view-collected-logs"></a>Visualización de los datos recopilados

Los registros de diagnóstico pueden tardar hasta 10 minutos en habilitarse y aparecer.

> [!NOTE]
> Si necesita todos los datos de registro de auditoría para cumplimiento u otros propósitos, recopílelos y almacénelos en un almacenamiento económico, como el almacenamiento de blobs. Use la categoría de registro *kube-audit-admin* para recopilar y almacenar un conjunto significativo de datos de registro de auditoría con fines de supervisión y alerta.

En Azure Portal, vaya al clúster de AKS y seleccione **Registros** en el lado izquierdo. Si aparece, cierre la ventana *Consultas de ejemplo*.

En el lado izquierdo, elija **Registros**. Para ver los registros de *kube-audit*, escriba la siguiente consulta en el cuadro de texto:

```
AzureDiagnostics
| where Category == "kube-audit"
| project log_s
```

Es probable que se devuelvan muchos registros. Para reducir el ámbito de la consulta y ver los registros sobre el pod de NGINX creado en el paso anterior, agregue una instrucción *where* adicional para buscar *nginx*, como se muestra en la consulta de ejemplo siguiente:

```
AzureDiagnostics
| where Category == "kube-audit"
| where log_s contains "nginx"
| project log_s
```

Para ver los registros de *kube-audit-admin*, escriba la consulta siguiente en el cuadro de texto:

```
AzureDiagnostics
| where Category == "kube-audit-admin"
| project log_s
```

En este ejemplo, la consulta muestra todos los trabajos de creación en *kube-audit-admin*. Es posible que se devuelvan muchos resultados y, para reducir el ámbito de la consulta y ver los registros sobre el pod de NGINX creado en el paso anterior, agregue una instrucción *where* adicional para buscar *nginx*, como se muestra en la consulta de ejemplo siguiente.

```
AzureDiagnostics
| where Category == "kube-audit-admin"
| where log_s contains "nginx"
| project log_s
```


Para más información acerca de cómo consultar y filtrar datos de registro, consulte el artículo de [Visualización o análisis de los datos recopilados con la búsqueda de registros de Log Analytics][analyze-log-analytics].

## <a name="log-event-schema"></a>Esquema de evento de registro

AKS registra los siguientes eventos:

* [AzureActivity][log-schema-azureactivity]
* [AzureDiagnostics][log-schema-azurediagnostics]
* [AzureMetrics][log-schema-azuremetrics]
* [ContainerImageInventory][log-schema-containerimageinventory]
* [ContainerInventory][log-schema-containerinventory]
* [ContainerLog][log-schema-containerlog]
* [ContainerNodeInventory][log-schema-containernodeinventory]
* [ContainerServiceLog][log-schema-containerservicelog]
* [Heartbeat][log-schema-heartbeat]
* [InsightsMetrics][log-schema-insightsmetrics]
* [KubeEvents][log-schema-kubeevents]
* [KubeHealth][log-schema-kubehealth]
* [KubeMonAgentEvents][log-schema-kubemonagentevents]
* [KubeNodeInventory][log-schema-kubenodeinventory]
* [KubePodInventory][log-schema-kubepodinventory]
* [KubeServices][log-schema-kubeservices]
* [Perf][log-schema-perf]

## <a name="log-roles"></a>Roles de registro

| Role                     | Descripción |
|--------------------------|-------------|
| *aksService*             | El nombre para mostrar en el registro de auditoría de la operación de plano de control (desde hcpService). |
| *masterclient*           | El nombre para mostrar en el registro de auditoría de MasterClientCertificate, el certificado que obtiene desde az aks get-credentials. |
| *nodeclient*             | El nombre para mostrar de ClientCertificate, que los nodos de agente usan. |

## <a name="next-steps"></a>Pasos siguientes

En este artículo, aprendió a habilitar y revisar los registros de los componentes del plano de control de Kubernetes en el clúster de AKS. Para seguir supervisando y solucionando problemas, también puede [ver los registros de Kubelet][kubelet-logs] y [habilitar el acceso al nodo SSH][aks-ssh].

<!-- LINKS - external -->
[kubectl-create]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#create

<!-- LINKS - internal -->
[cli-quickstart]: kubernetes-walkthrough.md
[portal-quickstart]: kubernetes-walkthrough-portal.md
[log-analytics-overview]: ../azure-monitor/logs/log-query-overview.md
[analyze-log-analytics]: ../azure-monitor/logs/log-analytics-tutorial.md
[kubelet-logs]: kubelet-logs.md
[aks-ssh]: ssh.md
[az-feature-register]: /cli/azure/feature#az_feature_register
[az-feature-list]: /cli/azure/feature#az_feature_list
[az-provider-register]: /cli/azure/provider#az_provider_register
[log-schema-azureactivity]: /azure/azure-monitor/reference/tables/azureactivity
[log-schema-azurediagnostics]: /azure/azure-monitor/reference/tables/azurediagnostics
[log-schema-azuremetrics]: /azure/azure-monitor/reference/tables/azuremetrics
[log-schema-containerimageinventory]: /azure/azure-monitor/reference/tables/containerimageinventory
[log-schema-containerinventory]: /azure/azure-monitor/reference/tables/containerinventory
[log-schema-containerlog]: /azure/azure-monitor/reference/tables/containerlog
[log-schema-containernodeinventory]: /azure/azure-monitor/reference/tables/containernodeinventory
[log-schema-containerservicelog]: /azure/azure-monitor/reference/tables/containerservicelog
[log-schema-heartbeat]: /azure/azure-monitor/reference/tables/heartbeat
[log-schema-insightsmetrics]: /azure/azure-monitor/reference/tables/insightsmetrics
[log-schema-kubeevents]: /azure/azure-monitor/reference/tables/kubeevents
[log-schema-kubehealth]: /azure/azure-monitor/reference/tables/kubehealth
[log-schema-kubemonagentevents]: /azure/azure-monitor/reference/tables/kubemonagentevents
[log-schema-kubenodeinventory]: /azure/azure-monitor/reference/tables/kubenodeinventory
[log-schema-kubepodinventory]: /azure/azure-monitor/reference/tables/kubepodinventory
[log-schema-kubeservices]: /azure/azure-monitor/reference/tables/kubeservices
[log-schema-perf]: /azure/azure-monitor/reference/tables/perf