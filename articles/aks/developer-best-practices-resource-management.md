---
title: Procedimientos recomendados de administración de recursos
titleSuffix: Azure Kubernetes Service
description: Obtenga más información acerca de los procedimientos recomendados para desarrolladores de aplicaciones para la administración de recursos en Azure Kubernetes Services (AKS).
services: container-service
author: zr-msft
ms.topic: conceptual
ms.date: 03/15/2021
ms.author: zarhoads
ms.openlocfilehash: 2cd2bab05346f66b933512e677f1d38f4514796c
ms.sourcegitcommit: 5f482220a6d994c33c7920f4e4d67d2a450f7f08
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/08/2021
ms.locfileid: "107105279"
---
# <a name="best-practices-for-application-developers-to-manage-resources-in-azure-kubernetes-service-aks"></a>Procedimientos recomendados para desarrolladores de aplicaciones para administrar recursos en Azure Kubernetes Services (AKS)

A medida que desarrolla y ejecuta aplicaciones en Azure Kubernetes Service (AKS), hay algunas áreas clave a tener en cuenta. El modo de administrar las implementaciones de aplicaciones puede repercutir negativamente en la experiencia del usuario final de los servicios que proporciona. Para que el proceso sea correcto, tenga en cuenta algunos procedimientos recomendados que puede seguir a medida que desarrolla y ejecuta aplicaciones en AKS.

Este artículo se centra en la ejecución del clúster y las cargas de trabajo desde la perspectiva de un desarrollador de aplicaciones. Para obtener información acerca de los procedimientos recomendados, consulte [Procedimientos recomendados para el aislamiento y la administración de recursos en Azure Kubernetes Service (AKS)][operator-best-practices-isolation]. En este artículo, aprenderá lo siguiente:

> [!div class="checklist"]
> * Solicitudes y límites de recursos de pod
> * Formas de desarrollar e implementar aplicaciones con Bridge to Kubernetes y Visual Studio Code.
> * Cómo usar la herramienta `kube-advisor` para comprobar si existen problemas con las implementaciones.

## <a name="define-pod-resource-requests-and-limits"></a>Definición de los límites y solicitudes de recursos del pod

> **Guía de procedimientos recomendados**
> 
> Configure las solicitudes y los límites de pod en todos los pods de los manifiestos de YAML. Si el clúster de AKS usa *cuotas de recursos* y no define estos valores, es posible que la implementación se rechace.

Use las solicitudes y los límites de pod para administrar los recursos de proceso dentro de un clúster de AKS. Las solicitudes y los límites de pod informan al programador de Kubernetes sobre los recursos de proceso que se asignarán a un pod.

### <a name="pod-cpumemory-requests"></a>Solicitudes de CPU/memoria de pod
Las *solicitudes de pod* definen una cierta cantidad de CPU y memoria que necesita el pod periódicamente.

En las especificaciones de su pod, es un **procedimiento recomendado y muy importante**  definir estas solicitudes y límites en función de la información anterior. Si no incluye estos valores, el scheduler de Kubernetes no puede tener en cuenta los recursos que requieren sus aplicaciones para ayudarlo a tomar decisiones de programación.

Supervise el rendimiento de la aplicación para ajustar las solicitudes de pod. 
* Si subestima las solicitudes de pod, su aplicación puede recibir un rendimiento degradado debido a la programación excesiva de un nodo. 
* Si se sobrestiman las solicitudes, su aplicación puede tener mayores dificultades para ser programada.

### <a name="pod-cpumemory-limits"></a>Límites de CPU/memoria de pod** 
Los *límites de pods* establecen la cantidad máxima de CPU y memoria que puede usar un pod. 

* Los *límites de memoria* definen qué pods se deben terminar cuando los nodos son inestables por haber recursos insuficientes. Sin los límites adecuados, los pods establecidos se terminarán hasta que se levante la presión de los recursos. 
* Aunque un pod puede exceder el *límite de CPU* periódicamente, el pod no se terminará por exceder el límite de CPU. 

Los límites de pod definen cuándo ha perdido un pod el control del consumo de recursos. Cuando supera el límite, el pod se marca para su terminación. Este comportamiento mantiene el estado del nodo y minimiza el impacto en los pods que comparten el nodo. Al no establecer un límite de pod, el valor predeterminado es el valor más alto disponible en un nodo determinado.

Evite establecer un límite de pod superior al que pueden admitir los nodos. Cada nodo de AKS reserva una cierta cantidad de CPU y memoria para los componentes básicos de Kubernetes. La aplicación puede intentar consumir demasiados recursos en el nodo para que otros pods se ejecuten correctamente.

Supervise el rendimiento de la aplicación en distintos momentos del día o de la semana. Determine cuándo se produce la máxima demanda y ajuste en consonancia los límites del pod para que haya recursos necesarios para las necesidades máximas.

> [!IMPORTANT]
>
> En las especificaciones de su pod, defina estas solicitudes y límites en función de la información anterior. Si no se incluyen estos valores, el programador de Kubernetes no tiene en cuenta los recursos que las aplicaciones necesitan para ayudar en la programación de decisiones.

Si el Scheduler coloca un Pod en un nodo con recursos insuficientes, el rendimiento de la aplicación se degradará. El administrador de clústeres **debe** definir *cuotas de recursos* en un espacio de nombres que requiere el establecimiento de límites y solicitudes de recursos. Para obtener más información, consulte [cuotas de recursos en clústeres de AKS][resource-quotas].

Al definir un límite o solicitud de CPU, el valor se mide en unidades de CPU. 
* CPU *1.0* equivale a un núcleo de CPU virtual subyacente en el nodo. 
    * Se usa la misma unidad de medida para las GPU.
