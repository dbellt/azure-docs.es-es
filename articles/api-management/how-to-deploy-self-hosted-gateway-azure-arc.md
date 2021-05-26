---
title: Implementación de una puerta de enlace de Azure API Management en Azure Arc
description: Habilite Azure Arc para implementar una puerta de enlace de Azure API Management autohospedada.
author: v-hhunter
ms.author: v-hhunter
ms.service: api-management
ms.topic: article
ms.date: 05/25/2021
ms.openlocfilehash: 25a647df5d1afcb5212b4e717e1a70f9a68f4ac5
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110388461"
---
# <a name="deploy-an-azure-api-management-gateway-on-azure-arc-preview"></a>Implementación de una puerta de enlace de Azure API Management en Azure Arc (versión preliminar)

Con la integración entre Azure API Management y [Azure Arc en Kubernetes](../azure-arc/kubernetes/overview.md), puede implementar el componente de puerta de enlace de API Management como una [extensión en un clúster de Kubernetes que esté habilitado para Azure Arc](../azure-arc/kubernetes/extensions.md). 

La implementación de la puerta de enlace de API Management en un clúster de Kubernetes que esté habilitado para Arc amplía la compatibilidad de API Management para entornos híbridos y de varias nubes. Habilite la implementación mediante una extensión de clúster para que la administración y aplicación de directivas en el clúster habilitado para Arc sea una experiencia coherente.

[!INCLUDE [preview](./includes/preview/preview-callout-self-hosted-gateway-azure-arc.md)]

> [!NOTE]
> También puede implementar la puerta de enlace autohospedada [directamente en Kubernetes](./how-to-deploy-self-hosted-gateway-azure-kubernetes-service.md).

## <a name="prerequisites"></a>Requisitos previos

