---
title: Reparación automática de nodos de Azure Kubernetes Service (AKS)
description: Obtenga información sobre la funcionalidad de reparación automática de nodos y cómo AKS corrige los nodos de trabajo interrumpidos.
services: container-service
ms.topic: conceptual
ms.date: 03/11/2021
ms.openlocfilehash: 95783000739fd42e39ef1bbce88c3b8e62d4a8dc
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110073205"
---
# <a name="azure-kubernetes-service-aks-node-auto-repair"></a>Reparación automática de nodos de Azure Kubernetes Service (AKS)

AKS supervisa de manera continua el estado de mantenimiento de los nodos de trabajo y realiza reparaciones automáticas si son incorrectos. La plataforma de máquina virtual (VM) de Azure [realiza el mantenimiento de las VM][vm-updates] que experimentan problemas. 

AKS y las VM de Azure trabajan de forma conjunta para minimizar las interrupciones del servicio de los clústeres.

Este documento, obtendrá información acerca de cómo se comporta la funcionalidad de reparación automática del nodo para nodos Windows y Linux. 

## <a name="how-aks-checks-for-unhealthy-nodes"></a>Comprobaciones de nodos en mal estado en AKS

AKS usa las siguientes reglas para determinar si un nodo es incorrecto y necesita reparación: 
* El nodo notifica un estado **NotReady** en comprobaciones consecutivas durante un período de 10 minutos.
* El nodo no notifica ningún estado en un plazo de 10 minutos.

Puede comprobar manualmente el estado de mantenimiento de los nodos con kubectl.

```
kubectl get nodes
```

## <a name="how-automatic-repair-works"></a>Funcionamiento de la reparación automática

> [!Note]
> AKS inicia las operaciones de reparación con la cuenta de usuario **aks-remediator**.

Si AKS identifica un nodo incorrecto que permanece de este modo durante 10 minutos, AKS realiza las siguientes acciones:

1. Reinicie el nodo.
1. Si el reinicio no se realiza correctamente, restablece la imagen inicial del nodo.
1. Si la nueva imagen no se restablece correctamente, crea y restablece la imagen inicial de un nuevo nodo.

Los ingenieros de AKS están investigando correcciones alternativas si la reparación automática no se realiza correctamente. 

Si AKS encuentra varios nodos incorrectos durante una comprobación de estado, cada nodo se repara individualmente antes de que comience otra reparación.


## <a name="limitations"></a>Limitaciones

En muchos casos, AKS puede determinar si un nodo está en un estado incorrecto e intentar reparar el problema, pero hay casos en los que AKS no puede reparar el problema o no puede detectar que hay un problema. Por ejemplo, AKS no puede detectar problemas si no se notifica el estado de un nodo debido a un error en la configuración de red.

## <a name="next-steps"></a>Pasos siguientes

Use [Availability Zones][availability-zones] para aumentar la alta disponibilidad con las cargas de trabajo del clúster de AKS.

<!-- LINKS - External -->

<!-- LINKS - Internal -->
[availability-zones]: ./availability-zones.md
[vm-updates]: ../virtual-machines/maintenance-and-updates.md