* Puede definir fracciones medidas en milinúcleos. Por ejemplo, *100 m* es *0,1* de un núcleo de vCPU subyacente.

En el siguiente ejemplo básico para un solo pod de NGINX, el pod solicita *100 m* de tiempo de CPU y *128 Mi* de memoria. Los límites de recursos para el pod se definen en *250 m* de CPU y *256 Mi* de memoria:

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: mypod
spec:
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
```

Para obtener más información acerca de las asignaciones y medidas de recursos, consulte [Administración de recursos de proceso para contenedores][k8s-resource-limits].

## <a name="develop-and-debug-applications-against-an-aks-cluster"></a>Desarrollo y depuración de aplicaciones en un clúster de AKS

> **Guía de procedimientos recomendados** 
>
> Los equipos de desarrollo deben realizar la implementación y depuración en un clúster de AKS mediante Bridge to Kubernetes.

Con Bridge to Kubernetes, desarrolle, depure y pruebe aplicaciones directamente en un clúster de AKS. Los desarrolladores de un equipo colaboran para realizar compilaciones y pruebas a lo largo del ciclo de vida de la aplicación. Puede seguir usando herramientas existentes, como Visual Studio o Visual Studio Code, con la extensión Bridge to Kubernetes. 

El uso de este proceso de desarrollo y pruebas integrado con Bridge to Kubernetes reduce la necesidad de entornos de prueba locales, como [minikube][minikube]. En su lugar, desarrolle y pruebe con un clúster de AKS, incluso con clústeres protegidos y aislados. 

> [!NOTE]
> Bridge to Kubernetes está pensado para su uso con aplicaciones que se ejecutan en nodos y pods de Linux.

## <a name="use-the-visual-studio-code-vs-code-extension-for-kubernetes"></a>Uso de Visual Studio Code (VS Code) para Kubernetes

> **Guía de procedimientos recomendados** 
>
> Instale y use la extensión de VS Code para Kubernetes al escribir manifiestos de YAML. También puede usar la extensión para la solución de implementación integrada, lo que puede ayudar a los propietarios de aplicaciones que interactúan con poca frecuencia con el clúster de AKS.

La [extensión de Visual Studio Code para Kubernetes][vscode-kubernetes] le ayuda a desarrollar e implementar aplicaciones en AKS. Esta extensión proporciona:
* Recursos de IntelliSense para Kubernetes, gráficos de Helm y plantillas. 
* Funcionalidades de examen, implementación y edición para recursos de Kubernetes desde VS Code. 
* Una comprobación de IntelliSense para los límites o solicitudes de recursos que se van a establecer en las especificaciones del pod:

    ![Advertencia de la extensión de VS Code para Kubernetes que indica que faltan los límites de memoria.](media/developer-best-practices-resource-management/vs-code-kubernetes-extension.png)

## <a name="regularly-check-for-application-issues-with-kube-advisor"></a>Comprobación de forma periódica de problemas de aplicaciones con kube-advisor

> **Guía de procedimientos recomendados** 
> 
> Ejecute periódicamente la versión más reciente de la herramienta de código abierto `kube-advisor` para detectar problemas en el clúster. Ejecute `kube-advisor` antes de aplicar cuotas de recursos en un clúster de AKS existente para buscar pods que no tengan definidas solicitudes y límites de recursos.

La herramienta [kube-advisor][kube-advisor] es un proyecto de código abierto de AKS asociado que examina un clúster de Kubernetes e informa sobre los problemas identificados. Una comprobación útil consiste en identificar los pods sin límites y solicitudes de recursos vigentes.

Aunque la herramienta `kube-advisor` puede informar sobre la falta de límites y solicitudes de recursos en PodSpecs para las aplicaciones Windows y Linux, la propia `kube-advisor` debe programarse en un pod de Linux. Use un [selector de nodos][k8s-node-selector] en la configuración del pod para programar la ejecución de un pod en un grupo de nodos con un sistema operativo específico.

En un clúster de AKS que hospeda muchos equipos y aplicaciones de desarrollo, le resultará fácil realizar un seguimiento de los pods con los límites y solicitudes de recursos. Como procedimiento recomendado, ejecute `kube-advisor` de forma periódica en los clústeres de AKS.

## <a name="next-steps"></a>Pasos siguientes

Este artículo se centra en cómo ejecutar el clúster y las cargas de trabajo desde la perspectiva de un operador de clústeres. Para obtener información acerca de los procedimientos recomendados, consulte [Procedimientos recomendados para el aislamiento y la administración de recursos en Azure Kubernetes Service (AKS)][operator-best-practices-isolation].

Para implementar algunos de estos procedimientos recomendados, consulte los artículos siguientes:

* [Desarrollo con Bridge to Kubernetes][btk]
* [Comprobación de problemas con kube-advisor][aks-kubeadvisor]

<!-- EXTERNAL LINKS -->
[k8s-resource-limits]: https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/
[vscode-kubernetes]: https://github.com/Azure/vscode-kubernetes-tools
[kube-advisor]: https://github.com/Azure/kube-advisor
[minikube]: https://kubernetes.io/docs/setup/minikube/

<!-- INTERNAL LINKS -->
[aks-kubeadvisor]: kube-advisor-tool.md
[btk]: /visualstudio/containers/overview-bridge-to-kubernetes
[operator-best-practices-isolation]: operator-best-practices-cluster-isolation.md
[resource-quotas]: operator-best-practices-scheduler.md#enforce-resource-quotas
[k8s-node-selector]: concepts-clusters-workloads.md#node-selectors
