---
title: 'Ubicaciones personalizadas: Kubernetes habilitado para Azure Arc'
services: azure-arc
ms.service: azure-arc
ms.date: 04/05/2021
ms.topic: conceptual
author: shashankbarsin
ms.author: shasb
description: En este artículo, encontrará información general y algunos conceptos sobre la funcionalidad Ubicaciones personalizadas de Kubernetes habilitado para Azure Arc
ms.openlocfilehash: 1235c6adfe97a943ef77584a6a0c8683d691be91
ms.sourcegitcommit: 56b0c7923d67f96da21653b4bb37d943c36a81d6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/06/2021
ms.locfileid: "106450903"
---
# <a name="custom-locations-on-top-of-azure-arc-enabled-kubernetes"></a>Ubicaciones personalizadas basadas en Kubernetes habilitado para Azure Arc

Como extensión de la construcción de ubicaciones de Azure, *Ubicaciones personalizadas* proporciona una método para que los administradores de inquilinos usen sus clústeres de Kubernetes habilitados para Azure Arc como ubicaciones de destino para implementar instancias de servicios de Azure. Entre los ejemplos de recursos de Azure se incluyen SQL Managed Instance habilitado para Azure Arc e Hiperescala de PostgreSQL habilitada para Azure Arc.

De forma similar a las ubicaciones de Azure, los usuarios finales dentro del inquilino con acceso a Ubicaciones personalizadas pueden implementar recursos allí mediante el proceso privado de su empresa.

[ ![Capas de la plataforma de Arc](./media/conceptual-arc-platform-layers.png) ](./media/conceptual-arc-platform-layers.png#lightbox)

Puede visualizar las ubicaciones personalizadas como una capa de abstracción sobre el clúster de Kubernetes habilitado para Azure Arc, la Conexión de clústeres y las extensiones de clúster. Ubicaciones personalizadas crea los [RoleBindings y ClusterRoleBindings](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#rolebinding-and-clusterrolebinding) pormenorizados necesario para que otros servicios de Azure accedan al clúster. Estos otros servicios de Azure requieren acceso al clúster para administrar los recursos que el cliente quiere implementar en sus clústeres.

[!INCLUDE [preview features note](./includes/preview/preview-callout.md)]

## <a name="architecture"></a>Architecture

Cuando el administrador habilita la característica Ubicaciones personalizadas en el clúster, se crea un ClusterRoleBinding en el mismo, lo que autoriza la aplicación de Azure AD utilizada por el proveedor de recursos de ubicaciones personalizadas (RP). Una vez autorizado, el RP de ubicaciones personalizadas puede crear los ClusterRoleBindings o RoleBindings que necesiten otros RP de Azure para crear los recursos personalizados en este clúster. Las extensiones de clúster instaladas en el clúster determinan la lista de RP que se van a autorizar.

[ ![Uso de ubicaciones personalizadas](./media/conceptual-custom-locations-usage.png) ](./media/conceptual-custom-locations-usage.png#lightbox)

Cuando el usuario crea una instancia del servicio de datos en el clúster: 
1. La solicitud PUT se envía a Azure Resource Manager.
1. La solicitud PUT se reenvía al RP de Data Services habilitado para Azure Arc. 
1. El RP captura el archivo `kubeconfig` asociado con el clúster de Kubernetes habilitado para Azure Arc, en el que existe la ubicación personalizada. 
   * Se hace referencia a la ubicación personalizada como `extendedLocation` en la solicitud PUT original. 
1. El RP de Data Services habilitado para Azure Arc usa `kubeconfig` para comunicarse con el clúster a fin de crear un recurso personalizado del tipo Data Services habilitado para Azure Arc en el espacio de nombres asignado a la ubicación personalizada. 
   * El operador de Data Services habilitado para Azure Arc se implementó mediante la creación de una extensión de clúster antes de que existiera la ubicación personalizada. 
1. El operador de Data Services habilitado para Azure Arc lee el nuevo recurso personalizado creado en el clúster y crea el controlador de datos, que se traduce en la concreción del estado deseado en el clúster. 

La secuencia de pasos para crear la instancia administrada de SQL y la instancia de PostgreSQL es idéntica a la descrita anteriormente.

## <a name="next-steps"></a>Pasos siguientes

* Use la guía de inicio rápido para [conectar un clúster de Kubernetes a Azure Arc](./quickstart-connect-cluster.md).
* [Cree una ubicación personalizada](./custom-locations.md) en el clúster de Kubernetes habilitado para Azure Arc.