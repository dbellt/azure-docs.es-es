---
title: Instalación de Linkerd en Azure Kubernetes Service (AKS)
description: Aprenda a instalar y usar Linkerd para crear una malla de servicio en un clúster de Azure Kubernetes Service (AKS).
author: paulbouwer
ms.topic: article
ms.date: 10/09/2019
ms.author: pabouwer
zone_pivot_groups: client-operating-system
ms.openlocfilehash: b2b859515481281b571c9803329c6e41cd365ef5
ms.sourcegitcommit: 2e123f00b9bbfebe1a3f6e42196f328b50233fc5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/27/2021
ms.locfileid: "108070054"
---
# <a name="install-linkerd-in-azure-kubernetes-service-aks"></a>Instalación de Linkerd en Azure Kubernetes Service (AKS)

[Linkerd][linkerd-github] es una malla de servicio de código abierto y un [proyecto en incubación de CNCF][linkerd-cncf]. Linkerd es una malla de servicio ultraligera que proporciona características entre las que se incluyen la administración del tráfico, la identidad y la seguridad de los servicios, la seguridad, la confiabilidad y la observabilidad. Para más información sobre Linkerd, consulte la documentación oficial [Linkerd FAQ][linkerd-faq] (Preguntas frecuentes de Linkerd) y [Linkerd Architecture][linkerd-architecture] (Arquitectura de Linkerd).

En este artículo se explica cómo instalar Linkerd. El binario de cliente `linkerd` de Linkerd se instala en la máquina cliente y sus componentes se instalan en un clúster de Kubernetes en AKS.

> [!NOTE]
> Estas instrucciones son aplicables a la versión `stable-2.6.0` de Linkerd.
>
> Las versiones `stable-2.6.x` de Linkerd se pueden ejecutar en las versiones `1.13+` de Kubernetes. Puede encontrar versiones estables y preliminares de Linkerd en [GitHub - Linkerd Releases][linkerd-github-releases] (GitHub: Linkerd > Versiones).

En este artículo aprenderá a:

> [!div class="checklist"]
> * Descarga e instalación del binario de cliente linkerd de Linkerd
> * Instalación de Linkerd en AKS
> * Validación de la instalación de Linkerd
> * Acceso al panel
> * Desinstalación de Linkerd de AKS

## <a name="before-you-begin"></a>Antes de empezar

En los pasos detallados en este artículo se da por hecho que creó un clúster de AKS (Kubernetes `1.13` y versiones posteriores, con RBAC de Kubernetes habilitado) y que estableció una conexión `kubectl` con el clúster. Si necesita ayuda con cualquiera de estos elementos, consulte el [inicio rápido de AKS][aks-quickstart].

Todos los pods de Linkerd deben estar programados para ejecutarse en nodos de Linux: esta es la configuración predeterminada en el método de instalación que se detalla a continuación y no requiere ninguna otra configuración.

En este artículo, las instrucciones de instalación de Linkerd se dividen en varios pasos discretos. El resultado final presenta la misma estructura que las [instrucciones][linkerd-getting-started] de introducción oficiales de Linkerd.

::: zone pivot="client-operating-system-linux"

[!INCLUDE [Linux - download and install client binary](includes/servicemesh/linkerd/install-client-binary-linux.md)]

::: zone-end

::: zone pivot="client-operating-system-macos"

[!INCLUDE [MacOS - download and install client binary](includes/servicemesh/linkerd/install-client-binary-macos.md)]

::: zone-end

::: zone pivot="client-operating-system-windows"

[!INCLUDE [Windows - download and install client binary](includes/servicemesh/linkerd/install-client-binary-windows.md)]

::: zone-end

## <a name="install-linkerd-on-aks"></a>Instalación de Linkerd en AKS

Antes de instalar Linkerd, ejecutaremos comprobaciones previas para determinar si el plano de control se puede instalar en nuestro clúster de AKS:

```console
linkerd check --pre
```

Debería ver algo parecido a lo siguiente, que indica que el clúster de AKS es un destino de instalación válido para Linkerd:

```console
kubernetes-api
--------------
√ can initialize the client
√ can query the Kubernetes API

kubernetes-version
------------------
√ is running the minimum Kubernetes API version
√ is running the minimum kubectl version

pre-kubernetes-setup
--------------------
√ control plane namespace does not already exist
√ can create Namespaces
√ can create ClusterRoles
√ can create ClusterRoleBindings
√ can create CustomResourceDefinitions
√ can create PodSecurityPolicies
√ can create ServiceAccounts
√ can create Services
√ can create Deployments
√ can create CronJobs
√ can create ConfigMaps
√ no clock skew detected

pre-kubernetes-capability
-------------------------
√ has NET_ADMIN capability
√ has NET_RAW capability

pre-linkerd-global-resources
----------------------------
√ no ClusterRoles exist
√ no ClusterRoleBindings exist
√ no CustomResourceDefinitions exist
√ no MutatingWebhookConfigurations exist
√ no ValidatingWebhookConfigurations exist
√ no PodSecurityPolicies exist

linkerd-version
---------------
√ can determine the latest version
√ cli is up-to-date

Status check results are √
```

