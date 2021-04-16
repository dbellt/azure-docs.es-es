---
title: 'Azure RBAC: Kubernetes habilitado para Azure Arc'
services: azure-arc
ms.service: azure-arc
ms.date: 04/05/2021
ms.topic: conceptual
author: shashankbarsin
ms.author: shasb
description: En este artículo se proporciona información general y algunos conceptos de la funcionalidad Azure RBAC en Kubernetes habilitado para Azure Arc
ms.openlocfilehash: 7eb55ed819b6487697b5c2d64cdbabe2bbdae8a3
ms.sourcegitcommit: 56b0c7923d67f96da21653b4bb37d943c36a81d6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/06/2021
ms.locfileid: "106450896"
---
# <a name="azure-rbac-on-azure-arc-enabled-kubernetes"></a>Azure RBAC en Kubernetes habilitado para Azure Arc

Los tipos de objeto [ClusterRoleBinding y RoleBinding](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#rolebinding-and-clusterrolebinding) de Kubernetes ayudan a definir la autorización en Kubernetes de forma nativa. Con Azure RBAC, puede usar Azure Active Directory (Azure AD) y las asignaciones de roles de Azure para controlar las comprobaciones de autorización en el clúster.

Con esta característica, todas las ventajas de las asignaciones de roles de Azure, como los registros de actividad que muestran todos los cambios de Azure RBAC en un recurso de Azure, ahora se pueden aplicar al clúster de Kubernetes habilitado para Azure Arc.

## <a name="architecture---azure-rbac-on-azure-arc-enabled-kubernetes"></a>Arquitectura de Azure RBAC en Kubernetes habilitado para Azure Arc

[ ![Arquitectura de Azure RBAC](./media/conceptual-azure-rbac.png) ](./media/conceptual-azure-rbac.png#lightbox)

Para enrutar todas las comprobaciones de acceso de autorización al servicio de autorización en Azure, se implementa un servidor de webhook ([Guard](https://github.com/appscode/guard)) en el clúster.

El `apiserver` del clúster está configurado para usar la [autenticación de tokens de webhook](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#webhook-token-authentication) y la [autorización de webhook](https://kubernetes.io/docs/reference/access-authn-authz/webhook/) para que las solicitudes `TokenAccessReview` y `SubjectAccessReview` se enruten al servidor de webhook de Guard. Las solicitudes `TokenAccessReview` y `SubjectAccessReview` se desencadenan cuando se envían solicitudes de recursos de Kubernetes al `apiserver`.

Después, Guard realiza una llamada `checkAccess` en el servicio de autorización de Azure para ver si la entidad de Azure AD solicitante tiene acceso al recurso correspondiente. 

Si existe un rol en la asignación que permite este acceso, se envía una respuesta `allowed` del servicio de autorización a Guard. A su vez, Guard envía una respuesta `allowed` a `apiserver`, lo que permite a la entidad autora de la llamada acceso al recurso de Kubernetes solicitado.


Si no existe un rol de la asignación que permita este acceso, se envía una respuesta `denied` del servicio de autorización a Guard. Guard envía una respuesta `denied` al `apiserver`, lo que devuelve a la entidad autora de la llamada un error 403 prohibido sobre el recurso solicitado.

## <a name="next-steps"></a>Pasos siguientes

* Use la guía de inicio rápido para [conectar un clúster de Kubernetes a Azure Arc](./quickstart-connect-cluster.md).
* [Configure Azure RBAC](./azure-rbac.md) en el clúster de Kubernetes habilitado para Azure Arc.