* [Conecte el clúster de Kubernetes](../azure-arc/kubernetes/quickstart-connect-cluster.md) en una [región de Azure Arc compatible](../azure-arc/kubernetes/overview.md#supported-regions).
* Instalación de la extensión `k8s-extension` de la CLI de Azure:

    ```azurecli
    az extension add --name k8s-extension
    ```
    Si ya instaló el módulo `k8s-extension`, actualícelo a la última versión:

    ```azurecli
    az extension update --name k8s-extension
    ```
* [Creación de una instancia de Azure API Management](./get-started-create-service-instance.md).
* [Aprovisione un recurso de puerta de enlace en la instancia de Azure API Management](./api-management-howto-provision-self-hosted-gateway.md).

## <a name="deploy-the-api-management-gateway-extension-using-azure-cli"></a>Implementación de la extensión de la puerta de enlace de API Management mediante la CLI de Azure

1. Vaya a la instancia de API Management en Azure Portal.
1. Seleccione **Puertas de enlace** en el menú de navegación lateral.
1. Seleccione y abra el recurso de puertas de enlace aprovisionadas de la lista.
1. En el recurso de puertas de enlace aprovisionadas, haga clic en **Implementación** en el menú de navegación lateral.
1. Tome nota de los valores de **token** y de la **dirección URL de configuración** para realizar el siguiente paso.
1. En la CLI de Azure, implemente la extensión de la puerta de enlace mediante el comando `az k8s-extension create`. Complete los valores `token` y `configuration URL`.
    * En el siguiente ejemplo se usa la configuración de la extensión `service.Type='NodePort'`. Consulte más [configuraciones de extensión disponibles](#available-extension-configurations).

    ```azurecli
    az k8s-extension create --cluster-type connectedClusters --cluster-name <cluster-name> \
      --resource-group <rg-name> --name <extension-name> --extension-type Microsoft.ApiManagement.Gateway \
      --scope namespace --target-namespace <namespace> \
      --configuration-settings gateway.endpoint='<Configuration URL>' \
      --configuration-protected-settings gateway.authKey='<token>' --release-train preview
    ```

    > [!TIP]
    > La marca `-protected-` de `authKey` es opcional, pero se recomienda usarla. 

1. Compruebe el estado de la implementación mediante el siguiente comando de la CLI:
    ```azurecli
    az k8s-extension show --cluster-type connectedClusters --cluster-name <cluster-name> --resource-group <rg-name> --name <extension-name>
    ```
1. Vuelva a la lista de **Puertas de enlace** para comprobar que el estado de la puerta de enlace muestre una marca de verificación verde con un recuento de nodos. Este estado significa que los pods implementados de la puerta de enlace autohospedada:
    * Se están comunicando correctamente con el servicio de API Management.
    * Que tienen un "latido" regular.

## <a name="deploy-the-api-management-gateway-extension-using-azure-portal"></a>Implementación de la extensión de la puerta de enlace de API Management mediante Azure Portal

1. En Azure Portal,vaya al clúster conectado de Azure Arc.
1. En el menú de la izquierda, seleccione **Extensiones (versión preliminar)**  >  **+ Agregar** > **Puerta de enlace de API Management (versión preliminar)** .
1. Seleccione **Crear**.
1. En la ventana **Instalar la puerta de enlace de API Management**, configure la extensión de la puerta de enlace:
    * Seleccione la suscripción y el grupo de recursos de la instancia de API Management.
    * En **Detalles de la puerta de enlace**, seleccione la **instancia de API Management** y el **nombre de la puerta de enlace**. Escriba un ámbito de **Espacio de nombres** para la extensión y, opcionalmente, escriba un número de **réplicas**, si esta opción se admite en el nivel de servicio de API Management.
    * En **Configuración de Kubernetes**, seleccione la configuración predeterminada o una configuración diferente para el clúster. Para ver las opciones, consulte las [configuraciones de extensiones disponibles](#available-extension-configurations).

    :::image type="content" source="./media/how-to-deploy-self-hosted-gateway-azure-arc/deploy-gateway-extension-azure-arc.png" alt-text="Captura de pantalla de la implementación de la extensión en Azure Portal":::

1. En la pestaña **Supervisión**, puede habilitar la opción de supervisión para cargar solicitudes de seguimiento de métricas en la puerta de enlace y el back-end. Si esta opción está habilitada, seleccione un área de trabajo de **Log Analytics** existente.
1. Seleccione **Revisar e instalar** y, a continuación, **Instalar**.

## <a name="available-extension-configurations"></a>Configuraciones de extensiones disponibles

Las siguientes configuraciones de extensiones son **obligatorias**.

| Configuración | Descripción |
| ------- | ----------- | 
| `gateway.endpoint` | Dirección URL de configuración del punto de conexión de la puerta de enlace. |
| `gateway.authKey` | Token para el acceso a la puerta de enlace. | 
| `service.Type` | Configuración del servicio Kubernetes para la puerta de enlace: `LoadBalancer` `NodePort`, o `ClusterIP`. |

### <a name="log-analytics-settings"></a>Configuración de Log Analytics

Para habilitar la supervisión de la puerta de enlace autohospedada, configure las siguientes opciones de Log Analytics:

| Configuración | Descripción |
| ------- | ----------- | 
| `monitoring.customResourceId` | Id. de recursos de Azure Resource Manager para la instancia de API Management. |
| `monitoring.workspaceId` | Id. del área de trabajo de Log Analytics. | 
| `monitoring.ingestionKey` | Secreto con la clave de ingesta de Log Analytics. |

> [!NOTE]
> Si no ha habilitado Log Analytics: 
> 1. Consulte la guía de inicio rápido [Creación de un área de trabajo de Log Analytics](../azure-monitor/logs/quick-create-workspace.md). 
> 1. Obtenga información sobre dónde encontrar la [configuración del agente de Log Analytics](../azure-monitor/agents/log-analytics-agent.md).

## <a name="next-steps"></a>Pasos siguientes

* Para más información sobre la puerta de enlace autohospedada, consulte [Introducción a la puerta de enlace autohospedada de Azure API Management](self-hosted-gateway-overview.md).
* Descubra todas las [extensiones de Kubernetes habilitadas en Azure Arc](../azure-arc/kubernetes/extensions.md). 
* Obtenga más información sobre [Kubernetes habilitado para Azure Arc](../azure-arc/kubernetes/overview.md).