Ahora es el momento de instalar los componentes de Linkerd. Use los binarios `linkerd` y `kubectl` para instalar los componentes de Linkerd en el clúster de AKS. Se creará automáticamente un espacio de nombres `linkerd` y los componentes se instalarán en este espacio de nombres.

```console
linkerd install | kubectl apply -f -
```

Linkerd implementa una serie de objetos. Verá la lista en la salida del comando `linkerd install` anterior. La implementación de los componentes de Linkerd debe tardar aproximadamente un minuto en completarse, en función del entorno del clúster.

En este punto, ya ha implementado Linkerd en el clúster de AKS. Para asegurarse de que la implementación de Linkerd ha sido correcta, vaya a la siguiente sección para [validar la instalación de Linkerd](#validate-the-linkerd-installation).

## <a name="validate-the-linkerd-installation"></a>Validación de la instalación de Linkerd

Confirme que los recursos se han creado correctamente. Use los comandos [kubectl get svc][kubectl-get] y [kubectl get pod][kubectl-get] para consultar el espacio de nombres `linkerd`, donde se instalaron los componentes de Linkerd con el comando `linkerd install`:

```console
kubectl get svc --namespace linkerd --output wide
kubectl get pod --namespace linkerd --output wide
```

En la siguiente salida de ejemplo se muestran los servicios y pods (programados en nodos de Linux) que deben estar ejecutándose ahora:

```console
NAME                     TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)             AGE  SELECTOR
linkerd-controller-api   ClusterIP   10.0.110.67    <none>        8085/TCP            66s  linkerd.io/control-plane-component=controller
linkerd-destination      ClusterIP   10.0.224.29    <none>        8086/TCP            66s  linkerd.io/control-plane-component=controller
linkerd-dst              ClusterIP   10.0.225.148   <none>        8086/TCP            66s  linkerd.io/control-plane-component=destination
linkerd-grafana          ClusterIP   10.0.61.124    <none>        3000/TCP            65s  linkerd.io/control-plane-component=grafana
linkerd-identity         ClusterIP   10.0.6.104     <none>        8080/TCP            67s  linkerd.io/control-plane-component=identity
linkerd-prometheus       ClusterIP   10.0.27.168    <none>        9090/TCP            65s  linkerd.io/control-plane-component=prometheus
linkerd-proxy-injector   ClusterIP   10.0.100.133   <none>        443/TCP             64s  linkerd.io/control-plane-component=proxy-injector
linkerd-sp-validator     ClusterIP   10.0.221.5     <none>        443/TCP             64s  linkerd.io/control-plane-component=sp-validator
linkerd-tap              ClusterIP   10.0.18.14     <none>        8088/TCP,443/TCP    64s  linkerd.io/control-plane-component=tap
linkerd-web              ClusterIP   10.0.37.108    <none>        8084/TCP,9994/TCP   66s  linkerd.io/control-plane-component=web

NAME                                      READY   STATUS    RESTARTS   AGE   IP            NODE                            NOMINATED NODE   READINESS GATES
linkerd-controller-66ddc9f94f-cm9kt       3/3     Running   0          66s   10.240.0.50   aks-linux-16165125-vmss000001   <none>           <none>
linkerd-destination-c94bc454-qpkng        2/2     Running   0          66s   10.240.0.78   aks-linux-16165125-vmss000002   <none>           <none>
linkerd-grafana-6868fdcb66-4cmq2          2/2     Running   0          65s   10.240.0.69   aks-linux-16165125-vmss000002   <none>           <none>
linkerd-identity-74d8df4b85-tqq8f         2/2     Running   0          66s   10.240.0.48   aks-linux-16165125-vmss000001   <none>           <none>
linkerd-prometheus-699587cf8-k8ghg        2/2     Running   0          65s   10.240.0.41   aks-linux-16165125-vmss000001   <none>           <none>
linkerd-proxy-injector-6556447f64-n29wr   2/2     Running   0          64s   10.240.0.32   aks-linux-16165125-vmss000000   <none>           <none>
linkerd-sp-validator-56745cd567-v4x7h     2/2     Running   0          64s   10.240.0.6    aks-linux-16165125-vmss000000   <none>           <none>
linkerd-tap-5cd9fc566-ct988               2/2     Running   0          64s   10.240.0.15   aks-linux-16165125-vmss000000   <none>           <none>
linkerd-web-774c79b6d5-dhhwf              2/2     Running   0          65s   10.240.0.70   aks-linux-16165125-vmss000002   <none>           <none>
```

Linkerd proporciona un comando a través del binario de cliente `linkerd` para validar que el plano de control de Linkerd se ha instalado y configurado correctamente.

```console
linkerd check
```

Debería ver algo parecido a lo siguiente, para indicar que la instalación se realizó correctamente:

```console
kubernetes-api
--------------
√ can initialize the client
√ can query the Kubernetes API

kubernetes-version
------------------
√ is running the minimum Kubernetes API version
√ is running the minimum kubectl version

linkerd-config
--------------
√ control plane Namespace exists
√ control plane ClusterRoles exist
√ control plane ClusterRoleBindings exist
√ control plane ServiceAccounts exist
√ control plane CustomResourceDefinitions exist
√ control plane MutatingWebhookConfigurations exist
√ control plane ValidatingWebhookConfigurations exist
√ control plane PodSecurityPolicies exist

linkerd-existence
-----------------
√ 'linkerd-config' config map exists
√ heartbeat ServiceAccount exist
√ control plane replica sets are ready
√ no unschedulable pods
√ controller pod is running
√ can initialize the client
√ can query the control plane API

linkerd-api
-----------
√ control plane pods are ready
√ control plane self-check
√ [kubernetes] control plane can talk to Kubernetes
√ [prometheus] control plane can talk to Prometheus
√ no invalid service profiles

linkerd-version
---------------
√ can determine the latest version
√ cli is up-to-date

control-plane-version
---------------------
√ control plane is up-to-date
√ control plane and cli versions match

Status check results are √
```

## <a name="access-the-dashboard"></a>Acceso al panel

Linkerd incluye un panel que proporciona información sobre la malla de servicio y las cargas de trabajo. Para acceder al panel, use el comando `linkerd dashboard`. Este comando utiliza [kubectl port-forward][kubectl-port-forward] para crear una conexión segura entre la máquina cliente y los pods correspondientes en el clúster de AKS. A continuación, abre automáticamente el panel en el explorador predeterminado.

```console
linkerd dashboard
```

El comando también crea un reenvío de puerto y devuelve un vínculo para los paneles de Grafana.

```console
Linkerd dashboard available at:
http://127.0.0.1:50750
Grafana dashboard available at:
http://127.0.0.1:50750/grafana
Opening Linkerd dashboard in the default browser
```

## <a name="uninstall-linkerd-from-aks"></a>Desinstalación de Linkerd de AKS

> [!WARNING]
> La eliminación de Linkerd de un sistema en ejecución puede provocar problemas relacionados con el tráfico entre los servicios. Antes de continuar, asegúrese de que su sistema esté bien aprovisionado para funcionar correctamente sin Linkerd.

En primer lugar, debe quitar los proxies del plano de datos. Quite de los espacios de nombres de carga de trabajo cualquier [anotación][linkerd-automatic-proxy-injection] de la inserción automática de proxies y cambie las implementaciones de carga de trabajo. Las cargas de trabajo ya no deben tener ningún componente de plano de datos asociado.

Por último, quite el plano de control, como se indica a continuación:

```console
linkerd install --ignore-cluster | kubectl delete -f -
```

## <a name="next-steps"></a>Pasos siguientes

Para explorar más opciones de instalación y configuración de Linkerd, consulte las siguientes guías oficiales de Linkerd:

- [Linkerd - Helm installation][linkerd-install-with-helm] (Linkerd: Instalación de Helm)
- [Linkerd - Multi-stage installation to cater for role privileges][linkerd-multi-stage-installation] (Linkerd: Instalación en varias fases para mantener los privilegios de rol)

También puede seguir otros escenarios mediante:

- [Linkerd emojivoto demo][linkerd-demo-emojivoto] (Demostración de emojivoto de Linkerd)
- [Linkerd books demo][linkerd-demo-books] (Demostración de books de Linkerd)

<!-- LINKS - external -->

[linkerd]: https://linkerd.io/
[linkerd-cncf]: https://landscape.cncf.io/?selected=linkerd
[linkerd-faq]: https://linkerd.io/2/faq/
[linkerd-architecture]: https://linkerd.io/2/reference/architecture/
[linkerd-getting-started]: https://linkerd.io/2/getting-started/
[linkerd-overview]: https://linkerd.io/2/overview/
[linkerd-github]: https://github.com/linkerd/linkerd2
[linkerd-github-releases]: https://github.com/linkerd/linkerd2/releases/

[linkerd-install-with-helm]: https://linkerd.io/2/tasks/install-helm/
[linkerd-multi-stage-installation]: https://linkerd.io/2/tasks/install/#multi-stage-install
[linkerd-automatic-proxy-injection]: https://linkerd.io/2/features/proxy-injection/

[linkerd-demo-emojivoto]: https://linkerd.io/2/getting-started/#step-5-install-the-demo-app
[linkerd-demo-books]: https://linkerd.io/2/tasks/books/

[helm]: https://helm.sh

[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[kubectl-port-forward]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#port-forward

<!-- LINKS - internal -->
[aks-quickstart]: ./kubernetes-walkthrough.md
