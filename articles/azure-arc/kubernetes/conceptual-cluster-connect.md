---
title: Conexión de clúster - Kubernetes habilitado para Azure Arc
services: azure-arc
ms.service: azure-arc
ms.date: 04/05/2021
ms.topic: conceptual
author: shashankbarsin
ms.author: shasb
description: En este artículo se proporciona información general conceptual de la funcionalidad Conexión de clúster de Kubernetes habilitado para Azure Arc.
ms.openlocfilehash: 716ffe0c1f123c2a6abe8f407f6435e157ba5b6a
ms.sourcegitcommit: 56b0c7923d67f96da21653b4bb37d943c36a81d6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/06/2021
ms.locfileid: "106450895"
---
# <a name="cluster-connect-on-azure-arc-enabled-kubernetes"></a>Conexión de clúster de Kubernetes habilitado para Azure Arc

La característica de *conexión de clúster* de Kubernetes habilitado para Azure Arc proporciona conectividad con `apiserver` del clúster sin necesidad de habilitar ningún puerto de entrada en el firewall. Un agente de proxy inverso que se ejecuta en el clúster puede iniciar una sesión de forma segura con el servicio de Azure Arc en modo de salida. 

La conexión de clúster permite a los desarrolladores tener acceso a sus clústeres desde cualquier lugar para un desarrollo y una depuración interactivos. También permite a los usuarios y administradores de clústeres acceder a sus clústeres o administrarlos desde cualquier lugar. Incluso puede utilizar agentes/ejecutores hospedados de Azure Pipelines, acciones de GitHub o cualquier otro servicio de CI/CD hospedado para implementar aplicaciones en clústeres locales, sin necesidad de agentes autohospedados.

[!INCLUDE [preview features note](./includes/preview/preview-callout.md)]

## <a name="architecture"></a>Architecture

[ ![Arquitectura de conexión de clústeres](./media/conceptual-cluster-connect.png) ](./media/conceptual-cluster-connect.png#lightbox)

En el lado del clúster, un agente de proxy inverso llamado `clusterconnect-agent` implementado como parte del gráfico Helm del agente realiza llamadas salientes al servicio de Azure Arc para establecer la sesión.

Cuando el usuario llama a `az connectedk8s proxy`:
1. El archivo binario del proxy de Azure Arc se descarga y se pone en marcha como un proceso en la máquina cliente. 
1. El proxy de Azure Arc captura un archivo `kubeconfig` asociado al clúster de Kubernetes habilitado para Azure Arc en el que se invoca a `az connectedk8s proxy`.
    * El proxy de Azure Arc usa el token de acceso de Azure del llamador y el nombre del identificador de Azure Resource Manager. 
1. El archivo `kubeconfig`, guardado en la máquina por el proxy de Azure Arc, apunta la dirección URL del servidor a un punto de conexión en el proceso del proxy de Azure Arc.

Cuando un usuario envía una solicitud mediante este archivo `kubeconfig`:
1. El proxy de Azure Arc asigna el punto de conexión que recibe la solicitud al servicio de Azure Arc. 
1. Después, el servicio de Azure Arc reenvía la solicitud a `clusterconnect-agent` que está en ejecución en el clúster. 
1. `clusterconnect-agent` pasa la solicitud al componente `kube-aad-proxy`, que realiza la autenticación de Azure AD en la entidad que realiza la llamada. 
1. Después de la autenticación de Azure AD, `kube-aad-proxy` usa la característica de [suplantación de usuario](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#user-impersonation) de Kubernetes para reenviar la solicitud a `apiserver` del clúster.

## <a name="next-steps"></a>Pasos siguientes

* Utilice el inicio rápido para [conectar un clúster de Kubernetes a Azure Arc](./quickstart-connect-cluster.md).
* [Acceda al clúster](./cluster-connect.md) de forma segura desde cualquier lugar mediante la conexión de clúster.