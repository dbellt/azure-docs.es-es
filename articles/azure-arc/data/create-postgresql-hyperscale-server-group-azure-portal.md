---
title: Creación de un grupo de servidores de Hiperescala de PostgreSQL habilitada para Azure Arc desde Azure Portal
description: Creación de un grupo de servidores de Hiperescala de PostgreSQL habilitada para Azure Arc desde Azure Portal
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: TheJY
ms.author: jeanyd
ms.reviewer: mikeray
ms.date: 04/28/2021
ms.topic: how-to
ms.openlocfilehash: 5cc99acf303b99e138edd0418d2d49824bbf27fe
ms.sourcegitcommit: fc9fd6e72297de6e87c9cf0d58edd632a8fb2552
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108294176"
---
# <a name="create-an-azure-arc-enabled-postgresql-hyperscale-server-group-from-the-azure-portal"></a>Creación de un grupo de servidores de Hiperescala de PostgreSQL habilitada para Azure Arc desde Azure Portal

En este documento se describen los pasos para crear un grupo de servidores de Hiperescala de PostgreSQL en Azure Arc desde Azure Portal.

[!INCLUDE [azure-arc-common-prerequisites](../../../includes/azure-arc-common-prerequisites.md)]

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="getting-started"></a>Introducción
Si ya está familiarizado con los temas siguientes, puede omitir este párrafo.
Hay temas importantes que puede que le interese leer antes de continuar con la creación:
- [Información general sobre los servicios de datos habilitados para Azure Arc](overview.md)
- [Modos de conectividad y requisitos](connectivity.md)
- [Conceptos de almacenamiento de Kubernetes y configuración de almacenamiento](storage-configuration.md)
- [Modelo de recursos de Kubernetes](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/scheduling/resources.md#resource-quantities)

Si prefiere probar las cosas sin aprovisionar un entorno completo por su cuenta, empiece a trabajar rápidamente con [Azure Arc JumpStart](https://azurearcjumpstart.io/azure_arc_jumpstart/azure_arc_data/) en Azure Kubernetes Service (AKS), AWS Elastic Kubernetes Service (EKS), Google Cloud Kubernetes Engine (GKE) o en una máquina virtual de Azure.


## <a name="deploy-an-arc-data-controller-configured-to-use-the-direct-connectivity-mode"></a>Implementación de un controlador de datos de Arc configurado para usar el modo de conectividad directa

Requisito: antes de implementar un grupo de servidores de Hiperescala de PostgreSQL habilitada para Azure Arc que se operará desde Azure Portal, primero debe implementar un controlador de datos de Azure Arc configurado para usar el modo de conectividad *directa*.
Para implementar un controlador de datos de Arc, complete las instrucciones de estos artículos:
1. [Implementación del controlador de datos: modo de conexión directa (requisitos previos)](deploy-data-controller-direct-mode-prerequisites.md)
1. [Implementación del controlador de datos de Azure Arc | Modo de conexión directa](deploy-data-controller-direct-mode.md)


## <a name="preliminary-and-temporary-step-for-openshift-users-only"></a>Paso preliminar y temporal solo para usuarios de OpenShift
Implemente este paso antes de pasar al siguiente. Para implementar el grupo de servidores Hiperescala de PostgreSQL en Red Hat OpenShift en un proyecto distinto del predeterminado, debe ejecutar los comandos siguientes en el clúster para actualizar las restricciones de seguridad. Este comando concede los privilegios necesarios a las cuentas de servicio que ejecutarán el grupo de servidores Hiperescala de PostgreSQL. La restricción de contexto de seguridad (SCC) arc-data-scc es la que ha agregado al implementar el controlador de datos de Azure Arc.

```Console
oc adm policy add-scc-to-user arc-data-scc -z <server-group-name> -n <namespace name>
```

**Server-group-name es el nombre del grupo de servidores que se va a crear en el siguiente paso.**

Para obtener más detalles sobre las SCC en OpenShift, consulte la [documentación de OpenShift](https://docs.openshift.com/container-platform/4.2/authentication/managing-security-context-constraints.html). 

Siga con el paso siguiente.

## <a name="deploy-an-azure-arc-enabled-postgresql-hyperscale-server-group-from-the-azure-portal"></a>Implementación de un grupo de servidores de Hiperescala de PostgreSQL habilitada para Azure Arc desde Azure Portal

Para implementar y operar un grupo de servidores de Hiperescala de Postgres habilitada para Azure Arc desde Azure Portal debe implementarlo en un controlador de datos de Arc configurado para usar el modo de conectividad *directa*. 

> [!IMPORTANT]
> No puede operar un grupo de servidores de Hiperescala de PostgreSQL habilitada para Azure Arc desde Azure Portal si lo implementó en un controlador de datos de Azure Arc configurado para usar el modo de conectividad *indirecta*. 

Después de implementar un controlador de datos de Arc habilitado para el modo de conectividad directa:
1. Abra un explorador con la siguiente dirección URL [https://portal.azure.com](https://portal.azure.com).
2. En la ventana de búsqueda de la parte superior de la página, busque "*azure arc postgres*" en Azure Market Place y seleccione **Grupos de servidores de Azure Database for PostgreSQL: Azure Arc**.
3. En la página que se abre, haga clic en **+ Crear** en la esquina superior izquierda. 
4. Rellene el formulario como si fuera a implementar otro recurso de Azure.


### <a name="important-parameters-you-should-consider-are"></a>Los parámetros importantes que debe tener en cuenta son los siguientes:

- **Número de nodos de trabajo** que quiere implementar para escalar horizontalmente y lograr un mejor rendimiento. Antes de continuar, consulte los [conceptos sobre Hiperescala de Postgres](concepts-distributed-postgres-hyperscale.md). Por ejemplo, si implementa un grupo de servidores con dos nodos de trabajo, la implementación creará tres pods, uno para la instancia o nodo de coordinación y dos para las instancias o nodos de trabajo (uno para cada uno de los nodos de trabajo).

## <a name="next-steps"></a>Pasos siguientes

- Conéctese a su instancia de Hiperescala de PostgreSQL habilitada para Azure Arc: lea [Obtención de puntos de conexión y cadenas de conexión](get-connection-endpoints-and-connection-strings-postgres-hyperscale.md)
- Lea los conceptos y las guías paso a paso de Hiperescala de Azure Database for PostgreSQL para distribuir los datos entre varios nodos de Hiperescala de PostgreSQL y poder beneficiarse de posibles mejores rendimientos:
    * [Nodos y tablas](../../postgresql/concepts-hyperscale-nodes.md)
    * [Determinar el tipo de aplicación](../../postgresql/concepts-hyperscale-app-type.md)
    * [Elección de una columna de distribución](../../postgresql/concepts-hyperscale-choose-distribution-column.md)
    * [Colocación de tabla](../../postgresql/concepts-hyperscale-colocation.md)
    * [Distribución y modificación de tablas](../../postgresql/howto-hyperscale-modify-distributed-tables.md)
    * [Diseño de una base de datos multiinquilino](../../postgresql/tutorial-design-database-hyperscale-multi-tenant.md)*
    * [Diseño de un panel de análisis en tiempo real](../../postgresql/tutorial-design-database-hyperscale-realtime.md)*

    > \* En los documentos anteriores, omita las secciones **Inicio de sesión en Azure Portal** y **Creación de una instancia de Azure Database for PostgreSQL: Hiperescala (Citus)** . Implemente los pasos restantes en la implementación de Azure Arc. Esas secciones son específicas de Hiperescala (Citus) de Azure Database for PostgreSQL que se ofrece como un servicio PaaS en la nube de Azure, pero las demás partes de los documentos se aplican directamente a Hiperescala de PostgreSQL habilitada para Azure Arc.

- [Escalado horizontal del grupo de servidores de Hiperescala de PostgreSQL habilitada para Azure Arc](scale-out-postgresql-hyperscale-server-group.md)
- [Conceptos de almacenamiento de Kubernetes y configuración de almacenamiento](storage-configuration.md)
- [Expansión de notificaciones de volúmenes persistentes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#expanding-persistent-volumes-claims)
- [Modelo de recursos de Kubernetes](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/scheduling/resources.md#resource-quantities)


